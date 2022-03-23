转载自：

[经典SQL练习题(MySQL版)_廖致君的博客-CSDN博客_mysql练习](https://blog.csdn.net/paul0127/article/details/82529216)



## 数据表结构

**学生表 Student**

```sql
create table Student(Sid varchar(6), Sname varchar(10), Sage datetime, Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
```

**成绩表 SC**

```sql
create table SC(Sid varchar(10), Cid varchar(10), score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

**课程表 Course**

```sql
create table Course(Cid varchar(10),Cname varchar(10),Tid varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```

**教师表 Teacher**

```sql
create table Teacher(Tid varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```



## 题目

### 排行榜设计

score表：自增主键、用户id、用户分区、用户分数

**根据分区查整体排名**

```mysql
select s.user_id, @curRank := @curRank + 1 as rank_no
from score s,
     (select @curRank := 0) r
where s.region_id = 1
order by s.score desc
limit 100
```

**根据排名查人**

```mysql
select t.*
from (select s.user_id, @curRank := @curRank + 1 as rank_no
      from (select * from score order by score desc) s,
           (select @curRank := 0) r
      where s.region_id = 1) as t
where t.rank_no=1
```





### 关注设计

自增主键、用户 id、用户关注的人 id

**查询用户关注的人**

```mysql
select followed_id
from follow
where me_id = 1
```

**查询用户的粉丝**

```mysql
select me_id
from follow
where followed_id = 3
```

**查询互关的人**

```mysql
select a.followed_id
from follow as a
         inner join follow as b
                    on a.followed_id = b.me_id and b.followed_id = a.me_id
where a.me_id = 2
```

**查询用户粉丝数和关注数**

```mysql
SELECT (SELECT COUNT(followed_id) as count FROM follow WHERE followed_id = 1) as Followers,
       (SELECT COUNT(me_id) as count FROM follow WHERE me_id = 1)             as Following
```



### 行转列

子查询

```mysql
select user,
       year,
       IFNULL((select amount from test t where quarter = 1 and t.year = test.year), 0)            as '季度1',
       IFNULL((select ifnull(amount, 0) from test t where quarter = 2 and t.year = test.year), 0) as '季度2',
       IFNULL((select ifnull(amount, 0) from test t where quarter = 3 and t.year = test.year), 0) as '季度3',
       IFNULL((select ifnull(amount, 0) from test t where quarter = 4 and t.year = test.year), 0) as '季度4'
from test
group by user, year
```

sum case when

```mysql
select user,
       year,
       sum(CASE quarter when 1 then amount else 0 end)  '季度1',
       sum(CASE quarter when 2 then amount else 0 end)  '季度2',
       sum(CASE quarter when 3 then amount else 0 end)  '季度3',
       sum(CASE quarter when 4 then amount else 0 end)  '季度4'
from test
group by user, year
```



### 查询 "01" 课程比 "02" 课程成绩高的学生的信息及课程分数

首先从 student 表选择所有学生信息， sc 表分别选择两个课程的成绩。

where 语句控制 01 课程比 02 课程成绩高，同时将 select 回来的三个结果连在一起

```sql
select s.*, sc_01.score as score_01, sc_01.score as score_02
from student s,
     (select Sid, score from sc where Cid = 01) sc_01,
     (select Sid, score from sc where Cid = 02) sc_02
where sc_01.Sid = sc_02.Sid
  and sc_01.score > sc_02.score
  and s.Sid = sc_01.Sid;
```



```sql
+------+--------+---------------------+------+----------+----------+
| Sid  | Sname  | Sage                | Ssex | score_01 | score_02 |
+------+--------+---------------------+------+----------+----------+
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |     70.0 |     60.0 |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |     50.0 |     30.0 |
+------+--------+---------------------+------+----------+----------+
2 rows in set (0.00 sec)
```



### 查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

从 student 表和 sc 表选择信息。

按学生编号分组后，求出 score 的平均值，并筛选。

```sql
select s.sid, sname, avg(score) as avg_score
from student as s,
     sc
where s.sid = sc.sid
group by s.sid
having avg_score > 60;
```



```sql
+------+--------+-----------+
| sid  | sname  | avg_score |
+------+--------+-----------+
| 01   | 赵雷   |  89.66667 |
| 02   | 钱电   |  70.00000 |
| 03   | 孙风   |  80.00000 |
| 05   | 周梅   |  81.50000 |
| 07   | 郑竹   |  93.50000 |
+------+--------+-----------+
5 rows in set (0.00 sec)
```



### 查询 SC 表存在成绩的学生信息

查询 student 表。

筛选条件放在where语句里。

```sql
select *
from student s
where Sid in (select Sid from sc where score is not null)
```



```sql
+------+--------+---------------------+------+
| Sid  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
7 rows in set (0.00 sec)
```



### 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )

因为有空值，所以用 join。且因为空值是在 sc 里，让 sc left join student 表。

同时对学生编号分组，统计每个人的选课总数和总成绩。

```sql
select s.Sid, s.Sname, count(Cid) as count_course, sum(score) as sum_score
from student s
         left join sc on s.Sid = sc.Sid
group by s.Sid
```



```sql
+------+--------+--------------+-----------+
| sid  | sname  | 选课总数     | 总成绩    |
+------+--------+--------------+-----------+
| 01   | 赵雷   |            3 |     269.0 |
| 02   | 钱电   |            3 |     210.0 |
| 03   | 孙风   |            3 |     240.0 |
| 04   | 李云   |            3 |     100.0 |
| 05   | 周梅   |            2 |     163.0 |
| 06   | 吴兰   |            2 |      65.0 |
| 07   | 郑竹   |            2 |     187.0 |
| 08   | 王菊   |            0 |      NULL |
+------+--------+--------------+-----------+
8 rows in set (0.00 sec)
```



### 查带有成绩的学生信息

行转列时，使用 sum(case when ... then ...)

```sql
select s.Sid,
       s.Sname,
       count(Cid)                                       as count_course,
       sum(score)                                       as sum_score,
       sum(case when Cid = 01 then score else null end) as score_01,
       sum(case when Cid = 02 then score else null end) as score_02,
       sum(case when Cid = 03 then score else null end) as score_03
from student s,
     sc
where s.Sid = sc.Sid
group by s.Sid
```



```sql
+------+--------+--------------+-----------+----------+----------+----------+
| sid  | sname  | 选课总数     | 总成绩    | score_01 | score_02 | score_03 |
+------+--------+--------------+-----------+----------+----------+----------+
| 01   | 赵雷   |            3 |     269.0 |     80.0 |     90.0 |     99.0 |
| 02   | 钱电   |            3 |     210.0 |     70.0 |     60.0 |     80.0 |
| 03   | 孙风   |            3 |     240.0 |     80.0 |     80.0 |     80.0 |
| 04   | 李云   |            3 |     100.0 |     50.0 |     30.0 |     20.0 |
| 05   | 周梅   |            2 |     163.0 |     76.0 |     87.0 |     NULL |
| 06   | 吴兰   |            2 |      65.0 |     31.0 |     NULL |     34.0 |
| 07   | 郑竹   |            2 |     187.0 |     NULL |     89.0 |     98.0 |
+------+--------+--------------+-----------+----------+----------+----------+
7 rows in set (0.00 sec)
```



### 查询「李」姓老师的数量

```sql
select count(Tid)
from teacher t
where Tname like "李%"
```



```sql
+--------------+
| count(tname) |
+--------------+
|            1 |
+--------------+
1 row in set (0.00 sec)
```



### 查询学过「张三」老师授课的同学的信息

个人解法

```sql
select s.*
from student s,
     teacher t,
     sc,
     course c
where sc.Sid = s.Sid
  and c.Tid = t.Tid
  and c.Cid = sc.Cid
  and t.Tname = '张三'
```

优质解法

```sql
select *
from student
where sid in (
    select sid
    from sc,
         course,
         teacher
    where sc.cid = course.cid
      and course.tid = teacher.tid
      and tname = '张三'
)
```



```sql
+------+--------+---------------------+------+
| Sid  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
6 rows in set (0.00 sec)
```



### 查询没有学全所有课程的同学的信息

```sql
select *
from student
where sid in (
    select sid
    from sc
    group by sid
    having count(sc.Cid) < 3
)
```



```sql
+------+--------+---------------------+------+
| Sid  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
3 rows in set (0.00 sec)
```



### 查询和 "01" 号的同学学习的课程完全相同的其他同学的信息

```sql
select *
from student
where sid in (
    select sid
    from sc
    where cid in (select cid from sc where sid = '01')
      and sc.Sid <> '01'
    group by sid
    having count(sc.Cid) >= 3
)
```



```mysql
+------+--------+---------------------+------+
| Sid  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
+------+--------+---------------------+------+
3 rows in set (0.00 sec)
```



### 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

```mysql
select *
from Student
where Sid in (
    select distinct Sid
    from SC
    where Cid in (
        select Cid
        from SC
        where Sid = '01'
    )
)
```

```mysql
+------+--------+---------------------+------+
| Sid  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
7 rows in set (0.00 sec)
```



