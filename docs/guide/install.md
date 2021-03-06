## 前端

```shell
# 环境依赖
# 1.  node ^v11.2.0 https://nodejs.org/zh-cn/download/
# 2.  npm ^6.4.1
git clone https://github.com/hyperf-admin/hyperf-admin-frontend.git
cd hyperf-admin-frontend
npm i
npm run dev
```

!> 请根据实际情况修改`vue.config.js`中的代理 `proxy.target`地址

```shell
# 打包
npm run build:prod
npm run build:test
```

## 后端

```shell
# 环境依赖 php ^7.1 composer swoole 
composer create-project hyperf/hyperf-skeleton hyperf-admin
cd hyperf-admin
# 移除日志配置, admin 底层已配置
rm config/autoload/logger.php
# 设置基础环境配置 .env 具体见下方
# hyperf-admin 为分包的模式, 此处引入的是完整仓库, 实际项目请按需引入
composer require hyperf-admin/hyperf-admin
composer i
# 启动 热重启参考 https://github.com/daodao97/hyperf-watch
composer watch
```

`.evn`
```bash
APP_NAME=hyperf-admin
ENV=dev

REDIS_HOST=localhost
REDIS_AUTH=(null)
REDIS_PORT=6379
REDIS_DB=0

HYPERF_ADMIN_DB_HOST=localhost
HYPERF_ADMIN_DB_PORT=3306
HYPERF_ADMIN_DB_NAME=hyperf_admin
HYPERF_ADMIN_DB_USER=root
HYPERF_ADMIN_DB_PWD=root

LOCAL_DB_HOST=localhost
```

## nginx配置

```nginx
upstream backend {
    server 127.0.0.1:9511;
}

server {
    listen 80;
    server_name hyperf-admin.com;
    index index.html;
    root /opt/www/hyperf-admin-front/dist;
    access_log /usr/local/var/log/nginx/hyperf-admin.access.log;
    error_log /usr/local/var/log/nginx/hyperf-admin.error.log;

    location ~ /api/(.*) {
        proxy_http_version 1.1;
        proxy_set_header Connection "keep-alive";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host hyperf-admin.com;
        proxy_pass http://backend/$1$is_args$args;
    }

    location / {
        root /opt/www/hyperf-admin-front/dist/default;
        index index.html;
    }

    location ~ /(.*) {
        set $module $1;
        if ($module ~* '^$') {
            set $module default;
        }
        try_files $uri $uri/ /$module/index.html;
    }
}
```
