---
title: Loader Instructions
category: advanced
---

# Loader instructions

## What is Loader?

Loader is a data import tool to load data to TiDB.

[Download the Binary](http://download.pingcap.org/tidb-enterprise-tools-latest-linux-amd64.tar.gz)

## Why did we develop Loader?

Since tools like mysqldump will take us days to migrate massive amounts of data, we used the mydumper/myloader suite of Percona to multi-thread export and import data. During the process, we found that mydumper works well. However, as myloader lacks functions of error retry and savepoint, it is inconvenient for us to use. Therefore, we developed loader, which reads the output data files of mydumper and imports data to TiDB through mysql protocol.

## What can Loader do?

+ Multi-thread import data

+ Support mydumper data format

+ Support error retry

+ Support savepoint

+ Improve the speed of importing data through system variable

## Usage

### Parameter description
```
  -L string: the log level setting. It can be set as debug, info, warn, error, fatal (default: "info")
  
  -P int: the port of TiDB (default: 4000)

  -V boolean: prints version and exit
  
  -c string: config file
  
  -checkpoint-schema string: the database name of checkpoint. In the execution process, loader will constantly update this database. After recovering from an interruption, loader will get the process of the last run through this database. (default: "tidb_loader")

  -d string: the storage directory of data that need to import (default: "./")
  
  -h string: the host of TiDB (default: "127.0.0.1")  
  
  -p string: the account and password of TiDB
  
  -pprof-addr string: the pprof address of Loader. It tunes the perfomance of Loader (default: ":10084")

  -t int: the number of thread,increase this as TiKV nodes increase (default: 16)
  
  -u string: the user name of TiDB (default: "root")
```

### Configuration file

Apart from command line parameters, you can also use configuration files. The format is shown as below:

```toml
# Loader log level
log-level = "info"

# Loader log file
log-file = ""

# Directory of the dump to import
dir = "./"

# Loader pprof addr
pprof-addr = ":10084"

# We saved checkpoint data to tidb, which schema name is defined here.
checkpoint-schema = "tidb_loader"

# Number of threads restoring concurrently for worker pool. Each worker restore one file at a time, increase this as TiKV nodes increase
pool-size = 16

# An alternative database to restore into
#alternative-db = ""
# Database to restore
#source-db = ""

# DB config
[db]
host = "127.0.0.1"
user = "root"
password = ""
port = 4000

# [[route-rules]]
# pattern-schema = "shard_db_*"
# pattern-table = "shard_table_*"
# target-schema = "shard_db"
# target-table = "shard_table"

```

### Usage

Command line parameter:

    ./bin/loader -d ./test -h 127.0.0.1 -u root -P 4000

Or use configuration file "config.toml":

    ./bin/loader -c=config.toml
    
### Note

+ If you use the default checkpoint-schema, after importing the data of a database, drop the tidb_loader database before you begin to import the next database. 
+ It is recommended to specify the `checkpoint-schema = "tidb_loader"` parameter when importing data.
