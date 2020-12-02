---
layout: post
title: firewalld with docker
date: 2020-12-01 16:07
tags: [docker firewalld kvm linux]
category:
---

When using `firewalld` and `docker`, docker containers can not connect to outside.
This happens when I install `kvm`, `kvm` use `firewalld` for network, so I have to install `firewalld`.

## First, add a `firewalld` policy

`firewall-cmd --zone=public --permanent --add-masquerade`

## Restart service

Must restart `firewalld` first, then `docker`. Everytime restart `firewalld`, it is better to restart `docker`.

`sudo systemctl restart firewalld.service`
`sudo systemctl restart docker.service`

> **解释一下这个问题**
>
> 默认情况下，系统的主网卡是在 public zone 里面的，这个是`firewalld`默认设置的。
> 对于 docker container 访问互联网，使用了`docker0`这个 interface，然后 NAT 桥接到主网卡上。
> 这时候，主网卡类似与一个路由器。`--add-masquerade`选项作用是支持转发时更换源 IP 地址。

_masquerade 大家应该都比较熟悉，其作用就是 ip 地址伪装，也就是 NAT 转发中的一种，具体处理方式是将接收到的请求的源地址设置为转发请求网卡的地址，这在路由器等相关设备中非常重要，比如大家很多都使用的是路由器连接的局域网，而想上互联网就得将我们的 ip 地址给修改一下，要不大家都是 192.168.1.XXX 的内网地址，那请求怎么能正确返回呢？所以在路由器中将请求实际发送到互联网的时候就会将请求的源地址设置为路由器的外网地址，这样请求就能正确地返回给路由器了，然后路由器再根据记录返回给我们发送请求的主机了，这就是 masquerade。_

因此如果使用`--add-masquerade`选项，就可以支持 docker 访问外部互联网。

对于`docker`使用 port map 映射出来的端口，貌似`docker`自己进行了管理，不用特殊配置就可以使用。
