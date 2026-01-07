1,安装  
install keepalived haproxy psmisc -y  
2,haproxy配置  
`vi /etc/haproxy/haproxy.cfg`  
注意：将 <IP address> 替换为 KubeSphere 集群各控制平面节点的私网 IP 地址    
2.1,重启、开机自启   
`systemctl restart haproxy`  
`systemctl enable haproxy`  
3,haproxy配置  
`vi /etc/keepalived/keepalived.conf`  
将以下参数替换为实际值：  
<NIC>                  当前负载均衡器的网卡名称。  
<source IP address>    当前负载均衡器的 IP 地址。  
<peer IP address>      另一台负载均衡器的 IP 地址。  
<floating IP address>   用作浮动 IP 地址的虚拟 IP 地址。  
  
  
## 问题1，启动服务报错  
Jan 07 14:25:18 k8s-hap01 haproxy[16058]: [ALERT]    (16058) : config : parsing [/etc/haproxy/haproxy.cfg:4] : 'pidfile' already specified. Continuing.  
Jan 07 14:25:18 k8s-hap01 haproxy[16058]: [WARNING]  (16058) : config : parsing [/etc/haproxy/haproxy.cfg:28] : backend 'kube-apiserver' : 'option tcplog' directive is ignored in backends.  
Jan 07 14:25:18 k8s-hap01 haproxy[16058]: [ALERT]    (16058) : Binding [/etc/haproxy/haproxy.cfg:21] for frontend kube-apiserver: cannot bind socket (Permission denied) for [0.0.0.0:6443]  
Jan 07 14:25:18 k8s-hap01 haproxy[16058]: [ALERT]    (16058) : [/usr/sbin/haproxy.main()] Some protocols failed to start their listeners! Exiting.  
  
分析：  
由 SELinux 引起。SELinux 默认只允许特定服务绑定特定的端口。  
  
解决：  
1，临时或永久关闭SELinux(推荐)  
临时：`setenforce 0`  
永久：`sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config`。需要重启系统  
2，将 6443 端口添加到 SELinux 允许 HAProxy 使用的端口列表中  
# 安装 semanage 工具（如果没装的话）  
yum install -y policycoreutils-python-utils  
# 添加端口  
semanage port -a -t http_port_t -p tcp 6443  
# 如果已经存在，使用 -m 修改  
semanage port -m -t http_port_t -p tcp 6443  