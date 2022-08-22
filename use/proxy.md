# ⚡ 反代配置

{% hint style="info" %}
**WEB服务端口默认为:9501**

**WS服务端口默认为:9502**
{% endhint %}

### 从零开始

#### 安装 nginxWebUI

安装podman

```bash
# centos
yum install podman -y
#ubuntu
sudo apt-get install podman
```

安装nginxWebUI

<pre class="language-bash"><code class="lang-bash"># centos
<strong>podman pull cym1102/nginxwebui:latest
</strong>
cd / &#x26;&#x26; mkdir home &#x26;&#x26; cd /home &#x26;&#x26; mkdir nginxWebUI

podman run -itd   -v /home/nginxWebUI:/home/nginxWebUI   -e BOOT_OPTIONS="--server.port=8080"   --privileged=true   --net=host   cym1102/nginxwebui:latest

#ubuntu 
<strong>sudo podman pull cym1102/nginxwebui:latest
</strong>
cd / &#x26;&#x26; mkdir home &#x26;&#x26; cd /home &#x26;&#x26; mkdir nginxWebUI

sudo podman run -itd   -v /home/nginxWebUI:/home/nginxWebUI   -e BOOT_OPTIONS="--server.port=8080"   --privileged=true   --net=host   cym1102/nginxwebui:latest</code></pre>

然后访问服务器IP:8080 进入可视化管理面板，如下图

![](<../.gitbook/assets/image (3).png>)

接下来点击 反向代理 进行创建

先反代WEB服务

![](<../.gitbook/assets/image (11).png>)

再反代WS服务

![](<../.gitbook/assets/image (22).png>)

提交后就能用了

### 宝塔反代

#### WEB服务

创建网站后，添加反向代理，目标URL为:http(s)://IP:WEB服端口号(http://127.0.0.1:9501)

![](<../.gitbook/assets/image (20).png>)

#### WS服务

再新建一个网站，绑定域名后点击配置文件进行修改

![](<../.gitbook/assets/image (12).png>)

在server模块前加上以下代码,如上图所示

```nginx
upstream sf_websocket {
    #Super-Forum WEBSOCKET Server 的 IP 及 端口
    ip_hash;
    # 服务器IP:WS服务端口号
    server 127.0.0.1:9502;
}
```

然后在server模块里，最后加上下面一行代码,如图所示

![](<../.gitbook/assets/image (2).png>)

```nginx
location / {
        # WebSocket Header
        proxy_http_version 1.1;
        proxy_set_header Upgrade websocket;
        proxy_set_header Connection "Upgrade";
        
        # 将客户端的 Host 和 IP 信息一并转发到对应节点  
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
    
        # 客户端与服务端无交互 60s 后自动断开连接，请根据实际业务场景设置
        proxy_read_timeout 60s ;
        
        # 执行代理访问真实服务器
        proxy_pass http://sf_websocket;
    }
```
