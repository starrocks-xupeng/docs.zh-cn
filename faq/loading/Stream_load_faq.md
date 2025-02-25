# Stream Load 常见问题

## 1. Stream Load 是否支持识别文本文件中首行的列名？或者是否支持指定不读取第一行？

Stream Load 不支持识别文本中首行的列名，首行对 Stream Load 来说也只是普通数据。当前也不支持指定不读取首行，如果需要导入的文本文件的首行为列名，可以使用如下四种方式处理：

- 在导出工具中修改设置，重新导出不带列名的文本文件。

- 使用 `sed -i '1d' filename` 等命令删除文本文件的首行。

- 在 Stream Load 执行语句中，使用 `-H "where: 列名 != '列名称'"` 把首行过滤掉。需要注意的是，当前系统会先转换、然后再做过滤，因此如果首行字符串转其他数据类型失败的话，会返回 `null`。所以，这种方式要求 StarRocks 表中的列不能有设置为 `NOT NULL` 的。

- 在Stream Load 执行语句中加入 `-H "max_filter_ratio:0.01"`，这样可以给导入作业设置一个 1% 或者更小、但能容错超过 1 行的容错率，从而将首行的错误忽视掉。您也可以根据实际数据量设置一个更小的容错率，但是要保证 1 行以上的容错。设置容错率后，返回结果的 `ErrorURL` 依旧会提示有错误，但导入作业整体会成功。容错率不宜设置过大，避免漏掉其他数据问题。

## 2. 当前业务的分区键对应的数据不是标准的 DATE 和 INT 类型，比如是 202106.00 的格式，如果需要使用 Stream Load 把这些数据导入到 StarRocks 中，需要如何转换？

StarRocks 支持在导入过程中进行数据转换，具体请参见[导入过程中完成数据转换](/loading/Etl_in_loading.md)。

假设待导入数据文件 `TEST` 为 CSV 格式，并且包含 `NO`、`DATE`、`VERSION`、`PRICE` 四列，但是其中 `DATE` 列是不规范的 202106.00 格式。如果在 StarRocks 中需使用的分区列为 `DATE`，那么首先需要在 StarRocks 中创建一张表，这里假设 StarRocks 表包含 `NO`、`VERSION`、`PRICE`、`DATE` 四列。您需要指定 `DATE` 列的数据类型为 DATE、DATETIME 或 INT。然后，在 Stream Load 执行语句中，通过指定如下设置来实现列之间的转换：

```Plain
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

`DATE_1` 可以简单地看成是先占位进行取数，然后通过 left() 函数进行转换，赋值给 StarRocks 表中的 `DATE` 列。特别需要注意的是，必须先列出 CSV 文件中所有列的临时名称，然后再使用函数进行转换。支持列转换的函数为标量函数，包括非聚合函数和窗口函数。

## 3. 数据质量问题报错 "ETL_QUALITY_UNSATISFIED; msg:quality not good enough to cancel" 应该怎么解决？

请参见[导入通用常见问题](/faq/loading/Loading_faq.md)。

## 4. 导入状态为 "Label Already Exists" 应该怎么解决？

请参见[导入通用常见问题](/faq/loading/Loading_faq.md)。
