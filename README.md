# Notary version2 prototype2

Launch a notary sandbox with:

```shell
docker-compose up -d
docker-compose exec client /bin/bash
```

Setup client:

```shell
ln -s /certs/reg-ca/ca.crt /usr/local/share/ca-certificates/reg-ca.crt
update-ca-certificates
mkdir -p ~/.docker
cat >~/.docker/nv2.json <<EOF
{
  "enabled": true,
  "verificationCerts": [
  ],
  "insecureRegistries": [
  ]
}
EOF
```

Setup nv2 folder with signing key and cli:

```shell
mkdir nv2-demo
cd nv2-demo/
openssl req \
  -x509 \
  -sha256 \
  -nodes \
  -newkey rsa:2048 \
  -days 365 \
  -subj "/CN=registry.wabbit-networks.io/O=wabbit-networks inc/C=US/ST=Washington/L=Seattle" \
  -addext "subjectAltName=DNS:registry.wabbit-networks.io" \
  -keyout ./wabbit-networks.key \
  -out ./wabbit-networks.crt
alias docker="docker nv2"
```

Build and sign an image:

```shell
export REPO=registry.wabbit-networks.io/net-monitor
export IMAGE=${REPO}:v1
docker build \
  -t $IMAGE \
  https://github.com/wabbit-networks/net-monitor.git#main
docker notary --enabled
docker notary sign \
  --key ./wabbit-networks.key \
  --cert ./wabbit-networks.crt \
  $IMAGE
docker push $IMAGE
```

Inspect image on registry:

```shell
oras discover $IMAGE
```

Pull signed image:

```shell
docker pull $IMAGE
```

Trust wabbit key:

```shell
cat >~/.docker/nv2.json <<EOF
{
  "enabled": true,
  "verificationCerts": [
    "/root/nv2-demo/wabbit-networks.crt"
  ],
  "insecureRegistries": [
  ]
}
EOF
docker pull $IMAGE
```

Exit the sandbox:

```shell
exit
```

Cleanup:

```shell
docker-compose down
```

Also cleanup volumes:

```shell
docker-compose down -v
```
