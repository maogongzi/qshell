#Introduction
`qupload` is used to synchronize all local files in a given folder to qiniu bucket.
Some of it's features listed below：

1. A snapshot will be taken before synchronization getting started.
2. The successfully synchornized files will be recorded during synchronization, next time the user can resume the synchronization flow from the last interrumption point.
3. Show the progress of the synchronization flow so that user knows which one is the current file being synchronized and it's percentage done.

#Format
```
qshell qupload [<ThreadCount>] <LocalUploadConfig>
```

#Parameters

|name|description|optional|
|-----------|--------------------------|-----|
|ThreadCount|threads used to upload files in parallel|Y|
|LocalUploadConfig|config file, which includes parameters like the name of the target bucket, AccessKey, SecretKey, etc. for more details please take a look the descriptions about the config file.|N|

#Config file

`qupload` works together with a config file described as below:

```
{
   "src_dir"            :   "<LocalPath>",
   "access_key"         :   "<Your AccessKey>",
   "secret_key"         :   "<Your SecretKey>",
   "bucket"             :   "<Bucket>",
   "file_list"          :   "<FileList>",
   "zone"               :   "<Zone>",
   "ignore_dir"         :   false,
   "key_prefix"         :   "<Key Prefix>",
   "up_host"            :   "<Upload Host>",
   "overwrite"          :   false,
   "check_exists"       :   false,
   "check_hash"         :   false,
   "check_size"         :   false,
   "skip_file_prefixes" :   ".git,bin",
   "skip_path_prefixes" :   "hello/,temp/",
   "skip_fixed_strings" :   ".svn",
   "skip_suffixes"      :   ".DS_Store,.exe",
   "rescan_local"       :   true,
   "log_file"           :   "upload.log",
   "log_level"          :   "info"
}
```

|name|description|optional|
|-----------|------------|------------|
|src_dir|the local full path, the program will synchronize all files inside this folder; When you are using it on Windows platform, be sure that the `src_dir` follows the Windows naming convention, e.g. `D:\\jemy\\backup`, which means you need to use two backslashes for each folder level.|N|
|access_key|AccessKey of your corresponding qiniu account|N|
|secret_key|SecretKey of your corresponding qiniu account|N|
|bucket|the target bucket to be synchronized with, can be either public or private|N|
|file_list|list of the files to be synchronized, all the files should be located inside `src_dir`, the program will try to resolve the file list if it's not given.|Y|
|zone|the physical area to host the bucket, options can be `nb`, `bc` and `aws`, `nb` by default.|Y|
|up_host|domain for uploading files, optional, http://up.qiniu.com by default.|Y|
|ignore_dir|whether to flatten the folder structure when files are to be saved into the bucket, false by default.|Y|
|key_prefix|add a prefix for the file name when it's to be saved, an empty string by default.|Y|
|overwrite|override the existing file, by default don't.|Y|
|check_exists|whether to check in advance if there is a file with the same name before uploading a file. by default don't.|Y|
|check_hash|compare the local file hash with the remote file hash giving that `check_exists` set to `true`, by default don't, in order to save time.|Y|
|check_size|giving that `check_exists` set to `true` and `check_hash` set to `false`, you can set `check_size` to `true` to make a simple size checking, so that you may know whether the local file size equals to the remote file size or not. by default don't.|Y|
|skip_file_prefixes|skip all the files with a name prefix from the given list(do not take relative path into count.)|Y|
|skip_path_prefixes|skip all the files with a path prefix from the given list|Y|
|skip_fixed_strings|skip all the files with it's path including a string from the given list|Y|
|skip_suffixes|skip all the files with a name suffix from the given list.|Y|
|rescan_local|new added files will not be scanned and synchronized until it's set to `true`.|Y|
|log_level|uploading log level, options can be `debug`,`info`,`warn`,`error`, `info` by default.|Y|
|log_file|uploading log file, options can be `stdout`,`stderr`,`local/log/file`, `stdout` by default.|Y|

**Note: optional parameters is not required to be set in the config file unless you have to use them**

**about the options of `up_host`(in general, you are not suggested to modify it since the default option is already the best.): **

before choosing a value for `up_host`, please take the following options into consideration, also, it's suggested to `ping` the choosed domain for a connection status report.

|scenario|value|
|------|-------|
|domestic CDN speed-up uploading(with a poor network connection)| http://upload.qiniu.com |
|general domestic uploading| http://up.qiniu.com |
|upload back to China from abroad| http://up.qiniug.com |
|default| http://up.qiniu.com |

**the relationship among `check_exists`, `overwrite`, `check_hash` and `check_size`**

By default, the program itself will not check whether there is already a file with the same name, therefore, the uploading process will fail if a file with the same name exists, at this moment, the program either overrides the existing file if `override` is set to `true` or reports and error `{"error":"file exists"}`.

如果设置了`check_exists`为`true`, 那么同步程序在上传每个文件之前, 会检查空间中是否存在同名文件, 如果发现同名文件的话, 它会根据`check_hash`和`check_size`的值来做进一步的匹配检查, 如果最后确认文件内容或者大小一致, 那么就不再重复同步这个文件, 否则会根据`overwrite`的状态决定是否去强制覆盖.

其中`check_hash`设置为`true`的时候会在`check_exists`发现文件已在空间存在的情况下, 计算本地文件的hash来和空间中的文件hash进行匹配, 如果一致则不重复同步, 否则在`overwrite`为`true`的情况下会去强制覆盖.

因为计算本地文件的hash在文件较大的情况下可能会比较耗时, 如果你确信空间中如果存在同名文件的话, 那么它的内容肯定是和本地一致的, 那么做简单的文件大小检查可以节约很多时间, 这个`check_size`为`true`将在`check_exists`为`true`, 同时`check_hash`为`false`的时候生效, 仅检查本地文件大小是否和远程的一致, 如果不一致, 那么在`overwrite`为`true`的情况下会去强制覆盖.

**关于`skip_file_prefixes`, `skip_path_prefixes`, `skip_fixed_strings`和`skip_suffixes`**

 `skip_file_prefixes` 根据文件名（不带相对路径）的前缀来跳过, 不上传匹配的文件
 `skip_path_prefixes` 根据文件路径（相对路径）的前缀来跳过, 不上传匹配的文件
 `skip_fixed_strings` 根据文件路径（相对路径）中包含的字符串来跳过, 不上传路径中含有指定字符串列表中字符串的文件
 `skip_suffixes` 根据文件的后缀或者目录的后缀来跳过, 不上传文件或者匹配目录下的文件
 
 `skip_file_prefixes`, `skip_path_prefixes`, `skip_fixed_strings`和 `skip_suffixes` 里面都可以指定多个要忽略的文件或目录前缀或者后缀或者子字符串, 彼此用`逗号`分开, 另外字符串两边的空白字符会被忽略.
 
 比如对于目录  `/Users/jemy/Temp/upload` 下的文件, `src_dir`设置为 `/Users/jemy/Temp/upload`
 
 那么
 
```
 jemy@jemy-MacBook ~/Temp/upload $ tree
 .
 ├── cloud-market-1.jpg
 ├── cloud-market-2.jpg
 ├── hello.txt
 └── test
     └── cloud-market-3.jpg
```
 
我们可以设置`skip_suffixes` 为 `.txt` 来忽略txt文件, 或者`skip_path_prefixes`为`test`来跳过`test`文件夹不上传.
 
注意：`skip_file_prefixes`, `skip_path_prefixes`, `skip_fixed_strings` 和 `skip_suffixes` 均不支持通配符.

**关于`ignore_dir`和`key_prefix`**

所谓的`ignore_dir`的意思, 举个例子, 我们要同步目录`/Users/jemy/Temp`目录下的文件, 该目录下的文件结构如下：
```
.
├── f1
│   └── world.txt
└── hello.txt
```
如果我们忽略文件路径(ignore_dir=true), 那么保存在七牛空间中的文件名将是`world.txt`和`hello.txt`.如果不忽略文件路径(ignore_dir=false), 那么保存在七牛空间中的文件名将是`f1/world.txt`和`hello.txt`.

如果指定`key_prefix`这个参数, 那么程序将在本地的文件名前面加上这个字符串.这个`key_prefix`在`ignore_dir`的基础上生效, 比如我们忽略文件路径(ignore_dir=true)的情况下, `key_prefix`指定为`2015/01/18/`, 那么最终保存在七牛的文件名将是`2015/01/18/hello.txt`和`2015/01/18/world.txt`, 如果我们不忽略文件路径(ignore_dir=false)的情况下, `key_prefix`指定为`2015/01/18/`, 那么最终保存在七牛的文件名将是`2015/01/18/f1/world.txt`和`2015/01/18/hello.txt`.

# 场景化文件同步

下面是这个路径(`/Users/jemy/Temp7`)的文件树状结构：

```
├── frog.png
├── funny.png
├── golang.png
└── sub1
    ├── golang.png
    └── sub2
        └── funny.png
```

**想要同步本地文件到七牛, 最简单的方式**

> 注意: 如果系统是WINDOWS的话, `"src_dir"` 的参数应该是 `"D:\\jemy\\Temp7"`

```
{
    "src_dir"         :    "/Users/jemy/Temp7",
    "access_key"     :    "ELUs327kxVPJrGCXqWaexyioc0xYZyrIpbM6Wh6x",
    "secret_key"    :    "LVzZY2SqOQ_I_kM1n02ygACVBArDvOWtiLkDtKip",
    "bucket"        :    "if-pbl"
}
```

上面的配置文件将同步本地目录`/Users/jemy/Temp7`下面的文件到空间`if-pbl`中, 所有的保存在空间中的文件将以`相对于`路径`/Users/jemy/Temp7`的文件路径来命名.所以, 保存在空间中的文件名是：

```
frog.png
funny.png
golang.png
sub1/golang.png
sub1/sub2/funny.png
```

**想要给同步到七牛的空间加上统一前缀**

```
{
    "src_dir"         :    "/Users/jemy/Temp7",
    "access_key"     :    "ELUs327kxVPJrGCXqWaexyioc0xYZyrIpbM6Wh6x",
    "secret_key"    :    "LVzZY2SqOQ_I_kM1n02ygACVBArDvOWtiLkDtKip",
    "bucket"        :    "if-pbl",
    "key_prefix"    :    "2016/03/07/"
}
```

那么, 保存在空间中的文件名是：

```
2016/03/07/frog.png
2016/03/07/funny.png
2016/03/07/golang.png
2016/03/07/sub1/golang.png
2016/03/07/sub1/sub2/funny.png
```

**想要忽略本地文件的相对路径, 比如本地存在一些层级, 但是层级里面的文件和外面的文件名称和内容都相同的情况下, 为了避免重复上传不需要的文件, 可以使用忽略本地文件的相对路径的方法.这种情况下, 也可以附带额外的前缀.**

```
{
    "src_dir"         :    "/Users/jemy/Temp7",
    "access_key"     :    "ELUs327kxVPJrGCXqWaexyioc0xYZyrIpbM6Wh6x",
    "secret_key"    :    "LVzZY2SqOQ_I_kM1n02ygACVBArDvOWtiLkDtKip",
    "bucket"        :    "if-pbl",
    "key_prefix"    :    "2016/03/08/",
    "ignore_dir"    :    true
}
```

那么, 保存在空间中的文件名是：

```
2016/03/08/frog.png
2016/03/08/funny.png
2016/03/08/golang.png
```
由于`sub1`和`sub2`里面的文件和上一个层级相同, 默认会被忽略（通过默认不支持同名文件覆盖上传实现）
