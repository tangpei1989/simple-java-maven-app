# 内存泄露问题测定

命令查询采集是否有单独的容器运行
```
docker ps | grep tomcatcollect
```
- [ ] 表示存在内存溢出的风险

![image](https://note.youdao.com/yws/public/resource/074ca7c423e9d4692373c4d731fb319b/xmlnote/BCFEB60B48434DD982309BD9B7A28715/439)
- [x] 表示已经经过排查

![image](https://note.youdao.com/yws/public/resource/074ca7c423e9d4692373c4d731fb319b/xmlnote/6AA8F0BF8E44408696F9257256069C58/174)



# 解决办法
(**脚本中ip地址请替换成项目上的应用服务器ip**)

1. 新建tomcatcollect镜像
```
docker run -d -m 8G -p 8098:8080 --log-driver=none -v /var/log/dripping/tomcatcollect:/usr/local/tomcat/logs -e config_url=192.168.40.126:2222 -e profile=dev -l company=ewell -l product=dripping --name tomcatcollect tomcat:ewell.20171026
```
![image](https://note.youdao.com/yws/public/resource/074ca7c423e9d4692373c4d731fb319b/xmlnote/C7683777CBEC4A4EBDE86DDDC5D73A39/185)
2. 从原tomcat镜像中复制collect程序包
```
mkdir collect-change;for var in $(docker exec tomcat find /usr/local/tomcat/webapps -name '*collect.war');do docker cp tomcat:$var collect-change;done;ls collect-change
```
![image](https://note.youdao.com/yws/public/resource/074ca7c423e9d4692373c4d731fb319b/xmlnote/21507082FED546C1AC7170335340AE8B/173)
3. 将collect程序包复制到tomcatcollect镜像中
```
for var in $(find collect-change/ -type f);do docker cp $var tomcatcollect:/usr/local/tomcat/webapps;sleep 10;done
```
4. 访问tomcatcollect的包管理网址,确认collect程序包正常运行(识别红框部分均是true) 

网址访问

    http://[应用服务器ip]:8098/manager/html
    登录信息 admin/ewell
        
![image](https://note.youdao.com/yws/public/resource/074ca7c423e9d4692373c4d731fb319b/xmlnote/5C4E88EA54E6463793F6707009D05BA2/199)
5. 访问tomcat的包管理网址，删除采集的运行包

网址访问

    http://[应用服务器ip]:8080/manager/html
    登录信息 admin/ewell
