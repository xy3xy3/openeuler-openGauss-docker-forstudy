用于数据库实验，基于openeuler打包的openGauss，docker

仅限于自己测试，生成环境使用可能并不安全

默认可外部链接的账号`superuser`,默认密码`OGSql@123`,默认密码可以构建镜像前搜索`OGSql@123`修改

官方下载链接：https://opengauss.org/zh/download/

# 轻量版

在`openGauss-Lite-6.0.0-openEuler22.03-x86_64`文件夹

构建镜像
```
docker build -t opengauss:6.0.0-openEuler .
```

运行容器（未持久化）
```
docker run -d -p 5432:5432 --name opengauss-container opengauss:6.0.0-openEuler
```

运行容器（有持久化）
本指令为了设置权限初始化似乎比较麻烦，有优化方法可以pr~
`/www/wwwroot/opengauss/data`改为你实际想设置的存储路径
```
docker run -d --name temp-opengauss opengauss:6.0.0-openEuler
docker cp temp-opengauss:/opt/openGauss/data /www/wwwroot/opengauss/data
docker stop temp-opengauss
docker rm temp-opengauss
sudo chmod -R 700 /www/wwwroot/opengauss/data
sudo chown -R 1000:1000 /www/wwwroot/opengauss/data
docker run -d \
    -p 5432:5432 \
    --name opengauss-container \
    -v /www/wwwroot/opengauss/data:/opt/openGauss/data \
    opengauss:6.0.0-openEuler
```

docker仓库地址：https://hub.docker.com/repository/docker/xy3666/opengauss/general

如果你不想自己构建
```
docker pull xy3666/opengauss:latest
```

之后对上面的指令替换docker名字即可

```
docker run -d -p 5432:5432 --name opengauss-container xy3666/opengauss:latest
```

自定义密码，通过`GAUSS_SUPERUSER_PASSWORD`环境变量设置
```
docker run -d -p 5432:5432 --name opengauss-container -e GAUSS_SUPERUSER_PASSWORD=NewPassword@123 xy3666/opengauss:latest
```

带挂载目录版指令

```
docker run -d --name temp-opengauss xy3666/opengauss:latest
docker cp temp-opengauss:/opt/openGauss/data /www/wwwroot/opengauss/data
docker stop temp-opengauss
docker rm temp-opengauss
sudo chmod -R 700 /www/wwwroot/opengauss/data
sudo chown -R 1000:1000 /www/wwwroot/opengauss/data
docker run -d \
    -p 5432:5432 \
    --name opengauss-container \
    -v /www/wwwroot/opengauss/data:/opt/openGauss/data \
    xy3666/opengauss:latest
```

主机链接
先安装链接客户端
```
sudo apt install postgresql-client
```

测试链接
```
psql -h 127.0.0.1 -d postgres -p 5432 -U superuser
```

# 企业版

