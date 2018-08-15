# RemoteStatement interface 为数据库操作入口

```
public interface RemoteStatement extends Remote {
    public RemoteResultSet executeQuery(String qry) throws RemoteException;

    public int executeUpdate(String cmd) throws RemoteException;
}
```