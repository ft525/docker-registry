## 部署私人倉庫 (Registry)




#### 1. 複製 docker login 預設帳密 (docker-registry / docker-registry)，或自訂 (請使用 Bcrypt - Apache v2.4 onwards)
cp -p ./etc/registry27/auth/htpasswd.default ./etc/registry27/auth/htpasswd


#### 1.1 調整專案名稱
vi .env => COMPOSE_PROJECT_NAME


#### 1.2 使用 insecure registry (HTTP) (不建議)
需要調整 client 端的 docker 設定
參考: https://docs.docker.com/registry/insecure/


#### 2. 產生自簽憑證，Common Name 請填該 domain (不含 port)
openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout ./data/certs/default.key \
    -x509 -days 3650 -out ./data/certs/default.crt

還需要將自簽憑證匯入信任的憑證，方法請參考下面連結

參考: https://docs.docker.com/registry/insecure/#use-self-signed-certificates


#### 2.1 使用合格憑證
請將憑證放到 ./data/certs/default.crt
請將私鑰放到 ./data/certs/default.key


#### 啟動 (要在專案目錄下執行)
docker-compose up -d


#### 關閉 (要在專案目錄下執行)
docker-compose down




###### 參考
https://docs.docker.com/registry/deploying/ (部署)
https://docs.docker.com/registry/configuration/ (設定)




## 附錄
#### Push image to private registry
1. 先正確的 tag image
docker tag SOURCE_IMAGE[:TAG] [REGISTRY_HOST/][USERNAME/]IMAGE_NAME[:TAG]
	例: docker tag xxx my.registry.com:5000/kyo/my-image:1.0

2. Push image (需要先登入 docker login REGISTRY_HOST)
docker push my.registry.com:5000/kyo/my-image:1.0

// 參考
https://stackoverflow.com/questions/28349392/how-to-push-a-docker-image-to-a-private-repository
https://docs.docker.com/engine/reference/commandline/tag/
https://docs.docker.com/engine/reference/commandline/push/


#### Delete image from private registry
// 參考
https://blog.gss.com.tw/index.php/2020/07/17/dockerregistry/
https://docs.docker.com/registry/spec/api/#deleting-an-image


#### Registry API
// 參考
https://docs.docker.com/registry/spec/api/#detail
