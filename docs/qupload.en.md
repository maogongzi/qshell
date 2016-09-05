#Introduction
`qupload`是用来将本地目录中的所有文件同步到七牛空间中的命令。
`qupload` is used to synchronize all local files in a given folder to qiniu bucket.
有如下一些特点：
Some of it's features listed below：

1. 同步之前，本地路径中文件快照功能
1. A snapshot will be taken before synchronization getting started.
2. 同步过程中，记录下同步成功的文件位置，在被中止后，下次重新同步时，可以从中止处开始
2. The successfully synchornized files will be recorded during synchronization, next time the user can resume the synchronization flow from the last interrumption point.
3. 同步过程的进度显示，可以看到当前同步的文件是所有文件中的第几个，以及同步进度百分比
3. Show the progress of the synchronization flow so that user knows which one is the current file being synchronized and it's percentage done.

#格式
#Format
```
qshell qupload [<ThreadCount>] <LocalUploadConfig>
```

#参数
#Parameters

|参数名|描述|可选参数|
|name|description|optional|
|-----------|--------------------------|-----|
|ThreadCount|并发上传的协程数量|Y|
|ThreadCount|threads used to upload files in parallel|Y|
|LocalUploadConfig|数据同步的配置文件，该配置文件里面包含了一些诸如目标空间名称，AccessKey，SecretKey等信息，详情参考配置文件的讲解|N|
|LocalUploadConfig|config file, which includes parameters like the name of the target bucket, AccessKey, SecretKey, etc. for more details please take a look the descriptions about the config file.|N|

#配置
#Config file

`qupload`功能需要配置文件的支持，配置文件的内容如下：
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

|参数名|描述|可选参数|
|name|description|optional|
|-----------|------------|------------|
|src_dir|本地同步路径，为全路径格式，程序将同步该目录下面所有的文件；在Windows系统下面使用的时候，注意`dest_dir`的设置遵循`D:\\jemy\\backup`这种方式。也就是路径里面的`\`要有两个（`\\`）|N|
|src_dir|the local full path, the program will synchronize all files inside this folder; When you are using it on Windows platform, be sure that the `src_dir` follows the Windows naming convention, e.g. `D:\\jemy\\backup`, which means you need to use two backslashes for each folder level.|N|
|access_key|七牛账号对应的AccessKey|N|
|access_key|AccessKey of your corresponding qiniu account|N|
|secret_key|七牛账号对应的SecretKey|N|
|secret_key|SecretKey of your corresponding qiniu account|N|
|bucket|同步数据的目标空间名称，可以为公开空间或私有空间|N|
|bucket|the target bucket to be synchronized with, can be either public or private|N|
|file_list|待同步文件列表，该文件列表内容必须是相对于`src_dir`的文件相对路径列表，可以不指定，程序将自动获取`src_dir`下面的文件列表。|Y|
|file_list|list of the files to be synchronized,|Y|
|zone|空间所属的区域位置，可选值为`nb`，`bc`和`aws`，默认为`nb`|Y|
|up_host|上传域名，可选设置，默认为 http://up.qiniu.com|Y|
|ignore_dir|保存文件在七牛空间时，使用的文件名是否忽略本地路径，默认为false|Y|
|key_prefix|在保存文件在七牛空间时，使用的文件名的前缀，默认为空字符串|Y|
|overwrite|是否覆盖空间中已有的同名文件，默认不覆盖。|Y|
|check_exists|每个文件上传之前是否检查空间中是否存在同名文件，默认为false，不检查|Y|
|check_hash|在`check_exists`设置为`true`的情况下生效，是否检查本地文件hash和空间文件hash一致，默认不检查，节约同步时间|Y|
|check_size|在`check_exists`设置为`true`的情况下，如果`check_hash`为`false`，那么你可以设置`check_size`为`true`做简单的大小检查，以查看本地文件和空间文件大小是否一致，默认不检查|Y|
|skip_file_prefixes|跳过所有文件名（不带相对路径）以该前缀列表里面字符串为前缀的文件|Y|
|skip_path_prefixes|跳过所有文件路径（相对路径）以该前缀列表里面字符串为前缀的文件|Y|
|skip_fixed_strings|跳过所有文件路径（相对路径）中包含该字符串列表中字符串的文件|Y|
|skip_suffixes|跳过所有以该后缀列表里面字符串为后缀的文件或者目录|Y|
|rescan_local|默认情况下，本地新增的文件不会被同步，需要手动设置为true才会去检测新增文件。|Y|
|log_level|上传日志输出级别，可选值为`debug`,`info`,`warn`,`error`,默认`info`|Y|
|log_file|上传日志的输出文件，可选值为`stdout`,`stderr`,`本地日志文件名`，默认`stdout`|Y|

**备注：可选参数可以根据个人需要设置，不使用时不必写在配置文件中**

**关于`up_host`的可选值(一般不需要自己设置，默认的效果即是最好的！！！)：**

在选择`up_host`的时候，可以参考如下选项，另外在上传之前，可以用`ping`来测试一下所选择的域名来判断当前的网络情况。

|场景|值|
|------|-------|
|国内CDN加速上传（网络不佳的情况下）| http://upload.qiniu.com |
|国内常规上传| http://up.qiniu.com |
|海外上传到国内| http://up.qiniug.com |
|默认，不设置| http://up.qiniu.com |

**关于`check_exists`，`overwrite`，`check_hash`，`check_size`的关系**

默认情况下，同步程序不会检查本地文件是否已经在空间中有一个同名文件，这样如果空间中已经存储同名文件，那么上传默认就会失败，在这个时候，如果`overwrite`设置为`true`的话，会覆盖远程文件，否则会报错`{"error":"file exists"}`。

如果设置了`check_exists`为`true`，那么同步程序在上传每个文件之前，会检查空间中是否存在同名文件，如果发现同名文件的话，它会根据`check_hash`和`check_size`的值来做进一步的匹配检查，如果最后确认文件内容或者大小一致，那么就不再重复同步这个文件，否则会根据`overwrite`的状态决定是否去强制覆盖。

其中`check_hash`设置为`true`的时候会在`check_exists`发现文件已在空间存在的情况下，计算本地文件的hash来和空间中的文件hash进行匹配，如果一致则不重复同步，否则在`overwrite`为`true`的情况下会去强制覆盖。

因为计算本地文件的hash在文件较大的情况下可能会比较耗时，如果你确信空间中如果存在同名文件的话，那么它的内容肯定是和本地一致的，那么做简单的文件大小检查可以节约很多时间，这个`check_size`为`true`将在`check_exists`为`true`，同时`check_hash`为`false`的时候生效，仅检查本地文件大小是否和远程的一致，如果不一致，那么在`overwrite`为`true`的情况下会去强制覆盖。

**关于`skip_file_prefixes`，`skip_path_prefixes`，`skip_fixed_strings`和`skip_suffixes`**

 `skip_file_prefixes` 根据文件名（不带相对路径）的前缀来跳过，不上传匹配的文件
 `skip_path_prefixes` 根据文件路径（相对路径）的前缀来跳过，不上传匹配的文件
 `skip_fixed_strings` 根据文件路径（相对路径）中包含的字符串来跳过，不上传路径中含有指定字符串列表中字符串的文件
 `skip_suffixes` 根据文件的后缀或者目录的后缀来跳过，不上传文件或者匹配目录下的文件
 
 `skip_file_prefixes`，`skip_path_prefixes`，`skip_fixed_strings`和 `skip_suffixes` 里面都可以指定多个要忽略的文件或目录前缀或者后缀或者子字符串，彼此用`逗号`分开，另外字符串两边的空白字符会被忽略。
 
 比如对于目录  `/Users/jemy/Temp/upload` 下的文件，`src_dir`设置为 `/Users/jemy/Temp/upload`
 
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
 
我们可以设置`skip_suffixes` 为 `.txt` 来忽略txt文件，或者`skip_path_prefixes`为`test`来跳过`test`文件夹不上传。
 
注意：`skip_file_prefixes`，`skip_path_prefixes`，`skip_fixed_strings` 和 `skip_suffixes` 均不支持通配符。

**关于`ignore_dir`和`key_prefix`**

所谓的`ignore_dir`的意思，举个例子，我们要同步目录`/Users/jemy/Temp`目录下的文件，该目录下的文件结构如下：
```
.
├── f1
│   └── world.txt
└── hello.txt
```
如果我们忽略文件路径(ignore_dir=true)，那么保存在七牛空间中的文件名将是`world.txt`和`hello.txt`。如果不忽略文件路径(ignore_dir=false)，那么保存在七牛空间中的文件名将是`f1/world.txt`和`hello.txt`。

如果指定`key_prefix`这个参数，那么程序将在本地的文件名前面加上这个字符串。这个`key_prefix`在`ignore_dir`的基础上生效，比如我们忽略文件路径(ignore_dir=true)的情况下，`key_prefix`指定为`2015/01/18/`，那么最终保存在七牛的文件名将是`2015/01/18/hello.txt`和`2015/01/18/world.txt`，如果我们不忽略文件路径(ignore_dir=false)的情况下，`key_prefix`指定为`2015/01/18/`，那么最终保存在七牛的文件名将是`2015/01/18/f1/world.txt`和`2015/01/18/hello.txt`。

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

**想要同步本地文件到七牛，最简单的方式**

> 注意: 如果系统是WINDOWS的话，`"src_dir"` 的参数应该是 `"D:\\jemy\\Temp7"`

```
{
    "src_dir"         :    "/Users/jemy/Temp7",
    "access_key"     :    "ELUs327kxVPJrGCXqWaexyioc0xYZyrIpbM6Wh6x",
    "secret_key"    :    "LVzZY2SqOQ_I_kM1n02ygACVBArDvOWtiLkDtKip",
    "bucket"        :    "if-pbl"
}
```

上面的配置文件将同步本地目录`/Users/jemy/Temp7`下面的文件到空间`if-pbl`中，所有的保存在空间中的文件将以`相对于`路径`/Users/jemy/Temp7`的文件路径来命名。所以，保存在空间中的文件名是：

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

那么，保存在空间中的文件名是：

```
2016/03/07/frog.png
2016/03/07/funny.png
2016/03/07/golang.png
2016/03/07/sub1/golang.png
2016/03/07/sub1/sub2/funny.png
```

**想要忽略本地文件的相对路径，比如本地存在一些层级，但是层级里面的文件和外面的文件名称和内容都相同的情况下，为了避免重复上传不需要的文件，可以使用忽略本地文件的相对路径的方法。这种情况下，也可以附带额外的前缀。**

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

那么，保存在空间中的文件名是：

```
2016/03/08/frog.png
2016/03/08/funny.png
2016/03/08/golang.png
```
由于`sub1`和`sub2`里面的文件和上一个层级相同，默认会被忽略（通过默认不支持同名文件覆盖上传实现）
