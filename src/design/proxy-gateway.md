# Proxy

Proxy là cổng để kết nối các hệ thống bên ngoài với các service được implement trên Scyna. Đây thường là kết nối Backend-to-Backend và được xác thực bằng hai giá trị `Client-ID` và `Client-Secret`. Với mỗi cặp `Client-ID`/`Client-Secret`, hệ thống sẽ được cấp quyền sử dụng một số Endpoint nhất định.

<p align="center">
<img src="../images/scyna-proxy.png"  width="100%">
</p>

# Gateway

Giống như Proxy, Gateway là cổng vào cho các request xuất phát từ các ứng dụng (Frontend). Scyna hỗ trợ cơ chế xác thực cho người dùng cuối bằng một loại Endpoint đặc biệt là `AuthService`. Các ứng dụng có thể  linh hoạt xây dựng các cơ chế xác thực cho riêng mình bằng cách implement `AuthService` của Application đó. Một `AuthService` có thể được sử dụng bởi nhiều Application. Đây cũng chính là cơ chế SSO cho các cho các Application. Luồng tạo một xác thực được mô tả như hình bên dưới

<p align="center">
<img src="../images/scyna-gateway.png"  width="100%">
</p>

Khi một Authentication đã được tạo cho một người dùng nào đó rồi thì luồng xác thực sẽ được diễn ra như mô tả bên dưới

<p align="center">
<img src="../images/scyna-gateway-2.png"  width="100%">
</p>