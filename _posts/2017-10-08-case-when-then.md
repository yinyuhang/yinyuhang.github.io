---
layout: post
title: "ORDER BY 子句中的 CASE WHEN THEN"
subtitle: "随记"
date: 2017-10-08 15:11:05
tag: 
    - 随记
    - SQL
---

## THEN 后跟数字
对于SQL：
```
create table tb(col int);
insert tb
select 1 union all select 2 union all select 3;
 
/**实现2、1、3这样排序**/
 
select *
from tb
order by
  case col when 2 then 0
           when 1 then 1
           when 3 then 2
           else        3
   end;
 
### ---------------- OUTPUT ---------------

col         
----------- 
2
1
3
 
（所影响的行数为 3 行）
```
这里可以理解为分组排序，比如说，学生排队，凡是姓张的，我们给他们贴个标签“1”，凡是姓李的，我们给他们贴个标签“2”，凡是姓王的，我们给他们贴个标签“3”。然后按照 1 2 3 排序

## THEN 后跟列名

```
# table tt(bmid int,boardid int,parentid int)
select * from tt;

### ---------------- OUTPUT ---------------

bmid        boardid     parentid    
----------- ----------- ----------- 
1           1           9
2           2           17
5           3           18
9           0           0
15          0           0
16          4           17
17          0           0
18          0           0
5           3           18
```

```
select * from tt order by (case when parentid=0 then bmid else parentid end),parentid,bmid,boardid;

### ---------------- OUTPUT ---------------

bmid        boardid     parentid    
----------- ----------- ----------- 
9           0           0
1           1           9
15          0           0
17          0           0
2           2           17
16          4           17
18          0           0
5           3           18
5           3           18
```
而上一个SQL等同于
```
select (case when parentid=0 then bmid else parentid end) as myid, * 
from tt
order by myid,parentid,bmid,boardid;

### ---------------- OUTPUT ---------------

myid        bmid        boardid     parentid    
----------- ----------- ----------- ----------- 
9           9           0           0
9           1           1           9
15          15          0           0
17          17          0           0
17          2           2           17
17          16          4           17
18          18          0           0
18          5           3           18
18          5           3           18
```

#### 参考资料：
* [https://bbs.csdn.net/topics/50045003](https://bbs.csdn.net/topics/50045003)
* [https://bbs.csdn.net/topics/310106857](https://bbs.csdn.net/topics/310106857)