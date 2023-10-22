> 跟着提示做即可

### Referer

HTTP来源地址是HTTP表头的一个字段，用来表示从哪儿链接到目前的网页，采用的格式是URL。换句话说，借着HTTP来源地址，目前的网页可以检查访客从哪里而来，这也常被用来对付伪造的跨网站请求。 而dereferer则是将HTTP来源地址信息剥离，所以网站将无法识别访客从何而来。


### X-Forwarded-For

X-Forwarded-For一般是每一个非[透明代理](https://www.baidu.com/s?wd=%E9%80%8F%E6%98%8E%E4%BB%A3%E7%90%86&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)转发请求时会将上游服务器的IP地址追加到X-Forwarded-For的后面，使用英文逗号分割

简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项，正如上面所述,当你使用了代理时,web服务器就不知道你的真实IP了,为了避免这个情况,代理服务器通常会增加一个叫做x_forwarded_for的头信息,把连接它的客户端IP(即你的上网机器IP)加到这个头信息里,这样就能保证网站的web服务器能获取到真实IP

```HTTP
X-Forwarded-For: 1.1.1.1, 2.2.2.2, 3.3.3.3
```

### X-Real-IP

X-Real-IP一般是最后一级代理将上游IP地址添加到该头中

当有多个代理时候，可以在第一个反向代理上配置“proxy_set_header X-Real-IP $remote_addr” 获取真实客户端IP；


X-Forwarded-For是多个IP地址，而X-Real-IP是一个
