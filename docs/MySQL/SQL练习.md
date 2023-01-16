### 事务隔离级别（4种）

|                                       | 脏读  | 不可重复读 | 幻读  |
|---------------------------------------|-----|-------|-----|
| Read uncommitted                      | ✔   | ✔     | ✔   |
| Read committed _(Oracle, Sql Server)_ | ✖   | ✔     | ✔   |
| Repeatable read _(MySQL)_             | ✖   | ✖     | ✔   |
| Serializable                          | ✖   | ✖     | ✖   |

### 事务传播行为（7种）

* PROPAGATION_REQUIRED：如果当前存在事务，就加入该事务；如果当前没有事务，就创建一个新事务。**【常用】**
* PROPAGATION_SUPPORTS：如果当前存在事务，就加入该事务；如果当前不存在事务，就以非事务执行。
* PROPAGATION_MANDATORY：如果当前存在事务，就加入该事务；如果当前不存在事务，就抛出异常。
* PROPAGATION_REQUIRES_NEW：无论当前存不存在事务，都创建新事务。
* PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
* PROPAGATION_NEVER：以非事务方式执行操作，如果当前存在事务，则抛出异常。
* PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。

### Access数据库

Access数据库包含有七个数据库操作对象，它们分别是：表、查询、窗体、报表、页、宏、模块

### 名词解释

事务：事务是一个不可分割的操作序列，是数据库环境中的逻辑工作单位。

数据字典：数据字典是对系统中数据的详细描述，它提供对数据库数据描述的集中管理。

数据库系统：数据库系统是实现有组织地、动态地存储大量关联数据、方便多用户访问的计算机软件、硬件和数据资源组成的系统。

### SQL语言具有什么功能？

数据定义、数据操纵、数据控制和SQL语句嵌入。

***

### 题目

- ***S(S#, SN, SEX, AGE, DEPT)***
- ***C(C#, CN)***
- ***SC(S#, C#, GRADE)***

① 查询所有姓王学生的姓名和性别

② 统计学生选课数据库中开出的课程总数

③ 查询每个学生选修每门课程的有关课程数据（姓名、课程名和成绩等）

④ 从学生选课库中查询出被3名以上（不含3名）学生选修的所有课程信息

⑤ 从学生选课库中查询最多选修了1门课（含未选任何课程）的全部学生信息

⑥ 查询所有与“张建”同年出生的学生姓名、年龄和性别（假设只有一个“张建”）

⑦ 从学生选课库中查询出每门课程被选修的学生人数，并按所选人数的降序排列出课程号和选课人数

⑧ 统计学生选课数据库中学生的总人数

⑨ 查询学生姓名及其所选课程的课程号和成绩

⑩ 从学生选课库中查询出被2至4名学生选修的所有课程信息

⑩① 从学生选课库中查询出选修至少两门课程的学生序号

***

- ***学生（学号，姓名，性别，专业，奖学金）***
- ***课程（课程号，名称，学分）***
- ***学习（学号，课程号，分数）***

① 检索不学课程号为“C135”课程的学生信息，包括学号、姓名和专业

② 检索至少学过课程号为“C135”和“C219”的学生信息，包括学号、姓名和专业

③ 从学生表中删除成绩出现过0分的所有学生信息

④ 定义“英语”专业学生所学课程的信息视图AAA，包括学号、姓名、课程号和分数

⑤ 检索没有获得奖学金、同时至少有一门课程成绩在95分以上的学生信息，包括学号、姓名和专业

⑥ 检索没有任何一门课程成绩在80分以下的所有学生的信息，包括学号、姓名和专业

⑦ 对成绩得过满分（100分）的学生，如果没有获得奖学金的，将其奖学金设为1000元

⑧ 定义学生成绩得过满分（100分）的课程视图AAA，包括课程号、名称和学分

***

### 答案

1.1(不同)

```sql
SELECT SN, SEX
FROM S
WHERE SN LIKE '王%';
SELECT SN, SEX
FROM S
WHERE SN LIKE '王*';
```

1.2(相同)

```sql
SELECT COUNT(*)
FROM C;
SELECT COUNT(*) AS 课程总数
FROM C;
```

1.3(相同)

```sql
SELECT s.SN, c.CN, sc.GRADE
FROM S s,
     C c,
     SC sc
WHERE s.S# = sc.S#
  AND c.C# = sc.C#;
SELECT s.SN, c.CN, sc.GRADE
FROM S s,
     C c,
     SC sc
WHERE s.S# = sc.S#
  AND c.C# = sc.C#;
```

1.4(不同)

```sql
SELECT c.*
FROM C c,
     SC sc
WHERE c.C# = sc.C#
GROUP BY sc.C#
HAVING COUNT(sc.S#) > 3;
SELECT c.*
FROM C c
WHERE EXISTS(
              SELECT sc.C# FROM SC sc WHERE c.C# = sc.C# GROUP BY sc.C# HAVING COUNT(*) > 3
          );
```

1.5(不同)

```sql
SELECT s.*
FROM S s,
     SC sc
WHERE s.S# = sc.S#
GROUP BY sc.S#
HAVING COUNT(sc.C#) <= 1;
SELECT s.*
FROM S s
WHERE s.S# IN (SELECT S#
               FROM SC
               GROUP BY S#
               HAVING COUNT(*) = 1)
   OR NOT EXISTS(
        SELECT sc.* FROM SC sc WHERE s.S# = sc.S#
    );
```

1.6(不同)

```sql
SELECT s.SN, s.AGE, s.SEX
FROM S s
WHERE s.AGE IN (SELECT ss.AGE
                FROM S ss
                WHERE ss.SN = '张建');
SELECT s.SN, s.AGE, s.SEX
FROM S s
WHERE s.AGE = (SELECT ss.AGE
               FROM S ss
               WHERE ss.SN = '张建');
```

1.7(不同)

```sql
SELECT C#, COUNT(S#) AS SCOUNT
FROM SC
GROUP BY C#
ORDER BY SCOUNT DESC;
SELECT c.C#, COUNT(c.C#) AS 人数
FROM C c,
     SC sc
WHERE c.C# = sc.C#
GROUP BY c.C#
ORDER BY 人数 DESC;
```

1.8(相同)

```sql
SELECT COUNT(*)
FROM S;
SELECT COUNT(*) AS 学生总人数
FROM S;
```

1.9(相同)

```sql
SELECT s.SN, sc.C#, sc.GRADE
FROM S s,
     SC sc
WHERE s.S# = sc.S#;
SELECT s.SN, sc.C#, sc.GRADE
FROM S s,
     SC sc
WHERE s.S# = sc.S#;
```

1.10(不同)

```sql
SELECT c.*
FROM C c,
     SC sc
WHERE c.C# = sc.C#
GROUP BY sc.C#
HAVING COUNT(sc.S#) BETWEEN 2 AND 4;
SELECT c.*
FROM C c
WHERE EXISTS(
              SELECT sc.C# FROM SC sc WHERE c.C# = sc.C# GROUP BY sc.C# HAVING COUNT(*) BETWEEN 2 AND 4
          );
```

1.11(不同)

```sql
SELECT S#
FROM SC
GROUP BY S#
HAVING COUNT(C#) >= 2;
SELECT DISTINCT c1.S#
FROM SC c1,
     SC c2
WHERE c1.S# = c2.S#
  AND c1.C# <> c2.C#;
```

***

2.1(相同)

```sql
SELECT s.学号, s.姓名, s.专业
FROM 学生 s
WHERE s.学号 NOT IN (SELECT sc.学号
                     FROM 学习 sc
                     WHERE sc.课程号 = 'C135');
SELECT s.学号, s.姓名, s.专业
FROM 学生 s
WHERE s.学号 NOT IN (SELECT sc.学号
                     FROM 学习 sc
                     WHERE sc.课程号 = 'C135');
```

2.2(不同)

```sql
SELECT s.学号, s.姓名, s.专业
FROM 学生 s
WHERE s.学号 IN (SELECT sc.学号
                 FROM 学习 sc
                 WHERE sc.课程号 = 'C135'
                    OR sc.课程号 = 'C219');
SELECT 学号, 姓名, 专业
FROM 学生
WHERE 学号 in (SELECT X.学号
               FROM 学习 AS X,
                    学习 AS Y
               WHERE X.学号 = Y.学号
                 AND X.课程号 = 'C135'
                 AND X.课程号 = 'C219');
```

2.3(相同)

```sql
DELETE
FROM 学生 s
WHERE s.学号 IN (SELECT sc.学号
                 FROM 学习 sc
                 WHERE sc.分数 = 0);
DELETE
FROM 学生 s
WHERE s.学号 IN (SELECT sc.学号
                 FROM 学习 sc
                 WHERE sc.分数 = 0);
```

2.4(相同)

```sql
CREATE VIEW AAA(学号, 姓名, 课程号, 分数) AS
SELECT s.学号, s.姓名, sc.课程号, sc.分数
FROM 学生 s,
     学习 sc
WHERE s.专业 = '英语'
  AND s.学号 = sc.学号
WITH CHECK OPTION;
CREATE VIEW AAA(学号, 姓名, 课程号, 分数) AS
SELECT s.学号, s.姓名, sc.课程号, sc.分数
FROM 学生 s,
     学习 sc
WHERE s.专业 = '英语'
  AND s.学号 = sc.学号
WITH CHECK OPTION;
```

2.5(不同)

```sql
SELECT s.学号, s.姓名, s.专业
FROM 学生 s
WHERE s.奖学金 = '0元'
  AND s.学号 NOT IN (SELECT sc.学号
                     FROM 学习 sc
                     GROUP BY sc.学号
                     HAVING MAX(sc.分数) < 95);
SELECT s.学号, s.姓名, s.专业
FROM 学生 s,
     学习 sc
WHERE s.学号 = sc.学号
  AND sc.课程号 = 课程.课程号
  AND 奖学金 <= 0
  AND 分数 > 95;
```

2.6(相同)

```sql
SELECT s.学号, s.姓名, s.专业
FROM 学生 s
WHERE s.学号 IN (SELECT sc.学号
                 FROM 学习 sc
                 GROUP BY sc.学号
                 HAVING MIN(sc.分数) >= 80);
SELECT s.学号, s.姓名, s.专业
FROM 学生 s
WHERE s.学号 NOT IN (SELECT sc.学号
                     FROM 学习 sc
                     WHERE 分数 < 80);
```

2.7(不同)

```sql
UPDATE 学生 s
SET s.奖学金 = '1000元'
WHERE s.奖学金 = '0元'
  AND s.学号 IN (SELECT sc.学号
                 FROM 学习 sc
                 GROUP BY sc.学号
                 HAVING MAX(分数) = 100);
UPDATE 学生 s
SET s.奖学金 = 1000
WHERE s.奖学金 <= 0
  AND s.学号 IN (SELECT sc.学号
                 FROM 学习 sc
                 WHERE sc.分数 = 100);
```

2.8(不同)

```sql
CREATE VIEW AAA(课程号, 名称, 学分) AS
SELECT c.课程号, c.名称, c.学分
FROM 课程 c,
     学习 sc
WHERE c.课程号 = sc.课程号
GROUP BY sc.学号
HAVING MAX(分数) = 100
WITH CHECK OPTION;
CREATE VIEW AAA(课程号, 名称, 学分) AS
SELECT c.课程号, c.名称, c.学分
FROM 课程 c
WHERE c.课程号 IN (SELECT sc.课程号
                   FROM 学习 sc
                   WHERE sc.分数 = 100)
WITH CHECK OPTION;
```