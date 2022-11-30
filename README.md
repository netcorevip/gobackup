<p align="center">
<img src="https://gobackup.github.io/images/gobackup.svg" width="160" />

<h1 align="center">GoBackup</h1>
<p align="center">Simple tool for backup your databases, files to cloud storages.</p>
<p align="center">
   <a href="https://github.com/huacnlee/gobackup/actions?query=workflow%3AGo"><img src="https://github.com/huacnlee/gobackup/workflows/Go/badge.svg" alt="Build Status" /></a>
   <a href="https://github.com/huacnlee/gobackup/releases"><img src="https://img.shields.io/github/v/release/huacnlee/gobackup?label=Version&color=1" alt="GitHub release (latest by date)"></a>
   <a href="https://hub.docker.com/r/huacnlee/gobackup"><img src="https://img.shields.io/docker/v/huacnlee/gobackup?label=Docker&color=blue" alt="Docker Image Version (latest server)"></a>
</p>
</p>

GoBackup is a fullstack backup tool design for web servers similar with [backup/backup](https://github.com/backup/backup), work with Crontab to backup automatically.

You can write a config file, run `gobackup perform` command by once to dump database as file, archive config files, and then package them into a single file.

It's allow you store the backup file to local, FTP, SCP, S3 or other cloud storages.

GoBackup 是一个类似 [backup/backup](https://github.com/backup/backup) 的一站式备份工具，为中小型服务器／个人服务器而设计，配合 Crontab 以实现定时备份的目的。

使用 GoBackup 你可以通过一个简单的配置文件，一次（执行一个命令）将服务器上重要的（数据库、配置文件）东西导出、打包压缩，并备份到指定目的地（如：本地路径、FTP、云存储...）。

详细中文介绍：https://ruby-china.org/topics/34094

https://gobackup.github.io/

## Features

- No dependencies.
- Multiple Databases source support.
- Multiple Storage type support.
- Archive paths or files into a tar.

## Current Support status

### Databases

- MySQL
- PostgreSQL
- Redis - `mode: sync/copy`
- MongoDB

### Archive

Use `tar` command to archive many file or path into a `.tar` file.

### Compressor

- Tgz - `.tar.gz`
- Uncompressed - `.tar`

### Encryptor

- OpenSSL - `aes-256-cbc` encrypt

### Storages

- Local
- FTP
- SCP - Upload via SSH copy
- [Amazon S3](https://aws.amazon.com/s3)
- [Aliyun OSS](https://www.aliyun.com/product/oss)
- [Google Cloud Storage](https://cloud.google.com/storage)
- [Backblaze B2 Cloud Storage](https://www.backblaze.com/b2)
- [Cloudflare R2](https://www.cloudflare.com/products/r2)
- [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces)
- [QCloud COS](https://cloud.tencent.com/product/cos)
- [UCloud US3](https://docs.ucloud.cn/ufile/introduction/concept)
- [Qiniu Kodo](https://www.qiniu.com/products/kodo)

## Install (macOS / Linux)

```bash
$ curl -sSL https://git.io/gobackup | bash
```

after that, you will get `/usr/local/bin/gobackup` command.

```bash
$ gobackup -h
NAME:
   gobackup - Easy full stack backup operations on UNIX-like systems

USAGE:
   gobackup [global options] command [command options] [arguments...]

VERSION:
   1.2.0

COMMANDS:
     perform
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```

## Configuration

GoBackup will seek config files in:

- ~/.gobackup/gobackup.yml
- /etc/gobackup/gobackup.yml

Example config: [gobackup_test.yml](https://github.com/huacnlee/gobackup/blob/master/gobackup_test.yml)

```yml
models:
  gitlab:
    compress_with:
      type: tgz
    store_with:
      type: scp
      path: ~/backup
      host: your-host.com
      private_key: ~/.ssh/id_rsa
      username: ubuntu
      password: password
      timeout: 300
    databases:
      gitlab:
        type: mysql
        host: localhost
        port: 3306
        database: gitlab_production
        username: root
        password:
        additional_options: --single-transaction --quick
      gitlab_redis:
        type: redis
        mode: sync
        rdb_path: /var/db/redis/dump.rdb
        invoke_save: true
        password:
    archive:
      includes:
        - /home/git/.ssh/
        - /etc/mysql/my.conf
        - /etc/nginx/nginx.conf
        - /etc/nginx/conf.d
        - /etc/redis/redis.conf
        - /etc/logrotate.d/
      excludes:
        - /home/ubuntu/.ssh/known_hosts
        - /etc/logrotate.d/syslog
  gitlab_repos:
    store_with:
      type: local
      path: /data/backups/gitlab-repos/
    archive:
      includes:
        - /home/git/repositories
```

## Usage

```bash
$ gobackup perform
[Modal: ruby_china] 2022/11/30 13:11:28 WorkDir: /tmp/gobackup/1669785088876548728/ruby_china
[Database] 2022/11/30 13:11:28 => database | postgresql : postgresql
[PostgreSQL] 2022/11/30 13:11:28 -> Dumping PostgreSQL...
[PostgreSQL] 2022/11/30 13:11:39 dump path: /tmp/gobackup/1669785088876548728/ruby_china/postgresql/postgresql/ruby-china.sql
[Database] 2022/11/30 13:11:39
[Database] 2022/11/30 13:11:39 => database | redis : redis
[Redis] 2022/11/30 13:11:39 -> Invoke save...
[Redis] 2022/11/30 13:11:39 Copying redis dump to /tmp/gobackup/1669785088876548728/ruby_china/redis/redis
[Database] 2022/11/30 13:11:40
[Archive] 2022/11/30 13:11:40 => includes 7 rules
[Compressor] 2022/11/30 13:11:41 => Compress | tgz
[Compressor] 2022/11/30 13:12:09 -> /tmp/gobackup/1669785088876548728/2022.11.30.13.11.41.tar.gz
[Encryptor] 2022/11/30 13:12:09 => Encrypt | openssl
[Encryptor] 2022/11/30 13:12:15 -> /tmp/gobackup/1669785088876548728/2022.11.30.13.11.41.tar.gz.enc
[Storage] 2022/11/30 13:12:15 => Storage | oss
[OSS] 2022/11/30 13:12:15 endpoint: oss-cn-hongkong.aliyuncs.com
[OSS] 2022/11/30 13:12:15 bucket: ruby-china-backup
[OSS] 2022/11/30 13:12:15 -> Uploading backups/2022.11.30.13.11.41.tar.gz.enc...
[Modal] 2022/11/30 13:12:15 Cleanup temp dir...
```

## Backup schedule

You may want run backup in scheduly, you need Crontab:

```bash
$ crontab -l
0 0 * * * /usr/local/bin/gobackup perform >> ~/.gobackup/gobackup.log
```

> `0 0 * * *` means run at 0:00 AM, every day.

And after a day, you can check up the execute status by `~/.gobackup/gobackup.log`.

## License

MIT
