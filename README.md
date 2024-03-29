# note

## linux command
```sh
# Listen 8000 prot
nc -l localhost 8000

# run command in background
# 在背景執行命令，可以登出後繼續執行
nohup my_program &

# find file by name
find . -name deployment.yaml

# 檢查 port 佔用
# for osx
lsof -n -i:50051
lsof -n -i | grep "LISTEN"
#
# for linux
netstat -tulp | grep 50051

# jobs fg bg
# use [ctrl] + [z] to suspend a running command and move to background
#
# show background jobs
jobs
# move the job to the foreground and continue
fg %${job_num}
# continue the background job
bg %${job_num}

# tar
# NOTE: mac 在使用 tar 壓縮時會多出 "._*" 至 "*.tar" 之中
# See: https://superuser.com/questions/259703/get-mac-tar-to-stop-putting-filenames-in-tar-archives
# See: https://apple.stackexchange.com/questions/94343/prevent-doubleapple-files-creation-when-tarring-directories-snow-leopard
# Solution:
# COPYFILE_DISABLE=1 tar -czvf xxx.tar.gz file1 file2
#
# alias tarc="COPYFILE_DISABLE=1 tar --exclude '\\.DS_Store' -zcf"
# alias tar="COPYFILE_DISABLE=1 tar"
#
# my-tar ${name} ${file1}[ ${file2}...]
# create "${name}.tar" from ${file1}[ ${file2}...]
function my-tar() { tar -cvf ${1}.tar ${@:2}; }
#
# my-untar ${name}[ ${path}]
function my-untar() {
    path=${2:-"tmp-$(date -u +"%Y%m%d%H%M%S")"}&&\
    mkdir ${path}&&\
    tar -xvf ${1} -C ${path};
}
#
# my-tgz ${name} ${file1}[ ${file2}...]
# create "${name}.tgz" from ${file1}[ ${file2}...]
function my-tgz() { tar -czvf ${1}.tgz ${@:2}; }
#
# my-untgz ${name}[ ${path}]
function my-untgz() {
    path=${2:-"tmp-$(date -u +"%Y%m%d%H%M%S")"}&&\
    mkdir ${path}&&\
    tar -xzvf ${1} -C ${path};
}

# screen
# 開啟新的 screen, screen 當 ssh 登出後會保留
screen

# 從 screen 分離出來 detach
# [ctrl] + [a], [d]

# 列出當前 screen
screen -ls

# 切換至某個 screen 若只有一個不用編號
screen -r [$n]
```

## ssh
```sh
# connect to ssh server
ssh ${account}@${ssh_server_host} -p ${port}
# with ssh config alias
ssh ${ssh_alias}

# generate public/private ssh key pair.
# path: ~/.ssh/
ssh-keygen -t rsa -b 4096
ssh-keygen -t rsa -b 4096 -m PEM
ssh-keygen -t ed25519 -b 4096
ssh-keygen -t ed25519 -b 4096 -m PEM

# change private ssh key password.
ssh-keygen -p

# copy public key to ssh server "~/.ssh/authorized_keys"
ssh-copy-id -i ${public_key_path} ${account}@${ssh_server_host}

# ssh config alias
echo '
Host ${ssh_alias}
    User ${user}
    Hostname ${ip}
    ServerAliveInterval 30
    # IdentityFile ${ssh_private_key_path}
    # Port 22
' >> ~/.ssh/config

# ssh tunnel 跳板
# 將指定的 local addr 收的 request 送至 remote 所指定的 addr
ssh -L ${local_addr}:${target_addr} ${remote}
ssh -L 50051:localhost:50051 ${ssh_alias}
ssh -L 50051:50051 ${ssh_alias}

# scp (ssh copy)
scp ${from} ${to}
scp ${ssh_alias}:${remote_path} ${local_path}
scp ${local_path} ${ssh_alias}:${remote_path}


# mysql
# official mysql client CLI
# Install on mac:
#   brew install mysql-client
#
mysql -h ${host} -P ${port} -u ${user} --password="${password}"
mysql -h 127.0.0.1 -P 3306 -u root --password='password'

# mysqlsh (mysql shell)
# official mysql advanced client CLI
# Download: https://dev.mysql.com/downloads/shell/
#
# connect on sql mode, exit enter: "\exit"
mysqlsh --sql "${mysql_dsn}"
mysqlsh --sql "mysql://${user}[:${password}]@${host}[:${port}][/${database}]"
mysqlsh --sql 'mysql://root:password@127.0.0.1:3306'

# mycli
# third-party mysql client CLI
# Install: pip3 install -U mycli
# Refer: https://www.mycli.net/
#
mycli "${mysql_dsn}"
mycli "mysql://${user}[:${password}]@${host}[:${port}][/${database}]"
mycli 'mysql://root:password@127.0.0.1:3306'

# mysqldump
# options:
#   -d --no-data: only dump schema, no data
#   -t --no-create-info: only dump data, no schema
#   -A --all--database
#   --databases ${db1} ${db2}
#   -e --extended-insert: 合併 insert 語法
#   --set-gtid-purged=OFF
#
# backup all table schemas and data in the database
mysqldump -h ${host} -P ${port} -u ${user} --password="${password}" ${database} > database_backup.sql
#
# backup all table data in the database
mysqldump -h ${host} -P ${port} -u ${user} --password="${password}" ${database} --no-create-info > database_backup.sql
#
# apply from backup file
mysqldump -h ${host} -P ${port} -u ${user} --password="${password}" ${database} < database_backup.sql


# mongosh
# Install on mac:
#   brew install mongosh
# Refer: https://docs.mongodb.com/mongodb-shell/install/#std-label-mdb-shell-install
mongosh "${mongo_dsn}"
mongosh --host "${host}[${port}]" --username "${user}" --password "${password}"
mongosh --host "rs0/127.0.0.1:27017" --username "root" --password "password"
mongosh "mongodb://${user}:${password}@${host}[${port}][/${database}][?replicaSet=rs0&readPreference=secondaryPreferred&retryWrites=false"]
mongosh "mongodb://root:password@127.0.0.1:27017/?replicaSet=rs0&readPreference=secondaryPreferred&retryWrites=false"


# localstack
pip install localstack
pip uninstall localstack
#
localstack --version
localstack start -d
localstack stop
localstack status services
#
# localstack use awscli
alias aws.local='aws --endpoint-url=http://localhost:4566'

```

# ES ElasticSearch
```js
GET /

GET /_cluster/health

// 修改 lifecycle 間隔時間 default: "10m"
// 目前會遇到錯誤 [xpack.monitoring.exporters.found-user-defined.auth.password] setting was deprecated in Elasticsearch and will be removed in a future release! See the breaking changes documentation for the next major version.
PUT /_cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval":"10s"
  }
}

GET /_cat/indices?v&index=*my_search_name*
GET /_cat/aliases?v&alias=*my_search_name*

# create index
PUT /test_idx_20210901_v1
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "title": {
        "type": "wildcard"
      }
    }
  },
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}

// change aliases
// If alias has multiple indexes, one of the indexes must be specified as "is_write_index"
POST /_aliases
{
  "actions" : [
    {
      "remove": {
        "index": "my_idx_v1",
        "alias": "my_idx_alias"
      }
    },
    {
      "add": {
        "index": "my_idx_v2",
        "alias": "my_idx_alias",
        "is_write_index": true
      }
    }
  ]
}

POST /test_idx_20210901/_doc
{
  "id": "my_id_1",
  "title": "my_title_1"
}

# get index info
GET /test_idx_20210901

GET /test_idx_20210901/_count

GET /test_idx_20210901/_doc/$_id

GET /test_idx_20210901/_search
{
  "sort":[{"_id":"desc"}],
  "from": 0,
  "size": 20,
  "timeout": "1s",
  "profile": true
}

POST /test_idx_20210901/_delete_by_query
{
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}

POST /_reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}

DELETE /test_idx_20210901_v2

```

## kubectl kubernetes k8s
```sh
# create ~/.kube/config
aws eks update-kubeconfig --name=${name}

kubectl get namespace

# show config
kubectl config get-contexts

# set default namespace
kubectl config set-context --current --namespace=${namespace}

kubectl [-n ${namespace}] get pod

kubectl delete pod ${pod_id}

kubectl [-n ${namespace}] get deployment
kubectl [-n ${namespace}] get job
kubectl [-n ${namespace}] get cronjob

kubectl rollout restart deploy ${deployment} [-n ${namespace}]

# port forward
kubectl [-n ${namespace}] port-forward deployment/${deployment} ${local_port}:${deployment_port}

kubectl logs ${pod_name}
kubectl logs ${pod_name} -c ${container_name}
kubectl logs ${pod_name} --all-containers

kubectl get secret
kubectl get secret ${secret_name} -o yaml
kubectl get secret ${secret_name} -o json

kubectl [-n ${namespace}] apply -f deployment.yaml

kubectl [-n ${namespace}] describe deployment ${deployment}

```

## k9s
```sh
# Source: https://github.com/derailed/k9s
# install
brew install k9s

# version
k9s version

# install with golang
go install -tags '' -ldflags="-X 'cmd.version=v0.25.18'" github.com/derailed/k9s@v0.25.18

```

## mongo
```js
NumberLong(new Date());

db.getCollection("my_collection").find().pretty();

db.getCollection("my_collection").updateOne({
    "_id": ObjectId("609de0d90455a0411a3b9a75")
  }, {
    $set: {
      "latestVersion": "1.1.7",
      "minVersion": "0.1.0",
    }
  }
);

db.getCollection("my_collection").count();

db.getCollection("my_collection").deleteOne({
  "id": NumberLong("442759920625822934"),
});

```

## git
```sh
# git log 比較兩個 tag 之間版本差異
git log --oneline v0.6.0..v0.8.0  | grep "Merge"

# 顯示最近的標籤包含 commit hash
# output: v0.3.0-1-g2d3e6a6
git describe

# 顯示最近的標籤，如果當前位置與標籤位置相同，則只顯示標籤
# output: v0.3.0
# output: v0.3.0-1-g2d3e6a6
git describe --tags

# 顯示最近的標籤，去除 commit hash
# output: v0.3.0
git describe --abbrev=0

# 顯示所有分支中最新的標籤
git describe --tags $(git rev-list --tags --max-count=1)

git describe --tags --dirty --always

```

## gpg
```sh
# install gpg2 on mac
brew install gpg

gpg --version

```


## docker
```sh
# docker repository: [${registry}/][${namespace}/]${image}
# default registry: hub.docker.com

# docker tag
# copy image
docker tag ${repository}[:${tag}] ${new_repository}[:${new_tag}]

# docker push
# push image to remote registry
docker push ${repository}[:${tag}]

# docker build
# options:
#   --no-cache
#   --build-arg ${arg_name}=${arg_value}
#   --progress=plain
#   -t ${repository}
#   -f ${dockerfile}
# env:
#   DOCKER_BUILDKIT=1
docker build ${path} [${options}]
docker build . [--no-cache] [--progress=plain] [-t ${repository}] [-f ${dockerfile}]

# docker image
# show all image
docker image ls -a
# remove useless images
docker image prune -a

# docker volume
# show all volume
docker volume ls
# remove useless volumes
docker volume prune

# docker network
# show all network
docker network ls
# remove useless networks
docker network prune

# docker remove all useless resource
docker system prune -a --volumes

# docker ps
# show all container
docker ps -a

# docker run
# options:
#   -d, --detach ## Run container in background and print container id
#   -p${local_port}:${container_port} ## Map local port to container port
#   -t, --tty ## Allocate a pseudo-TTY
#   --platform "${platform}" ## e.g. --platform "linux/amd64"
#   -e ${env_key}=${env_value} ## set env
#   --name ${name}
# run
docker run [${options}] ${repository}
# run in background
docker run -d ${repository}
# run and execute command
docker run -it ${repository} ${command}
# run ubuntu and execute bash
docker run -it ubuntu bash
# run mysql in background
docker run -d -p 3306:3306 -e 'MYSQL_ROOT_PASSWORD=password' mysql:5.7
docker run -d -p 3306:3306 -e 'MYSQL_ROOT_PASSWORD=password' mysql:8.0
# run redis in background
docker run -d -p 6379:6379 redis:6.2


# docker exec
# execute command
docker exec -it ${container_id} sh

```

## docker-compose
```sh
# run
docker-compose up

# run in the background
docker-compose up -d

# stop
docker-compose stop

# stop and remove
docker-compose down -v

```

## MySQL
```sql

-- Character Set & Collation
--
-- Collation name rules:
--   ai: accent insensitive. e.g. 'á' = 'a'
--   as: accent sensitive. e.g. 'á' != 'a'
--   ci: case insensitive.
--   cs: case sensitive.
--   bin: binary.
--
-- Set Connection Collation
SET NAMES 'utf8mb4' COLLATE 'utf8mb4_0900_as_cs';

-- Create Database
--
-- in mysql 8.0
CREATE DATABASE `db_name`;
CREATE DATABASE `db_name` CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
CREATE DATABASE `db_name` CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_as_cs;
--
-- in mysql 5.7
CREATE DATABASE `db_name` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE `db_name` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE DATABASE `db_name` CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

DROP TABLE IF EXISTS `table_name`;
CREATE TABLE IF NOT EXISTS `table_name` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `uid` varchar(255) NOT NULL,
  `value` varchar(255) NOT NULL DEFAULT '',
  `value2` varchar(255) DEFAULT NULL,
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uq_uid` (`uid`),
  KEY `idx_value_value2` (`value`,`value2`)
);

SET @@session.time_zone = '+00:00';
SET @@session.time_zone = '+08:00';
SELECT @@session.time_zone;


-- User Management
SELECT `user`, `host`, `account_locked` FROM `mysql`.`user`;

CREATE USER `user_name`@`localhost` IDENTIFIED BY 'password';
CREATE USER `user_name`@`%` IDENTIFIED BY 'password';

-- lock unlock user
ALTER USER `user_name`@`%` ACCOUNT LOCK;
ALTER USER `user_name`@`%` ACCOUNT UNLOCK;

-- update password
ALTER USER `user_name`@`localhost` IDENTIFIED BY 'password';

DROP USER `user_name`@`localhost`;

-- GRANT 授權
-- show current user grant.
SHOW GRANTS;
-- show specified user grant.
SHOW GRANTS FOR `user_or_role`@`localhost`;
SHOW GRANTS FOR `user1`@`%`;
SHOW GRANTS FOR `r_role1`@`%`;

-- GRANT PRIVILEGE
-- 使用 WITH GRANT OPTION 使用者可以將自己的權限給其他使用者
GRANT ALL PRIVILEGES ON *.* TO `user_or_role`@`%` WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `db_name`.`table_name` TO `user_or_role`@`%`;
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON *.* TO `user_or_role`@`%`;
GRANT `r_role_name` to `user_or_role`@`%`;

-- REVOKE PRIVILEGE
-- revoke all privileges in all database from the user_or_role
REVOKE ALL PRIVILEGES, GRANT OPTION FROM `user_or_role`@`%`;
REVOKE GRANT OPTION ON *.* FROM `user_or_role`@`%`;
REVOKE ALL PRIVILEGES ON *.* FROM `user_or_role`@`%`;
REVOKE `r_role_name` from `user_or_role`@`%`;

-- ROLE
-- role supports in mysql 8.0
CREATE ROLE `r_role_name`;
DROP ROLE `r_role_name`;
-- 活化使用者的角色
SET DEFAULT ROLE ALL TO `user_name`;

-- 刷新使用者設定以套用
FLUSH PRIVILEGES;

```
