## 部署私人倉庫 (Registry)

### 1. 複製 docker-compose.yml 並調整內容

```bash
cp -p docker-compose.yml.sample docker-compose.yml
```


#### 1.1 複製 docker login 預設帳密 (docker-registry / docker-registry)，或自訂 (請使用 Bcrypt - Apache v2.4 onwards)

```bash
cp -p ./etc/registry/auth/htpasswd.default ./etc/registry/auth/htpasswd
```


#### 1.2 複製 .env & 調整專案名稱

```bash
cp -p .env.sample .env
vi .env	# 調整 COMPOSE_PROJECT_NAME
```


#### 1.3 使用 insecure registry (HTTP) (不建議)

需要調整 client 端的 docker 設定
	Docker Desktop for Windows 或 Docker Desktop for Mac (Settings > Docker Engine) ||
	調整 /etc/docker/daemon.json (Linux) {
		"insecure-registries" : ["myregistrydomain.com:5000"]
	}

Docker 會先嘗試使用 HTTPS，會忽略憑證錯誤; 如果 HTTPS 不可用才會改用 HTTP

參考: https://docs.docker.com/registry/insecure/


### 2. 產生自簽憑證，Common Name 請填該 domain (不含 port)

openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout ./data/pki/default.key \
    -x509 -days 3650 -out ./data/pki/default.crt

還需要將自簽憑證匯入信任的憑證，方法請參考下面連結 (TODO: 目前好像不需要了，只需執行步驟 1.4。待確認)

參考: https://docs.docker.com/registry/insecure/#use-self-signed-certificates


#### 2.1 使用合格憑證

請將憑證放到 ./data/pki/default.crt
請將私鑰放到 ./data/pki/default.key


### 啟動 (要在專案目錄下執行)

```cmd
docker-compose up -d
```


### 關閉 (要在專案目錄下執行)

```cmd
docker-compose down
```


### 參考

https://docs.docker.com/registry/deploying/ (部署)
https://docs.docker.com/registry/configuration/ (設定)




## 附錄

### Push image to private registry

```
0. 登入 & 登出 private registry
docker login [(-u | --username) USERNAME] [((-p | --password) PASSWORD | --password-stdin)] REGISTRY_HOST
docker logout REGISTRY_HOST

1. 正確的 tag image
docker tag SOURCE_IMAGE[:TAG] [REGISTRY_HOST/][USERNAME/]IMAGE_NAME[:TAG]
	例: docker tag damp-haproxy my.registry.com:5000/damp/haproxy:1.0

2. Push image (需要先登入 private registry)
docker push NAME[:TAG]
	例: docker push my.registry.com:5000/damp/haproxy:1.0


// 參考
https://stackoverflow.com/questions/28349392/how-to-push-a-docker-image-to-a-private-repository
https://docs.docker.com/engine/reference/commandline/tag/
https://docs.docker.com/engine/reference/commandline/push/
```


### Registry APIs

```
// 顯示 repository 列表
GET /v2/_catalog

// 顯示 repository 的 tag 列表 (name 從 /v2/_catalog 查看)
GET /v2/{name}/tags/list

/*
	取得 manifests 清單

	需添加額外 Accept header 才能取得刪除用的 digest (在 response header 裡的 Docker-Content-Digest)
	reference 可以是 tag 或 digest
*/
GET /v2/{name}/manifests/{reference}
	Accept: application/vnd.docker.distribution.manifest.v2+json

// 刪除 image (reference 只能是 digest; 需先啟用刪除功能)
DELETE /v2/{name}/manifests/{reference}

// 垃圾回收 (刪除 image 後; 請在 registry 容器裡執行)
[/bin/]registry garbage-collect [--dry-run] [--delete-untagged] /etc/docker/registry/config.yml		# 舊版 Registry 2.8
[/bin/]registry garbage-collect [--dry-run] [--delete-untagged] /etc/distribution/config.yml		# 新版 Distribution 3.1

// 刪除空的 repository (垃圾回收後; 請在 registry 容器裡執行)
rm -R /var/lib/registry/docker/registry/v2/repositories/{name}


// 參考
https://distribution.github.io/distribution/spec/api/ (API 說明)
https://docs.docker.com/registry/spec/api/#detail
https://docs.docker.com/registry/spec/api/#deleting-an-image
https://docs.docker.com/registry/garbage-collection/
https://stackoverflow.com/questions/43666910/remove-docker-repository-on-remote-docker-registry
```
