## nginxとCloudFlareのSSLの組み合わせ方

### 方法1 snakeoilとFullを組み合わせる
簡単 (CloudFlare<=>nginx間は暗号化されるがホスト名の照合はない)

nginx側ではDebian/Ubuntuの自己署名証明書を設定して
```
ssl_certificate     /etc/ssl/certs/ssl-cert-snakeoil.pem;
ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
```

CloudFlare側ではmodeとしてFullを選択する

### 方法2 
厳格 (CloudFlare<=>nginx間は暗号化さホスト名も照合される)

CloudFlare側ではmodeとしてFull (strict)を選択する

CloudFlareでorigin certificateを発行する

発行したcertificateをnginx側に設定する

