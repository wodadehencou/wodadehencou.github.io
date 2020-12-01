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


