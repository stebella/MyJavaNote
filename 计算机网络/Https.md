基于HTTP协议，通过SSL或TLS提供加密处理数据、验证对方身份以及数据完整性保护

通过抓包可以看到数据不是明文传输，而且HTTPS有如下特点：

1. 内容加密：采用混合加密技术，中间者无法直接查看明文内容
2. 验证身份：通过证书认证客户端访问的是自己的服务器
3. 保护数据完整性：防止传输的内容被中间人冒充或者篡改