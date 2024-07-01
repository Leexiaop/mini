# npm 包的安装机制

npm 包的安装，对于一个前言程序员来说再常用不过了，就是执行命令npm install.那么当你敲了npm install这条命令后，到底做了什么呢？这就是npm 包的安装机制。

首先，在项目的根目录下敲命令npm install.

这时，就会检查config配置。获取npm的配置，其优先级为：项目级的.npmrc文件>用户级的.npmrc>全局的.npmrc>npm内置的.npmrc文件。

然后检查项目中有没有package-lock.json文件：
    +   如果有lock文件，则检查lock文件和package.json文件中声明的版本是不是一致。
        1. 如果一致，直接使用lock文件中的信息，从缓存中或者网络资源加载所需要的依赖包。
        2. 如果不一致则会根据npm的版本进行处理。
            -   npm v5.0.x版本根据package-lock.json下载
            -   npm v5.1.0-v5.4.2:当package.json声明的依赖版本规范有符合跟新版本时，会忽略package-lock.json,按照package.jsonan安装，并更新package-lock.json.
            -   v5.4.2以上，当package.json声明的依赖版本规范与package-lock.json安装版本兼容时，则根据package-lock.json安装；如果package.json声明的依赖版本规范与package-lock.json安装版本不兼容时，则按照package.json安装。并跟新package-lock.json.

如果没有package-lock.json文件，则根据package.json文件递归构建依赖树，然后，按照构建好依赖树，下载完整的以来资源。在下载时会检查是否有相关缓存。
    +   如果有，则将缓存解压到node_modules中。
    +   如果没有，则先从npm远程库下载资源，检查包的完整性。并将其添加到缓存。同时解压到node_modules中。
    +   最后生成package-lock.json 文件

构建依赖树时，当前依赖项目无论是直接依赖还是子依赖的依赖，都应该遵循扁平化原则，优先将其放置到node_modules根目录下，在这个过程中，遇到相同模块，应先判断已放置的依赖树中的模块版本是否符合对新模块版本的要求。如果符合，则跳过，不符合则在当前模块的node_modules下放置该模块。


# npm 的缓存机制

在一个中型前端项目中所依赖的npm包很可能已经是海量。如果每次都要通过网络资源获取，这个时间成本将会是非常巨大的。所以，借助缓存的思想，还是一个比较好的解决思路。

我们通过命令:`npm config get cache`命令来得到npm包的缓存目录。每台电脑的位置不一样，以实际为准。

当进入到_cacache文件夹后，就会看到三个文件，这些就是npm包的缓存内容：

+ content-v2：存放的都是二进制文件。可以将这些文件扩展为.tgz文件，解压后得到的就是npm包的资源文件。
+ index-v5：这里存放的是一些描述性文件，事实上这里存档的是content-v2文件的索引。
+ temp: 这个文件，我也不知道是干啥的

当执行npm install 的时候，会通过pacote将相应的包资源解压到node_modules下面，npm下载依赖时，会将依赖下载到缓存中。再将其解压到项目的node_modules下，pacote依赖npm-registry-fetch来下载包资源。npm-registry-fetch可以通过设置cache属性再给定的路径下根据IETF RFC7234生成缓存数据。

接着，在每次安装的时候，根据package-lock.json中存储的integrity、version、name信息生成唯一的key，这个key就是index-v5下的缓存记录，如果发现有缓存，就会找到tar包的hash值，根据hash值找到对应的tar包，再次通过pacote将对应的二进制文件解压到对应的项目node_modules下，这样就能节省网络下载资源的时间。

注：这里的npm缓存机制,是从npm v5版本开始的。之前版本的缓存模块在~/.npm文件夹中以模块名的形式直接存储，存储结构是{cache}/{name}/version.