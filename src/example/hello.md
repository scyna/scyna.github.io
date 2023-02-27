# Hello Example

Ví dụ này giúp các bạn làm quen với việc tạo xây dựng một microservice đơn giản dựa sử dụng Scyna. Các bạn cần cài đặt môi trường runtime của Scyna để có thể chạy được các ví dụ. 

Source code của ví dụ https://github.com/scyna/example/tree/main/go/hello

### 1. API

Ở ví dụ này chúng ta xây dựng một microservice thực hiện 2 endpoint đơn giản nhất là **Hello** và **Add**. Protobuf định nghĩa hai endpoint này được lưu trong file [proto/hello.proto](https://github.com/scyna/example/blob/main/go/hello/proto/hello.proto)

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

Lệnh này sẽ sinh ra file `proto/hello.pb.go`

## 2. Test

Theo tinh thần của TDD, chúng ta sẽ viết test trước khi implement logic.

#### Hello

Endpoint `Hello` chỉ làm việc rất đơn giản là nhận 1 tên và trả lại lời chào với tên nhận được. Các rule sau cầ n được tuân thủ cho dữ liệu đầu vào:
- `Name` phải không được rỗng
- `Name` có độ dài từ 3 đến 50 ký tự

Scyna hỗ trợ `EndpointTest` để chúng ta có thể viết test cho các endpoint implement trên Scyna. Test cho `Hello` sẽ được lưu trong file `test/hello_test.go` và có nội dung cơ bản sau.

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
#### Add

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


## 3. Code

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
