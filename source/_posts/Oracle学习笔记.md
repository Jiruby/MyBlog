---
title: Oracle学习笔记
date: 2020-04-19 12:40:27
tags:
	- Oracle
categories: 学习笔记
---

## Oracle总结

### 一、数据库架构

```
1.一个数据库下一般只有一个实例
2.实例下可以创建多个表空间
3.一个表空间下可以创建多个用户（创建用户时给用户分配表空间）
4.一个用户下可以创建多张表
```

### 二、数据类型

|        |  mysql  |      oracle      |
| :----: | :-----: | :--------------: |
|  数字  |   int   |      number      |
| 字符串 | varchar | varchar/varchar2 |

### 三、增删改

|              |      mysql      |     oracle     |
| :----------: | :-------------: | :------------: |
| 默认事务级别 | repeatable read | read committed |
|     提交     |    自动提交     |  需手动commit  |

### 四、序列

```sql
-- 创建序列
create sequence s_序列名
[increase by n]
[start with n]

-- 使用序列完成主键自增
insert into person values(select s_person.nextval,'测试');

-- 查询当前序列  （dual伪表 补齐结构）
select s_person.currval from dual;
```

### 五、函数

#### 5.1 单行行数

``` sql
-- 	1. 字符函数
-- 转大写
select uppper('jim') from dual;  
-- 转小写
select lower('JIM') from dual;

--	2. 数值函数
-- 四舍五入
select round(21.54,1) from dual;  --21.5  
-- 截取
select trunc(55.29,1) from dual;  --55.2

--	3. 日期函数
-- 字符串与日期相互转换
-- fm 移除0开头
-- 24 24小时制
select to_char(sysdate,'fm yyyy-mm-dd hh24:mi:ss') from dual;
--to_date('','')

-- 	4. 时间计算
--	两个时间点之间的月数
select months_between(sysdate,hiredate) from emp;

--	5. 滤空     
-- 	mysql中使用ifnull()
select e.sal*12+nvl(comm,0) onyear from emp;

-- 	6. case when  条件表达式
--	等值
select ename , 
       case ename
	when 'SMITH' then '曹操'
	when 'KING' then '刘备'
	else '无名'
	end
	from emp;	
--  范围
select 
       e1.ename, d1.dname ,e1.sal ,
                 case 
                   when e1.sal >3001 then 5
                   when e1.sal >2001 then 4
                   when e1.sal>1404 then 3
                     when e1.sal>1201 then 2
                       when e1.sal>700 then 1
                         end   
       ,e2.ename ,d2.dname,e2.sal,
                  case 
                   when e2.sal >3001 then 5
                   when e2.sal >2001 then 4
                   when e2.sal>1404 then 3
                     when e2.sal>1201 then 2
                       when e2.sal>700 then 1
                         end 
from emp e1 , emp e2 ,dept d1 ,dept d2
where e1.mgr=e2.empno and e1.deptno=d1.deptno and d2.deptno=e2.deptno;
```

#### 5.2 多行函数（聚合函数）

| count | sum  |  avg   |  max   |  min   |
| :---: | :--: | :----: | :----: | :----: |
| 计数  | 求和 | 平均值 | 最大值 | 最小值 |

### 六、分组查询

1. where与having比较

   |      |      where       |      having      |
   | :--: | :--------------: | :--------------: |
   | 作用 | 过滤分组前的数据 | 过滤分组后的数据 |

2. select 后的列 只能是 **聚合函数**  和 **group by** 后的分组字段

    

### 七、多表查询

|    名称     |                语法                |
| :---------: | :--------------------------------: |
|  笛卡尔积   |             from a , b             |
|   内连接    |    from a inner join b on  条件    |
|  等值连接   |       from a , b where 条件        |
| 左/右外连接 |  from a left/right join b on 条件  |
| oracle方言  | from a , b where a. id (+)= b. uid |

关于外连接，主表方显示全部数据， （+）方使用 空值 补全

### 八、子查询

1.单行单列

= , > ,< ,>= ,<=

2.多行单列

in

3.多行多列

作为表

```sql
-- 查询每个部门中工资最的员工姓名、工资、部门名
select d.dname,e.ename,e.sal
from (select deptno,min(sal) msal from emp group by deptno) t ,
     emp e, dept d
where t.deptno=e.deptno and t.msal=e.sal and e.deptno=d.deptno;
```

### 九、分页查询

先排序后分页需要 3 层，不排序只需要 2 层，oracle分页查询格式固定，本质是子查询

```sql
select * from
  (select rownum rn,t.* from
    	(select * from emp order by sal desc) t
    where rownum<11
    )
where rn>5;
```

### 十、PLSQL

过程化编程语言

主要用于编写存储过程和存储函数

基本语法结构

```sql
declare
	i number(2) :=10;
	s varchar2(5) :='小智';
	ena emp.ename%type;   -- 引用型变量
	erow emp%rowtype;	 -- 记录型变量
begin
	dbms_output.put_line(i);
	dbms_output.put_line(s);
	select ename into ena from emp where empno='7788';
	dbms_output.put_line(ena);
	select * into erow from emp where empno='7788';
	dbms_output.put_line(erow.ename|| '的工作是：'|| erow.job);
end;
```

if分支

```sql
declare
	age number(3):=&num ;
begin
	if age<18 then
	dbms_output.put_line('未成年');
	elsif age<28 then      -- 注意是 elsif
	dbms_output.put_line('青年');
	elsif age<58 then
	dbms_output.put_line('中年');
	else
	dbms_output.put_line('老年');
	end if;
end;
```

loop循环

```sql
-- 循环输出1-10的数字
-- 1. while循环
declare
	i number:=1;
begin
	while i<=10 loop
	dbms_output.put_line(i);
	i:=i+1;
	end loop;
end;
-- 2. exit循环  （重点）
declare
	i number:=1;
begin
	loop
	exit when i>10;
	dbms_output.put_line(i);
	i:=i+1;
	end loop;
end;
-- 3. for循环
declare
	
begin
	for i in 1..10 loop
	dbms_output.put_line(i);
	end loop;
end;
```

cursor游标

```sql
-- 使用游标方式输出emp表中员工编号和姓名
declare 
	cursor cc is select * from emp;
	erow emp%rowtype;
begin
	open cc;
		loop
			fetch cc into erow;
			exit when cc%notfound;
			dbms_output.put_line(erow.empno||'  '||erow.ename);
		end loop;
	close cc;
end;


-- 给指定部门的员工+工资
declare 
  cursor cc(dno emp.deptno%type) is
  select empno from emp where deptno=dno;
  eno emp.empno%type;
begin
   open cc(10);
        loop
          fetch cc into eno;
          exit when cc%notfound;
          update emp set sal=sal+100 where empno=eno;
          commit;
        end loop;
   close cc;
end;
 
select * from emp where deptno=10;
```

### 十一、存储过程、存储函数

事先编写好的一段PL/SQL语句，存储在数据库内，供直接调用。

```sql
-- 给指定员工涨工资
create or replace procedure p1(eno emp.empno%type) 
is

begin
	update emp set sal=sal+100 where empno=eno;
  	commit;
end;

-- 计算指定员工的年薪
create or replace function f_yearsal(eno emp.empno%type) return number 
is
	ss number(10);
begin
	select (sal*12+nvl(comm,0)) into ss from emp where empno=eno;
	return ss;
end;

-- 测试f_yearsal
declare 
 s number(10);
begin
 s:=f_yearsal(7788) ;
 dbms_output.put_line(s);
end;

-- 使用存储过程计算年薪
create or replace procedure p_yearsal(eno emp.empno%type, yearsal out number) 
is
	s number(10);
	c emp.comm%type;
begin
	select sal*12,nvl(comm,0) into s,c from emp where empno=eno;
	yearsal:= s + c;
end;
```

JAVA调用存储过程、存储函数

```java
  {?= call <procedure-name>[(<arg1>,<arg2>, ...)]}   //存储函数
  {call <procedure-name>[(<arg1>,<arg2>, ...)]} 	//存储过程

//加载驱动
Class.forName("oracle.jdbc.driver.OracleDriver");
//获取连接
Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.16.100:1521:orcl", "scott", "tiger");
//存储函数
CallableStatement cs = connection.prepareCall("{?=call f_yearsal(?)}");
//设置参数
cs.setObject(2,7788);
//设置out返回值类型
cs.registerOutParameter(1, OracleTypes.NUMBER);
//执行
cs.execute();
//获取2返回值
System.out.println(cs.getObject(1));
cs.close();
connection.close();
```

### 十二、触发器

1.语句级触发器

2.行级触发器   （多一个 for each row  可以用 :old :new 行级对象）

基本格式

```sql
create or replace trigger t_update 
before
update
on emp 
for each row
begin
 	 if :new.sal-:old.sal>1000 then
   	 raise_application_error(-20001,'每次涨工资不能超过1000');
   	 end if;
end; 
```

