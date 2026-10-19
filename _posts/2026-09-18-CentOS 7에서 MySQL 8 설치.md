---
layout: post
title: "CentOS 7에서 MySQL 8 설치"
date: 2026-09-18
last_modified_at: 2026-09-18
categories: study
tags: ["Blog", "study", "code", "linux", "정보보안"]
---

## [Web_Server]

Web_DB vm

- Mysql, Oracle downlods
    - 일반적으로 다운로드 했을 경우(터미널)

```java

woori is not in the sudoers file.  This incident will be reported. -> 에러 문구 출력
= 사용자가 sudo 명령어를 사용할 수 없음. -> root 권한 필요한 작업
```

1. su -
2. visudo
3. ALL 어쩌고 부분에 woori(사용자) ALL=(ALL) ALL 추가
4. esc
5. :wq + enter
6. exit(로그아웃)
7. sudo ls /root(다시 로그인)
-> 위 명령어 제대로 작동하면, 사용자가 sudo 권한을 성공적으로 얻은 것!
visudo -> 문법 오류 자동으로 검사

[mysql 설치]
sudo yum localinstall https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

```java
sudo yum install mysql-server
```

```java
sudo systemctl start mysqld
```

```java
sudo systemctl enable mysqld
-> 설치 중 에러 발생 ([woori@localhost ~]$ sudo systemctl start mysql
```

```java
Failed to start mysql.service: Unit not found.)
-> (에러 2.  Failing package is: mysql-community-libs-8.0.37-1.el7.x86_64
```

```java
GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql)
-> GPG 키 재구성
```

```java
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

```java
sudo rm -f /etc/yum.repos.d/mysql-community.repo
-> 마지막에 에러 Error: Nothing to do
```

```java
sudo rm -f /etc/yum.repos.d/mysql-community.repo(저장소 파일 삭제)
```

```java
sudo yum install https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm(저장소 재설정)
-> 에러 
/var/tmp/yum-root-gX7sBx/mysql80-community-release-el7-3.noarch.rpm: does not update installed package.
Error: Nothing to do
-> 이미 설치된 패키지 인듯.
```

```java
sudo yum install https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm(sql 저장소 추가)
sudo yum install mysql-community-server(패키지 다시 설치)
```

-> 해도 안됨 Loading mirror speeds from cached hostfile

```java
base: mirror.anigil.com
```

```java
extras: mirror.anigil.com
```

```java
updates: mirror.anigil.com
No package mysql-community-server available.
Error: Nothing to do
```

-> Mysql community Server 패키지를 찾을 수 없음

```java
cd /etc/yum.repos.d/
ls -l mysql*.repo
```

-> sql 저장소 설정 파일 확인

```java
[woori@localhost yum.repos.d]$ ls -l mysql*.repo
-rw-r--r--. 1 root root 2108 Apr 24  2019 mysql-community-source.repo
```

-> 저장소 설정 파일 확인 완료. mysql-community-source.repo만 존재함
-> 새로 생성해야 함 저장소 설정 파일

처음으로 돌아가서~

1. exit
2. 터미널 다시 접속
3. sudo vi /etc/yum.repos.d/mysql-community.repo
4. [mysql80-community]
name=MySQL 8.0 Community Server
baseurl=https://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
-> 그대로 붙여넣기
5. sudo yum install mysql-community-server
-> 서버 다시 생성

끗..!

2024-06-01T11:10:40.397389Z 0 [ERROR] [MY-010457] [Server] --initialize specifipecified but the data directory has files in it. Aborting.
2024-06-01T11:10:40.397483Z 0 [ERROR] [MY-013236] [Server] The designated data data directory /var/lib/mysql/ is unusable. You can remove all files that the at the server added to it.
2024-06-01T11:10:40.397676Z 0 [ERROR] [MY-010119] [Server] Aborting
2024-06-01T11:10:40.398773Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Sqld: Shutdown complete (mysqld 8.0.37)  MySQL Community Server - GPL.
2024-06-01T11:10:40.852847Z 0 [Warning] [MY-011070] [Server] 'Disabling symboliymbolic links using --skip-symbolic-links (or equivalent) is the default. Cons. Consider not using this option as it' is deprecated and will be removed in a future release.
2024-06-01T11:10:40.852909Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolid in a future release.
2024-06-01T11:10:40.858132Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mqld (mysqld 8.0.37) starting as process 54404
2024-06-01T11:10:40.882529Z 1 [ERROR] [MY-011011] [Server] Failed to find valid data directory.
2024-06-01T11:10:40.883000Z 0 [ERROR] [MY-010020] [Server] Data Dictionary inity initialization failed.
2024-06-01T11:10:40.883053Z 0 [ERROR] [MY-010119] [Server] Aborting
2024-06-01T11:10:40.885200Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Sqld: Shutdown complete (mysqld 8.0.37)  MySQL Community Server - GPL.

[woori@localhost ~]$ sudo yum-config-manager --disable mysql80-community
Loaded plugins: fastestmirror, langpacks
=========================== repo: mysql80-community ============================
[mysql80-community]
async = True
bandwidth = 0
base_persistdir = /var/lib/yum/repos/x86_64/7
baseurl = http://repo.mysql.com/yum/mysql-8.0-community/el/7/x86_64/
cache = 0
cachedir = /var/cache/yum/x86_64/7/mysql80-community
check_config_file_age = True
compare_providers_priority = 80
cost = 1000
deltarpm_metadata_percentage = 100
deltarpm_percentage =
enabled = 0
enablegroups = True
exclude =
failovermethod = priority
ftp_disable_epsv = False
gpgcadir = /var/lib/yum/repos/x86_64/7/mysql80-community/gpgcadir
gpgcakey =
gpgcheck = True
gpgdir = /var/lib/yum/repos/x86_64/7/mysql80-community/gpgdir
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
hdrdir = /var/cache/yum/x86_64/7/mysql80-community/headers
http_caching = all
includepkgs =
ip_resolve =
keepalive = True
keepcache = False
mddownloadpolicy = sqlite
mdpolicy = group:small
mediaid =
metadata_expire = 21600
metadata_expire_filter = read-only:present
metalink =
minrate = 0
mirrorlist =
mirrorlist_expire = 86400
name = MySQL 8.0 Community Server
old_base_cache_dir =
password =
persistdir = /var/lib/yum/repos/x86_64/7/mysql80-community
pkgdir = /var/cache/yum/x86_64/7/mysql80-community/packages
proxy = False
proxy_dict =
proxy_password =
proxy_username =
repo_gpgcheck = False
retries = 10
skip_if_unavailable = False
ssl_check_cert_permissions = True
sslcacert =
sslclientcert =
sslclientkey =
sslverify = True
throttle = 0
timeout = 30.0
ui_id = mysql80-community/x86_64
ui_repoid_vars = releasever,
basearch
username =

- ----------------------------------------------------- (정말 많은 삽질 이후 알아낸 방법들,,,)

`[root@localhost ~]# sudo yum update mysql-community-server`

`Loaded plugins: fastestmirror, langpacks`

`Loading mirror speeds from cached hostfile`

- base: ftp.jaist.ac.jp
- extras: ftp.kaist.ac.kr
- updates: ftp.kaist.ac.kr
Package(s) mysql-community-server available, but not installed.
No packages marked for update
`sudo yum install mysql-community-server`

Exiting on user command
Your transaction was saved, rerun it with:
yum load-transaction /tmp/yum_save_tx.2024-06-02.06-28.74KLJ_.yumtx
`sudo yum load-transaction /tmp/yum_save_tx.2024-06-02.06-28.74KLJ_.yumtxsudo rpm --import <https://repo.mysql.com/RPM-GPG-KEY-mysql`>

 → gpg 추가
`sudo yum load-transaction /tmp/yum_save_tx.2024-06-02.06-28.74KLJ_.yumtx` → 다시 설치

Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
GPG key retrieval failed: [Errno 14] curl#37 - "Couldn't open file /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql"
`sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql <https://repo.mysql.com/RPM-GPG-KEY-mysql`>
`sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-mysqlsudo yum load-transaction /tmp/yum_save_tx.2024-06-02.06-28.74KLJ_.yumtxsudo yum --nogpgcheck load-transaction /tmp/yum_save_tx.2024-06-02.06-28.74KLJ_.yumtx`

`yum --enablerepo=mysql80-community install mysql-community-server` 

설치 다른 버젼

https://www.infracody.com/2022/12/centos-7-mysql-57-rpm-gpg-key-error.htmlhttps://blog.naver.com/hsunryou/221988309317 

-> 매우 도움 많이 됨

Installed:
mysql-community-libs.x86_64 0:8.0.37-1.el7

mysql-community-libs-compat.x86_64 0:8.0.37-1.el7

mysql-community-server.x86_64 0:8.0.37-1.el7

Dependency Installed:
mysql-community-client.x86_64 0:8.0.37-1.el7

mysql-community-client-plugins.x86_64 0:8.0.37-1.el7

mysql-community-common.x86_64 0:8.0.37-1.el7

mysql-community-icu-data-files.x86_64 0:8.0.37-1.el7

Replaced:
mariadb-libs.x86_64 1:5.5.68-1.el7

Complete!

-- 설치 완!

`sudo systemctl start mysqld` ->서버 시작
`sudo systemctl status mysqld` -> 서버 상태 확인
`sudo grep 'temporary password' /var/log/mysqld.log` -> 임시 비번 확인
---- 비번 안바뀜
`mysql -u root -p` 비번 정책 확인
SHOW VARIABLES LIKE 'validate_password%'; -> 명령 실행

SET GLOBAL validate_password.policy = LOW;
SET GLOBAL validate_password.length = 6; -> 정책 완화

`sudo mysql_secure_installation` -> 비번 다시 시도
이후
SET GLOBAL validate_password.policy = MEDIUM;
SET GLOBAL validate_password.length = 8; -> 비번 정책 복원
`EXIT;` -> mysql 로그아웃

mysql 임시 비번
zqA5_spF3e#M