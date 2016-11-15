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

The program will check whether there is an existing copy using the same name in the remote bucket before start uploading a file if `check_exists` is set to `true`, further checking will be made to the found copy based on `check_hash` and `check_size`, finally, the synchronization process will be cancelled if it's confirmed that the content or size of the two files is the same, or the copy will be overrided depending on `overwrite`.

To make it more clear, given that `check_exists` and `check_hash` both set to `true` and a possible remote copy has been found, the program will calculate the hashes of both the local file and remote copy then compare them, synchronization will be cancelled if the hashes match with each other, otherwise it will forcibly override the existing remote copy when `overwrite` set to `true`.

Since it's a time-consuming task to calculate file hash if it is very big in size, you may want to simply apply a file-size checking to save your time if you are sure that there would be a copy with the same name and content in the remote bucket. Thus, the `check_size` option will come in to play when `check_exists` set to `true` and `check_hash` set to `false`, it will only check the sizes of the local file and it's possible remote copy and forcibly override the remote one when the sizes don't match and `overwrite` set to `true`.

**About `skip_file_prefixes`, `skip_path_prefixes`, `skip_fixed_strings` and `skip_suffixes`**

 `skip_file_prefixes` will skip uploading files according to their names(without relative paths)
 `skip_path_prefixes` will skip uploading files according to their relative paths
 `skip_fixed_strings` will skip uploading files if given strings found in their relative paths.
 `skip_suffixes` will skip uploading files or folders if they have a given suffix.

 `skip_file_prefixes`, `skip_path_prefixes`, `skip_fixed_strings` and `skip_suffixes` all support multiple values of files or folders or suffixes or strings separated by comma, the leading and trailing whitespaces will be ignored.

 For example, according to files inside of folder `/Users/jemy/Temp/upload`, given that `src_dir` set to `/Users/jemy/Temp/upload`,

 Then:
 
```
 jemy@jemy-MacBook ~/Temp/upload $ tree
 .
 ├── cloud-market-1.jpg
 ├── cloud-market-2.jpg
 ├── hello.txt
 └── test
     └── cloud-market-3.jpg
```
 
We can set `skip_suffixes` to `.txt` to ignore txt files, or `skip_path_prefixes` to `test` to ignore the whole `test` folder.
 
Notice: `skip_file_prefixes`, `skip_path_prefixes`, `skip_fixed_strings` and `skip_suffixes` all don't support wildcards.

**About`ignore_dir` and `key_prefix`**

The meaning of `ignore_dir`, for example, if we want to synchronize the files inside of the folder `/Users/jemy/Temp`, which has a tree structure like below:

```
.
├── f1
│   └── world.txt
└── hello.txt
```

If we ignore file paths(ignore_dir=true), then the files to be saved in the remote bucket will be `world.txt` and `hello.txt`, but if we dont' ignore file paths(ignore_dir=false), then the files to be saved will become `f1/world.txt` and `hello.txt`.

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
