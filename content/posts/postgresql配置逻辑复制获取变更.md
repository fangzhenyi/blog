---
title: postgresq配置逻辑复制并获取数据变更
date: 2023-08-05 20:38:35

description: postgresq配置逻辑复制并获取数据变更
keywords: postgresq 逻辑复制
tags:
  - postgresql
  - 逻辑复制
categories:
  - blog
collections:
  - blog

---
### 通过docker搭建postgresq

通过docker搭建

```
docker run -d \
	--name some-postgres \
	-e POSTGRES_PASSWORD=qazwsx123 \
	-e PGDATA=/var/lib/postgresql/data/pgdata \
	-v /var/postgres-data:/var/lib/postgresql/data -p 15432:5432 \
	postgres:13.5
```

然后修改配置文件

```
cd /var/postgres-data/pgdata
```
修改postgresql.conf
```
wal_level = 'logical';
max_replication_slots = 5; #该值要大于1
```
创建数据库
```
create database fzy;

```
创建有replication权限的用户
```
CREATE ROLE test_rep LOGIN  ENCRYPTED PASSWORD 'test_rep' REPLICATION;
GRANT CONNECT ON DATABASE fzy to test_rep;
```

重启数据库

```
docker restart some-postgres
```

创建表，执行命令

```
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);
```

执行写入命令

```
 insert into company values(1,'fzy',12,'aaa',1.2)
```

### 通过java程序捕捉变化

代码如下：

```
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.5</version>
        </dependency>
```

```

import org.postgresql.PGConnection;
import org.postgresql.PGProperty;
import org.postgresql.replication.PGReplicationStream;

import java.nio.ByteBuffer;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;
import java.util.concurrent.TimeUnit;

public class Main {
    public static void main(String[] args) throws SQLException, InterruptedException {
        String url = "jdbc:postgresql://xxx.xxx.xxx.xxx:15432/fzy";
        Properties props = new Properties();
        PGProperty.USER.set(props, "test_rep");
        PGProperty.PASSWORD.set(props, "test_rep");
        PGProperty.ASSUME_MIN_SERVER_VERSION.set(props, "9.4");
        PGProperty.REPLICATION.set(props, "database");
        PGProperty.PREFER_QUERY_MODE.set(props, "simple");

        Connection con = DriverManager.getConnection(url, props);
        PGConnection replConnection = con.unwrap(PGConnection.class);

        replConnection.getReplicationAPI()
                .createReplicationSlot()
                .logical()
                .withSlotName("test_slot")
                .withOutputPlugin("test_decoding")
                .make();

        PGReplicationStream stream =
                replConnection.getReplicationAPI()
                        .replicationStream()
                        .logical()
                        .withSlotName("test_slot")
                        .withSlotOption("include-xids", false)
                        .withSlotOption("skip-empty-xacts", true)
                        .withStatusInterval(20, TimeUnit.SECONDS)
                        .start();

        while (true) {
            //non blocking receive message
            ByteBuffer msg = stream.readPending();

            if (msg == null) {
                TimeUnit.MILLISECONDS.sleep(10L);
                continue;
            }

            int offset = msg.arrayOffset();
            byte[] source = msg.array();
            int length = source.length - offset;
            System.out.println(new String(source, offset, length));

            //feedback
            stream.setAppliedLSN(stream.getLastReceiveLSN());
            stream.setFlushedLSN(stream.getLastReceiveLSN());
        }
    }
}

```

最终效果如下

```
BEGIN
table public.company: INSERT: id[integer]:1 name[text]:'fzy' age[integer]:12 address[character]:'aaa                                               ' salary[real]:1.2
COMMIT
```