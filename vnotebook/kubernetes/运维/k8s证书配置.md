# k8s证书配置

## kubeadm token永不过期
```
kubeadm token create bxmauj.x7qn8ayj59h1pb05 --print-join-command --ttl=0
```
## kubernetes证书修改过期时间

```
cat << EOF >> ssl.conf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[v3_req]
keyUsage =critical, digitalSignature, keyEncipherment
extendedKeyUsage = TLS Web Server Authentication, TLS Web Client Authentication
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc.cluster.local
DNS.4 = kubernetes.default.svc
IP.1 = 10.17.0.7
IP.2 = 10.96.0.1
EOF
```
```
kubeadm alpha certs renew all
```
```
openssl req -new -key apiserver.key -out apiserver.csr -subj "/CN=kube-apiserver" -config ssl.conf
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -days 7300 -extensions v3_req -extfile ssl.conf

openssl req -new -key apiserver-kubelet-client.key -out apiserver-kubelet-client.csr -subj "/CN=apiserver-kubelet-client" -config ssl.conf
openssl x509 -req -days 7300 -in apiserver-kubelet-client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver-kubelet-client.crt  -extensions v3_req -extfile ssl.conf

openssl req -new -key front-proxy-client.key -out front-proxy-client.csr -subj "/CN=front-proxy-client" -config ssl.conf
openssl x509 -req -days 7300 -in front-proxy-client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out front-proxy-client.crt  -extensions v3_req -extfile ssl.conf

kubeadm init phase kubeconfig all
```

## 查看集群过期时间
```
openssl x509 -in signed.crt -noout -dates
```
## kubeadm更新集群证书
```
kubeadm alpha phase  certs all --config kubeadm-master.config
kubeadm alpha phase  kubeconfig all  --config kubeadm-master.config
kubeadm init phase certs etcd-server  --config kubeadm.yaml
```
## 日志采集
在部署ES之前，首先看一下docker 的log-driver配置，修改为json-file,默认的可能是journald。fluentd要读取/var/log/containers/目录下的log日志，这些日志是从/var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log链接过来的，如果log-driver是journald，就会读取不到:

```
vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=json-file --signature-verification=false'
```

## kubeconfig权限配置

```
# config
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: https://10.10.1.100:6443
    certificate-authority-data: <这里省略>
  name: k8s-dev
users:
- name: "devlog"
  user:
    token: <这里是解码后的token字符串>
contexts:
- context:
    cluster: k8s-dev
    user: "devlog"
  name: devlog-ct
preferences: {}
current-context: devlog-ct
```