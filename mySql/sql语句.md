## 常用 Sql

# 查询

```sql
SELECT column1, column2, ...
FROM table_name;
```

column1, column2 --查询的参数名
table_name --查询的表名称

在 go 中的应用：

```go
model.DB.Query("SELECT username, id, email FROM users")
```

查询 username, id, email 在 users 表中

# where

```sql
SELECT id, username, email FROM users WHERE id = ?
```

查询 id 与传入 id 相同数据的 id,username,email

```go
model.DB.QueryRow("SELECT id, username, email FROM users WHERE id = ?", id).Scan(&user.ID, &user.Username, &user.Email)
```

<!-- 通过id查询到指定数据并赋值给user结构 -->
