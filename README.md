## 数据库镜像
docker push registry.jingan.com:32008/ja/pgautoupgrade/pgautoupgrade:13-alpine-amd


## 打包
docker build -t registry.jingan.com:32008/ja/ja-label-studio:v1.0.6 --platform=linux/amd64 . 
## push
docker push registry.jingan.com:32008/ja/ja-label-studio:v1.0.6

