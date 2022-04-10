ติดตั้ง mysql ลงใน ubuntu data-server
https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04

sudo apt install mysql-server

sudo systemctl start mysql.service
### Local Virtual Machine Snapshot
192.168.18.5
data-server (ubuntu 20.04)
username: supakorn
password: lnwza007

192.168.18.4
crm-server (fedora 35)
username: supakorn
password: lnwza007

backend-server (ubuntu 35)
username: supakorn
password: lnwza007

### SQL Script on Data-Server
https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql
แก้ไฟล์ Config ใน /etc/mysql/mysql.conf.d
```
select user,host from mysql.user;

CREATE USER 'supakorn'@'192.168.18.4' IDENTIFIED BY 'lnwza007';
GRANT all privileges ON *.* TO 'supakorn'@'192.168.18.4';
flush privileges

create database napzz;
use napzz
create table customer(id int primary key,firstname varchar(100),lastname varchar(100),age int,)
inser ข้อมูลลงไป
```

ต่อ mysql แบบ remote ด้วยคำสั่ง
```
mysql -h 192.168.18.5 -P 3306 -u supakorn -p
```

ในเครื่อง Fedora ทดลองเปลี่ยน IP Address

```
nmcli show connection
```

ตอนนี้ตัว Bridge Router Pingไปหา Ip VM KubeVirt แต่ว่ามันไปไม่ได้เพราะมันใช้ MacVlan
https://github.com/kubevirt/kubevirt/issues/5483