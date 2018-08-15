在simpleDB中，每一个Table会被保存到一个独立文件中，在代码中对应RecordFile。

RecodeFile构造函数：
```
public RecordFile(TableInfo ti, Transaction tx) {
    this.ti = ti;
    this.tx = tx;
    filename = ti.fileName();
    // 检查文件大小如果为0，则调用appendBlock()添加一个block
    if (tx.size(filename) == 0)
        appendBlock();
    moveTo(0);
}
```