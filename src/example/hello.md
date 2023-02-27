# Hello Example

Ví dụ này giúp các bạn làm quyen với việc tạo xây dựng một microservice đơn giản dựa sử dụng Scyna. Các bạn cần cài đặt môi trường runtime của Scyna để có thể chạy được các ví dụ. 

Source code của ví dụ https://github.com/scyna/example/tree/main/go/hello

### 1. API

Ở ví dụ này chúng ta xây dựng một microservice thực hiện 2 endpoint đơn giản nhất là **Hello** và **Add**. Protobuf định nghĩa hai endpoint này được lưu trong file `proto/hello.proto`.

**Hello**

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

**Add**

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

Lệnh để dịch file `proto` sang code của Go:

```
protoc -I=. --go_out=. hello.proto
```

## 2. Code

##### [service/hello.go](https://github.com/scyna/example/blob/main/go/hello/service/hello.go)

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

##### [service/add.go](https://github.com/scyna/example/blob/main/go/hello/service/add.go)


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

## 3. Unit test

**test/hello_test.go**

```go
func TestHelloSuccess(t *testing.T) {
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{Name: "Alice"}).
		ExpectResponse(&proto.HelloResponse{Content: "Hello Alice"}).
		Run(t)
}

func TestHelloEmptyName(t *testing.T) {
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{}).
		ExpectError(scyna.REQUEST_INVALID).
		Run(t)
}

func TestHelloLongName(t *testing.T) {
	name := "Very long name will cause request invalid."
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{Name: name}).
		ExpectError(scyna.REQUEST_INVALID).
		Run(t)
}

func TestHelloShortName(t *testing.T) {
	scyna_test.EndpointTest(hello.HELLO_URL).
		WithRequest(&proto.HelloRequest{Name: "A"}).
		ExpectError(scyna.REQUEST_INVALID).
		Run(t)
}
```
**test/add_test.go**

```go
func TestAddSuccess(t *testing.T) {
	scyna_test.EndpointTest(hello.ADD_URL).
		WithRequest(&proto.AddRequest{A: 5, B: 73}).
		ExpectResponse(&proto.AddResponse{Sum: 78}).
		Run(t)
}

func TestAddTooBig(t *testing.T) {
	scyna_test.EndpointTest(hello.ADD_URL).
		WithRequest(&proto.AddRequest{A: 50, B: 73}).
		ExpectError(hello.ADD_RESULT_TOO_BIG).
		Run(t)
}
```
