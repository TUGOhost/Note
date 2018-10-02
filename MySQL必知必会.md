# 检索数据

将介绍如何使用SELECT语句从表中检索一个或多个数据列。

## 检索单个列

**输入**

```sql
SELECT prod_name FROM products;
```

**分析**

上述语句利用SELECT语句从products表中检索一个名为prod_name的列。所需的列名在SELECT关键字之后给出，FROM关键字指出从其中检索数据的表名。

## 检索多个列

在选择多个列时，一定要在列名之间加上逗号，但最后一个列名后不加。

**输入**

```sql
SELECT prod_id, prod_name, prod_price FROM products;
```

**分析**

与前一个列子一样，这条语句使用SELCT语句从表products中选择数据。

## 检索所有列

**输入**

```sql
SELECT * FROM products;
```

**分析**

如果给定一个通配符（*），则返回表中所有列。列的顺序一般是列在表中定义中出现的顺序。但有时候并不是这样的，表的模式的变化（如添加或删除列）可能会导致顺序的变化。

## 检索不同的行

使用`DISTINCT`关键字，指示MySQL只返回不同的值。

**输入**

```sql
SELECT DISTINCT vend_id FROM products;
```

## 限制结果

为了返回第一行或前几行，可使用LIMIT子句。

**输入**

```sql
SELECT prod_name FROM products LIMIT 5;
```

**分析**

此语句使用SELECT语句检索单个列。LIMIT 5指示MySQL返回不多于5行。

**输入**

```sql
SELECT prod_name FROM products LIMIT 5, 5;
```

**分析**

LIMIT 5, 5只是MySQL返回从行5开始的5行。

---

行0 检索出来的第一行为行0而不是行1

---

## 使用完全限定的表名

迄今为止使用的SQL例子只通过列名引用列。也可能会使用完全限定的名字来引用列（同时使用表名和列字）。

**输入**

```sql
SELECT products.prod_name FROM products;
```

这条语句等同于

```sql
SELECT prod_name FROM products;
```

表名也可以是完全限定的，如下所示：

**输入**

```sql
SELECT produrcts.prod_name FROM crashcourse.productsl
```

这条语句在功能上也等于刚使用的那条语句。

