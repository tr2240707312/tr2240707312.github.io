---
title: Hexo + GitHub完成个人博客部署
date: 2025-07-07 20:41:49
excerpt: 本文详细阐述了通过Hexo + GitHub部署个人在线博客的全部流程，有兴趣的朋友可以跟着动手尝试搭建属于自己的个人博客！
sticky: 
tags:
  - 博客搭建
  - 域名申请
categories:
  - [学习,环境部署]
---

`通常在线上部署个人博客需要租借云服务器，而通过借助Node.js特性以及GitHub Page的帮助，我们利用Hexo（一个基于 Node.js 的快速、简洁且高效的静态网站生成器）完成本地博客与静态资源的映射，将项目托管给GitHub，从而省下一笔租借云服务器的费用，Hexo的丰富生态也使得博客页面有了较大的定制化空间，这是我们选择使用Hexo + GitHub搭建个人博客的重要原因！`

# 1. 环境准备工作

**安装nvm，node.js，git**

> nvm是node.js版本管理工具（可以理解为node.js版的conda）；node.js使得JavaScript可以在服务端运行而不仅仅在浏览器，使得其可以用于开发后端服务，npm是node.js的包管理工具，类似于python环境中的pip；git用于与GitHub建立连接，通过配置密钥可以省略身份验证阶段，安全性更高的同时提高效率

## 1.1 nvm与node.js安装

nvm安装查询相关资料进行傻瓜式安装即可，当在命令提示符输入以下命令显示版本号即安装成功

```bash
nvm version
```

node.js可通过nvm直接安装，通过[node.js版本](https://nodejs.org/en/about/previous-releases) 查询可用版本，安装命令如下：

```bash
# 安装v18的node.js
nvm install 18
# 查询nvm拥有的node.js环境
nvm list
```

<img src="./static/img/hexo/1.png" alt="image-20250623164606527" style="zoom: 50%;" />

```bash
# 启用指定版本的node.js环境
nvm use 18.20.8
```

## 1.2 git配置

### 代理配置

> 访问GitHub时常会出现访问延时的情况，建议自己找到梯子，为git配置代理服务器

梯子确保可用后，在系统此处可查看代理服务器地址

<img src="./static/img/hexo/2.png" alt="image-20250629150439961" style="zoom: 50%;" />

```bash
# 通过如下方式配置代理服务器地址
git config --global http.proxy 'http://代理服务器地址:端口'
git config --global https.proxy 'https://代理服务器地址:端口'
# 通过如下方式查询是否配置成功
git config --global --get https.proxy
git config --global --get http.proxy
```

### 密钥配置

打开git bash并输入如下命令

```bash
ssh-keygen -t rsa -C "你的邮箱地址"
```

命令完成后若 `C:\Users\你的本机用户名\.ssh` 目录下拥有如下文件，说明本地私钥创建成功

<img src="./static/img/hexo/3.png" alt="image-20250623165204572" style="zoom:67%;" />

启动 SSH 代理（ssh-agent）并将私钥添加到代理中

```bash
eval "$(ssh-agent -s)" 
ssh-add ~/.ssh/id_rsa
```

查看该私钥对应的公钥（以ssh-rsa开头，以邮箱地址结尾）

```bash
# 查询私钥对应的公钥
cat ~/.ssh/id_rsa.pub
```

<img src="./static/img/hexo/4.png" alt="image-20250623165644488" style="zoom:67%;" />

打开GitHub进入个人Setting界面，将查询到的公钥信息复制并添加

<img src="./static/img/hexo/5.png" alt="image-20250623165928605" style="zoom:50%;" />

使用如下命令验证该环节是否成功

```bash
ssh -T git@github.com
```

<img src="./static/img/hexo/6.png" alt="image-20250623170124316" style="zoom:67%;" />

## 1.3 Hexo基本配置

Hexo 是一个基于 Node.js 的快速、简洁且高效的静态网站生成器。它允许你使用 Markdown 或其他标记语言编写文章，然后通过简单的命令将其转换为静态 HTML 页面，适合用于搭建博客、项目文档、个人网站等。

```bash
# 直接通过npm进行安装
npm install -g hexo-cli
# 初始化Hexo 项目，自动创建一个完整的博客目录结构，并安装必要的依赖文件
hexo init
# 安装git自动部署发布hexo插件
npm install hexo-deployer-git --save
# 启动本地服务器，验证hexo是否安装成功
hexo server
```

<img src="./static/img/hexo/7.png" alt="image-20250623171307314" style="zoom:67%;" />

<img src="./static/img/hexo/8.png" alt="image-20250623171331262" style="zoom:50%;" />

# 2. 利用GitHub管理

## 2.1 建立仓库，创建分支

1. 进入GitHub官网，创建一个新的仓库，仓库名称为“GitHub账号名.github.io”

   > 名称必须符合上述规则，否则可能会出错

   <img src="./static/img/hexo/9.png" alt="image-20250629204150588" style="zoom: 50%;" />

   ![image-20250629201442687](./static/img/hexo/10.png)

2. 将该项目克隆至本地（点进仓库即可看见克隆地址）

   <img src="./static/img/hexo/11.png" alt="image-20250629204842349" style="zoom:67%;" />

   在自定义的路径下，使用git工具完成项目克隆，未来在此项目下完成博客编写

   ```bash
   git clone 上述地址👆
   ```

3. 创建“code”分支，用于未来管理源码（否则当本地项目丢失后无法还原博客数据）

   ```bash
   # 本地创建code分支并完成分支切换（本地默认使用main分支）
   git checkout -b "code"
   ```

   <img src="./static/img/hexo/12.png" alt="image-20250629203023711" style="zoom: 67%;" />

   将本地分支推送至于远程，首次推送分支至远程需要本地内容有改动

   ```bash
   # 创建一个空文件（后续不需要可手动删除）
   touch init.md
   # 将当前目录下的文件添加进暂存区
   git add .
   # 将暂存区内容纳入版本库管理
   git commit -m "code分支初始化"
   # 推送新分支
   git push -u origin code
   ```

   通过GitHub仓库验证推送是否成功

   <img src="./static/img/hexo/13.png" alt="image-20250629211413286" style="zoom:50%;" />

## 2.2 创建Hexo项目

1. 创建一个空文件夹，执行如下命令，完成博客项目的初始化

   ```bash
   # 初始化Hexo 项目，自动创建一个完整的博客目录结构，并安装必要的依赖文件
   hexo init
   ```

2. 打开 Hexo 项目下的 `_config.yml`

   <img src="./static/img/hexo/14.png" alt="image-20250623172228084" style="zoom:67%;" />

   拉至文件最下方，对相关内容进行修改

   ```yaml
   # Deployment
   ## Docs: https://hexo.io/docs/one-command-deployment
   deploy:
     type: git
     repository: https://github.com/GitHub账号名/GitHub账号名.github.io
     branch: main
   ```


### 2.2.1 源码维护

1. 将该文件夹的所有内容剪切至我们本地的仓库下

   > 确保此时git属于code分支（在当前目录下打开git bash，若命令行处末尾显示code即说明当前处于code分支），若处于main分支则需要切换分支后再进行剪切，后续我们使用code分支完成源码的备份，使用main分支完成项目的静态资源维护

   <img src="./static/img/hexo/15.png" alt="image-20250629212144974" style="zoom:67%;" />

2. 将源码进行备份至GitHub仓库

   ```bash
   # 将修改内容全部加入暂存区
   git add .
   # 将暂存区内容送入版本库
   git commit -m "博客源码备份1"
   # 推送至远程仓库
   git push
   ```

   此时可在 code 分支看见我们推送的数据

   <img src="./static/img/hexo/16.png" alt="image-20250629213024701" style="zoom:67%;" />

### 2.2.1 静态资源维护

1. 安装hexo-deployer-git插件

   > 由于该插件是安装在本地的，因此每一个全新的hexo项目都需要重新安装

   ```bash
   # 安装git自动部署工具
   npm install hexo-deployer-git --save
   ```

2. 执行Hexo自动部署的三个步骤

   ```bash
   # 清理 Hexo 缓存
   hexo clean   
   # 重新生成静态文件
   hexo generate  
   # 完成本地项目部署至GitHub 
   #（由于在_config.yml中配置过，因此生成的静态资源会发布至main分支）
   hexo deploy    
   ```

3. 此时可在main分支查看部署的具体资源情况

   <img src="./static/img/hexo/17.png" alt="image-20250629215128189" style="zoom:67%;" />

   此时可以通过网址访问初始页面

   ```bash
   https://你的GitHub账户名.github.io/
   ```

   <img src="./static/img/hexo/18.png" alt="image-20250629215424932" style="zoom: 50%;" />

## 2.3 后续维护

总体而言，我们利用code分支管理项目源码，利用main分支管理静态页面。

每次项目中的数据更新后，我们需要使用add，commit，push命令同步至code分支上。

利用hexo clean,hexo generate，hexo deploy命令完成静态项目的构建与自动部署，此部分内容会被同步至main分支上。

## 2.4 绑定个人域名（非必须）

> 如果不希望自己的GitHub账号暴露在网址中，可以考虑申请租借域名，[腾讯云](https://buy.cloud.tencent.com/domain/price?type=overview)阿里云等平台都提供了此项服务

### 2.4.1 个人域名申请

从经济实惠的角度考虑，选择.top后缀的域名即可，域名申请需要实名认证，可能需要半天左右的时间，具体的不再赘述，只需跟着系统指引即可

<img src="./static/img/hexo/19.png" alt="image-20250630204211025" style="zoom:67%;" />

完成域名申请后，可在个人主页查询域名的具体情况，如下👇

<img src="./static/img/hexo/20.png" alt="image-20250630204451421" style="zoom:50%;" />

### 2.4.2 绑定GitHub Page

1. 在博客项目的source文件夹下下创建“CNAME”文件，内容填写为自己的域名（如上即xxx.top，不要包含https，www以及斜杠部分）

   <img src="./static/img/hexo/21.png" alt="image-20250702094707036" style="zoom: 67%;" />

   使用add，commit，push老三样将新添的内容推送至code分支

   <img src="./static/img/hexo/22.png" alt="image-20250630205441059" style="zoom:67%;" />

2. 进入腾讯云的域名管理界面，点击需要目标域名的“解析”

   <img src="./static/img/hexo/23.png" alt="image-20250701194319708" style="zoom:67%;" />

   点击“新手快速解析”

   <img src="./static/img/hexo/24.png" alt="image-20250701194454790" style="zoom: 50%;" />

   填写自己GitHub仓库的访问地址

   <img src="./static/img/hexo/25.png" alt="image-20250701194713777" style="zoom: 50%;" />

   完成后可看到如下两条记录，对腾讯云的操作到此结束

   <img src="./static/img/hexo/26.png" alt="image-20250701194856377" style="zoom:50%;" />

3. 进入GitHub仓库对应的Settings部分，选择Page选项，并将自己的域名填入保存

   <img src="./static/img/hexo/27.png" alt="image-20250630205714086" style="zoom:67%;" />

   当出现如下标志时，说明域名与该项目成功绑定

   <img src="./static/img/hexo/28.png" alt="image-20250701195148565" style="zoom:67%;" />

4. 使用绑定的域名进行访问，成功跳转至我们自己的项目

   <img src="./static/img/hexo/29.png" alt="image-20250701195253497" style="zoom:67%;" />

# 3. 从使用的角度出发认识Hexo

在Hexo生成的初始项目下，我们着重关注如下内容👇

```bash
.
├── _config.yml # 网站的配置文件，您可以在此配置大部分的参数。
├── scaffolds	# 模版文件夹，当新建文章时，Hexo会根据scaffold存储模板来创建文件。
├── source	# 资源文件夹，用于存放用户资源
|   ├── _drafts	# 草稿文件，自动部署时此部分不会生成静态资源
|   └── _posts	# 文章文件
```

本节将对各部分作用进行解释，并展示常见的用法（更加详细的内容请看[Hexo Fluid官方文档](https://hexo.fluid-dev.com/docs/guide/)），最后为项目换上较为流行的 Fluid 主题

## 3.1 _config.yml

该文件主要针对网页全体对象进行配置，如下👇

| 设置       | 描述                                 |
| :--------- | :----------------------------------- |
| `title`    | 网站标题（不同主题显示位置不一致）   |
| `subtitle` | 网站副标题（不同主题显示位置不一致） |

<img src="./static/img/hexo/30.png" alt="image-20250702095654608" style="zoom:67%;" />

| 设置                 | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `description`        | 网站描述（在搜索引擎搜索出来后展示的简短描述）<br /><img src="./static/img/hexo/31.png" alt="image-20250702100455980" style="zoom: 50%;" /> |
| `keywords`           | 网站的关键词，支持多个关键词【Yml列表】（一些主题可以用于站内搜索） |
| `author`             | 作者名称（网页左下角）<br /><img src="./static/img/hexo/32.png" alt="image-20250702100654273" style="zoom: 50%;" /> |
| `language`           | 网站使用的语言，默认为 `en`（即英文），中文简体为`zh-CN`     |
| `timezone`           | 网站时区， 电脑对应时区，对于中国大陆地区可以使用 `Asia/Shanghai` |
| `permalink`          | 文章的[永久链接](https://hexo.io/zh-cn/docs/permalinks)格式，设置为`blog/:categoty/:title/`即文章生成的静态页面放在`blog/分类名/文章名`目录下 |
| `permalink_defaults` | 永久链接中各部分的默认值                                     |
| `url`                | 网站的根目录，如`https://xxx.github.io`                      |
| `baseurl`            | 生成的页面中，所有相对路径（如 CSS、JS、图片）会基于`url`拼接`baserul`，因此当`baseurl`为`/`时，所有资源基于`https://xxx.github.io/`进行定位 |
| `theme`              | 主题设定，如后续切换为`fluid`主题则此处内容为`fluid`，关于主题更多的操作在后续更换主题时详细介绍 |

上述即为常用配置，需要完成更高等级的定制化请参考官方文档。

由于分类页面与标签页面默认不支持分页，因此建议再加入如下配置，使得其拥有分页功能

```yml
tag_generator:
  per_page: 10          # 标签页分页设置（关键配置）
  order_by: -date       # 按日期降序排列
category_generator:
  per_page: 10          # 分类页单独设置分页数量
  order_by: -date       # 按日期降序排列
```

## 3.2 scaffolds文件夹

初始情况下，Hexo为我们提供了三种模板

<img src="./static/img/hexo/33.png" alt="image-20250707203104866" style="zoom:50%;" />

当创建一篇新文章时，我们需要用如下命令，这样创建出的文章会按照目录结构存放

```bash
# 其中layout取值即文件夹中的模板名称（draft，page，post）
# 根据需要选取即可，默认选取post（可通过config文件进行修改）
hexo new [layout] <title>
```

`post.md文`件内部格式如下，即用该模板创建的文章会显式记录当前的标题以及文件创建日期，标签为空，后续可自由更改

```markdown
---
title: {{ title }}
date: {{ date }}
tags:
---

```

创建后的文件存放在`source`文件夹的`_post`目录下

<img src="./static/img/hexo/34.png" alt="image-20250707204427194" style="zoom:67%;" />

创建出的初始内容如下，`title`部分为传入参数，日期为创建时的日期。这部分信息在`Hexo`中称为[Front-matter](https://hexo.io/zh-cn/docs/front-matter)，更多参数详情请见官方文档，此处仅介绍用法

<img src="./static/img/hexo/35.png" alt="image-20250707204704275" style="zoom:67%;" />

当需要为文章划定分类或者添加标签时，在`Front-matter`中使用`tags`或`categories`关键字进行标签或者分类管理

```yaml
# 说明当前文章有[日常,美食]两种标签，标签之间层次相同，因此顺序不重要
# 在URL上为：xxx/blog/tags/日常/post-title
tags:
  - 日常
  - 美食
```

而分类之间根据写法的不同会有不同的层次关系

**写法一**

```yaml
# 此时Tennis与Baseball属于同一层级，当前文章同时隶属于这两个分类
# 在URL上为：xxx/categories/Tennis/post-title
categories:
  - Tennis
  - Baseball
```

**写法二**

```yaml
# 这种写法说明Baseball是Baseball是Sports的子类
# 在URL上为：xxx/categories/Sports/Baseball/post-title
categories:
  - [Sports,Baseball]
```

`个人认为最开始每篇文章的categories控制在一个即可，否则模糊了标签与分类的界限，后续熟悉了或者感觉确实有需要再进行操作。`

## 3.3 source文件夹

> 使用Typora作为Markdown编辑器即可，面对初始化主题或许有些不适配，但是当后续换成第三方提供的主题，如Fluid时，几乎完美适配，唯一的区别就是在文章顶部需要保留`Front-matter`的内容用于指定文章的标题，标签，分类等信息
>
> 此外关于图片信息，如果有条件的话建议利用PicGo将其存储到云空间，这样可以使得项目更为轻量级，具体可以参考[Typora+PicGo+腾讯云COS搭建图床](https://blog.csdn.net/xk1835217729/article/details/123958269)

+ `_post`目录下存放的为使用`hexo new post xxx`创建的文章

+ `_drafts`目录下存放的为使用`hexo new draft xxx`创建的草稿文章

+ `yourname`目录下存放的为使用`hexo new page yourname`创建的文章

  > 这部分创建的文件夹为自定义的`yourname`，文章名为`index.md`，不在博客正文中出现，但是可能在博客的“关于”界面需要使用，如后续部署的fluid主题就需要使用`hexo new page about`生成一个页面单独显示，具体在后续布置主题时介绍

## 3.4 Fluid主题安装

> Hexo原生主题十分老久，市面上存在大量开源主题，[Butterfly](https://butterfly.js.org/)，[NextX](https://theme-next.js.org/)，[Fluid](https://hexo.fluid-dev.com/)等，更多的主题可以参考官方文档或者直接咨询大模型，挑选自己喜爱的一款即可

各主题的迁移大同小异，此处以`Fluid`为例

1. 在博客目录下打开命令行，利用`npm`安装`fluid`主题

   ```bash
   npm install --save hexo-theme-fluid
   ```

   在`node_modules`文件夹下看见`hexo-theme-fluid`即安装成功

   <img src="./static/img/hexo/36.png" alt="image-20250707223123613" style="zoom:67%;" />

2. 进入`hexo-theme-fluid`文件夹，复制其`_config.yml`配置文件并将其重命名为`_config.fluid.yml`，移动至项目根目录

   <img src="./static/img/hexo/37.png" alt="image-20250707223340557" style="zoom:50%;" />

3. 打开`_config.yml`进行配置

   ```yaml
   # 将主题进行修改
   theme: fluid
   ```

   此时在本地使用命令`hexo server`后已能顺利访问到修改后的主题

   <img src="./static/img/hexo/38.png" alt="image-20250707223646403" style="zoom:50%;" />

4. 创建“关于”界面

   ```bash
   # 首次应用主题需要手动进行创建，否则该连接无内容
   hexo new page "关于本站"
   ```

   <img src="./static/img/hexo/39.png" alt="image-20250707230420537" style="zoom:50%;" />

## 3.5 更多定制化服务

一些细节的设置通过操作`_config.fluid.yml`与`_config.yml`实现，前者优先级更高，且配置文件内部几乎都有比较详尽的中文注释，根据需要进行修改即可，或参考[Hexo Fluid配置指南](https://hexo.fluid-dev.com/docs/guide/)

例如需要修改背景图片，则定位至`node_modules\hexo-theme-fluid\source\img`路径下添加自己的图片并修改配置文件路径即可，此外还有诸多插件可以用于丰富主题内容，如背景音乐等，具体的情况可在网上查询相关资料进行添加。

<img src="./static/img/hexo/40.png" alt="image-20250708102053016" style="zoom:67%;" />

<img src="./static/img/hexo/41.png" alt="image-20250708102013453" style="zoom: 67%;" />

关于背景图片固定，毛玻璃背景特效等可参考👉[Hexo fluid 全屏背景图随日夜模式切换以及正文底页毛玻璃效果](https://4rozen.github.io/archives/Hexo/60191.html)
