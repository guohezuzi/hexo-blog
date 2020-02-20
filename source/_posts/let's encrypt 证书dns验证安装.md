---
title: let's encrypt 证书dns验证安装
date: 1999-02-24
tags:
---

# let's encrypt 证书dns验证安装

./certbot-auto --no-bootstrap --no-self-upgrade certonly --manual -d www.guohezuzi.cn --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory


