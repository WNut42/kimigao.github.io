---
layout: post
title: "PostgreSQL 如何读取外部数据"
date: 2018-04-18
comments: true
categories: [PostgreSQL]
---

最近在整理阿里云上 PostgreSQL 中的数据，其中一部分是给用户发通知的日志记录，这部分数据积攒了几年了，量比较大，占用空间较大，而且也导致在数据库全量备份时比较缓慢，本想着将之前旧数据删除掉，但感觉直接删除这部分数据比较可惜，所以就希望能有地方（PostgreSQL之外）能存储这部分数据，同时又可以在必要的时候（可能是帮运营查看之前数据）方便查询这部分数据。

查看了阿里云的文档，发现阿里云上的 PostgreSQL 支持 oss_fdw 扩展，可以实现在 PostgreSQL 中装载 oss 上的数据到数据库，同时也支持把数据库中数据写到 oss 上。所以满足了上面👆既能将 PostgreSQL 中数据存储到外部，又可以方便查询的需求。

在介绍 oss_fdw 之前，需要先了解下面👇几个概念。

#### SQL/MED

SQL/MED （management of external data）是 sql 语言中管理外部数据的一个扩展标准。它通过定义一个外部数据包装器和数据连接类型来管理外部数据。PostgreSQL 从 9.1 开始提供对 SQL/MED 的支持，通过 SQL/MED 可以连接到各种异构数据库或者 PostgreSQL 数据库。其相当于一套连接其他数据源的框架和标准。

#### 外部数据包装器（foreign data wrapper）

外部数据包装器（即 FDW），相当于定义了外部数据驱动。PostgreSQL 之所以可以访问外部数据是在 FDW 的帮助下被访问的。一个 FDW 是一个库，它可以与一个外部数据源通讯，并隐藏连接到数据源和从它获取数据的细节。PostgreSQL 目前支持很多 FDW，我们也可以在其他第三方产品中找到其他类型的 FDW（例如阿里云的 oss_fdw），具体可以参考 [这里](https://wiki.postgresql.org/wiki/Foreign_data_wrappers)。当然如果这些现有的 FDW 都不能满足你的需求，可以自己编写一个，可以参考 [这里](http://www.postgres.cn/docs/10/fdwhandler.html)。

#### 外部服务器（server）

外部数据服务器（即 server），相当于定义一个外部数据源，需要制定外部数据源的 FDW。

要访问外部数据，首先需要建立一个外部服务器对象，它根据它支持的外部数据包装器所使用的一组选项定义了如何连接到一个特定的外部数据源。

####  外部表（foreign table）

创建外部服务器后，我们需要创建一个或者多个外部表，它们定义了外部数据的结构。一个外部表可以在查询中像一个普通表一样使用，但是在 PostgreSQL 服务器中外部表没有存储数据。不管使用什么外部数据包装器，PostgreSQL 会要求外部数据包装器从外部数据源获取数据，或者在更新命令的情况下传达数据到外部数据源。

#### 用户映射（user mapping）

访问远程数据可能需要在外部数据源的授权。这些信息通过一个*用户映射*提供，它基于当前的PostgreSQL角色提供了附加的数据例如用户名和密码。



通过上面几个概念的了解，可以知道 oss_fdw 其实是阿里云开发的基于 PostgreSQL 的外部数据包装器的扩展，oss_fdw 和其他 fdw 的接口一样，提供对外部数据源 oss 的数据封装，用户可像使用数据表一样通过 oss_fdw 读取 oss 上存放的文件。和其他 fdw 一样，oss_fdw 提供独有的参数用于连接和解析 oss 上的文件数据。

具体通过下面代码实例了解一下，由于我是在 Rails 项目中应用 oss_fdw，所以代码都写在迁移文件中。

- 开启 oss_fdw / 创建外部服务

```sql
class AddFdwServer < ActiveRecord::Migration[5.1]
  def up
    if Rails.env.production?
      enable_extension 'oss_fdw'

      connection.execute <<-SQL
        CREATE SERVER ossserver FOREIGN DATA WRAPPER oss_fdw OPTIONS (host 'oss-cn-beijing.aliyuncs.com', id '***', key '***', bucket 'oss-fdw')
      SQL
    else
      enable_extension 'file_fdw'

      connection.execute <<-SQL
        CREATE SERVER file_server FOREIGN DATA WRAPPER file_fdw
      SQL
    end
  end

  def down
    if Rails.env.production?
      disable_extension 'oss_fdw'

      connection.execute <<-SQL
        DROP SERVER IF EXISTS ossserver;
      SQL
    else
      disable_extension 'file_fdw'

      connection.execute <<-SQL
        DROP SERVER IF EXISTS file_server;
      SQL
    end
  end
end
```

- 创建外部表

```sql
class CreateOssNotificationLogs < ActiveRecord::Migration[5.1]
  def up
    execute_sql = <<-SQL
      CREATE FOREIGN TABLE oss_logs (
        "id" integer,
        "request" json ,
        "response" text ,
        "status" integer,
        "created_at" timestamp,
        "updated_at" timestamp
      )
    SQL

    if Rails.env.production?
      execute_sql << " SERVER ossserver OPTIONS ( dir 'logs/', delimiter ',', format 'csv', encoding 'utf8');"
    else
      system 'touch /tmp/oss_logs.csv'
      execute_sql << " SERVER file_server OPTIONS  ( filename '/tmp/oss_logs.csv', format 'csv' );"
    end

    connection.execute execute_sql
  end

  def down
    connection.execute "DROP FOREIGN TABLE oss_logs"
  end
end
```

上面代码可以看到，只有在 production 环境下，才会开启 oss_fdw，开发或者测试环境下，可以使用 file_fdw，file_fdw 是 PostgreSQL 支持的基于文件（一般为 CSV）作为外部数据源的外部数据包装器。



##### 参考文章：

- [使用 oss_fdw 读写外部数据文本文件](https://help.aliyun.com/document_detail/44461.html)
- [PostgreSQL 外部数据定义](http://www.postgres.cn/docs/10/ddl-foreign-data.html)
- [SQL/MED](https://en.wikipedia.org/wiki/SQL/MED)
- [Foreign data wrappers](https://wiki.postgresql.org/wiki/Foreign_data_wrappers)
