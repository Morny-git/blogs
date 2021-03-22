#### 反向代理

项目只买一个域名，然后访问多态服务器。需要做的是让域名对应的服务器具有代理反转的功能。这里的服务器做为代理，让客户端能通过访问服务器来访问其他服务器。

```
location /one/ {
  proxy_pass       http://server:port/two/;
  proxy_redirect   default;
}
```

客户端访问server/one  ，服务端实际访问serverport/two.返回给client 的也是serverport/two

#### 重定向

```
location /one/ {
  proxy_pass       http://server:port/two/;
  proxy_redirect   http://server:port/two/   /one/;
}
```

客户端访问server/one  ，服务端实际访问serverport/two.返回给client 的也是serve/one



#### 负载均衡

- 轮询。nginx负载默认的方式。按照时间先后顺序分配到不同的服务器

  ```
  upstream test-server{
  	server	localhost:1001;
  	server	localhsot:1002;
  }
  ```

- 权重。指定每个服务器的权重比例.适用于服务器性能不同

  ```
  upstream test-server{
  	server	localhost:1001 weight=1;
  	server	localhsot:1002 weight=2;
  }
  ```

- iphash 每个请求根据访问的ip的hash结果分配，这样每个访问固定在一台机器上。和weight配合

  ```
  upstream test-server{
   	ip_hash; 
  	server	localhost:1001 weight=1;
  	server	localhsot:1002 weight=2;
  }
  ```

- 最少连接。将请求分配到连接数最少的服务上

  ```
  upstream test-server{
   	least_conn; 
  	server	localhost:1001 weight=1;
  	server	localhsot:1002 weight=2;
  }
  ```

- fair.按照后端服务器的响应时间来分配请求，响应时间短的优先分配

  ```
  upstream test-server{
   	fair; 
  	server	localhost:1001 weight=1;
  	server	localhsot:1002 weight=2;
  }
  ```

  