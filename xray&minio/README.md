# Xray & MinIO

This guide base of Debain.

## Update system

```sh
apt update
apt upgrade -y
apt autopurge -y
```

## Install Docker

```sh
apt install -y ca-certificates curl gnupg lsb-release
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" \
  | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## Run the Nginx container

```sh
mkdir -p /data/nginx
docker run --name nginx -d nginx
docker cp nginx:/etc/nginx/conf.d /data/nginx/.
docker cp nginx:/usr/share/nginx/html /data/nginx/.
docker rm -f nginx
docker run --name root-nginx-1 -v /data/nginx/html:/usr/share/nginx/html -p 80:80 -d nginx
```

## Get a TLS certificate from Let's Encrypt

```sh
curl https://get.acme.sh | sh
./.bashrc
acme.sh --upgrade --auto-upgrade
acme.sh --set-default-ca --server letsencrypt
```

Test:

`acme.sh --issue --test -d acgxcloud.com -w /data/nginx/html --keylength ec-256`

Cover:

`acme.sh --issue --force -d acgxcloud.com -w /data/nginx/html --keylength ec-256`

Get cert file:

```sh
mkdir -p /data/cert/default
acme.sh --install-cert -d acgxcloud.com \
  --cert-file /data/cert/default/cert.pem \
  --key-file /data/cert/default/key.pem \
  --reloadcmd "docker restart root-nginx-1"
```
