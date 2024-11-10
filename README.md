# การติดตั้ง Oracle21c บน CentOS 7

## Step 1. Preinstall
เตรียม os user สำหรับลง database โดยในที่นี้จะใช้เป็น user oracle เตรียมพร้อมตัว install โดยในขั้นตอนแรกๆแนะนำให้ใช้งาน root user โดยใช้คำสั่ง
```bash
yum -y update
```

เตรียม file preinstall และ install Download ตัว rpm โดยใช้ curl
```bash
curl -o oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm https://yum.oracle.com/repo/OracleLinux/OL8/appstream/x86_64/getPackage/oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
```

ทำการ loacal install
```bash
yum -y localinstall oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
```
หรือ
```bash
rpm -ivh --nodeps oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
```

## Step 2. Install base rpm
Download file install ที่เป็น rpm จาก https://www.oracle.com/database/technologies/oracle-database-software-downloads.html
โดย ใช้ rpm ที่เป็น 21c — Enterprise Edition Linux x86–64
ทำการ install โดยใช้

1. Install Common Prerequisites: Install these packages that are generally required by Oracle Database:
```bash
sudo yum install -y nano binutils compat-libcap1 compat-libstdc++ gcc gcc-c++ glibc glibc-devel ksh libaio libaio-devel libX11 libXau libXi libXtst libgcc libstdc++ libstdc++-devel libxcb make smartmontools sysstat
```
2. Create Oracle User and Groups The preinstall package would typically create the oracle user and groups for you, but we can do it manually:
```bash
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper 
#groupadd -g 54324 backupdba
#groupadd -g 54325 dgdba
#groupadd -g 54326 kmdba
#groupadd -g 54327 asmdba
#groupadd -g 54328 asmoper
#groupadd -g 54329 asmadmin
#groupadd -g 54330 racdba

useradd -u 54321 -g oinstall -G dba,oper oracle
```
3.Configure Kernel Parameters Manually set kernel parameters in `/etc/sysctl.conf` as follows:
```bash
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.panic_on_oops = 1
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```
Apply the settings:
```bash
sudo sysctl -p
```

4. Configure Shell Limits for the Oracle User Add the following limits to `/etc/security/limits.conf`:
```bash
oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc     16384
oracle   hard   nproc     16384
oracle   soft   stack     10240
oracle   hard   stack     32768
oracle   hard   memlock   134217728
oracle   soft   memlock   134217728
```
5.Retry Installing Oracle Database RPM Once the environment is configured, attempt to install the oracle-database-ee-21c RPM again:
```bash
sudo rpm -ivh --nodeps oracle-database-ee-21c-1.0-1.ol8.x86_64.rpm
```

## Step 3. Creating and Configuring
หลังจาก install เรียบร้อยจะทำการเริ่ม config database โดยใช้คำสั่ง
```bash
/etc/init.d/oracledb_ORCLCDB-21c configure
```

## Step 4. Firewall
ทำการเพิ่ม port 1521 เข้าไป
```bash
firewall-cmd --add-port=1521/tcp --permanent
```
จากนั้นทำการ reload
```bash
firewall-cmd --reload
```
ตรวจสอบโดยใช้คำสั่ง
```bash
sudo firewall-cmd --list-ports
```

## Step5.Edit Environment variables
ในขั้นตอนนี้จะเป็นการตั้งค่า ORACLE_HOME , PATH ,ORACLE_SID , LANG โดยจะต้องทำการสลับ user โดยใช้คำสั่ง
```bash
su oracle
```
ทำการแก้ไขไฟล์ .bashrc
```bash
nano ~/.bashrc
```
เพิ่มคำสั่งต่อไปนี้ในไฟล์
```bash
export LANG=en_US.UTF-8
export ORACLE_HOME=/opt/oracle/product/21c/dbhome_1 
export PATH=$PATH:/opt/oracle/product/21c/dbhome_1/bin 
export ORACLE_SID=ORCLCDB
```
หลังจาก save แล้วทำการ refresh ด้วยคำสั่ง
```bash
source ~/.bashrc
```

## Step 6. Listener
จากนั้นเราจะทำการตรวจสอบตัว listener ว่าทำงานปกติหรือไม่ด้วยคำสั่ง
```bash
lsnrctl status
```
หาก listener ยังไม่ถูกเปิดจะมีข้อความหน้าตาแบบนี้
```
[oracle@localhost ~]$ lsnrctl status

LSNRCTL for Linux: Version 21.0.0.0.0 - Production on 01-NOV-2024 03:30:54

Copyright (c) 1991, 2021, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 21.0.0.0.0 - Production
Start Date                01-NOV-2024 02:57:07
Uptime                    0 days 0 hr. 33 min. 49 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /opt/oracle/homes/OraDBHome21cEE/network/admin/listener.ora
Listener Log File         /opt/oracle/diag/tnslsnr/localhost/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=localhost)(PORT=5500))(Security=(my_wallet_directory=/opt/oracle/admin/ORCLCDB/xdb_wallet))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "25cc73bee5191762e065000000000001" has 1 instance(s).
  Instance "ORCLCDB", status READY, has 1 handler(s) for this service...
Service "ORCLCDB" has 1 instance(s).
  Instance "ORCLCDB", status READY, has 1 handler(s) for this service...
Service "ORCLCDBXDB" has 1 instance(s).
  Instance "ORCLCDB", status READY, has 1 handler(s) for this service...
Service "orclpdb1" has 1 instance(s).
  Instance "ORCLCDB", status READY, has 1 handler(s) for this service...
The command completed successfully
```
ให้เราทำการ start โดยใช้คำสั่ง
```bash
lsnrctl start
```
แสดงข้อความต่อไปนี้
```
[oracle@localhost ~]$ lsnrctl start

LSNRCTL for Linux: Version 21.0.0.0.0 - Production on 01-NOV-2024 03:31:55

Copyright (c) 1991, 2021, Oracle.  All rights reserved.

TNS-01106: Listener using listener name LISTENER has already been started
```

## Step 7. ทดลองใช้ sqlplus
ทำลองเข้าใช้งาน database ด้วยคำสั่ง
```bash
sqlplus / as sysdba
```
ทำการกำหนด password ให้กับ user system ด้วยคำสั่ง
```bash
alter user system identified by <กำหนด password>;

```

## แหล่งที่มาของข้อมูล
- [การติดตั้ง oracle database บน centos 7](https://medium.com/@jackchawanwit/%E0%B8%9A%E0%B8%B1%E0%B8%99%E0%B8%97%E0%B8%B6%E0%B8%81-%E0%B8%81%E0%B8%B2%E0%B8%A3%E0%B8%95%E0%B8%B4%E0%B8%94%E0%B8%95%E0%B8%B1%E0%B9%89%E0%B8%87-oracle-database-%E0%B8%9A%E0%B8%99-centos-7-e49b648fe68)
- [Oracle Database 21c Installation On Oracle Linux 8 (OL8)](https://oracle-base.com/articles/21c/oracle-db-21c-installation-on-oracle-linux-8)



