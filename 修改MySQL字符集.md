# 0. 缘起

开始用MySQL8.x，没注意字符集的事，只设了utf8mb4，COLLATE用了默认的utf8mb4_0900_ai_ci。

后来发现，每个现场的数据库，基本都是低版本，没有utf8mb4_0900_ai_ci这个字符集。

为了统一，将之前的utf8mb4_general_ci改为utf8mb4_0900_ai_ci。



总结：

之前的字符集：utf8mb4, utf8mb4_0900_ai_ci

修改后的字符集：utf8mb4, utf8mb4_general_ci



# 1. 思路

- 修改database的字符集
- 修改表的字符集
- 修改列的字符集

# 2. 操作

## 2.1 全库备份，有备无患

略

## 2.2 修改库的字符集

ALTER DATABASE SJZC CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;



## 2.3 修改表的字符集

### 2.3.1 生成SQL

```sql 
SELECT
	CONCAT( 'ALTER TABLE ', table_name, ' CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;' ) 
FROM
	information_schema.TABLES AS T,
	information_schema.`COLLATION_CHARACTER_SET_APPLICABILITY` AS C 
WHERE
	C.collation_name = T.table_collation 
	AND T.table_schema = 'SJZC' 
	AND ( C.CHARACTER_SET_NAME != 'utf8mb4' OR C.COLLATION_NAME != 'utf8mb4_general_ci' );
```

### 2.3.2 执行

略

## 2.4 修改列的字符集

### 2.4.1 生成varchar列字符集

```sql
SELECT
	CONCAT(
		'ALTER TABLE `',
		table_name,
		'` MODIFY `',
		column_name,
		'` ',
		DATA_TYPE,
		'(',
		CHARACTER_MAXIMUM_LENGTH,
		') CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci',
		( CASE WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL' ELSE '' END ),
		';' 
) 
FROM
	information_schema.COLUMNS 
WHERE
	TABLE_SCHEMA = 'SJZC' 
	AND DATA_TYPE = 'varchar' 
	AND ( CHARACTER_SET_NAME != 'utf8mb4' OR COLLATION_NAME != 'utf8mb4_general_ci' );
```



### 2.4.2 生成其他列字符集(text等)

```sql
SELECT
	CONCAT( 'ALTER TABLE `', table_name, '` MODIFY `', column_name, '` ', DATA_TYPE, ' CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci', ( CASE WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL' ELSE '' END ), ';' ) 
FROM
	information_schema.COLUMNS 
WHERE
	TABLE_SCHEMA = 'SJZC' 
	AND DATA_TYPE != 'varchar' 
	AND ( CHARACTER_SET_NAME != 'utf8mb4' OR COLLATION_NAME != 'utf8mb4_general_ci' );
```

### 2.4.3 执行

略