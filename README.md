# วิธีติดตั้ง Oracle 11g r2 บน ubuntu server 20.04LTS

## สร้าง Swap File
1. ทำการปืดการใช้งาน swap file ชั่วคราวก่อน
```sh
sudo swapoff -a
```
2. สร้าง Swap file ขึ้นมาใหม่
```sh
sudo dd if=/dev/zero of=/swapfile bs=1G count=8
sudo mkswap /swapfile
```
3. ทำการเปิดการใช้งาน Swap file ขึ้นมาอีกครั้ง
```sh
sudo swapon /swapfile
```
4. ตรวจสอบ Swap file ว่าตรงกับค่าที่กำหนดไว้หรือไม่
```sh
grep SwapTotal /proc/meminfo
```
`
#Output
SwapTotal:       8388604 kB
`

## สร้าง User group ของ Oracle
1. ใช้ user root ในการติดตั้ง
```sh
sudo su
```
2. สร้าง Oracle Inventory group
```sh
groupadd oinstall
```
3. สร้าง Oracle DBA group:
```sh
groupadd dba
```
4. สร้าง home directory ของ Oracle user:
```sh
mkdir /home/oracle/
```
5. สร้าง directory สำหรับติดตั้ง Oracle:
```sh
mkdir -p /u01/app/oracle
```
6. สร้าง User สำหรับ Oracle และ Directory สำหรับ User 
```sh
useradd -g oinstall -G dba -d /home/oracle -s /bin/bash oracle
```
7. ตั้งค่า Password ให้กับ Oracle user
```sh
passwd oracle
```
8. ตั้งค่าให้ User oracle เป็น Owner ของ Directory ในข้อ 4 และ 5
```sh
chown -R oracle:oinstall /home/oracle

chown -R oracle:oinstall /u01/app/oracle
```
9. สร้าง directory สำหรับ Oracle Inventory:
```sh
mkdir -p /u01/app/oraInventory
```
10. ตั้งค่าให้ oracle user เป็น owner ของ Oracle Inventory directory:
```sh
chown -R oracle:oinstall /u01/app/oraInventory
```

## ตั้งค่า kernel parameters
1. แก้ไขไฟล์  `/etc/sysctl.conf` ตามตัวอย่างต่อไปนี้
```sh
nano /etc/sysctl.conf
```

```sh
#File: /etc/sysctl.conf
# ============================

# Oracle 11g

# ============================

# semaphores: semmsl, semmns, semopm, semmni

kernel.sem = 250 32000 100 128

kernel.shmall = 2097152

kernel.shmmni = 4096

# Replace kernel.shmmax with the half of your memory size in bytes

# if lower than 4 GB minus 1

# 2147483648 is 2 GigaBytes (4 GB of RAM / 2)

kernel.shmmax=2147483648

#

# Max number of network connections. Use sysctl -a | grep ip_local_port_range to check.

net.ipv4.ip_local_port_range = 9000  65500

#

net.core.rmem_default = 262144

net.core.rmem_max = 4194304

net.core.wmem_default = 262144

net.core.wmem_max = 1048576

#

# The maximum allowed value, set to avoid overhead and input/output errors

fs.aio-max-nr = 1048576

# 512 * Processes

fs.file-max = 6815744

fs.suid_dumpable = 1

#

# To allow dba to allocate hugetlbfs pages

# 1001 is your oinstall group, you can check this id with the grep oinstall /etc/group command

vm.hugetlb_shm_group = 1001
```
2. ทำการ Aplly การตั้งค่าไฟล์ `/etc/sysctl.conf`
```sh
sysctl -p
```

## ตั้งค่า shell limits
แก้ไขไฟล์ `/etc/security/limits.conf`
```sh
# Oracle
# File: /etc/security/limits.conf

oracle           soft    nproc   2047

oracle           hard    nproc   16384

oracle           soft    nofile  1024

oracle           hard    nofile  65536

oracle           soft    stack   10240
```

## ตรวจสอบการตั้งค่า PAM
ตรวจสอบให้แน่ใจว่า `/etc/pam.d/login` มีการตั้งค่าตามนี้
```sh
cat /etc/pam.d/login | grep pam_limits.so
```
```
#Output
session    required   pam_limits.so
```

## ตั้งค่า shell profile
```sh
nano /etc/profile
```
เติมคำสั่งต่อไปนี้ลงไปในไฟล์ `/etc/profile`
```
# File: /etc/profile
...
if [ $USER = "oracle" ]; then

        if [ $SHELL = "/bin/ksh" ]; then

              ulimit -p 16384

              ulimit -n 65536

        else

              ulimit -u 16384 -n 65536

        fi

fi
```

## ติดตั้ง Package ที่จำเป็น
1. อัพเดท ubuntu
```sh
apt update
```
2. ทำการตั้ง Package ต่อไปนี้
```sh
apt install alien -y
apt install autoconf -y
apt install automake -y
apt install autotools-dev -y
apt install binutils -y
apt install bzip2 -y
apt install doxygen -y
apt install elfutils -y
apt install expat -y
apt install gawk -y
apt install gcc -y
apt install gcc-multilib -y
apt install g++-multilib -y
apt install libelf-dev -y
apt install libltdl-dev -y
apt install libodbcinstq4-1 libodbcinstq4-1:i386 -y
apt install libpth-dev -y
apt install libpthread-stubs0-dev -y
apt install libstdc++5 -y
apt install make -y
apt install openssh-server -y
apt install rlwrap -y
apt install rpm -y
apt install sysstat -y
apt install unixodbc -y
apt install unixodbc-dev -y
apt install unzip -y
apt install x11-utils -y
apt install zlibc -y
apt install libaio1 -y
apt install libaio-dev -y
apt install ia32-libs -y
apt install lib32z1 -y
apt install libmotif4 -y
```

## สร้าง symbolic links
1. รันคำสั่งต่อไปนี้
```sh
mkdir /usr/lib64

ln -s /etc /etc/rc.d

ln -s /usr/bin/awk /bin/awk

ln -s /usr/bin/basename /bin/basename

ln -s /usr/bin/rpm /bin/rpm

ln -s /lib/x86_64-linux-gnu/libgcc_s.so.1 /usr/lib64/

ln -s /usr/lib/x86_64-linux-gnu/libc_nonshared.a /usr/lib64/

ln -s /usr/lib/x86_64-linux-gnu/libpthread_nonshared.a /usr/lib64/

ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib64/

ln -s /usr/lib/x86_64-linux-gnu /usr/lib64

ln -sf /bin/bash /bin/sh

ln -s /etc/rc0.d /etc/rc.d/rc0.d

ln -s /etc/rc2.d /etc/rc.d/rc2.d

ln -s /etc/rc3.d /etc/rc.d/rc3.d

ln -s /etc/rc4.d /etc/rc.d/rc4.d

ln -s /etc/rc5.d /etc/rc.d/rc5.d

ln -s /etc/rc6.d /etc/rc.d/rc6.d

ln -s /etc/init.d /etc/rc.d/init.d
```
2. สร้างไฟล์ Red Hat Linux release
```sh
echo 'Red Hat Linux release 5' > /etc/redhat-release
```

## ตั้งค่า oracle user profile
1. ทำการ Login เข้าสู่ user oracle
```sh
su oracle
cd
```
2. แก้ไขไฟล์ `.bashrc`
```sh
nano ~/.bashrc
```
ตั้งค่าตามตัวอย่างต่อไปนี้
```
# File: ~/.bashrc
# Oracle Settings

TMP=/tmp; export TMP

TMPDIR=$TMP; export TMPDIR

# Enter your hostname

ORACLE_HOSTNAME=ubuntu-oracle11; export ORACLE_HOSTNAME

ORACLE_UNQNAME=ORADB11G; export ORACLE_UNQNAME

ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE

ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1; export ORACLE_HOME

ORACLE_SID=ORADB11G; export ORACLE_SID

PATH=/usr/sbin:$PATH; export PATH

PATH=$ORACLE_HOME/bin:$PATH; export PATH

LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/lib64; export LD_LIBRARY_PATH

CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH

umask 022
```
3. ทำการ Apply ไฟล์ด้วคำสั่ง
```sh
source ~/.bashrc
```
4. ทำการ restart หนึ่งครั้ง

## ติดตั้ง Oracle
1. ทำการ Download File ไปวางใน `/home/oracle`
- linux.x64_11gR2_database_1of2.zip
- linux.x64_11gR2_database_2of2.zip
3. ทำการ Set Owner ให้กับไฟล์ติดตั้ง
```sh
chown oracle:oinstall linux.x64_11gR2_*.zip
```
4. ย้ายไฟล์ติดตั้งไปอยู่ที่ `ORACLE_HOME`
```sh
mv linux.x64_11gR2_database_*.zip /u01/app/oracle/product/11.2.0/dbhome_1
```
5. ทำการ Login ด้วย user oracle
```sh
su oracle
```
6. ทำการ unzip
```sh
cd $ORACLE_HOME

unzip linux.x64_11gR2_database_1of2.zip

unzip linux.x64_11gR2_database_2of2.zip
```
7. ทำการติดตั้ง
```sh
cd ${ORACLE_HOME}/database

./runInstaller -ignorePrereq -waitforcompletion -silent                        \
    -responseFile ${ORACLE_HOME}/database/response/db_install.rsp               \
    oracle.install.option=INSTALL_DB_SWONLY                                    \
    ORACLE_HOSTNAME=${ORACLE_HOSTNAME}                                         \
    UNIX_GROUP_NAME=oinstall                                                   \
    INVENTORY_LOCATION=${ORA_INVENTORY}                                        \
    SELECTED_LANGUAGES=en,en_GB                                                \
    ORACLE_HOME=${ORACLE_HOME}                                                 \
    ORACLE_BASE=${ORACLE_BASE}                                                 \
    oracle.install.db.InstallEdition=EE                                        \
    oracle.install.db.OSDBA_GROUP=dba                                          \
    oracle.install.db.OSBACKUPDBA_GROUP=dba                                    \
    oracle.install.db.OSDGDBA_GROUP=dba                                        \
    oracle.install.db.OSKMDBA_GROUP=dba                                        \
    oracle.install.db.OSRACDBA_GROUP=dba                                       \
    SECURITY_UPDATES_VIA_MYORACLESUPPORT=false                                 \
    DECLINE_SECURITY_UPDATES=true
    ```
