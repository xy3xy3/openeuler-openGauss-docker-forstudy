用于数据库实验，基于openeuler打包的openGauss，docker

仅限于自己测试，生成环境使用可能并不安全

默认可外部链接的账号`superuser`,默认密码`OGSql@123`

构建镜像
```
docker build -t opengauss:6.0.0-openEuler .
```

运行镜像
```
docker run -d -p 5432:5432 --name opengauss-container opengauss:6.0.0-openEuler
```