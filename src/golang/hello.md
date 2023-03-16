# Hello

Trong ví dụ này chúng ta sẽ xây dựng một microservice đơn giản với Scyna:
- Sử dụng Protobuf để định nghĩa API cho các endpoint của microservice
- Viết test cho các endpoint
- Implement logic cho các endpoint
- Deploy thành docker container

Microservice này sẽ thực hiện 2 chức năng đơn giản sau:
- `Hello`: nhận vào tên một người và trả lại lời chào người đó
- `Add`: nhận vào 2 giá trị nguyên và trả ra tổng của 2 số nguyên đó

## 1. Hello

### API

Scyna sử dụng Google Protobuf để định nghĩa các API, file `hello.proto` sẽ chứa định nghĩa của các endpoint.

```protobuf
message HelloRequest
{
  string name = 1;
}

message HelloResponse 
{
  string content = 1;
}
```

Lệnh để dịch file `hello.proto` sang code của Go:

```bash
protoc -I=. --go_out=. hello.proto
```

Sau khi dịch, `protoc` sẽ sinh ra file `hello.pb.go` chứa các định nghĩa giao thức bằng ngôn ngữ `golang` sẵn sàng cho Hello Service sử dụng.

### Test

Theo tinh thần của TDD, chúng ta sẽ viết test trước khi implement logic. Endpoint `Hello` chỉ làm việc rất đơn giản là nhận một tên và trả lại lời chào với tên nhận được. Các rule sau cần được tuân thủ cho dữ liệu đầu vào:
- `Name` phải không được rỗng
- `Name` có độ dài từ 3 đến 50 ký tự

Chúng ta sẽ viết các test case để kiểm định các rule trên sử dụng `EndpointTest` được Scyna hỗ trợ để viết test case cho các endpoint implement trên Scyna. Test cho `Hello` sẽ được lưu trong file `hello_test.go` và có nội dung cơ bản sau.

```go
func TestHello_Success(t *testing.T) {
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{Name: "Alice"}).
		ExpectResponse(&proto.HelloResponse{Content: "Hello Alice"}).
		Run(t)
}

func TestHello_EmptyName(t *testing.T) {
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{}).
		ExpectError(scyna.REQUEST_INVALID).
		Run(t)
}

func TestHello_LongName(t *testing.T) {
	name := "Very long name will cause request invalid."
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{Name: name}).
		ExpectError(scyna.REQUEST_INVALID).
		Run(t)
}

func TestHello_ShortName(t *testing.T) {
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{Name: "A"}).
		ExpectError(scyna.REQUEST_INVALID).
		Run(t)
}
```

#### Implementation

```go
func HelloHandler(ctx scyna.Context, request *proto.HelloRequest) scyna.Error {
	ctx.Info("Receive HelloRequest")

	length := len(request.Name)
	if length < 3 || length > 50 {
		return scyna.REQUEST_INVALID
	}

	return ctx.OK(&proto.HelloResponse{Content: "Hello " + request.Name})
}
```

## 2. Add

### 2.1 API

```protobuf
message AddRequest
{
  int32 a = 1;
  int32 b = 2;
}

message AddResponse
{
  int32 sum = 1;
}
```

### 2.2 Test

Endpoint `Add` trả về tổng của 2 số nguyên đầu vào. Nếu kết quả lớn hơn 100 sẽ báo lỗi `ADD_RESULT_TOO_BIG`. Test cho endpoint `Add` như sau:

```go
func TestAdd_Success(t *testing.T) {
	scyna_test.EndpointTest(hello.ADD_URL).
		WithRequest(&proto.AddRequest{A: 5, B: 73}).
		ExpectResponse(&proto.AddResponse{Sum: 78}).
		Run(t)
}

func TestAdd_TooBig(t *testing.T) {
	scyna_test.EndpointTest(hello.ADD_URL).
		WithRequest(&proto.AddRequest{A: 50, B: 73}).
		ExpectError(hello.ADD_RESULT_TOO_BIG).
		Run(t)
}

```

### 2.3 Implementation

```go
func AddHandler(ctx scyna.Context, request *proto.AddRequest) scyna.Error {
	ctx.Info("Receive AddRequest")

	sum := request.A + request.B
	if sum > 100 {
		return ADD_RESULT_TOO_BIG
	}

	return ctx.OK(&proto.AddResponse{Sum: sum})
}
```

## 3. Deployment

### 3.1 Setup Script

Setup script được viết bằng `go` để tạo master data của microservice (module) và client trên Scyna Engine.

```go
func main() {
	scyna_setup.Init()
	scyna_setup.NewModule("ex_hello", "123456").Build()

	scyna_setup.NewClient("hello_test", "123456").
		UseEndpoint(service.ADD_URL).
		UseEndpoint(service.HELLO_URL).
		Build()
}
```

### 3.2 Main Function

```go
func main() {
	scyna.RemoteInit(scyna.RemoteConfig{
		ManagerUrl: "http://localhost:8081",
		Name:       "ex_hello",
		Secret:     "123456",
	})
	defer scyna.Release()

	scyna.RegisterEndpoint(service.ADD_URL, service.AddHandler)
	scyna.RegisterEndpoint(service.HELLO_URL, service.HelloHandler)

	scyna.Start()
}
```

### 3.3 Docker Container

```dockerfile
# Base image for Go build environment
FROM golang:1.19.1-alpine3.16 AS build-env

# Set environment variables
ENV GO111MODULE=on \
    WORKDIR=/go/src/app

# Set working directory
WORKDIR $WORKDIR

# Copy application source code
COPY example_folder $WORKDIR

# Download Go module dependencies
RUN go mod download


# Build the application
RUN go build -o /go/bin/app

# Final image
FROM alpine:3.16

# Set environment variables
ENV PASSWORD="123456789aA@#" \
    MANAGER_URL="https://localhost:8081"

# Copy the application binary from the build environment
COPY --from=build-env /go/bin/app /usr/local/bin/app

# Start the application
CMD /usr/local/bin/app --password ${PASSWORD} --managerUrl ${MANAGER_URL}
```

### 3.4 Test with cURL

```bash
curl --location --request POST 'http://localhost:8080/ex/hello/hello' \
--header 'Client-Id: hello_test' \
--header 'Client-Secret: 123456' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "Alice"
}'
```

```bash
curl --location --request POST 'http://localhost:8080/ex/hello/add' \
--header 'Client-Id: hello_test' \
--header 'Client-Secret: 123456' \
--header 'Content-Type: application/json' \
--data-raw '{
    "a": 23,
    "b": 5
}'
```

> ***NOTE***: để chạy được ví dụ, các bạn cần phải cài đặt môi trường runtime của Scyna, chi tiết hướng dẫn [tại đây](./../getting-started.md).

## 4. Reference

- Source code: https://github.com/scyna/example/tree/main/go/hello
- Setup for Golang developer.
  