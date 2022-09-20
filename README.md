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
