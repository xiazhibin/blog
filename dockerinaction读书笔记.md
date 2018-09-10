## --link outside_c_name:inside_name

  `docker run --rm --expose 5000 --expose 3333/udp mysql`
  `docker run --rm --link mysql:db app`
  
  那么就会在`container_b`里面设置一下环境变量，形式如
  ```bash
  <ALIAS>_PORT_<PORT NUMBER>_<PRO TOCOL TCP or UDP>
  <ALIAS>_PORT_<PORT NUMBER>_<PRO TOCOL TCP or UDP>_PORT
  <ALIAS>_PORT_<PORT NUMBER>_<PRO TOCOL TCP or UDP>_ADDR
  <ALIAS>_PORT_<PORT NUMBER>_<PRO TOCOL TCP or UDP>_PROTO
  ```
  
  所以container `app`会有一下环境变量：
  
 ```bash
 DB_PORT=udp://172.17.0.2:333
 
 DB_PORT_5000_TCP=tcp://172.17.0.2:5000
 DB_PORT_5000_TCP_PORT=50000
 DB_PORT_5000_TCP_ADDR=172.17.0.2
 DB_PORT_5000_TCP_PROTO=tcp
 
 DB_PORT_3333_UDP=udp://172.17.0.2:3333
 DB_PORT_3333_UDP_PORT=3333
 DB_PORT_3333_UDP_ADDR=172.17.0.2
 DB_PORT_3333_UDP_PROTO=udp
 
 DB_NAME=/random_string/db
 ```
## --net=host/bridge/none

- host 没有网络容器，并且对主机网络有完全的访问权。一般都不适用，除非是`--rm`这样的container，用一次执行以下命令
- bridge 最常见的网络容器，能够与`docker 0`接口进行通信，常见`5.4章`
- none 不允许外部链接，属于closed 网络
