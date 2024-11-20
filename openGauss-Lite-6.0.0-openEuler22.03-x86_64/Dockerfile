# 使用 openEuler 22.03 作为基础镜像
FROM openeuler/openeuler:22.03
# 重命名 openEuler.repo 为 openEuler.repo.bak
RUN mv /etc/yum.repos.d/openEuler.repo /etc/yum.repos.d/openEuler.repo.bak

# 将本地的 tinghua.repo 复制到容器中替换默认的 yum 源
COPY tinghua.repo /etc/yum.repos.d/

# 更新软件包索引，并安装必要的依赖项
RUN yum makecache && \
    yum install -y \
    bzip2 \
    python3 \
    readline-devel \
    libaio-devel \
    tar \
    findutils \
    sudo \
    cjson \
    libcgroup

# 设置必要的环境变量
ENV GAUSSHOME=/opt/openGauss
ENV GAUSSDATA=/opt/openGauss/data
ENV PATH=$GAUSSHOME/install/bin:$PATH
ENV LD_LIBRARY_PATH=/opt/openGauss/install/lib:/usr/lib:/usr/lib64
ENV GAUSS_SUPERUSER_PASSWORD=OGSql@123


# 创建安装目录
RUN mkdir -p /opt/openGauss

# 创建非 root 用户并赋予必要权限
RUN useradd -m gauss && \
    chown -R gauss:gauss /opt/openGauss

# 切换为非 root 用户
USER gauss

# 设置用户级别的 PATH 和 LD_LIBRARY_PATH
RUN echo 'export PATH=$GAUSSHOME/install/bin:$PATH' >> /home/gauss/.bashrc && \
    echo 'export LD_LIBRARY_PATH=/opt/openGauss/install/lib:/usr/lib:/usr/lib64' >> /home/gauss/.bashrc

# 将 openGauss 安装包拷贝到容器中
COPY --chown=gauss:gauss openGauss-Lite-6.0.0-openEuler22.03-x86_64.tar.gz /opt/openGauss/

# 解压 openGauss 安装包
RUN tar -zxf /opt/openGauss/openGauss-Lite-6.0.0-openEuler22.03-x86_64.tar.gz -C /opt/openGauss/

# 切换为非 root 用户并安装 openGauss
USER gauss
RUN echo $GAUSS_SUPERUSER_PASSWORD | sh /opt/openGauss/install.sh --mode single -D /opt/openGauss/data -R /opt/openGauss/install --start

# 修改 postgresql.conf 文件，允许监听所有 IP 地址，并设置密码加密类型为 md5
RUN sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" /opt/openGauss/data/postgresql.conf && \
    sed -i "s/#password_encryption_type = 2/password_encryption_type = 0/" /opt/openGauss/data/postgresql.conf  # 0 为 md5, 2 为 sha256

# 修改 pg_hba.conf 文件，允许所有 IP 地址通过密码连接，并使用 md5 进行身份验证
RUN echo "host    all             all             0.0.0.0/0               md5" >> /opt/openGauss/data/pg_hba.conf

# 切换回 root 用户并更新共享库缓存
USER root
RUN echo "/opt/openGauss/install/lib" >> /etc/ld.so.conf.d/opengauss.conf && \
    ldconfig

# 初始化数据库并创建超级用户
USER gauss
RUN /opt/openGauss/install/bin/gaussdb -D /opt/openGauss/data & \
    sleep 10 && \
    gsql -d postgres -c "CREATE USER superuser WITH PASSWORD '$GAUSS_SUPERUSER_PASSWORD';" && \
    gsql -d postgres -c "ALTER USER superuser WITH SUPERUSER CREATEROLE CREATEDB;" && \
    gsql -d postgres -c "GRANT ALL PRIVILEGES ON DATABASE postgres TO superuser;" && \
    gsql -d postgres -c "GRANT USAGE ON SCHEMA public TO superuser;" && \
    gsql -d postgres -c "GRANT CREATE ON SCHEMA public TO superuser;"

# 暴露默认的数据库端口
EXPOSE 5432

# 启动数据库
CMD ["/opt/openGauss/install/bin/gaussdb", "-D", "/opt/openGauss/data"]
