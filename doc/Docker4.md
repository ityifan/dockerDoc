# 1、管理防火墙

```powershell
 firewall-cmd --permanent --add-port=8080-8085/tcp  #新增端口
 
 firewall-cmd --reload #生效设置
 
 firewall-cmd --permanent --list-ports #查询开放的端口
 
 firewall-cmd --permanent --remove-port =8080-8085/tcp #关闭端口
 
 firewall-cmd --permanent --list-services #查看使用物联网的的程序 
```

# 2、安装docker

```powershell
yum -y update #更新yum源
yum install -y docker #安装docker

service docker start  #启动   
service docker stop  #关闭
service docker restart # 重启
```


