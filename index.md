开窗函数 在使用时对每行数据进行开窗 使用时耗费大量资源、

###### 思考   :是否可以不使用 over 实现其效果。。

如果要达到效果须在 select 中进行子查询得到 聚合值和原数据 组成新的表

###### 问题来了：！！！

Hive是否支持在select中 添加子查询呢？ 去官网看看

![hive_select_subquery](file://D:/Users/lidin/Documents/SomeThing/image/hive_select_subquery.png?lastModify=1597497010)

英语不好 红色部分愣是没看懂 翻译一下

![translate.png](file://D:/Users/lidin/Documents/SomeThing/image/translate.png?lastModify=1597497010)

这翻译还不如没有跟看不懂了  

还是自己试下把接下来终于要写代码了   hive版本3.1.2

##### 先建个表：

```
create table score
(
    uid        string,
    subject_id string,
    score      int
)
    row format delimited fields terminated by '\t';

insert into table scorevalues ('1001', '01', '90');
insert into table scorevalues ('1001', '02', '90');
insert into table scorevalues ('1001', '03', '90');
insert into table scorevalues ('1002', '01', '85');
insert into table scorevalues ('1002', '02', '85');
insert into table scorevalues ('1002', '03', '70');
insert into table scorevalues ('1003', '01', '70');
insert into table scorevalues ('1003', '02', '70');
insert into table scorevalues ('1003', '03', '80');
```

##### 需求：

​      每个人的每科分数，以及总分

```
--使用开窗函数很容易便实现此效果
select
*,
sum(score) over(partition by uid)
from score;
```

![resTable1](file://D:/Users/lidin/Documents/SomeThing/image/resTable1.png?lastModify=1597497010)

```
--使用子查询  开窗函数实际是对每条数据建立一个视图 取出值在于原表连接
select
*,
(select sum(score)
from score s1
where s1.uid = s2.uid  --开窗函数内层查询与外层查询进行where过滤，相当于对每条数据按照uid 分组再求聚合值
)
from score s2 ;

```

![resTable](file://D:/Users/lidin/Documents/SomeThing/image/resTable.png?lastModify=1597497010)



两种方法结果完全相同 那再增加些复杂度看看  进行累加  

```
--使用开窗函数很容易便实现此效果  使用开船函数进行累加 只需要再分区后加上排序即可
select
*,
sum(score) over(partition by uid order by score)
from score;
```

over()的方式实现很容易，那子查询呢

![{A72C4EFB-7152-4DBA-B90C-FA1EB85C7682}](file://D:/Users/lidin/Documents/SomeThing/image/%7BA72C4EFB-7152-4DBA-B90C-FA1EB85C7682%7D.png?lastModify=1597497010)

看到 hive中 支持 where 过滤 和 having 过滤  不可以使用 order by ！！！！

##### 问题又来了 ！！！怎么实现累加！！！！ （我压力好大，真的好难 T_T）

但是一法通万法通啊世上无难事啊  sort by 怎么实现的  就 a < b 呗 也就这么点事 那问题就解决了



```
--再过滤条件中添加个排序就行啊   结果跟 over()相同
select
*,
(select sum(score)
from score s1
where s1.uid = s2.uid
and s1.score <= s2.score   --升序排序 <= 降序排序 >= 实现
)
from score s2 ;
```



以此类推 我们已经成功实现了 聚合函数+over（分区，排序）  

那么接下来就到了 获取 前（后）行数据以及 自定义开窗范围 

###### ……嗯嗯嗯嗯嗯嗯 这个呢  就以后再说吧  