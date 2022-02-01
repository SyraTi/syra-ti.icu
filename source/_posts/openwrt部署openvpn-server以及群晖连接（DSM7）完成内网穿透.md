---
title: openwrt部署openvpn server以及群晖连接（DSM7）完成内网穿透
date: 2022-01-20 15:46:11
tags:
---
## 前情提要
公司里放了一台黑裙，但是没有公网IP，想到用openvpn拨到家里的路由器，再进行端口转发达到内网穿透，公网访问的目的。

**总之的总之三步走，需要openwrt上有server，以及群晖能够有client进行拨入，最后配置端口转发**
___

## 1. openwrt部署openvpn server
>openwrt官方提供了详实的文档，如下
[[OpenWrt Wiki] OpenVPN server](https://openwrt.org/docs/guide-user/services/vpn/openvpn/server)

但是这个版本生成的配置，因为用到了``<tls-crypt-v2>``，群晖没法识别。可以使用下面这个`` previous version of OpenWrt`` 的旧版本。
>![在这里插入图片描述](https://img-blog.csdnimg.cn/8196fece92354dfda722165d47ba5c54.png)
链接在这里↓
[[OpenWrt Wiki] OpenVPN server (old)](https://openwrt.org/docs/guide-user/services/vpn/openvpn/server?rev=1632708683)
___
接下来就是跟着文档走一遍, 我会对官方的文档进行一个翻译和加注：
> **1. 准备**
如果需要请先设置DDNS客户端。 安装依赖包. 指定一些临时的环境变量

```bash
# 安装包
opkg update
opkg install openvpn-openssl openvpn-easy-rsa
 
# 配置环境变量
OVPN_DIR="/etc/openvpn"
OVPN_PKI="/etc/easy-rsa/pki"
OVPN_PORT="1194"
OVPN_PROTO="udp"
OVPN_POOL="192.168.8.0 255.255.255.0"  # 注：这里可以指定openvpn的网段
OVPN_DNS="${OVPN_POOL%.* *}.1"
OVPN_DOMAIN="$(uci get dhcp.@dnsmasq[0].domain)"
 
# 拿到外网ip作为外网地址
. /lib/functions/network.sh
network_flush_cache
network_find_wan NET_IF
network_get_ipaddr NET_ADDR "${NET_IF}"
OVPN_SERV="${NET_ADDR}"
 
# 如果有ddns 则从ddns拿到全域名(FQDN)作为外网地址
NET_FQDN="$(uci -q get ddns.@service[0].lookup_host)"
if [ -n "${NET_FQDN}" ]
then OVPN_SERV="${NET_FQDN}"
fi

```
>**2. 秘钥管理**
使用 EasyRSA 管理 PKI. 必要时使用私钥密码保护.
```bash
# 配置环境变量
export EASYRSA_PKI="${OVPN_PKI}"
export EASYRSA_REQ_CN="ovpnca"
export EASYRSA_BATCH="1"
 
# 移除并且重新初始化pki目录
easyrsa init-pki
 
# 生成DH变量
easyrsa gen-dh
 
# 创建新的CA
easyrsa build-ca nopass
 
# 为服务端生成密钥对并在本地签名
easyrsa build-server-full server nopass
 
# 为客户端生成密钥对并在本地签名
easyrsa build-client-full client nopass  
#注：此处为生成客户端配置必要，如果需要多个客户端的，可以多次运行本行 只需要更改“client”, 如：
#easyrsa build-client-full client1 nopass 
#easyrsa build-client-full client2 nopass 
#easyrsa build-client-full client-dsm nopass
#注：更改后的名称会作为生成后配置文件的名称 
 
# 生成TLS PSK
openvpn --genkey --secret ${OVPN_PKI}/tc.pem
```

>**3. 防火墙**
将VPN网络视为私有网络，将VPN接口分配到LAN区域，以减少防火墙的设置。允许从WAN区域访问VPN服务器。

```bash
# 配置防火墙
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci del_list firewall.lan.device="tun+"
uci add_list firewall.lan.device="tun+"
uci -q delete firewall.ovpn
uci set firewall.ovpn="rule"
uci set firewall.ovpn.name="Allow-OpenVPN"
uci set firewall.ovpn.src="wan"
uci set firewall.ovpn.dest_port="${OVPN_PORT}"
uci set firewall.ovpn.proto="${OVPN_PROTO}"
uci set firewall.ovpn.target="ACCEPT"
uci commit firewall
/etc/init.d/firewall restart
```
>**4. VPN服务**
配置VPN服务，生成客户端配置文件。
```bash
# 配置环境变量
OVPN_DH="$(cat ${OVPN_PKI}/dh.pem)"
OVPN_TC="$(sed -e "/^#/d;/^\w/N;s/\n//" ${OVPN_PKI}/tc.pem)"
OVPN_CA="$(openssl x509 -in ${OVPN_PKI}/ca.crt)"
NL=$'\n'
 
# 配置VPN服务，生成客户端配置文件。 
#注：如果你清楚的知道你在做什么，可以更改下方ovpn配置相关的代码，可以省掉后续多个客户端单独修改配置的工作量
umask go=
ls ${OVPN_PKI}/issued \
| sed -e "s/\.\w*$//" \
| while read -r OVPN_ID
do
OVPN_KEY="$(cat ${OVPN_PKI}/private/${OVPN_ID}.key)"
OVPN_CERT="$(openssl x509 -in ${OVPN_PKI}/issued/${OVPN_ID}.crt)"
OVPN_EKU="$(openssl x509 -in ${OVPN_PKI}/issued/${OVPN_ID}.crt -purpose)"
case ${OVPN_EKU} in
(*"SSL server : Yes"*)
OVPN_CONF="\
port ${OVPN_PORT}
proto ${OVPN_PROTO}
server ${OVPN_POOL}
topology subnet
client-to-client
keepalive 10 60
persist-tun
persist-key
push \"dhcp-option DNS ${OVPN_DNS}\"
push \"dhcp-option DOMAIN ${OVPN_DOMAIN}\"
push \"redirect-gateway def1\"
push \"persist-tun\"
push \"persist-key\"
<dh>${NL}${OVPN_DH}${NL}</dh>"
OVPN_EXT="conf" ;;
(*"SSL client : Yes"*)
OVPN_CONF="\
nobind
client
remote ${OVPN_SERV} ${OVPN_PORT} ${OVPN_PROTO}
auth-nocache
remote-cert-tls server"
OVPN_EXT="ovpn" ;;
esac
cat << EOF > ${OVPN_DIR}/${OVPN_ID}.${OVPN_EXT}
user nobody
group nogroup
dev tun
${OVPN_CONF}
<tls-crypt>${NL}${OVPN_TC}${NL}</tls-crypt>
<key>${NL}${OVPN_KEY}${NL}</key>
<cert>${NL}${OVPN_CERT}${NL}</cert>
<ca>${NL}${OVPN_CA}${NL}</ca>
EOF
done
/etc/init.d/openvpn restart
# 此处就能够看到client.ovpn 把这个文件通过openwrt的备份功能导出来，上传到群晖即可
ls ${OVPN_DIR}/*.ovpn
```
到这里openwrt的openvpn服务端配置完毕，可以使用openvpn的备份功能（``System -> Backup/Flash Firmware``）. 从备份包导出客户端的配置文件。

---

## 2. 群晖连接openvpn
到了群晖这一步就比较简单，按照官网的流程，打开 ``控制面板->网络->网络界面->新增->创建VPN配置文件``，把刚才导出的client.ovpn上传即可

>注：其实这里因为群晖的错误提示非常的“友好”，连接失败只会提示“失败，请检查配置”或者“认证错误”，导致排错非常的困难，踩了不少坑。比如不支持的标签，属性等等，总之注意点都写在第一步了。如果还是有问题，请自行前往``/var/log/messages``进行openvpn相关日志的排查。

按照下图位置上传client.ovpn
![在这里插入图片描述](https://img-blog.csdnimg.cn/db605dfb49fe49aebf6ac6fe52d3c13c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55m96Iy2U3lyYVRp,size_15,color_FFFFFF,t_70,g_se,x_16)
这里只需要填写 “导入.ovpn” 文件这一项
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a403e5c14fe4f0493d5e95118e79b6c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55m96Iy2U3lyYVRp,size_19,color_FFFFFF,t_70,g_se,x_16)
记得勾选这两个选项
![在这里插入图片描述](https://img-blog.csdnimg.cn/88e52cab7a7144d9a9d482b3391b4903.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55m96Iy2U3lyYVRp,size_20,color_FFFFFF,t_70,g_se,x_16)
到这一步 openvpn的连接可以正常建立了。

---
## 3.配置端口转发
需要完成端口转发，首先我们得先把群晖的ip固定下来，否则ip一变动，端口转发规则就失效了。
需要完成openvpn客户端的ip固定，我们需要在server配置中添加规则。

**编辑服务端配置**
```bash
vim ${OVPN_DIR}/server.ovpn
```
**添加下面这一行，用来指定客户端config文件夹**
```bash
# 这里的文件夹路径可以自定
client-config-dir /etc/openvpn/ccd
```
**保存后新增客户端config 注意这里的“client”需要和.ovpn文件名相符，也就是easyrsa创建客户端秘钥时的名字**
```bash
vim /etc/openvpn/ccd/client
```
**添加下面这一行，来绑定ip**
```bash
# 这里的192.168.8.108 即为你想要绑定的ip
ifconfig-push 192.168.8.108 255.255.255.0
```
**保存后重启openvpn**
```bash
/etc/init.d/openvpn restart
```
___
接下来正式开始配置端口转发，进入openwrt,  菜单选择 ``Network->Firewall->Port Forwards``
![在这里插入图片描述](https://img-blog.csdnimg.cn/b76d34e11f01421aaf38f7c9a4c6c0cb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55m96Iy2U3lyYVRp,size_15,color_FFFFFF,t_70,g_se,x_16)
新增一条端口转发，按照下图填写
![在这里插入图片描述](https://img-blog.csdnimg.cn/2df447f9ae0a470c8d3a3f65c98dba4a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55m96Iy2U3lyYVRp,size_20,color_FFFFFF,t_70,g_se,x_16)
至此，应该就能够使用openwrt的端口转发达到内网穿透的目的了。



以上！まいど～

---