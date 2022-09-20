# วิธีติดตั้ง Oracle 11g r2 บน ubuntu server 20.04LTS

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
