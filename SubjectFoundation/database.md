# 第一章 绪论

- 数据模型

  - 概念模型
    人们把现实世界抽象为不依赖于计算机和DBMS的概念模型，用实体，属性，联系，表示出来。

  - 基本概念

    - 实体
      指客观存在可以相互区别的事物。如具体的人，事，物，一位老师，一个学生。也可以是抽象的概念或联系，如老师的授课，学生的选课，称为联系型实体。

    - 属性
      属性是指事物具有的某一特性。例如：教师实体可以由教师号，姓名，年龄组成。

    - 码
      码是指唯一标识实体的属性和属性集，例如教师号在教师中就是码。

    - 域
      域是指属性的取值范围，是具有相同数据类型的数据集合。如年龄为正整数，教师号为6位数字编号。

    - 实体型
      具有相同属性的实体必然具有共同的特征和性质。用实体名和属性名集合组成的形式，称为实体型。例如 ：教师（教师号，姓名，年龄）

    - 实体集
      实体集是指同型实体的集合，实体集用实体型来定义，每个实体透视实体型的实例或值。如：（‘123456’，tom，42）

    - 联系
      在现实世界事物之间是有联系的。在信息世界，联系指实体与实体之间，实体集内实体与实体之间以及实体的各属性之间的关系。

  - 表示方法
    矩形表示实体，椭圆表示属性，菱形表示联系。

  - ER模型的转换
    1：垂直分割，垂直合并：拆分属性，如把属性分为不变属性，变动信息。
    2：水平分割，水平合并：按类细分，如学生分为大学生，小学生。

- 数据库系统结构

  - 数据库管理系统的三级模式结构

    - 外模式

      又称子模式或用户模式，用户级模式。

      - 特点
        1是各个具体用户看到的数据视图，是数据库用户可以看见的和使用的局部数据的逻辑结构和特征。是用户与DB的接口。
        2可以有多个外模式。
        3不同用户看到的外模式不同，无法看到数据库其余数据，是保护数据库安全性的有力措施。
        4面向最终用户和应用程序​

    - 模式

      又称逻辑模式，还称概念模式。

      - 特点
        1是数据库中全体数据的逻辑结构和特征的描述。它包括数据的逻辑结构，数据之间的联系和数据的完整性要求。
        2是所有用户的公共数据视图，一般只有DBA可以看到全部。
        3只有一个模式
        4由DBA定义与管理。​

    - 内模式

      也称存储模式，还称物理级模式，

      - 特点
        1是数据库的物理结构和存储方式
        2只有一个内模式
        3由DBA定义，现基本由DBMS定义。​

  - 数据库的二级映像功能与数据独立性

    为了实现这三个抽象层次之间的转换。

    - 外模式/模式映像
      外模式描述的是用户可以看到的局部数据的逻辑结构和特征，模式描述的是数据库中全体数据的逻辑结构和特征。一个模式可以对应多个外模式，每个外模式数据库都有一个外模式/模式映像。定义外模式与模式之间对应关系，通常定义包含在外模式的描述中。
      若数据库变动，DBA只需要变动映像，保持外模式不变。由于程序是基于外模式的，故不用变动，称为数据逻辑独立性。

    - 模式/内模式映像
      内模式描述了数据库的物理结构和存储方式，而且数据库只有一个内模式，和一个模式，所以模式/内模式映像是唯一的。数据库存储结构变动时，只用改变映像，模式和外模式不变，程序不变，称为数据物理独立性。

# 第二章 关系数据库

- 关系代数

  关系代数是以一个或两个关系作为输入，产生一个新的关系作为操作结果，即其运算对象是关系，结果也是关系。
  关系代数的运算符包括4类：集合运算符，专门的关系运算符，比较运算符和逻辑运算符（与或非）。
  比较运算符和逻辑运算符是用来辅助专门的关系运算的。
  ​传统的集合运算主要从行的角度进行，专门的关系运算有行有列。

  - 传统的集合运算
    并，交，差，广义笛卡尔积

  - 专门的关系运算

    选择，投影，连接，除。

    - 选择
      又称为限制，它是在关系R中选择满足条件的诸元组，记作б。
      例子：选择系别为CS的学生：бDEPT=‘CS'(S)

    - 投影
      关系R上的投影是选择若干属性列组成的新关系。
      例子：查询学生关系中都有哪些系别：∏DEPT（S）

    - 连接
      它是从两个关系的笛卡尔积中选择满足一定关系的元组。
      1等值连接：从R，S的笛卡尔积中选出A，B属性相等的列。
      2自然连接：R，S包含相同的属性列B，则按照B连接。​
      例子：R∞S=бR.B=S.B(R×S）

    - 除
      当R和S具有域相同的列B,C时，相除即为找到A为什么值的时候，他的象集包含S在B,C上的投影。
      R/S

- 关系演算

  - 元组关系演算语言ALPHA

    ALPHA主要有GET,PUT,HOLD,UPDATE,DELETE,DROP6条语句。
    基本格式：操作语句 工作空间（表达式) ：操作条件

    - 检索操作
      GET W(S.SNO,S.AGE):S.DEPT='IS'  DOWN S.AGE
      GET w(1)([S.SN](http://s.sn/)):S.DEPT='IS'  1表示检索出的元组个数。
      元组变量检索：1简化名字2有量词必须用
      RANGE Student X
      GET W([X.SN](http://x.sn/)):X.DEPT='IS'
      RANGE SC X
      GET W([S.SN](http://s.sn/)):存在X(X.SNO=S.SNO)​​​

    - 更新操作

      - 修改操作
        HOLD W(S.SNO,S.DEPT):S.SNO='444'
        MOVE 'IS' TO W.DEPT
        UPDATE W

      - 插入操作
        MOVE 'C8' TO W.CNO
        MOVE 'C4' TO W.Cpno
        PUT W(C)​

      - 删除操作
        HOLD W(S):S.SNO='22'
        DELETE W

# 第三章  关系数据库标准语言SQL

- SQL语言定义

  - 字段数据类型
    整数：Bigint,int,smallint,tinyint
    精确数值类型：Numeric，Decimal
    近似浮点数值：Float，Real
    日期类型：Datetime，SmallDateTime
    非Unicode的字符型数据：C哈日，Varchar，Text​​
    二进制：Binary，Varbinary，Image

  - 创建，修改和删除数据表

    - 定义基本表
      CREATE TABLE STUDENT(
          SNO CHAR(5),
      SN VARCHAR(8) NOT NULL,
      AGE INT NOT NULL CHECK(AGE>0)
      SEX CHAR(2) NOT NULL CHECK(SEX IN('男','女'));
      CONSTRAINT SN_U UNIQUE(SN),
      CONSTRAINT C_F FORIEGN KEY(CNO) REFERENCES C(CON)
      ​)

    - 修改基本表
      ALTER TABLE S ADD SCOME DATETIME
      ALTER TABLE S ALTER COLUMN AGE SMALLINT
      ALTER TABLE S DROP COLUMN SCOME
      ALTER TABLE SC NOCHECK CONSTAINT C_F​

    - 删除基本表
      DROP TABLE S

    - 设计，创建和维护索引

      索引是为了加快检索的一种存储结构（如b+树）
      聚集索引与物理地址顺序相同，检索块，一个表只有一个。

      - 创建索引
        CREATE UNIQUE INDEX S_SNO ON S(SNO)

      - 删除索引
        DROP INDEX S.S_SNO

- SQL数据查询

  - select子句基本使用
    SELECT  * FROM S

  - where子句基本使用
    SELECT DISTINCT SNO FROM SC WHERE DEPT='IS' AND  AGE <20
    1比较运算符=
    2确定范围：between and   not between  and
    3确定集合：in   not  in
    4字符匹配  like
    通配符：%任意字符串  _任何单个字符
    [a-z]   [^a-z]是反义​​​

  - 常用集函数
    COUNT,SUM,AVG,MAX,MIN

  - 分组查询
    GROUP BY
    SELECT CNO,COUNT(SNO) FROM SC GROUP BY CNO

  - 查询的排序
    SELECT S.SNO FROM S ORDER BY SCORE DESC
    DESC,ASC

  - 连接查询
    1等值连接和非等值连接
    2自然连接
    3广义笛卡尔积​
    外连接：left join ，right join，inner  join

  - 合并查询
    得到多个结果，合并多个元组结果。如student1和student2合并。

  - 嵌套查询
    使用括号嵌套。

  - 子查询别名表达式
    使用as
    SELECT  AVG(SCORE) FROM SC AS T1(SNO)

- SQL数据更新

  - 插入数据
    INSERT INTO SC(SNO,CNO)  VALUES('S7','C1');

  - 修改数据
    UPDATE S SET AGE=22 WHERE SNO='S3';

  - 删除数据
    DELETE FROM S WHERE SNO='S7';

# 第四章 关系数据库设计理论

- 范式
  1第一范式：每个属性都不可再分。
  2第二范式：所有非主属性都完全依赖于任意一个候选关键字。
  3第三范式：所有非主属性对任何候选关键字都不存在传递函数依赖。
  4BC范式：所有的函数依赖都含有码​

- 规范化

  - 函数依赖

    对于集合X,Y，对于X的每一个具体值，Y都有唯一的值对应，则X决定Y，Y函数依赖于X。

    - 平凡的函数依赖与非平凡的函数依赖
      当Y是X的子集时，必然存在Y函数依赖X，这种称为平凡。

    - 函数依赖与属性间的联系类型有关
      1若1：1相互依赖
      2m:1,则y依赖x。
      3m:n,无任何函数依赖。​

    - 函数依赖可以保证关系分解的无损连接性。
      R(X,Y,Z),Y,Z函数依赖于X，可分解R（X,Y),R(X,Z),再用自然连接合并可以得原式。

  - 函数依赖的基本性质
    1投影性：投影产生平凡依赖
    2扩张性：X→Y，W→Z，则(X,W)→(Y,Z)
    3合并性:X→Y，X→Z，则X→(Y,Z)
    4分解性:3反过来

  - 完全/部分函数依赖和传递/非传递函数依赖
    若Y函数依赖于X，对于任何X的子集，Y都不依赖，则为完全依赖。
    Y依赖X，X不依赖Y，Z依赖Y，则Z依赖X，称为传递依赖。

- 码

  若U完全依赖K，则K为R的候选码。
  包含在候选码中的属性为主属性。

  - 求候选码（是否依赖他人，关系全集是否依赖他）
    1定义法
    ​2规范求解法：1找到谁也不依赖的属性集合P
    2p是否空，空则看看联合键可以吗。
    若不空再试试集合中元素是否满足定义​，试试联合键。

# 第六章 数据库设计

- er转换为关系模式

  关系名（属性1,属性2......)

  - 基本原则(主键=实体键或联系中各实体组合键，属性为实体属性或联系的属性)
    1.一个实体转换为一个关系模式
    2.一个m:n联系转换为一个关系模式
    3.一个1：n联系转换为一个关系模式
    4.一个1：1联系可以转换为一个独立的关系模式
    5.3个以上多元联系转换为一个关系模式​​

### mysql例

CREATE TABLE `category_tbl`

(

​    `category_id` INT UNSIGNED AUTO_INCREMENT,

​    `category` VARCHAR(20) NOT NULL,

​    `value`  INT NOT NULL,

​    `percent` VARCHAR(20) NOT NULL,

​    `time` DATE NOT NULL,

​    PRIMARY KEY(`category_id`)

)ENGINE=InnoDB DEFAULT CHARSET=utf8;

# 数据库引擎

