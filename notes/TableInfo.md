TableInfo是关于一个Table的元数据

其成员包含
```
//字段元数据
private Schema schema;
//各字段名称偏移量，在构造函数中初始化
private Map<String, Integer> offsets;
//总偏移量
private int recordlen;
//表名
private String tblname;
```