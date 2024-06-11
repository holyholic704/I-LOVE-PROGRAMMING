# 同一域名部署多个 Vue 项目

## 需求

一般常用子域名来区分同一域名下的不同项目

- 输入 `http://a.xxx.com`，展示项目 a
- 输入 `http://b.xxx.com`，展示项目 b

很简单，适用于大部分需求，就是不适用我目前的一个需求，我的网站用的腾讯的免费的 SSL 证书只能绑定单个域名，要么再申请一个，要么就另想法子

目前构想中的前端项目有 3 个，意味着我得再申请 3 个域名，再依样画葫芦配置一下，其实也不复杂。但我还想尝试尝试别的方法，还有一个原因就是一个证书有效期只有 90 天，到期只能重新申请，我懒啊...

所以我的需求就是

- 输入 `http://xxx.com/a`，展示项目 a
- 输入 `http://xxx.com/b`，展示项目 b

## 前端

目前我有两个前端项目，一个没有做任何特殊处理，并且已上线，以下是新的前端项目，所以我的实际需求为

- 输入 `http://xxx.com`，展示主项目
- 输入 `http://xxx.com/test`，展示新项目

vue.config.js 中加入 publicPath

```javascript
module.exports = defineConfig({
  // 填写你想访问该项目的路径
  publicPath: '/test/'
})
```

添加 router 文件

```javascript
import VueRouter from "vue-router";

export default new VueRouter({
  // 与 vue.config.js 中的 publicPath 保持一致
  base: '/test/'
})
```

## nginx 配置

```bash
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  你的域名;

        location / {
            root   主项目地址;
            index  index.html index.htm;
        }

        location /test {
            # 注意使用二级路由的话，要用 alias
            alias  新项目地址;
            index  index.html index.htm;
            # 如果出现404，检查下面的配置是否正确
            try_files $uri $uri/ /test/index.html;
        }

        rewrite ^(.*)$ https://$host$1 permanent;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    # SSL配置，如没有SSL证书，可省略以下配置
    server {
        listen       443 ssl;
        server_name  你的域名;

        ssl_certificate      /usr/local/nginx/cert/你的证书.crt;
        ssl_certificate_key  /usr/local/nginx/cert/你的证书.key;

        ssl_session_cache    shared:SSL:50m;
        ssl_session_timeout  1d;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers  on;

        location / {
            root   主项目地址;
            index  index.html index.htm;
        }

        location /test {
            alias  新项目地址;
            index  index.html index.htm;
            try_files $uri $uri/ /test/index.html;
        }
    }
}
```

## 参考

- [一个域名部署多个vue项目的方法](https://blog.csdn.net/mynewdays/article/details/124465677)
- [nginx同域名下部署多个vue项目](https://blog.csdn.net/qq_35321405/article/details/96135145)
- [nginx配置同一域名同一端口下部署多个vue项目](https://blog.csdn.net/strggle_bin/article/details/137918013)
