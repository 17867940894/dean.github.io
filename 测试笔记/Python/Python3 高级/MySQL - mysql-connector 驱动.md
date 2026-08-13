使用 **mysql-connector** 来连接使用 MySQL， **mysql-connector** 是 **MySQL** 官方提供的驱动器。

我们可以使用 **pip** 命令来安装 **mysql-connector**：

```python
python -m pip install mysql-connector
```

## 创建数据库连接

创建数据库使用 "CREATE DATABASE" 语句，以下创建一个名为 runoob_db 的数据库：

```python
import mysql.connector

cfg = {
    "host": "127.0.0.1",
    "port": 3306,
    "user": "root",
    "password": "root",
    "database": "51_test",
    "charset": "utf8mb4"
}

conn = mysql.connector.connect(**cfg)
print("连接成功")
conn.close()
```

## 自定义工具函数声明

```python
import mysql.connector

# 数据库连接配置
cfg = {
    "host": "127.0.0.1",
    "port": 3306,
    "user": "root",
    "password": "root",
    "database": "runoob_db ",
    "charset": "utf8mb4",
    # 开启严格模式, 否则无法校验超长字符串
    "sql_mode": "TRADITIONAL"
}

# 初始化sites表
def init_table_sites(mysql_conn, mysql_cursor):
    mysql_cursor.execute("""DROP TABLE IF EXISTS sites""")
    mysql_cursor.execute("""CREATE TABLE sites (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    url VARCHAR(255)
    )""")
    mysql_conn.commit()

# 获取数据库连接
def get_connection():
    mysql_conn = mysql.connector.connect(**cfg)
    return mysql_conn

# 遍历查询结果
def fetchall(mysql_cursor):
    results = mysql_cursor.fetchall()
    for result in results:
        print(result)

# 执行SQL语句
def execute(mysql_conn, mysql_cursor, mysql_sql, *args):
    if args:
        mysql_cursor.execute(mysql_sql, args)
    else:
        mysql_cursor.execute(mysql_sql)
    return mysql_cursor

# 执行批量SQL语句
def executemany(mysql_conn, mysql_cursor, mysql_sql, params):
    if params:
        mysql_cursor.executemany(mysql_sql, params)
    return mysql_cursor
```

## 创建数据表

创建数据表使用 **"CREATE TABLE"** 语句，创建数据表前，需要确保数据库已存在，以下创建一个名为 **sites** 的数据表：

```python
if __name__ == '__main__':
    results = mysql_cursor.fetchall()
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE sites (name VARCHAR(255), url VARCHAR(255))")
```

### 主键设置

创建表的时候我们一般都会设置一个主键（PRIMARY KEY），我们可以使用 **"INT AUTO_INCREMENT PRIMARY KEY"** 语句来创建一个主键，主键起始值为 1，逐步递增。

如果我们的表已经创建，我们需要使用 **ALTER TABLE** 来给表添加主键：

```python
if __name__ == '__main__':
    results = mysql_cursor.fetchall()
    cursor = conn.cursor()
    cursor.execute("ALTER TABLE sites ADD COLUMN id INT AUTO_INCREMENT PRIMARY KEY")
```

## 插入数据

插入数据使用 **"INSERT INTO"** 语句：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
    val = ("RUNOOB", "https://www.runoob.com")
    execute(conn, cursor, sql, *val)
    conn.commit()

    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
```

### 批量插入

批量插入使用 **executemany()** 方法，该方法的第二个参数是一个元组列表，包含了我们要插入的数据：

向 sites 表插入多条记录。

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
    val = [
        ('Google', 'https://www.google.com'),
        ('Github', 'https://www.github.com'),
        ('Taobao', 'https://www.taobao.com'),
        ('stackoverflow', 'https://www.stackoverflow.com/')
    ]
    executemany(conn, cursor, sql, val)
    conn.commit()

    execute(conn, cursor, "SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
```

## 查询数据

查询数据使用 **SELECT** 语句：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
    val = [
        ('Google', 'https://www.google.com'),
        ('Github', 'https://www.github.com'),
        ('Taobao', 'https://www.taobao.com'),
        ('stackoverflow', 'https://www.stackoverflow.com/')
    ]
    executemany(conn, cursor, sql, val)
    conn.commit()

    execute(conn, cursor, "SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
```

读取指定的字段数据：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
    val = [
        ('Google', 'https://www.google.com'),
        ('Github', 'https://www.github.com'),
        ('Taobao', 'https://www.taobao.com'),
        ('stackoverflow', 'https://www.stackoverflow.com/')
    ]
    executemany(conn, cursor, sql, val)
    conn.commit()

    execute(conn, cursor, "SELECT name, url FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
```

读取一条数据，可以使用 **fetchone()** 方法：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
    val = [
        ('Google', 'https://www.google.com'),
        ('Github', 'https://www.github.com'),
        ('Taobao', 'https://www.taobao.com'),
        ('stackoverflow', 'https://www.stackoverflow.com/')
    ]
    executemany(conn, cursor, sql, val)
    conn.commit()

    cursor.execute("SELECT * FROM sites")
    result = cursor.fetchone()
    print(result)
    print("-----------------")
    fetchall(cursor)
    """
    问题原理： 使用 fetchone() 只读取了结果集中的第一条记录，但结果集中还有其他未读取的数据。
    当尝试关闭 cursor 时，MySQL connector 检测到存在未读取的结果，
    就会触发 Unread result found 错误。
    """
    cursor.close()
    conn.close()
```

### where 条件语句

如果我们要读取指定条件的数据，可以使用 **where** 语句：

读取 name 字段为 RUNOOB 的记录：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)

    sql = "SELECT * FROM sites WHERE name ='RUNOOB'"
    cursor.execute(sql)
    result = cursor.fetchall()
    if result:
        for row in result:
            print(row)
    else:
        executemany(conn, cursor, sql)
        conn.commit()
    cursor.execute("SELECT * FROM sites")
    result = cursor.fetchone()
    print("--------------")
    fetchall(cursor)
    cursor.close()
    conn.close()
```

也可以使用通配符 **%**：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)

    sql = "SELECT * FROM sites WHERE url LIKE '%oo%'"
    cursor.execute(sql)
    result = cursor.fetchall()
    if result:
        for row in result:
            print(row)
    else:
        executemany(conn, cursor, sql)
        conn.commit()
    cursor.execute("SELECT * FROM sites")
    result = cursor.fetchone()
    print("--------------")
    fetchall(cursor)
    cursor.close()
    conn.close()
```

为了防止数据库查询发生 SQL 注入的攻击，我们可以使用 **%s** 占位符来转义查询的条件：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)

    sql = "SELECT * FROM sites WHERE name = %s"
    cursor.execute(sql, ("RUNOOB",))
    result = cursor.fetchall()
    if result:
        for row in result:
            print(row)
    else:
        executemany(conn, cursor, sql)
        conn.commit()
    cursor.execute("SELECT * FROM sites")
    result = cursor.fetchone()
    print("--------------")
    fetchall(cursor)
    cursor.close()
    conn.close()
```

### 排序

查询结果排序可以使用 **ORDER BY** 语句，默认的排序方式为升序，关键字为 **ASC**，如果要设置降序排序，可以设置关键字 **DESC**。

降序分页排序

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)

    execute(conn, cursor, "SELECT name, url FROM sites order by name desc limit 3")
    fetchall(cursor)

    cursor.close()
    conn.close()
>>>
('Taobao', 'https://www.taobao.com')
('stackoverflow', 'https://www.stackoverflow.com/')
('RUNOOB', 'https://www.runoob.com')

进程已结束，退出代码为 0
```

指定起始位置，使用的关键字是 **OFFSET**：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)

    execute(conn, cursor, "SELECT name, url FROM sites"
                          " order by name desc limit 3 offset 1")
    fetchall(cursor)

    cursor.close()
    conn.close()
>>>
('stackoverflow', 'https://www.stackoverflow.com/')
('RUNOOB', 'https://www.runoob.com')
('Google', 'https://www.google.com')

进程已结束，退出代码为 0
```

## 删除记录

删除记录使用 **"DELETE FROM"** 语句：

删除 name 为 stackoverflow 的记录：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)
    print("--------------")
    sql = "DELETE FROM sites WHERE name = 'stackoverflow'"
    execute(conn, cursor, sql)
    conn.commit()
    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
>>>
(1, 'Google', 'https://www.google.com')
(2, 'Github', 'https://www.github.com')
(3, 'Taobao', 'https://www.taobao.com')
(4, 'stackoverflow', 'https://www.stackoverflow.com/')
(5, 'RUNOOB', 'https://www.runoob.com')
--------------
(1, 'Google', 'https://www.google.com')
(2, 'Github', 'https://www.github.com')
(3, 'Taobao', 'https://www.taobao.com')
(5, 'RUNOOB', 'https://www.runoob.com')

进程已结束，退出代码为 0
```

**注意：**要慎重使用删除语句，删除语句要确保指定了 WHERE 条件语句，否则会导致整表数据被删除。

为了防止数据库查询发生 SQL 注入的攻击，我们可以使用 **%s** 占位符来转义删除语句的条件：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)
    print("--------------")
    sql = "DELETE FROM sites WHERE name = %s"
    na = ("stackoverflow",)
    cursor.execute(sql, na)
    conn.commit()
    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
>>>
(1, 'Google', 'https://www.google.com')
(2, 'Github', 'https://www.github.com')
(3, 'Taobao', 'https://www.taobao.com')
(4, 'stackoverflow', 'https://www.stackoverflow.com/')
(5, 'RUNOOB', 'https://www.runoob.com')
--------------
(1, 'Google', 'https://www.google.com')
(2, 'Github', 'https://www.github.com')
(3, 'Taobao', 'https://www.taobao.com')
(5, 'RUNOOB', 'https://www.runoob.com')
```

## 更新表数据

数据表更新使用 **"UPDATE"** 语句：

将 name 为 RUNOOB的字段数据改为 ZH：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "UPDATE sites SET name = 'ZH' WHERE name = 'RUNOOB'"
    execute(conn, cursor, sql)
    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
>>>
(1, 'Google', 'https://www.google.com')
(2, 'Github', 'https://www.github.com')
(3, 'Taobao', 'https://www.taobao.com')
(4, 'stackoverflow', 'https://www.stackoverflow.com/')
(5, 'ZH', 'https://www.runoob.com')

进程已结束，退出代码为 0
```

**注意：**UPDATE 语句要确保指定了 WHERE 条件语句，否则会导致整表数据被更新。

为了防止数据库查询发生 SQL 注入的攻击，我们可以使用 %s 占位符来转义更新语句的条件：

```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    # 初始化sites表
    init_table_sites(conn, cursor)
    sql = "UPDATE sites SET name = %s WHERE name = %s"
    val = ("ZH", "RUNOOB")
    cursor.execute(sql, val)
    conn.commit()
    cursor.execute("SELECT * FROM sites")
    fetchall(cursor)

    cursor.close()
    conn.close()
>>>
(1, 'Google', 'https://www.google.com')
(2, 'Github', 'https://www.github.com')
(3, 'Taobao', 'https://www.taobao.com')
(4, 'stackoverflow', 'https://www.stackoverflow.com/')
(5, 'ZH', 'https://www.runoob.com')

进程已结束，退出代码为 0
```

## 删除表

删除表使用 **"DROP TABLE"** 语句， **IF EXISTS** 关键字是用于判断表是否存在，只有在存在的情况才删除：

```python
# 初始化sites表
def init_table_sites(mysql_conn, mysql_cursor):
    mysql_cursor.execute("""DROP TABLE IF EXISTS sites""")
    mysql_cursor.execute("""CREATE TABLE sites (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    url VARCHAR(255)
    )""")

    sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
    val = [
        ('Google', 'https://www.google.com'),
        ('Github', 'https://www.github.com'),
        ('Taobao', 'https://www.taobao.com'),
        ('stackoverflow', 'https://www.stackoverflow.com/'),
        ("RUNOOB", "https://www.runoob.com")
    ]
    executemany(mysql_conn, mysql_cursor, sql, val)
    mysql_conn.commit()
```

