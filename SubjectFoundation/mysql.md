### 数据表

##### 创建数据库

CREATE DATABASE `cloud_db_one` CHARACTER SET utf8 COLLATE utf8_bin;

##### 创建数据表

CREATE TABLE `dept`(
`dept_no` int(11) NOT NULL AUTO_INCREMENT,
`dept_name` varchar(500) DEFAULT NULL,
`db_source` varchar(500) DEFAULT NULL,
PRIMARY KEY(`dept_no`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;