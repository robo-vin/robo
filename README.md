### Robo.Vin

#### Go开发与运行环境配置

##### 1. 安装Go

下载：https://golang.google.cn/dl/，解压到/usr/local目录下，设置环境变量

```
tar -zxvf go1.17.2.linux-amd64.tar.gz
sudo mv go /usr/local/
nano ~/.bashrc，添加export PATH=$PATH:/usr/local/go/bin
source ~/.bashrc
go version
```
##### 2. 初始化

```
cd /home/adchen/goprojects/robovin
go mod init robo.vin

# 大陆访问受限，使用国内代理
go env -w GOPROXY=https://goproxy.cn,direct
go mod tidy

# 任意目录执行go run 脚本，设置GOWORK环境变量
# 1. 创建go.work文件
nano /home/adchen/goprojects/go.work
# 写入内容

```

##### 3. 安装依赖

```
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
go get -u github.com/go-redis/redis/v9
```