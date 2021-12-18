# 解决服务器端使用 let's encrypt 签发的证书时客户端无法认证https的问题

问题描述：

服务器使用了 let's encrypt 签发的证书，浏览器访问是没问题的，问题是另一台服务器Ubuntu16.04使用API访问https服务器时出错，提示：
```
ERROR: cannot verify wx.hb3344.com's certificate, issued by ‘CN=R3,O=Let's Encrypt,C=US’:
  Issued certificate has expired.
```

原理是看这个，解释的非常详细 [Will You Be Impacted by Let’sEncrypt DST Root CA X3 Expiration?](https://medium.com/geekculture/will-you-be-impacted-by-letsencrypt-dst-root-ca-x3-expiration-d54a018df257)

解决办法是在访问HTTPS服务器的服务器（。。。）禁用 `DST_Root_CA_X3.crt` :

```bash
sudo sed -i 's/mozilla\/DST_Root_CA_X3.crt/!mozilla\/DST_Root_CA_X3.crt/g' /etc/ca-certificates.conf
sudo update-ca-certificates
```


