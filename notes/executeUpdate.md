# RemoteStatement.executeUpdate操作最终调用到Planner.executeUpdate

```
public int executeUpdate(String cmd, Transaction tx) {
    Parser parser = new Parser(cmd);
    Object obj = parser.updateCmd();
    if (obj instanceof InsertData)
        return uplanner.executeInsert((InsertData) obj, tx);
    else if (obj instanceof DeleteData)
        return uplanner.executeDelete((DeleteData) obj, tx);
    else if (obj instanceof ModifyData)
        return uplanner.executeModify((ModifyData) obj, tx);
    else if (obj instanceof CreateTableData)
        return uplanner.executeCreateTable((CreateTableData) obj, tx);
    else if (obj instanceof CreateViewData)
        return uplanner.executeCreateView((CreateViewData) obj, tx);
    else if (obj instanceof CreateIndexData)
        return uplanner.executeCreateIndex((CreateIndexData) obj, tx);
    else
        return 0;
}
```

首先看创建数据库表操作executeCreateTable:

最最终调用到TableMgr.createTable

```
public void createTable(String tblname, Schema sch, Transaction tx) {
    TableInfo ti = new TableInfo(tblname, sch);
    // insert one record into tblcat
    RecordFile tcatfile = new RecordFile(tcatInfo, tx);
    tcatfile.insert();
    tcatfile.setString("tblname", tblname);
    tcatfile.setInt("reclength", ti.recordLength());
    tcatfile.close();

    // insert a record into fldcat for each field
    RecordFile fcatfile = new RecordFile(fcatInfo, tx);
    for (String fldname : sch.fields()) {
        fcatfile.insert();
        fcatfile.setString("tblname", tblname);
        fcatfile.setString("fldname", fldname);
        fcatfile.setInt("type", sch.type(fldname));
        fcatfile.setInt("length", sch.length(fldname));
        fcatfile.setInt("offset", ti.offset(fldname));
    }
    fcatfile.close();
}
```