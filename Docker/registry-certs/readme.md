
- create cerfificates
```
openssl req -utf8 -sha256 -new -x509 -days 3650 -keyout ca.key -out ca.pem -config openssl-registry.conf
openssl genrsa -out docker-key.pem 4096
openssl req -new -key docker-key.pem -out docker.csr  -subj "/CN=registry.lmy.com" -config openssl-registry.conf
openssl x509 -req -in docker.csr -CA ca.pem -CAkey  ca-key.pem -out docker.pem -CAcreateserial  -days 3650 -extensions v3_req -extfile openssl-registry.conf
```



- add ca to system 
  - CentOS 
    ```
    cp ca.pem /etc/pki/ca-trust/source/anchors/registry.pem 
    update-ca-trust extract
    ```
