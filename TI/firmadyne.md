# AttifyOS

### firmadyne

##### problem



### fat

https://blog.csdn.net/song_lee/article/details/104393933
https://github.com/liyansong2018/Old-Firmware-Analysis-Toolkit
18.04
错误：pexpect.exceptions.EOF: End Of File (EOF). Exception style platform.
command: /home/cuckoo/download/Old-Firmware-Analysis-Toolkit-master/firmadyne/scripts/getArch.shargs: ['/home/cuckoo/download/Old-Firmware-Analysis-Toolkit-master/firmadyne/scripts/[getArch.sh](http://getarch.sh/)', './images/1.tar.gz']buffer (last 100 chars): ''before (last 100 chars): ''after: match: Nonematch_index: Noneexitstatus: Noneflag_eof: Truepid: 9973child_fd: 5closed: Falsetimeout: 30delimiter: logfile: Nonelogfile_read: Nonelogfile_send: Nonemaxread: 2000ignorecase: Falsesearchwindowsize: Nonedelaybeforesend: 0.05delayafterclose: 0.1delayafterterminate: 0.1searcher: searcher_re:    0: re.compile('Password for user firmadyne: ')

##### 键值重复

删除该表所有data

##### 连接失败

pg_hba.conf全设trust

### lob/docker

https://github.com/firmadyne/firmadyne
image那一步，说无法kernel通信。类似这个：https://github.com/firmadyne/firmadyne/issues/21
错误：
/dev/mapper/control: open failed: Operation not permitted
Failure to communicate with kernel device-mapper driver.
Check that device-mapper is available in the kernel.
device mapper prerequisites not met
​

##### temp

/home/cuckoo/[WNAP320.zip](http://wnap320.zip/)
几个组件，extrctor，binwalk，postgredql，qemu。

### postgresql

\l  显示database
\d 显示表格关系
\d tabelname ,show table info
\c databasename 进入数据库
sudo -u postgres psql 进入psql
\q ,exit
service postgresql start
​
分号必须隔一个空格。

##### 增删改查

sudo insert  tablename columns()  values();
delete from tablename where a=b;
UPDATE table_name
SET column1 = value1, column2 = value2...., columnN = valueN
WHERE [condition];

select * from tablename;



