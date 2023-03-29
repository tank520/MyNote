## docker

1. MySQL

```sh
首次启动
docker run -d --name mysql -p 9506:3306 -v D:/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.41-debian

后续启动
docker run mysql

进入
docker exec -it [container-id] bash
```

2. 镜像构建并启动

```sh
基于dockerfile构建镜像
docker build -f Dockerfile.dev -t machine_cloud_api .

启动
docker run -itd --name machine_cloud_api -p 17171:17171 machine_cloud_api
```

3. 修改启动参数

```sh
docker update --参数名 参数值 [container-id]
```

