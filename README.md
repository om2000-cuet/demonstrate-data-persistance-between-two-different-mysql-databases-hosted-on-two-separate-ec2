<img alt="" class="bg lk ll c" width="700" height="904" loading="eager" role="presentation" src="https://miro.medium.com/v2/resize:fit:1400/1*ZbhuMsSRrfZbN65a4Gzc6Q.jpeg">
<br/><br/>
Create 2 EC2 Instance Name Old and New <br/>
 in the Old EC2 instance i have attached a 20GB EBS Volume<br/>
 in the New EC2 instance i have kept the default 8GB storage<br/>
 
 In Old EC2<br/>
ubuntu@ip-172-31-87-243:~$ sudo apt-get update<br/><br/>
 
 
 # Add Docker's official GPG key:
ubuntu@ip-172-31-87-243:~$ sudo apt-get update<br/>
sudo apt-get install ca-certificates curl gnupg<br/>
sudo install -m 0755 -d /etc/apt/keyrings<br/>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg<br/>
sudo chmod a+r /etc/apt/keyrings/docker.gpg<br/>

# Add the repository to Apt sources:<br/>
echo \<br/>
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \<br/>
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \<br/>
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null<br/>
sudo apt-get update<br/>
<br/><br/>

Install the latest Docker packages.<br/>
ubuntu@ip-172-31-87-243:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 <br/>
 
Verify that the Docker Engine installation is successful by running the hello-world image.<br/>
ubuntu@ip-172-31-87-243:~$ sudo docker run hello-world
 
Check volume<br/>

 
ubuntu@ip-172-31-87-243:~$ sudo df -h<br/><br/>

Filesystem      Size  Used Avail Use% Mounted on<br/>
/dev/root       7.6G  2.3G  5.3G  31% /<br/>
tmpfs           483M     0  483M   0% /dev/shm<br/>
tmpfs           194M  864K  193M   1% /run<br/>
tmpfs           5.0M     0  5.0M   0% /run/lock<br/>
/dev/xvda15     105M  6.1M   99M   6% /boot/efi<br/>
tmpfs            97M  4.0K   97M   1% /run/user/1000<br/>

<br/>

ubuntu@ip-172-31-87-243:~$ sudo lsblk<br/><br/>

NAME     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0      7:0    0  24.4M  1 loop /snap/amazon-ssm-agent/6312
loop1      7:1    0  55.6M  1 loop /snap/core18/2745
loop2      7:2    0  63.3M  1 loop /snap/core20/1879
loop3      7:3    0 111.9M  1 loop /snap/lxd/24322
loop4      7:4    0  53.2M  1 loop /snap/snapd/19122
xvda     202:0    0     8G  0 disk
├─xvda1  202:1    0   7.9G  0 part /
├─xvda14 202:14   0     4M  0 part
└─xvda15 202:15   0   106M  0 part /boot/efi
xvdb     202:16   0    20G  0 disk
<br/><br/>

ubuntu@ip-172-31-87-243:~$ pwd<br/>
/home/ubuntu

<br/><br/>
Creating a folder named mydata  in /home/ubuntu
<br/><br/>
ubuntu@ip-172-31-87-243:~$ mkdir mydata<br/>

ubuntu@ip-172-31-87-243:~$ ls<br/>
mydata<br/>
 <br/><br/>
Mount  /dev/xvdb  to  /home/ubuntu/mydata<br/>
ubuntu@ip-172-31-87-243:~$ sudo mount /dev/xvdb  /home/ubuntu/mydata<br/>
mount: /home/ubuntu/mydata: wrong fs type, bad option, bad superblock on /dev/xvdb, missing codepage or helper program, or other error.<br/><br/>

Creating FileSystem<br/>
ubuntu@ip-172-31-87-243:~$ sudo file -s /dev/xvdb<br/>
/dev/xvdb: data<br/>

ubuntu@ip-172-31-87-243:~$ sudo mkfs -t xfs /dev/xvdb<br/><br/>

meta-data=/dev/xvdb              isize=512    agcount=4, agsize=1310720 blks<br/>
         =                       sectsz=512   attr=2, projid32bit=1<br/>
         =                       crc=1        finobt=1, sparse=1, rmapbt=0<br/>
         =                       reflink=1    bigtime=0 inobtcount=0<br/>
data     =                       bsize=4096   blocks=5242880, imaxpct=25<br/>
         =                       sunit=0      swidth=0 blks<br/>
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1<br/>
log      =internal log           bsize=4096   blocks=2560, version=2<br/>
         =                       sectsz=512   sunit=0 blks, lazy-count=1<br/>
realtime =none                   extsz=4096   blocks=0, rtextents=0<br/>
<br/><br/>
 ubuntu@ip-172-31-87-243:~$ sudo file -s /dev/xvdb<br/>
/dev/xvdb: SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
<br/><br/>
Mounting the 20GB EBS volume to mydata folder<br/>
ubuntu@ip-172-31-87-243:~$ sudo mount /dev/xvdb  /home/ubuntu/mydata/

<br/><br/>
Creating Docker Compose file<br/>
ubuntu@ip-172-31-87-243:~$ vim docker-compose.yml
<br/><br/>
version: "3"

services:<br/>
  mysql:<br/>
    image: mysql:latest<br/>
    container_name: db<br/>
    environment:<br/>
      MYSQL_ROOT_PASSWORD: a<br/>
      MYSQL_DATABASE: db<br/>
      MYSQL_USER: a<br/>
      MYSQL_PASSWORD: a<br/>
    volumes:<br/>
      - ./mydata:/var/lib/mysql<br/>
    ports:<br/>
      - "3306:3306"<br/>
    restart: always<br/>

<br/><br/>
ubuntu@ip-172-31-87-243:~$ sudo docker compose up<br/>

ubuntu@ip-172-31-87-243:~$ ls<br/>
docker-compose.yml  mydata<br/>

ubuntu@ip-172-31-87-243:~$ sudo docker ps<br/>
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                  NAMES<br/>
120acc3b71d5   mysql:latest   "docker-entrypoint.s…"   10 minutes ago   Up 10 minutes   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   db<br/>
<br/><br/>

ubuntu@ip-172-31-87-243:~$ sudo docker exec -it db bash<br/>
bash-4.4# mysql -u root -p<br/>
Enter password:<br/>
Welcome to the MySQL monitor.  Commands end with ; or \g.<br/>
<br/>
mysql> show databases;<br/><br/>
+--------------------+<br/>
| Database           |<br/>
+--------------------+<br/>
| db                 |<br/>
| information_schema |<br/>
| mysql              |<br/>
| performance_schema |<br/>
| sys                |<br/>
+--------------------+<br/>
5 rows in set (0.00 sec)<br/>
<br/><br/>
mysql> use db;<br/>
Database changed<br/>
<br/><br/>
mysql> Create table person (
     Id INT auto_increment primary key,
     Name varchar(255) not null,
     Age int
     );<br/>
Query OK, 0 rows affected (0.04 sec)
<br/><br/>
mysql> show tables ;<br/>
+--------------+<br/>
| Tables_in_db |<br/>
+--------------+<br/>
| person       |<br/>
+--------------+<br/>
1 row in set (0.01 sec)<br/>
<br/>

<br/>
mysql> INSERT INTO person (name, age) VALUES ('John Doe', 30),
    ('Jane Smith', 25),
    ('Bob Johnson', 40),
    ('Alice Brown', 35),
('Eva Davis', 28);<br/>
Query OK, 5 rows affected (0.01 sec)<br/>
Records: 5  Duplicates: 0  Warnings: 0
<br/><br/>
mysql> select * from person;<br/>
+----+-------------+------+<br/>
| Id | Name        | Age  |<br/>
+----+-------------+------+<br/>
|  1 | John Doe    |   30 |<br/>
|  2 | Jane Smith  |   25 |<br/>
|  3 | Bob Johnson |   40 |<br/>
|  4 | Alice Brown |   35 |<br/>
|  5 | Eva Davis   |   28 |<br/>
+----+-------------+------+<br/>
5 rows in set (0.00 sec)
<br/><br/>
mysql> exit;
Bye<br/>
bash-4.4# exit<br/>
exit
<br/><br/>
ubuntu@ip-172-31-87-243:~$ ls<br/>
docker-compose.yml  mydata<br/>
ubuntu@ip-172-31-87-243:~$ cd mydata<br/>
ubuntu@ip-172-31-87-243:~/mydata$ ls<br/><br/>
'#ib_16384_0.dblwr'  '#innodb_temp'   binlog.000002   ca.pem            db               ibtmp1      mysql.sock           public_key.pem    sys<br/>
'#ib_16384_1.dblwr'   auto.cnf        binlog.index    client-cert.pem   ib_buffer_pool   mysql       performance_schema   server-cert.pem   undo_001<br/>
'#innodb_redo'        binlog.000001   ca-key.pem      client-key.pem    ibdata1          mysql.ibd   private_key.pem      server-key.pem    undo_002<br/>



<br/><br/>

/////////////////////////////////////////
 In New EC2
 <br/>
  sudo apt-get update
 <br/>
 
  # Add Docker's official GPG key:<br/>
sudo apt-get update<br/>
sudo apt-get install ca-certificates curl gnupg<br/>
sudo install -m 0755 -d /etc/apt/keyrings<br/>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg<br/>
sudo chmod a+r /etc/apt/keyrings/docker.gpg<br/>

# Add the repository to Apt sources:
echo \<br/>
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \<br/>
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \<br/>
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null<br/>
sudo apt-get update<br/>
<br/>
Install the latest Docker packages.<br/>
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin<br/>
<br/><br/>
Verify that the Docker Engine installation is successful by running the hello-world image.<br/>
sudo docker run hello-world<br/>
 
 Then attach the 20GB EBS volume to the New EC2 instance<br/>
 
 ubuntu@ip-172-31-85-206:~$ lsblk<br/><br/>
 
NAME     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS<br/>
loop0      7:0    0  24.4M  1 loop /snap/amazon-ssm-agent/6312<br/>
loop1      7:1    0  55.6M  1 loop /snap/core18/2745<br/>
loop2      7:2    0  63.3M  1 loop /snap/core20/1879<br/>
loop3      7:3    0 111.9M  1 loop /snap/lxd/24322<br/>
loop4      7:4    0  53.2M  1 loop /snap/snapd/19122<br/>
xvda     202:0    0     8G  0 disk<br/>
├─xvda1  202:1    0   7.9G  0 part /<br/>
├─xvda14 202:14   0     4M  0 part<br/>
└─xvda15 202:15   0   106M  0 part /boot/efi<br/>
xvdf     202:80   0    20G  0 disk<br/>
<br/>
ubuntu@ip-172-31-85-206:~$ sudo df -h<br/>

Filesystem      Size  Used Avail Use% Mounted on<br/>
/dev/root       7.6G  2.3G  5.3G  31% /<br/>
tmpfs           483M     0  483M   0% /dev/shm<br/>
tmpfs           194M  864K  193M   1% /run<br/>
tmpfs           5.0M     0  5.0M   0% /run/lock<br/>
/dev/xvda15     105M  6.1M   99M   6% /boot/efi<br/>
tmpfs            97M  4.0K   97M   1% /run/user/1000<br/>
<br/><br/>

ubuntu@ip-172-31-85-206:~$ mkdir another_folder<br/>

ubuntu@ip-172-31-85-206:~$ ls<br/>
another_folder<br/>
<br/>
ubuntu@ip-172-31-85-206:~$ sudo mount /dev/xvdf  /home/ubuntu/another_folder/<br/>

<br/>

ubuntu@ip-172-31-85-206:~$ lsblk<br/>
NAME     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS<br/>
loop0      7:0    0  24.4M  1 loop /snap/amazon-ssm-agent/6312<br/>
loop1      7:1    0  55.6M  1 loop /snap/core18/2745<br/>
loop2      7:2    0  63.3M  1 loop /snap/core20/1879<br/>
loop3      7:3    0 111.9M  1 loop /snap/lxd/24322<br/>
loop4      7:4    0  53.2M  1 loop /snap/snapd/19122<br/>
xvda     202:0    0     8G  0 disk<br/>
├─xvda1  202:1    0   7.9G  0 part /<br/>
├─xvda14 202:14   0     4M  0 part<br/>
└─xvda15 202:15   0   106M  0 part /boot/efi<br/>
xvdf     202:80   0    20G  0 disk /home/ubuntu/another_folder<br/>

<br/><br/>

ubuntu@ip-172-31-85-206:~$ sudo df -h<br/>
Filesystem      Size  Used Avail Use% Mounted on<br/>
/dev/root       7.6G  2.3G  5.3G  31% /<br/>
tmpfs           483M     0  483M   0% /dev/shm<br/>
tmpfs           194M  864K  193M   1% /run<br/>
tmpfs           5.0M     0  5.0M   0% /run/lock<br/>
/dev/xvda15     105M  6.1M   99M   6% /boot/efi<br/>
tmpfs            97M  4.0K   97M   1% /run/user/1000<br/>
/dev/xvdf        20G  363M   20G   2% /home/ubuntu/another_folder<br/>

<br/><br/>
ubuntu@ip-172-31-85-206:~$ cd another_folder<br/>
ubuntu@ip-172-31-85-206:~/another_folder$ ls<br/><br/>
'#ib_16384_0.dblwr'   auto.cnf        ca-key.pem        db               mysql.ibd            public_key.pem    undo_001<br/>
'#ib_16384_1.dblwr'   binlog.000001   ca.pem            ib_buffer_pool   mysql.sock           server-cert.pem   undo_002<br/>
'#innodb_redo'        binlog.000002   client-cert.pem   ibdata1          performance_schema   server-key.pem<br/>
'#innodb_temp'        binlog.index    client-key.pem    mysql            private_key.pem      sys<br/>
<br/><br/>
ubuntu@ip-172-31-85-206:~$ cd ..<br/>
ubuntu@ip-172-31-85-206:~$ vim docker-compose.yml<br/>
<br/>
version: "3"<br/>
<br/>
services:<br/>
  mysql:<br/>
    image: mysql:latest<br/>
    container_name: db<br/>
    environment:<br/>
      MYSQL_ROOT_PASSWORD: a<br/>
      MYSQL_DATABASE: db<br/>
      MYSQL_USER: a<br/>
      MYSQL_PASSWORD: a<br/>
    volumes:<br/>
      - ./another_folder:/var/lib/mysql<br/>
    ports:<br/>
      - "3306:3306"<br/>
    restart: always<br/>
ubuntu@ip-172-31-85-206:~$ sudo docker compose up<br/>
<br/>
ubuntu@ip-172-31-85-206:~$ sudo docker exec -it db bash<br/>
bash-4.4# mysql -u root -p<br/>
Enter password:<br/>
Welcome to the MySQL monitor.  Commands end with ; or \g.<br/>
<br/>
mysql> show databases;<br/>
+--------------------+<br/>
| Database           |<br/>
+--------------------+<br/>
| db                 |<br/>
| information_schema |<br/>
| mysql              |<br/>
| performance_schema |<br/>
| sys                |<br/>
+--------------------+<br/>
5 rows in set (0.03 sec)<br/>
<br/>
mysql> use db;<br/>
Reading table information for completion of table and column names<br/>
You can turn off this feature to get a quicker startup with -A<br/>
<br/>
Database changed<br/>
mysql> show tables;<br/>
+--------------+<br/>
| Tables_in_db |<br/>
+--------------+<br/>
| person       |<br/>
+--------------+<br/>
1 row in set (0.01 sec)<br/>
<br/>
mysql> select * from person;<br/>
+----+-------------+------+<br/>
| Id | Name        | Age  |<br/>
+----+-------------+------+<br/>
|  1 | John Doe    |   30 |<br/>
|  2 | Jane Smith  |   25 |<br/>
|  3 | Bob Johnson |   40 |<br/>
|  4 | Alice Brown |   35 |<br/>
|  5 | Eva Davis   |   28 |<br/>
+----+-------------+------+<br/>
5 rows in set (0.00 sec)
