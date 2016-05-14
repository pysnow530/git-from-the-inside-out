Translated from <http://maryrosecook.com/blog/post/git-from-the-inside-out>.

## git内部实现原理

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
        
`.git`目录及内容是由Git创建的。其它文件被称作working copy，由用户创建。

### 添加文件

### 创建提交

#### 创建树图

#### 创建提交对象

#### 将当前分支指向新提交

### 创建第二个提交

### 检出提交

### 创建分支

### 检出分支

### 检出与working copy不兼容的分支

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
