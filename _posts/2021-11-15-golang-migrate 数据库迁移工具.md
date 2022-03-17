---
layout: post
title:  "golang-migrate 数据库迁移工具"
date:   2021-11-15
description: 'golang-migrate 数据库迁移工具'
category: notes

---

rails 项目中自带 migration 是非常好用的数据库迁移工具，方便我们在开发中根据需求去记录数据表的创建、修改等历史版本

golang-migrate 是 go 项目中的数据库迁移工具，已经可以大概实现 rails 中 migration 的功能

macos 中可以用 homebrew 安装它 golang-migrate

```bash
brew install golang-migrate
```

创建迁移脚本，这会生成两个迁移脚本

```txt
 migrate create -ext sql -dir configs/db/migration -seq create_roles_table
```

001_create_sys_roless_table.down.sql

```
DROP TABLE IF EXISTS sys_roles;
```

001_create_sys_roles_table.up.sql

```sql
BEGIN;
-- 角色
CREATE TABLE IF NOT EXISTS sys_roles(
   "id" bigserial PRIMARY KEY,
   "name" varchar (50) NOT NULL,
   "status"  int NOT NULL DEFAULT (0),
   "weigh"  int NOT NULL DEFAULT (0),
   "data_scope" int NOT NULL DEFAULT (2),
   "remark" varchar (300),
   "creator_id" int DEFAULT (0),
   "updator_id" int DEFAULT (0),
   "created_at" timestamptz NOT NULL DEFAULT (now()),
   "updated_at" timestamptz NOT NULL DEFAULT (now()),
   "deleted_at" timestamptz DEFAULT NULL
);

COMMENT ON TABLE "sys_roles" IS '角色表';
COMMENT ON COLUMN "sys_roles"."status" IS 'must be positive';
COMMENT ON COLUMN "sys_roles"."data_scope" IS '数据范围';


COMMIT;
```



创建一个 `Makefile`  文件：:

```makefile
ifndef ENV
	DATABASE_HOST=localhost
	DATABASE_PORT=5432
	DATABASE_USER=xxx
	DATABASE_PASSWORD=xxx
	DATABASE_NAME=pmhuang_dev
endif

ifeq ($(ENV),dev)
	DATABASE_NAME=pmhuang_dev
endif

ifeq ($(ENV),test)
	DATABASE_NAME=pmhuang_test
endif

ifeq ($(ENV),prod)
	DATABASE_NAME=pmhuang_production
endif

createdb:
	createdb  -h $(DATABASE_HOST) -p$(DATABASE_PORT) -U$(DATABASE_USER) --owner=$(DATABASE_USER)  $(DATABASE_NAME)

dropdb:
	dropdb -h $(DATABASE_HOST) -p$(DATABASE_PORT) -U$(DATABASE_USER)  $(DATABASE_NAME)

migrateup:
	migrate -path configs/db/migration -database "postgresql://$(DATABASE_USER):$(DATABASE_PASSWORD)@$(DATABASE_HOST):$(DATABASE_PORT)/$(DATABASE_NAME)?sslmode=disable" -verbose up

migratereset:
	make dropdb && make createdb && make migrateup


.PHONY: dropdb	createdb	migrateup		migratereset
```



创建数据库:

```bash
make createdb
```

创建表：

```bash
make migrateup
```

回滚重置数据库：

```bash
make migratereset
```

删除数据库：

```bash
make dropdb
```

你也可以基于 docker 使用 go-migrate，这是一个 `Dockerfile`:

```docke
FROM ubuntu:latest
MAINTAINER andrew.zhyl@gmail.com

ENV  TZ="Asia/Shanghai"

RUN apt-get update -q \
  && apt-get -q -y install  ca-certificates tzdata curl postgresql-client vim bash make  --fix-missing --no-install-recommends

# tzdata 校正常见 Linux 系统时区
RUN ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime
RUN echo ${TZ} > /etc/timezone
RUN dpkg-reconfigure -f noninteractive tzdata

# 安装 go-migrate
RUN curl -L https://github.com/golang-migrate/migrate/releases/download/v4.11.0/migrate.linux-amd64.tar.gz | tar xvz
RUN mv ./migrate.linux-amd64 /usr/bin/migrate

RUN mkdir -p /opt/

WORKDIR /opt/

RUN mkdir -p ./log
COPY configs ./configs
COPY Makefile .

CMD ["/bin/bash"] 
```

本文仅为个人记录，不够详尽，有疑问可以看其它的资料

