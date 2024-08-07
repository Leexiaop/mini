# yarn包的安装机制

yarn 包的安装过程主要分为5部：

## 检测包

这一步主要是检查项目中是否存在一些npm相关的文件，比如package-lock.json文件等，如果存在，会提示用户可能存在冲突，当然也会检测系统os和CPU信息等。

## 解析包

这一步主要是解析依赖树种的每一个包的版本信息（package.json）。

1. 首先就是获取dependencies、devDependencies, optionalDependencies等内容，这些都是首层依赖。
2. 接着就是遍历首层依赖，获取包的版本信息。并且递归查找每个包下面的嵌套依赖的版本信息。将解析过的包和正在解析的包用set数据结构存储起来。这样就能保证同一个版本的包不会被重复解析。
    + 对于没有解析过的包A,首次尝试从yarn.lock文件中获取版本信息。并标记为已解析。
    + 如果yarn.lock文件中没有找到包A,则向registry发起请求，获取已知的满足版本要求的包信息，获取后，标记为已解析。

总之，通过这一步，就能确定所有依赖的具体版本信息以及下载地址。

## 获取包

首先会检查缓存中是否存在当前的依赖包，同时，将不存在的依赖包下载到缓存中。通过cacheFolder + slug + node_modules+ pkg.name来生成一个路径，判断系统中是否有该路径，来确认是不是缓存中存在相应的包。
对于不在缓存中的包，yarn会维持一个fetch队列，按照规则进行网络请求，如果下载的地址是一个file协议，或者是一个相对路径，说明该地址指向一个本地文件目录，此时就会从离线缓存中获取包，否则就从网络获取，最终通过fs.createWriteStream写入缓存目录。

## 链接包

链接包遵循扁平化原则，将项目依赖中包复制到项目的node_modules目录下。

## 构建包

如果依赖包中存在二进制文件，那么在这一步会被编译。