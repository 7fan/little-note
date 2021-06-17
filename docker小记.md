## PHP镜像
### 如何安装扩展

#### 初始化安装目录
```
$ docker-php-source extract
```
此操作实际执行为：将镜像中预置的PHP源码解压至`/usr/src/php`目录中

#### 查看已有扩展
```
$ php -m
# 或者
$ php -i
```

#### 启用扩展
```
$ docker-php-ext-enable xxx
```

#### 安装扩展4法
##### 安装核心扩展
> 即`/usr/src/php/ext`中已有的扩展
```
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd
```

##### 通过pecl安装
```
RUN pecl install redis-5.1.1 \
    && pecl install xdebug-2.8.1 \
    && docker-php-ext-enable redis xdebug

RUN apt-get update && apt-get install -y libmemcached-dev zlib1g-dev \
    && pecl install memcached-2.2.0 \
    && docker-php-ext-enable memcached
```

##### 通过源码安装
```
RUN curl -fsSL 'https://xcache.lighttpd.net/pub/Releases/3.2.0/xcache-3.2.0.tar.gz' -o xcache.tar.gz \
    && mkdir -p /tmp/xcache \
    && tar -xf xcache.tar.gz -C /tmp/xcache --strip-components=1 \
    && rm xcache.tar.gz \
    && docker-php-ext-configure /tmp/xcache --enable-xcache \
    && docker-php-ext-install /tmp/xcache \
    && rm -r /tmp/xcache
```

##### 通过第三方工具安装
```
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions && sync && \
    install-php-extensions gd xdebug
```

---

## 设置时区
### alpine
```
apk add --no-cache tzdata && cp -r -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### other
```
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' > /etc/timezone && \
    echo "[Date]\ndate.timezone=Asia/Shanghai" > /usr/local/etc/php/conf.d/timezone.ini

```

---

## mysql容器
```
$ docker pull mysql:8

$ mkdir mysql_data

$ docker run -dlt --name my_mysql -v mysql_data:/var/lib/mysql -e MYSQL_DATABASE=database_name -e MYSQL_USER=username -e MYSQL_PASSWORD=password -e MYSQL_ROOT_PASSWORD=root_password -p 3306:3306 --restart unless-stopped mysql
```

---

## 容器自启动
### 启动时指定
```
$ docker run -dit --restart unless-stopped --name 容器名 镜像名
```

### 运行时设置
```
$ docker update --restart unless-stopped 容器名
```

---

## 参考
[PHP官方镜像](https://hub.docker.com/_/php)
[第三方工具安装PHP扩展](https://github.com/mlocati/docker-php-extension-installer)
[容器自启动](https://docs.docker.com/config/containers/start-containers-automatically/)
