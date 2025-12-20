---
title: OpenSSl创建自签证书
date: 2025-10-14 22:42:58
update: 2025-10-14 22:42:58
tags: [linux]
index_img: /img/bg/openssl.jpg
excerpt: Openssl创建自签证书
sticky: 100
category: linux
---
# <center>Openssl创建自签证书</center>

#### <div style="background-color:cadetblue">一、openssl安装</div>
```sh
# ubuntu系统
sudo apt install openssl
```

#### <div style="background-color:cadetblue">一、创建自签证书</div>

生成一个 2048 位的 RSA 私钥
```sh
# 生成2048位私钥，使用AES256加密
openssl genrsa -out cicd.key 2048
```

生成一个 证书签名请求（Certificate Signing Request, CSR），其中包含公钥和身份信息，用于申请证书。
```sh
openssl req -new -key cicd.key -out cicd.csr
    # Country Name (2 letter code)：国家代码，如CN
    # State or Province Name：省份
    # Locality Name：城市
    # Organization Name：公司名称
    # Organizational Unit Name：部门名称
    # Common Name：非常重要，填写服务器域名或IP（如internal.company.com）
    # Email Address：邮箱地址
    # 其他可选字段直接按回车跳过
```

使用私钥（cicd.key）对 CSR（cicd.csr）进行自签名，生成一个 X.509 格式的证书文件 cicd.crt
```sh
# 生成2048位私钥，使用AES256加密
openssl x509 -req -days 3650 -in cicd.csr  -signkey cicd.key -out cicd.crt
```

#### <div style="background-color:cadetblue">三、测试</div>