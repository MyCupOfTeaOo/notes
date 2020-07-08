# 使用 tls 连接 rabbitmq(python client)

## 创建证书

```bash
git clone git@github.com:michaelklishin/tls-gen.git
cd tls-gen/basic && make
```
如果你需要加密码则
> ⚠️ 如果加了密码,rabbitmq.conf 千万要记得加 `ssl_options.password = 你的密码`
```bash
cd tls-gen/basic && make PASSWORD=你的密码
```
如果需要加CN
```bash
cd tls-gen/basic && make CN=你的CN
```

## 自我验证
### 查看rabbitmq的证书信息
`openssl s_client -connect localhost:5671 -cert .\result\client_certificate.pem -key .\result\client_key.pem -CAfile .\result\ca_certificate.pem
`

### 使用openssl验证
服务端
```bash
openssl s_server -accept 8443 -cert ./result/server_certificate.pem -key ./result/server_key.pem -CAfile ./result/ca_certificate.pem
```
客户端
```bash
openssl s_client -connect localhost:8443 -cert .\result\client_certificate.pem -key .\result\client_key.pem -CAfile .\result\ca_certificate.pem -verify 8 -verify_hostname 你的cn
```



## python代码
> ⚠️ 需要注意创建证书 `CN` 不要用 ip,不然会校验失败
### pika
```python
context = ssl.create_default_context(
    cafile="./result/ca_certificate.pem")
context.load_cert_chain("./result/client_certificate.pem",
                        "./result/client_key.pem",password="证书密码")
ssl_options = pika.SSLOptions(context, "证书cn")
conn_params = pika.ConnectionParameters(host="localhost",port=5671,
                                        ssl_options=ssl_options,credentials=pika.PlainCredentials("admin", "admin"))

```

### aio-pika
> aio-pika的源码中看了下不支持加password(除非你修改源码,或者每次运行输入password)
> aio-pika的源码中默认使用了host当做`CN`来验证,不过`CN`和`host`不一致就关闭验证(`no_verify_ssl=1`)

```python
connection = await aio_pika.connect_robust(
    host="localhost",
    port=5671,
    login="admin",
    password="admin",
    ssl=True,
    ssl_options=dict(
        cafile="./result/ca_certificate.pem",
        certfile="./result/client_certificate.pem",
        keyfile="./result/client_key.pem",
        no_verify_ssl=1
    ),
    loop=loop,
)

```