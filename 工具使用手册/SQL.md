### #1 了解SQL
- 概念
	- database数据库：保存有组织数据的容器（文件柜）
	- table表：结构化的文件
	- schema模式：关于数据库和表的布局及特性的信息
	- column列：表中的一个字段
	- row行

### #2 检索数据
- SELECT
```SQL
SELECT name FROM products
SELECT name, price FROM products
SELECT * FROM products
```

### #3 排序检索数据
- ORDER BY
	- 确保是最后一条子句
	- 按多个列排序（如果前者相同则比较后者）
	- 可以使用数字表示列的位置
```SQL
SELECT name FROM products ORDER BY price
SELECT name FROM products ORDER BY price, name
SELECT name FROM products ORDER BY 2, 3
SELECT name FROM products ORDER BY price DESC, name
```

### #4 过滤数据
- WHERE
```SQL
SELECT name FROM products WHERE price = 3.49
SELECT name FROM products WHERE price BETWEEN 5 AND 10
SELECT name FROM products WHERE price IS NULL
```

### 5 高级数据过滤
- 组合WHERE子句
	- SQL会优先处理AND操作符，再处理OR操作符
```SQL
SELECT name FROM products WHERE price=3.49 AND name='apple'
SELECT name FROM products WHERE price=3.49 OR name='apple'

SELECT name FROM products WHERE price=3
AND (name='apple' OR name='orange')

SELECT name FROM products WHERE name IN ('apple', 'orange')
SELECT name FROM products WHERE NOT name='apple'
```

### #6 用通配符进行过滤
- `%`：表示不限数量任意字符
```SQL
-- 匹配以1 Fish开头的name
SELECT name FROM products WHERE name LIKE '1 Fish%'
```
- `_`：表示一个任意字符
```SQL
SELECT name FROM products WHERE name LIKE '_ Fish'
```
- `[]`：指定对应位置可以匹配的单个字符
	- `^`：表示不包含
```SQL
-- 只有name为1 Fish和2 Fish符合
SELECT name FROM products WHERE name LIKE '[12] Fish'

SELECT name FROM products WHERE name LIKE '[^12] Fish'
```

### #7 创建计算字段
- `+`或者`||`：输出结果为字段值拼接
	- `TRIM()`：去掉拼接结果因为数据库输出格式而填充的空格
	- `LTRIM()`或`RTRIM()`：去除左侧或者右侧的空格
```SQL
-- 输出诸如：John(USA)
SELECT name+'('+country+')' FROM Vendors
SELECT RTRIM(name)+'('+RTRIM(country)+')' FROM Vendors

-- 特例：MySQL
SELECT CONCAT(name, '(', country, ')') FROM Vendors
```
- `AS`：别名，即为列命名或者重命名
```SQL
SELECT name+'('+country+')' AS title FROM Vendors
```
- 算术计算
```SQL
-- 总价 = 单价*数量
SELECT name, quantity, price, quantity*price AS final 
FROM OrderItems
```

### #8 使用数据处理函数
- 文本（字符串）处理函数
	- `UPPER()`：转大写
	- `LENGTH()`：求长度
	- `SOUNDEX()`：找到发音相近的字符串
```SQL
SELECT UPPER(name) AS name2 FROM Vendors
```
- 日期处理函数：各个DBMS不太相同，需要查询文档
- 数值处理函数：包括求绝对值、正余弦等

### #9 汇总数据
- 聚集函数
	- `COUNT()`
		- `COUNT(*)`：表示对表中行得数目进行计数
		- `COUNT(column)`：表示对非null值的行的计数
```SQL
SELECT AVG(price) AS avg_price FROM Products
SELECT COUNT(price) AS avg_price FROM Products WHERE id = 2
```
- 聚集不同值
	- `DISTINCT`：只包含不同的值
```SQL
SELECT AVG(DISTINCT price) AS avg_price FROM Products
```

### #10 分组数据
- `GOURP BY`：创建分组
	- 必须出现在WHERE之后
```SQL
SELECT id, COUNT(*) AS prods FROM Product GROUP BY id
```
- `HAVING`
	- 作用
		- 允许使用聚合函数来筛选
		- 过滤分组（在数据分组后进行过滤）
		- 在结果集中进行筛选
```SQL
SELECT id, COUNT(*) AS prods 
FROM Product 
GROUP BY id
HAVING COUNT(*)>=2

SELECT price FROM goods HAVING price>100
```

### #11 使用子查询```SQL
- 子查询：嵌套在其他查询中得查询
	- 子查询语句只能查询单个列
```SQL
SELECT id FROM Orders WHERE order_num IN (
	SELECT order_num FROM OrderItems
	WHERE prod_id = '001' 
)

SELECT name, (
	SELECT COUNT(*) FROM Orders WHERE Orders.id = Customers.id)
FROM Customers
```

#### #12 联结表
- 关系数据库
	- 一类数据一个表，从而减少重复数据的出现
	- 使用联结将多个关系表连接，可以有多种表达方式
- 内部联结
```SQL
SELECT name, price FROM Vendors, Products
WHERE Vendors.id = Products.id

-- 等效于以下这句(INNER可以省略)
SELECT name, price FROM Vendors 
INNER JOIN Products ON Vendors.id = Products.id
```
- 多表连结
```SQL
SELECT name, price FROM Vendors, Products, Customers
WHERE Vendors.id = Products.id
	AND Customers.id = Vendors.id
```

#### #13 创建高级联结
- 表别名
```SQL
SELECT name, price 
FROM Vendors AS V, Products AS P, Customers AS C
WHERE V.id = P.id AND C.id = V.id
```
- 自联结
```SQL
SELECT id, name FROM Customers
WHERE name = (SELECT name FROM Customers WHERE contact = "Jim")

-- equals below
SELECT c1.id, c1.name, FROM Customers AS c1, Customers As c2
WHERE c1.name = c2.name AND c2.Contact = "Jim"
```
- 自然联结：手动将所有表格的列信息一起展示
- 外部联结
	- 外部联结包括了没有关联行的行
	- 在使用`OUTER JOIN`时，必须使用FULL，RIGHT或LEFT关键字指定包括其所有行的表（`JOIN`关键字左边的表还是右边的表）
	- 使用`*=`表示左外部联结，`=*`表示右外部联结
```SQL
SELECT Customers.id, Orders.num FROM Customers
LEFT OUTER JOIN Orders ON Customers.id = Orders.id

-- equals below
SELECT Customers.id, Orders.num FROM Customers, Orders ON
WHERE Customers.id *= Orders.id
```

#### #14 组合查询
- UNION
	- 执行多个查询，并将结果集合并
	- UNION的结果会去除重复的行，而UNION ALL不会
```SQL
SELECT name, email FROM Customers 
WHERE country IN ('US', 'UK')
UNION
SELECT name, email FROM Customers
WHERE name = 'Fun'
```

#### #15 插入数据
- 直接插入
```SQL
INSERT INTO Customers VALUES('100','Tom',NULL,'NY')
INSERT INTO Customers(id,name,city) VALUES('100','Tom','NY')
```
- 将其他表格按格式插入：使用INSERT SELECT
	- 逐行搜索并插入到已经存在的表
```SQL
INSERT INTO Customers(id, name, email, city)
SELECT old_id, old_name, old_email, old_city FROM OldList
```
- 将其他表格按格式插入：使用SELECT INTO
	- 创建并插入到新表中
```SQL
SELECT * INTO CustCopy FROM Customers
```

#### #16 更新和删除数据
- 更新
```SQL
UPDATE Customers
SET country = 'UK', city = 'London'
WHERE name = 'Tom'
```
- 删除
```SQL
DELETE FROM Customers WHERE id = '100'
```

#### #17 创建和操纵表
- 创建
	- 对于MySQL，varchar必须替换为text
```SQL
CREATE TABLE Products(
	id CHAR(10) NOT NULL,
	name CHAR(254) NOT NULL,
	desc VARCHAR(1000) DEFAULT "..."
)
```
- 更新表
	- 极为小心！应该先备份再修改
```SQL
ALTER TABLE Vendors ADD phone CHAR(20)
ALTER TABLE Vendors DROP COLUMN phone
```
- 删除表
```SQL
DROP TABLE CustCopy
```

#### #18 使用视图
- 视图
	- 虚拟的表
	- 使用场景
		- 简化了复杂的SQL操作
		- 在关系数据库中，使用视图虚拟地创建联结表，同样简化了查询操作
		- 通过限定访问权限保护数据
		- 更改数据格式和表示
	- 规则
		- 像表一样唯一命名
		- 可以创建子视图（嵌套）
```SQL
SELECT name, contact FROM Customers
```
- 使用
```SQL
CREATE VIEW ProductCustmers AS 
SELECT name, contact, id FROM Customers, Orders
WHERE Customers.id = Orders.id

SELECT name contact FROM ProductCustmers WHERE id = '100'

DROP VIEW ProductCustmers
```

#### #19 存储过程
- 使用原因
	- 需要多条语句的情形。使用存储过程将这些语句合并为一个操作
- 创建存储过程：不同DBMS会有很大出入
```SQL
CREATE PROCEDURE MailingListCount
AS
DECLARE @cnt INTEGER
SELECT @cnt = COUNT(*)
FROM Customers
WHERE NOT email IS NULL
RETURN @cnt
```
- 执行存储过程
```SQL
EXECUTE AddNewProduct('ST001', 'Tower')
```

#### #20 管理事务处理
- 事务处理
	- 维护数据库的完整性
	- 保证SQL操作要么完全执行，要么不执行，不会中途停止
	- 如果发生错误，则进行回退
```SQL
BEGIN TRANSACTION
-- 事务内容
DELETE OrderItems WHERE num = 100
DELETE Orders WHERE num = 100
COMMIT TRANSACTION
```
- 使用保留点
	- 使得不必回退到起点
```SQL
BEGIN TRANSACTION

INSERT INTO Customers(id, num) VALUES('001', 'Tom');
SAVE TRANSACTION StartOrder;
INSERT INTO Orders(num, date) VALUES(100, '2001/1/1');
IF @@ERROR <> 0 ROLLBACK TRANSACTION StartOrder;
INSERT INTO OrderItems(num, quantity) VALUES(100, 1);
IF @@ERROR <> 0 ROLLBACK TRANSACTION StartOrder;

COMMIT TRANSACTION
```

#### #21 使用游标
- 结果集：SQL查询所检索出的结果
- 游标：存储在DBMS服务器上的数据库查询

#### #22 高级SQL特性
- 约束
	- 主键
	- 外键
	- 唯一约束
	- 检查约束
```SQL
CREATE TABLE Orders(
	num INTEGER NOT NULL PRIMARY KEY,
	id CHAR(10) NOT NULL REFERENCES Customers(id),
	quantity INTEGER NOT NULL CHECK(quantity>0)
)
```
- 索引
	- 用于排序数据以加快搜索和排序操作的速度
	- 索引能够改善某些检索操作的性能，但是降低数据插入修改删除的性能
	- 对于数据库管理员而言，索引应该定期检查
```SQL
CREATE INDEX prod_name_ind ON Products(prod_name);
```
- 触发器
	- 在特定表的数据库活动发生时自动执行
	- 使用场景
		- 确保输入数据的格式保持一致
		- 跟踪数据库操作并写入日志
	- 不同DBMS的触发器创建语法差异很大
```SQL
// 强制转大写
CREATE TRIGGER state ON Customers
FOR INSERT, UPDATE
AS UPDATE Customers SET state = Upper(state)
WHERE Customers.id = inserted.id
```
- 数据库安全
	- 权限授予与撤销