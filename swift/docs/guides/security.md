# 保护Swift

保护Swift连接包括三个方面：

- 用户 <-> Swift服务端
- Swift服务端 <-> Tiller服务端
- Swift服务端 <-> Chart仓库


## 通过SSL提供Swift API

通过SSL连接提供Swift API，需要通过 `--tls-cert-file` 和 `--tls-private-key-file` 标记来提供服务端证书对。如果使用自签名证书对，需要通过 `--tls-ca-file` 标记来传递CA证书。

```
  --tls-ca-file string                      包含CA证书的文件
  --tls-cert-file string                    包含服务端TLS证书的文件
  --tls-private-key-file string             包含服务端TLS私钥的文件
```

可以使用[onessl](https://github.com/kubepack/onessl)、openssl、cfssl等工具来生成证书对。下面是使用onessl来生成证书对的说明：

- 从项目的Github发布页面下载onessl二进制文件。

```console
# Linux amd64:
curl -fsSL -o onessl https://github.com/kubepack/onessl/releases/download/0.3.0/onessl-linux-amd64 \
  && chmod +x onessl \
  && sudo mv onessl /usr/local/bin/

# Linux arm64:
curl -fsSL -o onessl https://github.com/kubepack/onessl/releases/download/0.3.0/onessl-linux-arm64 \
  && chmod +x onessl \
  && sudo mv onessl /usr/local/bin/

# Mac OSX amd64:
curl -fsSL -o onessl https://github.com/kubepack/onessl/releases/download/0.3.0/onessl-darwin-amd64 \
  && chmod +x onessl \
  && sudo mv onessl /usr/local/bin/
```

- 使用onessl创建CA证书对。

```console
$ mkdir swift
$ cd swift
$ onessl create ca-cert

$ ls -b1
ca.crt
ca.key
```

- 使用上一步骤中创建的CA证书来创建服务端证书对。使用 `--domains` 标记来传递连接Swift服务端的域名。

```console
$ onessl create server-cert --domains=swift.kube-system.svc
Wrote server certificates in  /home/tamal/Desktop/swift

$ ls -b1
ca.crt
ca.key
server.crt
server.key
```

- 创建把证书上传到Kubernetes集群的密钥，挂载密钥的是Swift Deployment。

```console
$ kubectl create secret generic swift-ssl -n kube-system \
  --from-file=./ca.crt \
  --from-file=./server.crt \
  --from-file=./server.key

secret "swift-ssl" created
```


## 对Chart仓库使用认证

使用用户名密码认证（Basic Auth）、不记名令牌认证（Bearer Auth）或客户端证书认证（Client Cert Auth），Swift可以从Chart仓库下载Chart。 `InstallRelease` 和 `UpdateRelease` API在各自的API调用中支持以下输入参数：

| 参数                   | 是否必要 | 描述                                           |
|------------------------|----------| -----------------------------------------------|
| `chart_url`            | `必要的` | 要下载的Chart归档的URL。                       |
| `username`             | `可选的` | 用户名，用于对Chart仓库进行基础认证。          |
| `password`             | `可选的` | 密码，用于对Chart仓库进行基础认证。            |
| `token`                | `可选的` | 不记名令牌，用于对Chart仓库进行认证。          |
| `ca_bundle`            | `可选的` | PEM编码的CA包，用于签名Chart仓库的服务端证书。 |
| `client_certificate`   | `可选的` | PEM编码的数据，作为客户端证书传递给Chart仓库。 |
| `client_key`           | `可选的` | PEM编码的数据，作为客户端Key传递给Chart仓库。  |
| `insecure_skip_verify` | `可选的` | 跳过Chart仓库的证书认证。                      |


## 安全地连接到Tiller服务端

You can run Swift server in the same pod as the Tiller server and connect over localhost. This will ensure that traffic between Tiller server and Swift proxy is not visible to outside parties.

If you are using [SSL with your Tiller Server](https://github.com/kubernetes/helm/blob/master/docs/tiller_ssl.md), you can pass the ca certificate and/or client certificate pair to Swift server to secure connect to Tiller server over TLS.

```
  --tiller-ca-file string                   File containing CA certificate for Tiller server
  --tiller-client-cert-file string          File container client TLS certificate for Tiller server
  --tiller-client-private-key-file string   File containing client TLS private key for Tiller server
  --tiller-endpoint string                  Endpoint of Tiller server, eg, [scheme://]host:port
  --tiller-insecure-skip-verify             Skip certificate verification for Tiller server
```

The server certificate used with Tiller should have the following Subject Alternative Names (SANS) to ensure that Swift can connect to it whether running in the same namespace or in different namespaces.

| Subject Alternative Name (SANS) | Usage                                                    |
|---------------------------------|----------------------------------------------------------|
| tiller-svc                      | When tiller and swift both running in same namespace     |
| tiller-svc.tiller-namespace     |                                                          |
| tiller-svc.tiller-namespace.svc | When tiller and swift are running in different namespace |
