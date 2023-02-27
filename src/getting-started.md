# Setup environment for Developer

### Hệ điều hành

Ở đây, chúng tôi khuyên bạn nên sử dụng hệ điều hành Ubuntu. Hãy trải nghiệm nó với những điều mới mẻ và sự mạnh mẽ của nó.

### Install Docker

[TBD: Docker là gì? dùng docker có pros/cons gì?]

Bạn có thể thực hiện theo từng bước trong [trang cài đặt](https://docs.docker.com/engine/install/ubuntu/) của DockerIO.
Sau khi cài đặt docker bạn cần thực hiện [cấp quyền](https://docs.docker.com/engine/install/linux-postinstall/) cho docker.
Dưới đây, tôi đã tổng hợp lại các đoạn mã cài đặt, bạn chỉ việc copy và paste vào trong Terminal của hệ điều hành.

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release -y
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

### Portainer

Portainer là một công cụ quản lý Docker dựa trên web, cho phép người dùng quản lý các container, hình ảnh, mạng, khối lưu trữ và các tài nguyên Docker khác thông qua giao diện web đơn giản và dễ sử dụng.

Portainer cho phép người dùng quản lý các ứng dụng Docker một cách trực quan, giúp cho việc triển khai, cập nhật và quản lý ứng dụng trở nên dễ dàng hơn. Nó hỗ trợ nhiều nền tảng và cung cấp cho người dùng các tính năng như quản lý mạng, quản lý volume, xem lịch sử logs và thống kê sử dụng tài nguyên của container.

Bạn có thể thực hiện theo các bước tại trang [cài đặt](https://docs.portainer.io/start/install-ce/server/docker/linux) của Portainer.

```shell
docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Sau khi cài đặt xong bạn hãy vào trang [http://localhost:9000](http://localhost:9000) để kích hoạt Portainer và đặt mật khẩu.

### IDE/Code editor

Có rất nhiều sự lựa chọn Code Editor nào cho phù hợp. Ở đây tôi đưa cho bạn hai công cụ phổ biến trong các dự án PM:

- [VSCode](https://code.visualstudio.com/download) (free)
- [Intellij](https://www.jetbrains.com/idea/download/) (17 đồng/tháng)

### Data Editor

Giống như Code editor, Bạn sẽ có nhiều lựa chọn công cụ làm việc với dữ liệu và hệ thống dữ liệu ScyllaDB (anh em với Cassandra).

- [cqlsh]() (command line only)
- [DBeaver Community](https://dbeaver.io/download/) (Free or 10 đồng/tháng)
  - Nếu sử dụng bản CE thì hãy cài thêm [jdbc driver](https://medium.com/@erkansirin/adding-cassandra-jdbc-driver-to-dbeaver-community-edition-6d34fd727d20) cho cassandra nhé.
- [Datagrip](https://www.jetbrains.com/datagrip/download/#section=linux) (9.9 đồng/tháng)

### GoLang

[TBD: Golang là gì?]

Có nhiều cách cài golang, bạn có thể cài theo bất cứ cách nào. Chúng tôi khuyên bạn nên cài bằng package của golang

Bạn có thể thực hiện theo các bước trên [trang cài đặt](https://go.dev/doc/install) của Go.dev.

```shell
wget https://go.dev/dl/go1.20.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.20.1.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```

## ScyllaDB

Bạn có thể cài đặt trực tiếp từ package (gói cài đặt) của ScyllaDB hoặc có thể sử dụng thông qua docker.

### Cài đặt trực tiếp

Bạn hãy làm theo các bước trong [trang cài đặt](https://docs.scylladb.com/stable/getting-started/install-scylla/) của ScyllaBD

### Cài đặt qua docker

#### Chạy bằng docker run

Dưới đây là hai cách khởi chạy ScyllaDB bằng docker. Mặc định khi khởi động máy tính sẽ bật. Bạn có thể tắt khi không sử dụng đến.

```shell
docker run -d -p 9042:9042 --name scylla -d scylladb/scylla --restart=always
```

#### Chạy bằng docker compose

```xml
version: '3'

services:
  db:
    image: scylladb/scylla:5.1
    ports:
      - "9042:9042" #Default
      - "9142:9142" #SSL
      - "10000:10000" #Scylla REST API
      - "9180:9180" #Prometheus API
    volumes:
      #- ./data/micro-data:/var/lib/scylla
      # - ./scylla.yaml:/etc/scylla/scylla.yaml
    restart: always
```

## Cài đặt NATS Server

NATS Server cũng cung cấp nhiều cách để bạn có thể sử dụng NATS. Bạn có thể tham khảo tại [trang cài đặt](https://docs.nats.io/running-a-nats-service/introduction/installation) của NATS.

### Cài đặt trực tiếp

```shell
wget https://github.com/nats-io/nats-server/releases/download/v2.9.14/nats-server-v2.9.14-linux-amd64.tar.gz
tar -xvf nats-server-v2.9.14-linux-amd64.tar.gz
./nats-server-v2.9.14-linux-amd64/nats-server  -D -p 4222 -js -n jetstream -sd jetstream
```

### Cài đặt bằng docker

#### Cài bằng docker run

```shell
docker run --network host -p 4222:4222 nats -js
```

#### Cài bằng docker compose

```xml
version: '3'
services:
  nats:
    image: nats:2.9.14
    command: -js -m 8222
    restart: always
    ports:
      - 4222:4222
```

## Cài đặt Scyna 

### Engine
[TBD]

### Example
[TBD]
