Translated from <http://maryrosecook.com/blog/post/git-from-the-inside-out>.

## 彻底理解Git

本文主要解释git的工作原理。如果你是一个视频党，请移步[youtube视频](https://www.youtube.com/watch?v=fCtZWGhQBvo)。

本文假设你已经有能力使用git来对项目做版本控制。我们主要考察支撑git的图结构和指导git行为的图属性。在考察原理时，我们会创建真实的状态模型，而不是通过各种实验的结果妄做猜想。通过这个真实的状态模型，我们可以更直观地了解git已经做了什么，正在做什么，以及接下来要做什么。

本文结构组织为一系列的git动作，针对一个单独的项目展开。我们偶尔会观察一下git当前状态的图结构，并解释图属性及其产生的行为。

如果你读完本文后仍意犹未尽，可以看一下[我对Git的JavaScript实现](http://gitlet.maryrosecook.com/docs/gitlet.html) (包含大量注释) 。

### 创建项目

    ~ $ mkdir alpha
    ~ $ cd alpha
    
创建项目目录`alpha`。

    ~/alpha $ mkdir data
    ~/alpha $ printf 'a' > data/letter.txt
    
进入`alpha`目录，创建目录`data`。在`data`目录下，创建内容为`a`的文件`letter.txt`。终了，`alpha`的目录结构如下：

    alpha
    └── data
        └── letter.txt

### 初始化仓库

    ~/alpha $ git init
    Initialized empty Git repository
    
`git init`命令将当前目录加到Git仓库。为此，它会在当前目录下创建一个`.git`目录并写入一些文件。这些文件记录了Git配置和版本历史的所有东西。它们都是一些普通的文件，没什么特别。用户可以使用编辑器或shell对它们进行浏览或编辑。也就是说，用户可以像编辑他们的项目文件一样来浏览或编辑项目的版本历史。

现在，`alpha`的目录结构变成了这个样子：

    alpha
    ├── data
    │   └── letter.txt
    └── .git
        ├── objects
        etc...
        
`.git`目录及内容是由Git创建的。其它文件组成工作区，由用户创建。

### 添加文件

    ~/alpha $ git add data/letter.txt
    
添加文件`data/letter.txt`到Git。此操作有两个影响。

第一，它会在`.git/objects/`目录下创建一个新的blob文件。

这个blob文件包含了`data/letter.txt`文件压缩后的内容。文件名取自内容的哈希值。哈希意味着执行一段算法，将给定内容转换为更小的，且能唯一确定原内容的值的过程。例如，Git对`a`作哈希得到`2e65efe2a145dda7ee51d1741299f848e5bf752e`。哈希值的头两个字符用作对象数据库的目录名：`.git/objects/2e/`，剩下的字符用作blob文件的文件名：`.git/objects/2e/65efe2a145dda7ee51d1741299f848e5bf752e`。

注意刚才添加文件时Git是如何把它的内容保存到`objects`目录的。即使我们从工作区把`data/letter.txt`文件删掉，它的内容在Git内仍然不会丢失。

第二，它会将`data/letter.txt`文件添加到index。index是一个文件列表，它记录有我们想要跟踪的所有文件。它保存为`.git/index`文件，每一行维护一个文件名到（添加到index时的）文件内容哈希值的映射。执行`git add`命令后的index如下：

    data/letter.txt 2e65efe2a145dda7ee51d1741299f848e5bf752e
    
创建一个内容为`1234`的文件`data/number.txt`。

    ~/alpha $ printf '1234' > data/number.txt
    
现在工作区的目录结构：

    alpha
    └── data
        ├── letter.txt
        └── number.txt
        
将`data/number.txt`文件加入到Git。

    ~/alpha $ git add data
    
`git add`命令添加一个包含`data/number.txt`内容的blob对象，然后添加一个index项将`data/number.txt`指向刚刚创建的blob对象。执行完后的index：

    data/letter.txt 2e65efe2a145dda7ee51d1741299f848e5bf752e
    data/number.txt 274c0052dd5408f8ae2bc8440029ff67d79bc5c3
    
注意，虽然我们执行的是`git add data`，但只有`data`目录内的文件被加到index。`data`目录不会被加入。

    ~/alpha $ printf '1' > data/number.txt
    ~/alpha $ git add data
    
我们原打算在`data/number.txt`内写入`1`而不是刚才的`1234`，现在修正一下，然后将文件重新加到index。这条命令会为新的内容重新生成一个新的blob文件，并更新`data/number.txt`在index中的指向。

### 创建提交

    ~/alpha $ git commit -m 'a1'
              [master (root-commit) 774b54a] a1
              
我们创建了一个提交`a1`。Git打印出此次提交的简短描述。

提交命令对应三步操作。创建提交版本对应的文件内容的树图，创建一个提交对象，然后将当前分支指向该提交。

#### 创建树图

树图记录着index内对应文件 (即项目文件) 的位置和内容，Git通过树图来记录项目的当前状态。

树图有两类对象组成：blob和tree。

blob是在执行`git add`命令时创建的，用来保存文件内容。

tree是在创建提交时产生的，一个tree代表着工作区的一个目录。

创建提交后，记录`data`目录内容的tree对象如下：

    100664 blob 2e65efe2a145dda7ee51d1741299f848e5bf752e letter.txt
    100664 blob 56a6051ca2b02b04ef92d5150c9ef600403cb1de number.txt
    
第一行记录了恢复`data/letter.txt`文件需要的所有信息。第一部分表示该文件的权限，第二部分表示该行记录的是一个blob对象，第三行表示该blob的哈希值，第四行表示文件名。

第二行是`data/number.txt`文件的信息。

下面是`alpha`目录 (项目的根目录) 的树对象：

    040000 tree 0eed1217a2947f4930583229987d90fe5e8e0b74 data
    
这仅有的一行指向`data`这棵树。

![Tree graph for the a1 commit](images/1-a1-tree-graph.png)

上图中，`root`树指向了`data`树，而`data`树指向了`data/letter.txt`和`data/number.txt`这两个blob。

#### 创建提交对象

`git commit`在创建完树图后会创建一个提交对象。提交对象只是`.git/objects/`目录下的另一种文本文件：

    tree ffe298c3ce8bb07326f888907996eaa48d266db4
    author Mary Rose Cook <mary@maryrosecook.com> 1424798436 -0500
    committer Mary Rose Cook <mary@maryrosecook.com> 1424798436 -0500
    
    a1
    
第一行指向一棵树。通过这里的哈希值，我们可以找到一个指向工作区根目录（即alpha目录）的tree对象。最后一行是提交信息。

![a1 commit object pointing at its tree graph](images/2-a1-commit.png)

#### 将当前分支指向新提交

### 创建第二个提交

### 检出提交

### 创建分支

### 检出分支

### 检出与工作区不兼容的分支

### 合并祖先提交

### 合并子孙提交

### 合并来自不同提交线的两个提交

### 合并来自不同提交线且有相同修改文件的两个提交

### 移除文件

### 拷贝仓库

### 把仓库关联到其它仓库

### 从远程仓库取回分支

### 合并FETCH_HEAD

### 从远程仓库取回并合并分支

### 克隆仓库

### 推送分支到远程仓库的检出分支

### 克隆裸仓库

### 推送分支到裸仓库

### 总结
