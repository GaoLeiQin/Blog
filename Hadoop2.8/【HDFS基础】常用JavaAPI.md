**在实际应用中，我们少不了要用Java开发HDFS，怎样操作呢？直接贴代码**

```
public class HdfsOperation {

    //1.上传本地文件
    public void uploadFile() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);

        //本地文件
        Path src = new Path("D:\\HebutWinOS");
        //HDFS为止
        Path dst = new Path("/");

        hdfs.copyFromLocalFile(src, dst);
        System.out.println("Upload to"+conf.get("fs.default.name"));

        FileStatus files[]= hdfs.listStatus(dst);
        for(FileStatus file:files){
            System.out.println(file.getPath());
        }
    }

    //2.创建HDFS文件
    public void createFile() throws IOException {
        Configuration conf=new Configuration();
        FileSystem hdfs=FileSystem.get(conf);

        byte[] buff="hello hadoop world!\n".getBytes();

        Path dfs=new Path("/test");

        FSDataOutputStream outputStream=hdfs.create(dfs);
        outputStream.write(buff,0,buff.length);
    }

    //3.创建HDFS目录
    public void createDir() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);

        Path dfs = new Path("/TestDir");

        hdfs.mkdirs(dfs);
    }

    //4.重命名HDFS文件
    public void renameFile() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);

        Path frpaht = new Path("/test");    //旧的文件名
        Path topath = new Path("/test1");    //新的文件名

        boolean isRename = hdfs.rename(frpaht, topath);

        String result = isRename?"成功":"失败";
        System.out.println("文件重命名结果为："+result);
    }

    //5.删除HDFS文件
    public void deleteFile() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);

        Path delef = new Path("/test1");

        boolean isDeleted = hdfs.delete(delef,false);

        //递归删除
        //boolean isDeleted=hdfs.delete(delef,true);

        System.out.println("Delete?"+isDeleted);
    }

    //6.查看某个HDFS文件是否存在
    public void isExitsFile() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);
        Path findf = new Path("/test1");

        boolean isExists = hdfs.exists(findf);

        System.out.println("Exist?"+isExists);
    }

    //7.查看HDFS文件的最后修改时间
    public void lastModifyTime() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);

        Path fpath = new Path("/user/hadoop/test/file1.txt");

        FileStatus fileStatus = hdfs.getFileStatus(fpath);
        long modiTime = fileStatus.getModificationTime();

        System.out.println("file1.txt的修改时间是"+modiTime);
    }

    //8.读取HDFS某个目录下的所有文件
    public void getAllFile() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);

        Path listf = new Path("/user/hadoop/test");

        FileStatus stats[] = hdfs.listStatus(listf);
        for(int i = 0; i < stats.length; ++i) {
            System.out.println(stats[i].getPath().toString());
        }

        hdfs.close();
    }

    //9.查找某个文件在HDFS集群的位置
    public void FileClusterLocation() throws IOException {
        Configuration conf = new Configuration();
        FileSystem hdfs = FileSystem.get(conf);
        Path fpath = new Path("/user/hadoop/cygwin");

        FileStatus filestatus = hdfs.getFileStatus(fpath);
        BlockLocation[] blkLocations = hdfs.getFileBlockLocations(filestatus, 0, filestatus.getLen());

        int blockLen = blkLocations.length;

        for(int i=0;i<blockLen;i++){
            String[] hosts = blkLocations[i].getHosts();
            System.out.println("block_"+i+"_location:"+hosts[0]);
        }
    }

    //10.获取HDFS集群上所有节点名称信息
    public void getAllClusterNameInfo() throws IOException {
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(conf);

        DistributedFileSystem hdfs = (DistributedFileSystem)fs;
        DatanodeInfo[] dataNodeStats = hdfs.getDataNodeStats();

        for(int i=0;i<dataNodeStats.length;i++){
            System.out.println("DataNode_"+i+"_Name:"+dataNodeStats[i].getHostName());
        }
    }


}

```

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

参考链接：[ShardedJedis虚拟节点一致性哈希分析](http://m635674608.iteye.com/blog/2297632)