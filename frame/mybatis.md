# mybatis xml头约束

### 配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
2 <!DOCTYPE configuration
3         PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
4         "http://mybatis.org/dtd/mybatis-3-config.dtd">
```
### mapper文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
</mapper>
```

```
Dept findById(Integer deptNo);
List<Dept> findAll();
boolean addDept(Dept dept);
```

# mybatis语句语法

```
INSERT INTO dept(dept_name,db_source) VALUES(#{deptName},DATABASE())
```