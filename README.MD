## xray_docker_reality
xray reality docker 镜像 加入SHORT_IDS与SNI_FILTER，防偷流量

原版：https://github.com/wulabing/xray_docker/
参考：https://github.com/oldfriendme/xrayREALITY

> 新的命令中添加了日志大小的限制，可以调整 max-size=100m 的数量来调整日志的最大大小，需要限制日志大小的用户请重新创建容器

> 变更了镜像tag，不影响已经使用的用户

> 本 xray_docker_reality 仓库会每两天在 UTC+8 04:00 自动构建镜像

## docker-compose.yml
```
version: "3.8"

services:
  xray-reality:
    image: kwxos/xray_docker_reality:latest
    container_name: xray_reality
    restart: always

    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

    ports:
      - "8443:443"

    environment:
      SERVERNAMES: "www.microsoft.com images.microsoft.com"
      DEST: "www.microsoft.com:443"
      EXTERNAL_PORT: "8443"
      UUID: "54375107-f7eb-4786-ad4c-a746dd6d293c"
      SNI_FILTER: "true"
      SHORT_IDS: "6e a93c f1"

    volumes:
      - /root/reality:/data

    network_mode: bridge
```


部署后，手动补上SHORT_IDS，到对应的客户段的位置，比如，填入 `a93c`





## docker一键命令
```
docker run -d \
  --name xray_reality \
  --restart=always \
  --log-opt max-size=100m \
  --log-opt max-file=3 \
  -p $EXTERNAL_PORT:443 \
  -e EXTERNAL_PORT=8443 \
  -e UUID="54375107-f7eb-4786-ad4c-a746dd6d293c" \
  -e SERVERNAMES="www.microsoft.com images.microsoft.com" \
  -e DEST="www.microsoft.com:443" \
  -e SNI_FILTER="true" \
  -e SHORT_IDS="6e a93c f1" \
  -v "/root/reality:/data" \
  wulabing/xray_docker_reality:latest
```

## 环境变量
你可以设置以下环境变量来自定义 XRay 的配置。
* UUID：XRay 的唯一用户标识符。若未设置，脚本将自动生成一个随机 UUID。
* DEST：目标地址。默认值为 www.apple.com:443。
* SERVERNAMES：服务器名称列表。默认值为 www.apple.com images.apple.com。
* SHORT_IDS：Reality shortIds 列表，空格分隔。默认自动生成。
* PRIVATEKEY：私钥。若未设置，脚本将自动生成一个新的私钥和对应的公钥。
* NETWORK：网络类型。默认值为 tcp。
* SNI_FILTER：是否启用 sni-filter，默认 true。
* SNI_FILTER_BACKEND_PORT：sni-filter 转发的内部端口，默认 65433。
* CONFIG_DIR：持久化目录。默认值为 /data。

## 常量
* flow：xtls-rprx-vision
* security：reality
* shortIds：留空

## 注意事项
请确保在使用前已正确安装和配置 XRay 客户端。
为了保证安全和稳定，请勿将 UUID、私钥等敏感信息泄露给他人。
启用 sni-filter 时默认只放行 SERVERNAMES 的第一个域名。
