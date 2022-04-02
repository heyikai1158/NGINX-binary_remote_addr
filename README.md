# NGINX-binary_remote_addr
使用NGINX进行接口限流

## 配置nginx.conf

#### 定义了一个 mylimit 缓冲区（容器），请求频率为每秒 1 个请求（nr/s）

limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
server {
    listen  90;
    location / {
        # nodelay 不延迟处理
        # burst 是配置超额处理,可简单理解为队列机制
        # 上面配置同一个 IP 没秒只能发送一次请求（1r/s），这里配置了缓存3个请求，就意味着同一秒内只能允许 4 个任务响应成功，其它任务请求则失败（503错误）
        limit_req zone=mylimit burst=3 nodelay;
        proxy_pass http://localhost:7070;
    }
}

上述的7070端口即需要监听的端口地址,该端口号需要在SPRING项目的application中定义

listen端口即和7070端口所绑定

在启动项目后不再直接访问spring中定义的7070端口，改向访问新定义的90端口

例如原先访问localhost:7070/方法名

现在访问localhost:90/方法名

## 修改“因超限制访问”而指向的503页面(其他错误页面修改方式可照搬)
server {
        listen  90;
        error_page 503 /503.html;
        location = /503.html {
                root /root/test3g/test3g;
        }
        location / {
                return 503;
        }
}

#### 实例如下

#处理单个指定错误

error_page 403 http://www.baidu.com;

#处理一系列指定错误

error_page 400 502 503 504 http://example.com/notfound.html;

按照上述配置后，如果发生404错误，就会跳转到百度首页。

#更改响应状态码

error_page 404 =200 /error.html;

再次访问，发生404错误，但是响应码为200，成功隐藏了实际响应码。
