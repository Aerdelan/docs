## 使用 SQL 更新数据表

# 转换 excel 表格

新建表：
-- 创建临时表（如果还没有创建的话）
CREATE TEMPORARY TABLE temp_table (
name VARCHAR(255),
new_data_column VARCHAR(255)
);

保存 excel 为 csv 文件，执行：
LOAD DATA INFILE '/path/to/your/file.csv'
INTO TABLE temp_table
FIELDS TERMINATED BY ',' -- CSV 文件的字段分隔符
ENCLOSED BY '"' -- 如果字段值被双引号括起来，使用这个选项
LINES TERMINATED BY '\n' -- 行结束符
IGNORE 1 ROWS -- 如果 CSV 文件有表头，跳过第一行
(name, new_data_column); -- 对应 CSV 文件的列

使用 SQL 导入数据：
UPDATE your_table yt
JOIN temp_table tt ON yt.name = tt.name
SET yt.data_column = tt.new_data_column;
