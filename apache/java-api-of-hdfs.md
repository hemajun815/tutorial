# HDFS的Java Api

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；
- 官方Api文档：[http://hadoop.apache.org/docs/r2.8.2/api/index.html](http://hadoop.apache.org/docs/r2.8.2/api/index.html)。

## 使用实例

- 获取文件系统
  
  ```java
  public static FileSystem getFileSystem(String url, String user=null) {
    if(StringUtils.isBlank(url)){
      return null;
    }
    Configuration configuration = new Configuration();
    FileSystem fs = null;
    try {
      URI uri = new URI(url.trim());
      if (user == null)
        fs = FileSystem.get(uri, configuration);
      else
        fs = FileSystem.get(uri, configuration, user);
    } catch (URISyntaxException|IOException e) {
      System.out.println(e);
    }
    return fs;
  }
  ```

- 创建目录

  ```java
  public static boolean mkdir(String path) throws Exception{
    FileSystem fs = getFileSystem(path, "root");
    boolean b = fs.mkdirs(new Path(path));
    fs.close();
    return b;
  }
  ```

- 读文件

  ```java
  public static void readFile(String filePath) throws IOException{
    FileSystem fs = getFileSystem(filePath, "root");
    InputStream in = null;
    try{
      in = fs.open(new Path(filePath));
      IOUtils.copyBytes(in, System.out, 4096, false);
    }catch(Exception e){
      System.out.println(e.getMessage());
    }finally{
      IOUtils.closeStream(in);
    }
  }
  ```

- 上传文件
  
  ```java
  public static void putFile(String localPath,String hdfsPath) throws IOException{
    FileSystem fs = getFileSystem(hdfsPath, "root");
    fs.copyFromLocalFile(new Path(localPath), new Path(hdfsPath));
    fs.close();
  } 
  ```

- 下载文件

  ```java
  public static void getFile(String hdfsPath,String localPath) throws IOException{
    FileSystem fs = getFileSystem(hdfsPath,"root");
    Path hdfs_path = new Path(hdfsPath);
    Path local_path = new Path(localPath);
    fs.copyToLocalFile(hdfs_path, local_path);
    fs.close();
  }
  ```

- 递归删除

  ```java
  public static boolean deleteFile(String hdfsPath) throws IllegalArgumentException, IOException{
    FileSystem fs = getFileSystem(hdfsPath, "root");
    return fs.delete(new Path(hdfsPath), true);
  }
  ```

- 目录列表
  ```java
  public static String[] listFile(String hdfsPath){
    String[] files = new String[0];
    FileSystem fs = getFileSystem(hdfsPath,"root");
    Path path=new Path(hdfsPath);
    FileStatus[] st=null;
    try {
      st = fs.listStatus(path);
    } catch (FileNotFoundException e) {
      e.printStackTrace();
    } catch (IOException e) {
      e.printStackTrace();
    }
    files = new String[st.length];
    for(int i=0;i<st.length;i++){
      files[i]=st[i].toString();
    }
    return files;
  }
  ```
