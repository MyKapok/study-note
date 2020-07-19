# Git

版本控制是一种在开发过程中用于管理我们对文件、目录或工程的修改历史，方便查看更改历史记录，备份一边恢复以前的版本的软件工程技术。

## 版本控制

### 本地版本控制

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709140145294.png" alt="image-20200709140145294" style="zoom: 67%;" />

### 集中版本控制

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709140317749.png" alt="image-20200709140317749" style="zoom:67%;" />

### 分布式版本控制

​	 <img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709140428380.png" alt="image-20200709140428380" style="zoom:67%;" />

## 基本Linux命令

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709141013608.png" alt="image-20200709141013608" style="zoom:67%;" />

## Git的理论知识

git四个工作区域：工作目录（working Directory），暂存区(Stage/Index)，本地仓库(Repository)，远程仓库(Remote Directory);

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709142730005.png" alt="image-20200709142730005" style="zoom:67%;" />

Workspace:工作区，就是存放项目代码的地方

Index/Stage：暂存区，用于临时存放改动，事实上它是一个文件，保存即将提交到文件列表信息

Repository：仓库，就是安全存放数据的位置，这里面有你提交的所有版本数据。其中HEAD指向最新放入仓库的版本。

Remote：远程仓库，托管代码的服务器，可以简单地认为是项目组合中的一台电脑用于远程数据交换。

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709143356775.png" alt="image-20200709143356775" style="zoom:67%;" />

git 的工作流程：

1、在工作目录中添加，修改文件

2、将需要进行版本管理的文件放入暂存区；

3、将暂存区的文件提交到git仓库。

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709143751652.png" alt="image-20200709143751652" style="zoom:67%;" />![image-20200709143854396](C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709143854396.png)

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709143909995.png" alt="image-20200709143909995" style="zoom:67%;" />

## Git文件操作

### 文件4种状态

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709144647216.png" alt="image-20200709144647216" style="zoom:150%;" />

### 忽略文件

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709154830171.png" alt="image-20200709154830171" style="zoom:150%;" />

## Idea 集成Git



## git 常用命令

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709165821017.png" alt="image-20200709165821017" style="zoom:150%;" />

## SSH协议

安全通信方式的协议；

使用ssh协议通信时，推荐使用密钥的验证方式。

+ 你必须为自己创建一对密钥，并把公用密匙放在需要访问的服务器上。
+ 如果你要连接到ssh服务器上，客户端软件就会向服务器发出请求，请求用你的密匙进行安全验证。
+ 服务器收到之后，现在服务器上你的主目录下寻找你的公用密匙，然后进行比较，如果一样，服务器就用公用密匙加密并将其发送给客户端软件。
+ 客户端收到之后就可以用你的私人密匙解密再发送给服务器

 ![image-20200709212251649](C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709212251649.png)

![image-20200709212318095](C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200709212318095.png)

## 分支的概念

当创建一个本地仓库的时候，就会有一个master，默认head指向master。

每个分支里面互不影响，互不干扰，可以合并，删除分支。

  