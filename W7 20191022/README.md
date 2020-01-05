# Docker with WordPress

## Clone the example repository

```sh
git clone https://github.com/ayubiz/learn_docker_by_examples.git
cd ch09
```

- docker-compose.yml 中的 volumes 是為了持久化

```yml
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on: # 代表相依於 db 服務
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data: # 如果沒有指定值，則會隨機選擇主機一個位置來掛載
```

- 在背景啟動

```
sudo docker-compose up -d
```

- `docker-compose ps` 顯示所有由 
- Name 原則
  - ch09 為資料夾的名稱，能避免撞名
  - 結尾的 1 代表是第幾個副本

```
minidino-macbook:ch09 minidino$ docker-compose ps
      Name                    Command               State          Ports        
--------------------------------------------------------------------------------
ch09_db_1          docker-entrypoint.sh mysqld      Up      3306/tcp, 33060/tcp 
ch09_wordpress_1   docker-entrypoint.sh apach ...   Up      0.0.0.0:8000->80/tcp
```

- `docker volume ls`：顯示所有的 volume
- `docker volume inspect <VOLUME NAME>`：查看 volume 的詳細資料（包含存放的位置）
  - `Mountpoint` 為檔案存放的位置

```
minidino-macbook:ch09 minidino$ docker volume inspect ch09_db_data
[
    {
        "CreatedAt": "2019-10-22T10:55:48Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "ch09",
            "com.docker.compose.version": "1.24.1",
            "com.docker.compose.volume": "db_data"
        },
        "Mountpoint": "/var/lib/docker/volumes/ch09_db_data/_data",
        "Name": "ch09_db_data",
        "Options": null,
        "Scope": "local"
    }
]
```

- `scp -r <Mountpoint> <user>@<remote_IP>:<remote_path>`：轉移檔案到遠端機器上

```
scp -r /var/lib/docker/volumes/ch09_db_data/_data user@172.16.45.128:/ch09_db_data
```

- **remote_machine**
```
mv /ch09_db_data /var/lib/docker/volumes
yum install -y git
git clone https://github.com/ayubiz/learn_docker_by_examples.git
cd ch09

# specify mountpoint
docker compose 
```

## Docker swarm

- 將 container 分散在不同的機器上
- 但這並不是最主流的方法，現在大多用 K8S(Kubernetes)

- 產生 wordpress, db

```
docker-compose up --scale wordpress=3 db=2
```

## 雜記

- 第九週不上課，但可以自行思考一個小專案
- 灰度升級：為了確保服務不要中斷，不會同時升級所有機器。而是分匹升級，先確認沒問題後，再部署（升級）到其他裝置上。
