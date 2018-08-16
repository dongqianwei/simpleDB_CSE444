在simpleDB中，每一个Table会被保存到一个独立文件中，在代码中对应RecordFile。

```
package simpledb.record;

import simpledb.file.Block;
import simpledb.tx.Transaction;

/**
 * Manages a file of records.
 * There are methods for iterating through the records
 * and accessing their contents.
 *
 * @author Edward Sciore
 */
public class RecordFile {
    // 表元数据
    private TableInfo ti;
    // 当前事务对象
    private Transaction tx;
    // 表文件名
    private String filename;
    // 当前页管理单元
    private RecordPage rp;
    // 当前块序号
    private int currentblknum;

    /**
     * Constructs an object to manage a file of records.
     * If the file does not exist, it is created.
     *
     * @param ti the table metadata
     * @param tx the transaction
     */
    public RecordFile(TableInfo ti, Transaction tx) {
        this.ti = ti;
        this.tx = tx;
        filename = ti.fileName();
        //检查表文件大小。如果不存在则创建
        if (tx.size(filename) == 0)
        //如果文件为空，添加一个块
            appendBlock();
        // currentblknum = 0
        // 初始化rp指向第一个块
        moveTo(0);
    }

    /**
     * Closes the record file.
     */
    public void close() {
        // 释放对当前块的引用
        rp.close();
    }

    /**
     * Positions the current record so that a call to method next
     * will wind up at the first record.
     */
    public void beforeFirst() {
        moveTo(0);
    }

    /**
     * Moves to the next record. Returns false if there
     * is no next record.
     *
     * @return false if there is no next record.
     */
    public boolean next() {
        while (true) {
            // 如果当前块中有下一条Record，移动过去，返回true
            if (rp.next())
                return true;
            // 如果当前块已经是最后一个块，说明没有下一条Record了，返回false
            if (atLastBlock())
                return false;
            // 移动到下一个块
            moveTo(currentblknum + 1);
        }
    }

    /**
     * Returns the value of the specified field
     * in the current record.
     *
     * @param fldname the name of the field
     * @return the integer value at that field
     */
    public int getInt(String fldname) {
        return rp.getInt(fldname);
    }

    /**
     * Returns the value of the specified field
     * in the current record.
     *
     * @param fldname the name of the field
     * @return the string value at that field
     */
    public String getString(String fldname) {
        return rp.getString(fldname);
    }

    /**
     * Sets the value of the specified field
     * in the current record.
     *
     * @param fldname the name of the field
     * @param val     the new value for the field
     */
     // 设置当前块中的当前Record中当前字段的Int型值
    public void setInt(String fldname, int val) {
        rp.setInt(fldname, val);
    }

    /**
     * Sets the value of the specified field
     * in the current record.
     *
     * @param fldname the name of the field
     * @param val     the new value for the field
     */
    public void setString(String fldname, String val) {
        rp.setString(fldname, val);
    }

    /**
     * Deletes the current record.
     * The client must call next() to move to
     * the next record.
     * Calls to methods on a deleted record
     * have unspecified behavior.
     */
     // 将当前块中当前Record设置为EMPTY
    public void delete() {
        rp.delete();
    }

    /**
     * Inserts a new, blank record somewhere in the file
     * beginning at the current record.
     * If the new record does not fit into an existing block,
     * then a new block is appended to the file.
     */
    public void insert() {
        // 在当前块中找一个EMPTY的RECORD，如果找到，则设置为INUSE，并指向它
        while (!rp.insert()) {
            // 检查是否为最后一个BLOCK了，如果是，在文件末尾新增一个Block
            if (atLastBlock())
                appendBlock();
            // 移动到下一个Block，继续查找空闲Record
            moveTo(currentblknum + 1);
        }
    }

    /**
     * Positions the current record as indicated by the
     * specified RID.
     *
     * @param rid a record identifier
     */
    // 根据RID找到Record
    public void moveToRid(RID rid) {
        moveTo(rid.blockNumber());
        rp.moveToId(rid.id());
    }

    /**
     * Returns the RID of the current record.
     *
     * @return a record identifier
     */
    public RID currentRid() {
        int id = rp.currentId();
        return new RID(currentblknum, id);
    }

    private void moveTo(int b) {
        if (rp != null)
            rp.close();
        currentblknum = b;
        Block blk = new Block(filename, currentblknum);
        rp = new RecordPage(blk, ti, tx);
    }

    private boolean atLastBlock() {
        return currentblknum == tx.size(filename) - 1;
    }

    private void appendBlock() {
        RecordFormatter fmtr = new RecordFormatter(ti);
        tx.append(filename, fmtr);
    }
}
```