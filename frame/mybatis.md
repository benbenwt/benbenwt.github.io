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

<mapper namespace="com.weitao.provider.mapper.DeptMapper">
    <select id="findById" resultType="Dept" parameterType="Integer">
        select dept_no deptNo,dept_name deptName,db_source dbSource from dept where dept_no=#{deptNo}
    </select>

    <select id="findAll" resultType="Dept">
         select dept_no deptNo,dept_name deptName,db_source dbSource from dept
    </select>

    <insert id="addDept" parameterType="Dept">
        INSERT INTO dept(dept_name,db_source) VALUES(#{deptName},DATABASE())
    </insert>
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