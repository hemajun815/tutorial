# HBase的Java Api

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)、[Ubuntu16.04环境下安装配置HBase2.1.0集群](./installing-hbase2.1.0-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；

## 使用实例

判断表是否存在：

```java
public boolean tableExists(String tableName) throws Exception {
    HBaseAdmin hBaseAdmin = new HBaseAdmin(configuration);
    boolean ret = hBaseAdmin.tableExists(tableName);
    hBaseAdmin.close();
    return ret;
}
```

创建表：

```java
public void createTable(String tableName, String[] families) throws Exception {
    HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));
    if (!this.tableExists(hTableDescriptor.getNameAsString())) {
        for (String family : families) {
            hTableDescriptor.addFamily(new HColumnDescriptor(family));
        }
        HBaseAdmin hBaseAdmin = new HBaseAdmin(this.configuration);
        hBaseAdmin.createTable(hTableDescriptor);
        hBaseAdmin.close();
    }
}
```

使表可用：

```java
public void enableTable(String tableName) throws Exception {
    if (this.tableExists(tableName)) {
        HBaseAdmin hBaseAdmin = new HBaseAdmin(this.configuration);
        hBaseAdmin.enableTable(tableName);
        hBaseAdmin.close();
    }
}
```

使表不可用：

```java
public void disableTable(String tableName) throws Exception {
    if (this.tableExists(tableName)) {
        HBaseAdmin hBaseAdmin = new HBaseAdmin(this.configuration);
        hBaseAdmin.disableTable(tableName);
        hBaseAdmin.close();
    }
}
```

删除表：

```java
public void deleteTable(String tableName) throws Exception {
    if (this.tableExists(tableName)) {
        this.disableTable(tableName);
        HBaseAdmin hBaseAdmin = new HBaseAdmin(this.configuration);
        hBaseAdmin.deleteTable(tableName);
        hBaseAdmin.close();
    }
}
```

插入数据：

```java
public void put(String tableName, List<Put> puts) throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    hTable.put(puts);
    hTable.close();
}

public void put(String tableName, String rowKey, String family, String qualifier, String value)
        throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Put put = new Put(Bytes.toBytes(rowKey));
    put.add(Bytes.toBytes(family), Bytes.toBytes(qualifier), Bytes.toBytes(value));
    hTable.put(put);
    hTable.close();
}
```

删除数据：

```java
public void delete(String tableName, String rowKey) throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Delete delete = new Delete(Bytes.toBytes(rowKey));
    hTable.delete(delete);
    hTable.close();
}

public void delete(String tableName, String rowKey, String family) throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Delete delete = new Delete(Bytes.toBytes(rowKey));
    delete.deleteFamily(Bytes.toBytes(family));
    hTable.delete(delete);
    hTable.close();
}

public void delete(String tableName, String rowKey, String family, String qualifier) throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Delete delete = new Delete(Bytes.toBytes(rowKey));
    delete.deleteColumn(Bytes.toBytes(family), Bytes.toBytes(qualifier));
    hTable.delete(delete);
    hTable.close();
}
```

扫描表：

```java
public List<Result> scan(String tableName) throws Exception {
    HTable table = new HTable(this.configuration, tableName);
    Scan scan = new Scan();
    ResultScanner scanner = table.getScanner(scan);
    List<Result> list = new ArrayList<>();
    for (Result result : scanner) {
        list.add(result);
    }
    scanner.close();
    table.close();
    return list;
}
```

追加插入：

```java
public void append(String tableName, String rowKey, String family, String qualifier, String appendValue)
        throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Append append = new Append(Bytes.toBytes(rowKey));
    append.add(Bytes.toBytes(family), Bytes.toBytes(qualifier), Bytes.toBytes(appendValue));
    hTable.append(append);
    hTable.close();
}
```

修改满足条件的数据：

```java
public void checkAndPut(String tableName, String rowKey, String family, String qualifier, String value,
        String familyCheck, String qualifierCheck, String valueCheck) throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Put put = new Put(Bytes.toBytes(rowKey));
    put.add(Bytes.toBytes(family), Bytes.toBytes(qualifier), Bytes.toBytes(value));
    hTable.checkAndPut(Bytes.toBytes(rowKey), Bytes.toBytes(familyCheck), Bytes.toBytes(qualifierCheck),
            Bytes.toBytes(valueCheck), put);
    hTable.close();
```

删除满足条件的数据：

```java
public void checkAndDelete(String tableName, String rowKey, String family, String qualifier,
        String familyCheck, String qualifierCheck, String valueCheck) throws Exception {
    HTable hTable = new HTable(this.configuration, tableName);
    Delete delete = new Delete(Bytes.toBytes(rowKey));
    delete.deleteColumn(Bytes.toBytes(family), Bytes.toBytes(qualifier));
    hTable.checkAndDelete(Bytes.toBytes(rowKey), Bytes.toBytes(familyCheck), Bytes.toBytes(qualifierCheck),
            Bytes.toBytes(valueCheck), delete);
    hTable.close();
}
```
