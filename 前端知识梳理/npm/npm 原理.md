# npm 原理
## 一.npm install的机制
### 1.检查npm配置
- 项目内.npmrc文件
- 用户的.npmrc文件
- 全局.npmrc文件
- npm内置.npmrc文件
### 2.检查项目中是否有 package-lock.json 文件
- 有：检查 package-lock.json 和 package.json 中声明的依赖是否一致
	- 一致：直接使用 package-lock.json 中的信息，从缓存或网络资源中加载依赖
	- 不一致：按照 npm 版本进行处理
- 没有：则根据 package.json 递归构建依赖树并扁平化。然后按照构建好的依赖树下载完整的依赖资源，检查缓存
	- 有缓存：则将缓存内容解压到 node_modules 中
	- 无缓存：-   先从 npm 远程仓库下载包，校验包的完整性，并添加到缓存，同时解压到 node_modules。最后生成 package-lock.json

![[npm-install.png]]


## 二.npm 缓存
### 1.查看缓存
```
	npm config get cache
	缓存就在`_cacache`文件下
```

### 2.清除缓存
```
	npm cache clean --force
```

### 3.缓存文件
-   content-v2：npm包的二进制文件，后最改为.tgz可查看源文件
    
-   index-v5：npm包的索引
    
-   tmp
## 三.npm 私有化部署
**npm 中的源（registry），其实就是一个查询服务**
**确保高速、稳定的 npm 服务**，而且**使发布私有模块更加安全**。除此之外，审核机制也可以**保障私服上的 npm 模块质量和安全**。

![[npm私有化部署.png]]

## 四.npm 优化依赖安装

### 1.npm之前的依赖管理
比如 A 依赖于 B，B又依赖于C。安装时会先下载A，然后将B下载到A目录下node_module/下，依次类推，会出现嵌套过深的情况
- 项目依赖树的层级非常深，不利于调试和排查问题
- 依赖树的不同分支里，可能存在同样版本的相同依赖。比如直接依赖 A 和 B，但 A 和 B 都依赖相同版本的模块 C，那么 C 会重复出现在 A 和 B 依赖的 node_modules 中
- 安装结果浪费了较大的空间资源，也使得安装过程过慢，甚至会因为目录层级太深导致文件路径太长，最终在 Windows 系统下删除 node_modules 文件夹出现失败情况

#### A 依赖于 B v1.0
![[npm包1.png]]

#### 增加C ,C依赖于 Bv2.0
![[npm包2.png]]

#### 增加D D 也依赖 B v2.0
![[npm包3.png]]

因为A先安装，所以Bv1.0在项目顶端，而Bv2.0在C,D的项目下
所以**模块的安装顺序可能影响 node_modules 内的文件结构**

#### 增加E, E依赖于Bv1.0
![[npm包4.png]]

#### 更新模块 A 为 v2.0，而模块 A v2.0 依赖了 B v2.0
-   删除 A v1.0；
    
-   安装 A v2.0；
    
-   留下 B v1.0 ，因为 E v1.0 还在依赖；
    
-   把 B v2.0 安装在 A v2.0 下，因为顶层已经有了一个 B v1.0。
![[npm包5.png]]

这时模块 B v2.0 分别出现在了 A、C、D 模块下——重复存在了
希望的结构应该如下
![[npm包6.png]]


#### E v2.0 发布了，并且 E v2.0 也依赖了模块 B v2.0
-   删除 E v1.0；
    
-   安装 E v2.0；
    
-   删除 B v1.0；
    
-   安装 B v2.0 在顶层 node_modules 中，因为现在顶层没有任何版本的 B 了。

![[npm包7.png]]

可以明显看到出现了较多重复的依赖模块 B v2.0。可以删除 node_modules，重新安装，利用 npm 的依赖分析能力，得到一个更清爽的结构。
更优雅的方式是使用 npm dedupe 命令，Yarn 在安装依赖时会自动执行 dedupe 命令
![[npm包8.png]]

## 五.npm 常用命令
### 1.npm update: 包升级
npm update 包
- 版本说明
	- ~1.2.3：匹配最新的小版本依赖包，如1.2.x，不包括1.3.x ^4d9d86
	- ^1.2.3: 匹配最新的大版本包，如：1.x.x，不包括：2.x.x
	- -1.2.3：匹配最新的依赖包x.x.x

### 2.npm link：将本地包。链接到全局，方便调试
比如本地包开发升级，避免频繁的发布调试，可以使用npm link

npm-test：项目，依赖于package-test
package-test： npm 包

- package-test目录下执行 npm link，将包链接到全局
- npm-test目录下执行  npm link package-test，链接全局包

### 3.npx的作用
- 解决了 npm 的一些使用快速开发、调试，以及项目内使用全局模块的痛点
	- 传统模式使用ESlint
		1.  npm install eslint --save-dev
		2.  /node_modules/.bin/eslint --init
		3.  ./node_modules/.bin/eslint yourfile.js
	- npmx
		1.  npx eslint --init
		2.  npx eslint yourfile.js
- npx 执行模块时会优先安装依赖，但是在安装执行后便删除此依赖，这就避免了全局安装模块带来的问题
### 4.npm 配置的优先级

![[npm配置的优先级.png]]

### 5.npm ci
**在 CI 环境使用 npm ci 代替 npm install，一般会获得更加稳定、一致和迅速的安装体验**。
- 项目中必须存在 package-lock.json 或 npm-shrinkwrap.json；
-   npm ci 完全根据 package-lock.json 安装依赖，这可以保证整个开发团队都使用版本完全一致的依赖；
    
-   正因为 npm ci 完全根据 package-lock.json 安装依赖，在安装过程中，它不需要计算求解依赖满足问题、构造依赖树，因此安装过程会更加迅速；
    
-   npm ci 在执行安装时，会先删除项目中现有的 node_modules，然后全新安装；
    
-   npm ci 只能一次安装整个项目所有依赖包，无法安装单个依赖包；
    
-   如果 package-lock.json 和 package.json 冲突，那么 npm ci 会直接报错，并非更新 lockfiles；
    
-   npm ci 永远不会改变 package.json 和 package-lock.json。

## 六.Yarn
### 1.Yarn的特点
- 确定性:通过 yarn.lock 等机制，保证了确定性。即不管安装顺序如何，相同的依赖关系在任何机器和环境下，都可以以相同的方式被安装。在 npm v5 之前，没有 package-lock.json 机制
- 速度更快：Yarn 采用了请求排队的理念，类似并发连接池，能够更好地利用网络资源；同时引入了更好的安装失败时的重试机制。npm是按队列顺序按安装。
- 采用缓存机制，实现了离线模式：npm 目前也有类似实现
- 采用模块扁平安装模式：将依赖包的不同版本，按照一定策略，归结为单个版本，以避免创建多个副本造成冗余（npm 目前也有相同的优化）
### 2.命令比较
![[npm,yarn命令比较.jpg]]