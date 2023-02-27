# Example `Hello`
Ví dụ này giúp bạn nắm được cách xây dựng một microservice đơn giản với Scyna:
- Sử dụng `protobuf` để định nghĩa API cho các endpoint của microservice
- Viết test cho các endpoint
- Implement logic cho các endpoint
- Deploy

Ở ví dụ này chúng ta xây dựng một microservice thực hiện 2 endpoint sau
- `Hello`: nhận vào tên một người và trả lại lời chào người đó
- `Add`: nhận vào 2 giá trị nguyên và trả ra tổng của 2 số nguyên đó

Chú ý: để chạy được ví dụ, các bạn cần phải cài đặt môi trường runtime của Scyna, chi tiết hướng dẫn [tại đây](../setup/golang.md).

### A. Endpoint Hello

#### 1. API

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

Định nghĩa API sẽ được lưu trong file `proto/hello.proto`, lệnh để dịch file `proto` sang code của Go:

```
protoc -I=. --go_out=. hello.proto
```

Lệnh này sẽ sinh ra file `proto/hello.pb.go`


#### 2. Test

Theo tinh thần của TDD, chúng ta sẽ viết test trước khi implement logic. Endpoint `Hello` chỉ làm việc rất đơn giản là nhận một tên và trả lại lời chào với tên nhận được. Các rule sau cần được tuân thủ cho dữ liệu đầu vào:
- `Name` phải không được rỗng
- `Name` có độ dài từ 3 đến 50 ký tự

Chúng ta sẽ viết các test case để kiểm định các rule trên sử dụng `EndpointTest` được Scyna hỗ trợ để viết test case cho các endpoint implement trên Scyna. Test cho `Hello` sẽ được lưu trong file `test/hello_test.go` và có nội dung cơ bản sau.

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

#### 3. Implement

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

### B. Endpoint Add

#### 1. API

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

#### 2. Test

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

#### 3. Implement

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

### C. Implement `main.go` và deploy

#### 1. Main function

```go
const MODULE_CODE = "scyna_test"

func main() {
	scyna.RemoteInit(scyna.RemoteConfig{
		ManagerUrl: "http://localhost:8081",
		Name:       MODULE_CODE,
		Secret:     "123456",
	})
	defer scyna.Release()

	scyna.RegisterEndpoint(service.ADD_URL, service.AddHandler)
	scyna.RegisterEndpoint(service.HELLO_URL, service.HelloHandler)

	scyna.Start()
}
```

#### 1. Setup script

TBD

#### 2. Docker container

TBD

### D. Reference

- Source code: https://github.com/scyna/example/tree/main/go/hello
