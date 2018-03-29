## Docker笔记
### Docker相关报错信息及解决方法
1.  创建registry后，push镜像到私有仓库，报错：x509: cannot validate certificate for <ipaddress> because it doesn't contain any IP SANs

    分析：this is not a valid SSL cert, because it doesn't match the name (in this case, IP) being used to connect.
    - Connect by name (either implement DNS, or if your environment is very static, add the hostnames to /etc/hosts on the prometheus server)
    - Set insecure_skip_verify
    - Reissue the certs to include an IP SAN for the host's IP
    <a href="https://github.com/prometheus/prometheus/issues/1654">点击查看</a>
 
 ---
 2. 生成key和crt证书
        
    详情请看此<a href="https://note.youdao.com/share/?id=7296f906578560187364e544f66419ff&type=notebook#/EA9C98260F784BF486F262F7BF721FB8">教程</a>

        ```
        [weshzhu@weshzhu ~]$ openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout registry.key -out registry.crt
        ```
---
3. 登录\pull\push 到Docker出错
        > Error response from daemon:    Get https://ipaddress/v2/: dial tcp ipaddress:443: getsockopt: connection refused

---
4. Centos 安装Docker
       
       1. 备份内核

           ` mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak.$(date +%Y%m%d)`
       2. 更新

           `yum -y update -q`
       3. 安装docker
        
            ```
            yum remove docker docker-common docker-selinux docker-engine
            yum install -y yum-utils device-mapper-persistent-data lvm2
            yum-config-manager --add-repo http://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
            yum install -y docker-ce-17.12.0.ce
            usermod -a -G docker $USER
            ```
        
        4. 添加用户到docker组里
        
            ```
            #将用户添加到docker组里
            [weshzhu@weshzhu ~]$ sudo gpasswd -a ${USER} docker
            #重启docker
            [weshzhu@weshzhu ~]$ sudo service docker restart
            [weshzhu@weshzhu ~]$ docker images
            Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.35/images/json: dial unix /var/run/docker.sock: connect: permission denied
            ```
            会报如上错误,此时需要修改/var/run/docker.sock权限:

            `sudo chmod a+rw /var/run/docker.sock`
        
        5. 安装镜像
           `docker pull ngix`
           OK 成功了！

---
5. CentOs中docker 安装私有仓库，并通过https方式上传镜像

- 安装仓库registry, Tag为2 
    ```
    [weshzhu@weshzhu ~]$ docker pull registry:2
    2: Pulling from library/registry
    Digest: sha256:672d519d7fd7bbc7a448d17956ebeefe225d5eb27509d8dc5ce67ecb4a0bce54
    Status: Image is up to date for registry:2

    ```
    查看仓库，此时先不启动容器。
    ```
    [weshzhu@weshzhu certs]$ docker images
    REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
    registry              2                   d1fd7d86a825        4 weeks ago         33.3MB
    ```


- 通过OpenSSL工具生成自签名的证书，后面将用于对请求进行校验
    
    对于证书以及OpenSSL, 请移目<a href="http://www.cnblogs.com/guogangj/p/4118605.html">那些证书相关的玩意儿</a>

    首先找到OpenSSL工具配置文件openssl.cnf，对于Centos,目录在/etc/pki/tls/中
    ```
    [weshzhu@weshzhu ~]$ cd /etc/pki/tls/
    [weshzhu@weshzhu tls]$ ll
    total 12
    lrwxrwxrwx. 1 root root    49 Jan 26 19:10 cert.pem -> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
    drwxr-xr-x. 2 root root   193 Feb  7 21:42 certs
    drwxr-xr-x. 2 root root    74 Jan 26 19:10 misc
    -rw-r--r--. 1 root root 10955 Feb  7 20:12 openssl.cnf
    drwxr-xr-x. 2 root root     6 Aug  4  2017 private
    ```
    编辑openssl.cnf,在[v3_ca]下面添加：subjectAltName = IP:域名|IP地址

    ```
    [ v3_ca ]
    subjectAltName = IP:172.10.15.110
    ```
    否则将会报错：
    ```
    x509: cannot validate certificate for <ipaddress> because it doesn't contain any IP SANs
    ```
    这是因为在证书中，要包含一些信息，比如国家、机构等等，好像访问的私有仓库ip或者域名必须要有，否则不予通过，就会报上面的错误。如果有读者发现此处有错误，请在下方提出。谢谢！

    修改完openssl配置文件后，可以生产私有证书，要记住生成证书的目录，后面会用到。
    ```
    [weshzhu@weshzhu certs]$ sudo openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout registry.key -out registry.crt
    [sudo] password for weshzhu: 
    Generating a 2048 bit RSA private key
    .................................................................................................................................................+++
    .........................................................................................................................................................+++
    writing new private key to 'registry.key'
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [XX]:       #该处让交互输入国家、省等，可以直接Enter跳过
    State or Province Name (full name) []:
    Locality Name (eg, city) [Default City]:
    Organization Name (eg, company) [Default Company Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (eg, your name or your server's hostname) []:172.10.15.110  #该处输入私有仓库的ip地址或者域名
    Email Address []:
    ```
    当然，上面私有证书生成，可以通过`-subj`指定证书信息：
    ```
     sudo openssl req -subj '/C=CN/ST=BeiJing/L=HaiDian/CN=<Ipaddress> ' -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout registry.key -out
     # -subj 通过C指定国家、ST指定省份、L指定区、CN指定域名或者IP
    ```
    此方法并没有经过试验，请感兴趣的童靴验证一下。

    可以看到在当前目录中，有*.crt 和 *.key文件
    ```
    [weshzhu@weshzhu certs]$ ll
    total 8
    -rw-r--r--. 1 root root 1306 Feb  8 15:04 registry.crt
    -rw-r--r--. 1 root root 1704 Feb  8 15:04 registry.key
    ```

- 将生成的私有证书追加到系统的证书管理文件中，否则后面push和login和pull时会报如下错误：
    ```
    [weshzhu@weshzhu certs]# cat ./registry.crt >> /etc/pki/tls/certs/ca-bundle.crt
    ```

    ```
    未cat到系统的crt文件中
    [weshzhu@weshzhu ~]$ docker push 192.168.0.123/rabbitmq:3.7
    The push refers to repository [192.168.0.123/rabbitmq]
    Get https://<IpAddress>/v2/: x509: certificate signed by unknown authority
    ```
- 重启docker, 该步骤一定不要省略，否则有可能加载私钥失败
  systemctl restart docker

- 启动私有仓库镜像 registry
    注意：在启动时，有参数需要配置
    ```
    docker run -d -p 443:443 --name registry -v /deploy/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key registry:2
    ```
    -d  后台运行
    -p 443:443 将容器的端口443映射到主机的443端口
    --name 给容器起个名字 registry
    -v /deploy/certs:/certs 主机的目录/deploy/certs映射到容器的/certs ，目的是将生成的私有证书映射到容器中
    -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt 指定TLS协议验证的证书目录：该目录为容器的registry.crt所在的目录
    -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key 指定TLS协议使用的key的目录：该目录为容器的registry.key所在的目录
- 上面的步骤按顺序操作完成后，可以尝试docker push一个镜像到私有仓库中。
   查看docker安装了哪些镜像：
   ```
   [weshzhu@weshzhu certs]$ docker images
    REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
    registry              2                   d1fd7d86a825        4 weeks ago         33.3MB
    nginx                 latest              3f8a4339aadd        6 weeks ago         108MB

   ```    
   将上传的镜像重新Tag一下：
   ```
   docker tag nginx[:tag] [ipaddress]/nginx
   #:tag 为镜像的tag,如果该镜像的tag为latest，则可以省略
   #[ipaddress] 为私有仓库的ip地址或域名，也就是上面步骤在openssl中添加的`subjectAltName = IP:172.10.15.110`地址或域名
   
   ```
   查看镜像，发现多了一个以ip地址为开头的image
   ```
   [weshzhu@weshzhu certs]$ docker images
    REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
    registry              2                   d1fd7d86a825        4 weeks ago         33.3MB
    192.168.0.123/nginx   latest              3f8a4339aadd        6 weeks ago         108MB
    nginx                 latest              3f8a4339aadd        6 weeks ago         108MB

   ```
    此时push该带ip地址的镜像到私有仓库中
    ```
    [weshzhu@weshzhu certs]# docker push 172.10.15.110/nginx
    The push refers to repository [172.10.15.110/nginx]
    a103d141fc98: Pushed 
    73e2bd445514: Pushed 
    2ec5c0a4cb57: Pushing [===========================================>       ]   48.2MB/55.26MB

    ```
    ```
    [weshzhu@weshzhu certs]# docker push 172.10.15.110/nginx
    The push refers to repository [172.10.15.110/nginx]
    a103d141fc98: Pushed 
    73e2bd445514: Pushed 
    2ec5c0a4cb57: Pushed 
    latest: digest: sha256:926b086e1234b6ae9a11589c4cece66b267890d24d1da388c96dd8795b2ffcfb size: 948
    ```
    查看私有仓库有哪些镜像：
    `curl -X GET https://172.10.15.110:443/v2/_catalog`
    `{"repositories":["aios-nginx","api","inception_serving-ubuntu16.04","k8s-dns-dnsmasq-nanny-amd64","k8s-dns-kube-dns-amd64","k8s-dns-sidecar-amd64","mongo","pod","tensorflow","tinycore-node","web"]}`
    

### 简易教程
1. 覆盖掉目录/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
	cp /home/zsd/tls-ca-bundle.pem /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem

2. 删除/certs/中的registry.crt 和 registry.key
	rm /certs/registry.*

3. 删除docker中的registry容器
	docker stop registry
	docker rm registry

4. 修改openssl.cnf文件
	vi /etc/pki/tls/openssl.cnf
	在[v3_ca]下面添加 subjectAltName = IP:172.28.8.124
	
5. openssl生成私有证书
	openssl req [-subj "/C=CN/ST=BeiJing/L=Dongcheng/CN=172.28.8.124"] -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout registry.key -out registry.crt
	openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout registry.key -out registry.crt
	
6. 将生成证书内容追加到该服务器上的证书存放目录的内置信任的证书
	cat /certs/registry.crt >> /etc/pki/tls/certs/ca-bundle.crt
	
7. 重启docker
	systemctl restart docker

8. 运行registry
	docker run -d -p 443:443 --name registry -v /deploy/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key registry:2
	
9. push镜像到registry
	docker push 172.28.8.124/nginx
	
	a. Get https://172.28.8.124/v2/: x509: cannot validate certificate for 172.28.8.124 because it doesn't contain any IP SANs  未操作第4步
	b. Get https://<IpAddress>/v2/: x509: certificate signed by unknown authority  #未操作第6步