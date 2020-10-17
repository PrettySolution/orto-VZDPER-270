1. $ `docker pull grafana/grafana`
1. $ `docker stop grafana`
1. $ `docker rm grafana`
1. run a new container:
```
docker run -d \
--name="grafana" \
--network="network-monitor" \
-u $(id -u root):$(id -g root) \
-p 3000:3000 \
-v /monitor/grafana/config:/etc/grafana \
-v /monitor/grafana/data:/var/lib/grafana \
--restart unless-stopped \ 
grafana/grafana 
```