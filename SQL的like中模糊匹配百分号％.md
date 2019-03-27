SQL用like做模糊匹配，如下所示：

```sql
select *
from student
where name like '张%'
```

这样就能匹配到所有姓「张」的学生。

但这里有个问题，就是当我要匹配的内容中有「%」该怎么办？

这就用到`escape`了。

**比如：** 找出名字中含有「%」的论文：

```sql
select *
from paper
where name like '%\%%' escape '\'
```

`escape`的意思是：后面的内容`\`是前面字符串`%\%%`的转义字符。这样前面字符串中的第2个`%`将不再认为是通配符，而被认为是要匹配的内容。

`escape` 只作用于一个like，当出现多个like都需要转义时，每一个like后面都需要加`escape`。

**比如：** 找出名字 或 内容中含有「%」的论文：

```sql
select *
from paper
where name like '%\%%' escape '\' or content like '%\%%'
```

