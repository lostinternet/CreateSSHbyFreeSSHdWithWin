# CreateSSHbyFreeSSHdWithWin
Create ssh server on Win10 by freeSSHd
在Windows系统上搭建ssh服务器
* 前言
  为什么要学习使用ssh,我觉得有且不限于以下几点：
  1、安全的登陆远程主机
  2、使用公钥密钥对，实现无密码访问
  3、通过ssh proxy实现翻墙
  
  为什么要在Windows系统上搭建ssh服务器呢？
  不得不说，Windows在中国程序员使用量还是很大的，这也算是一种悲哀吧。
  但我相信让程序员多接触一些*nix的东西，就会有越来越多的人转移到*nix阵
  营。

* 服务器配置
  工欲善其事，必先利其器。要想在Windows下用好ssh还真不简单。

  我们首先需要一个能提供ssh服务的软件，选来选去，大家都推荐FreeSSHD。
  使用后，觉得这个软件还不错。（但是使用GUI软件，必竟不是程序员的上上
  选）。FreeSSHD安装使用都比较方便。
** FreeSSHD安装
    只需要按照安装程序一步一步来就好了, 可以全部采用默认设置。
    在安装最后一步时，会提示是否把FreeSSHd设置为系统服务，这个我建议选
    是，这样每次系统启动时，都可以自动启动freeSSHd服务了。
** freeSSHd设置
   安装完成freeSSHd后，我们还需要对其进行一下设置。
   以管理员权限打开freeSSHd（Win7以上系统），打开会，会直接进入
   freeSSHd Setting界面。我们可以按照下面步骤进行设置，其它设置可以以
   后慢慢看，都能看的懂，并不复杂。
*** 用户管理（Users）
    为了让其它人能够连接ssh服务器，我们需要为其它人建立用户。
    在Users选项卡中进行用户的设置。
    1、我们先点击Add...新建一个用户。
    在新建对话框中输入用户名，选择一个种验证方式。这里有三种验证方式，
    分别是：
    NT Authentication
    Password Stored as SHA1 hash
    Public Key(SSH only)
    个人建议使用优先级, Public Key(SSH only) > Password Stored as SHA1
    hash > NT Authentication.
    如果使用NT方式，需要输入域名。
    如果使用Password方式，需要输入密码并确认。
    如果使用Public Key方式，则需要客户端提供公钥(我们后面详细介绍)。
    
    最后，好要给用户选择权限。有Shell, SFTP, Tunneling三种，对与
    Tunneling，我还不熟，不敢乱讲，我们只选择Shell与SFTP就好了。

*** SSH
    freeSSHd不只可以建立SSH服务，还可以建立telnet服务，我们只关心SSH,
    所以这一步我们设置SSH选项卡。
    首先在监听地址中，输入服务器本机IP。
    在端口号中输入监听端口，这个使用默认的22即可。
    最大连接数，是同时可以连接服务器的用户数，根据实现情况设置即可。
    空闲超时时间，是指客户端多长时间没有用户动作，就由服务器自动断开连
    接，这个也是根据实际情况来设置。
    命令Shell是指客户端登陆ssh服务器后，使用哪个shell。Windows上大家就
    默认cmd.exe吧（当然，因为我的机器上安装了git，我就使用了
    git-cmd.exe），其实用cmd.exe就可以了。
    其它一切默认即可。

*** 验证管理（Authentication）
    Authentication是用来设置FreeSSHd所有接收到哪些验证方式以及使用
    public key验证方式时公钥的存放位置。
    对于使用password与public key两种方式的验证，都有三个选项可以选，
    Disabled/Allowed/Required，Disabled代表禁止使用此种方式登陆，
    Allowed代表允许使用此方式登陆，Required代表要求使用此方式登陆。
    这时我们都设置为Allowed更为灵活。
    还有就是我们可以设置Public key验证方式的公钥存位置，这个大家记得有
    这个，我们也先使用默认位置，具休用途后面还会说到。

*** 安全的FTP（SFTP）
    建立了ssh服务器，也就可以基于ssh服务来提供SFTP了，在SFTP选项卡中，
    可以设置SFTP的访问目录。根据实现应用设置就可以了。
    
    至此，我们的SSH服务器就设置完成了。
    我们还需要在任务管理器中，重启一下FreeSSHD服务。以后，每次更改
    freeSSHd设置后，我们都应该重新启动freeSSHd服务。



* 客户端配置
   我们使用两种客户端工具来登陆ssh服务器。一种是在服务器本机，使用
   Putty模拟Windows客户端登陆ssh服务器。一种是使用Android手机上的
   Termux模拟linux客户端登陆ssh服务器。因为服务器使用"Password Stored
   as SHA1 hash"模式比较简单，我们着重介绍一下使用"Public Key"方式。
** Putty客户端（Windows）
    Putty的相关内容，大家可以自行百度，这里不多介绍。
    Putty安装完成后，会在安装目录有Puttygen.exe。这个工具是用来生成密
    钥对的。点击Generate按钮开始生成密钥，此时，用鼠标在对话框的空白区
    域划动会加快密钥对的生成（因为生成算法会根据鼠标的位置计算生成因
    子）。
    生成密钥对后，我们可以编辑密钥对注释，还可以给密钥设置密码，我们为
    了不使用密码登陆，这里就可以忽略，不设置密码。
    私钥需要保存在我们本地，Putty登陆时设置中需要使用。
    公钥的格式并不能被freeSSHd所识别，因此我们可以新建一个文件文件，保
    存文件名要与我们ssh服务器的用户名相同，这一点很重要，文件中输入引
    号中的内容"ssh-rsa "，注意有空格。我们再从公钥中
    复制出公钥，粘贴到"ssh-rsa "后面，确保公钥内容在同一行上。把这个文
    件复制到freeSSHd中设置的公钥存放目录下。
    
    现在准备工具做完了，我们可以使用Putty.exe登陆ssh服务器了，首先在
    putty.exe的session设置项中输入服务器地址IP地址，端口号与服务器设置一致即
    可。连接方式使用默认的SSH。再到Connection->SSH->Auth设置项下选择私
    钥。选好后，单击Open按钮。就会弹出Putty的控制台窗口，提示我们输入
    用户名，我们输入用户名后，回车。不需要任何密码，我们就登陆了ssh服
    务器。就可以在控制台下做任何你想做的事情。

** Termux(Linux)
    Termux是个好东西呀，把Android手机打造成笔记本不是梦，关于Termux我
    会另写博客说明，这里只用她来模拟Linux。
    在Linux终端里，使用ssh-keygen -t rsa命令生成密钥对。命令会提示输入密
    钥文件名，按回车选择默认也可以，密钥对默认保存在home/.ssh中。接着
    会提示我们输入密钥的密码，我们直接回车，不使用密码。
    生成的私钥文件没有扩展名，公钥文件名与私钥文件名相同，
    但扩展名是.pub。我们把公钥上传到ssh服务器（上传方法多种，随便哪一
    种）freeSSHd设置的公钥目录中。再公钥文件改名，去掉扩展名即可。
    
    准备工作完成，在终端里输入ssh XXX.XXX.XXX.XXX（XXX.XXX.XXX.XXX是服
    务器ip地址） 连接ssh服务器。
    我们还是不需要输入密码就登陆了ssh服务器。
    
    是不是命令行用起来比Putty的GUI更简单，所以说，命令行才是程序员的最
    爱。


* freeSSHd使用注意事项
  最后，更次强调一下几点注意事项 
  1、每次更改完freeSSHD后，都要重启一下freeSSHd服务。
  2、对于新建用户，我们使用哪种验证方式比较好。
  个人建议使用优先级, Public Key(SSH only) > Password Stored as SHA1
  hash > NT Authentication.
  3、使用public key验证方式一定要设置公钥位置，在freeSSHd Setting的Authentication选项卡
  中设置公钥目录。
  4、公钥的名称必须与登陆ssh服务器的用户名一致
  5、公钥的RSA格式：rsa-ssh XXXXXXXXXXXXXXXXXXXXXXXXXXX
  注意前面的rsa-ssh是固定内容，跟个空格，更跟随一个公钥。
  如果是使用open ssh，则生成的公钥格式就是正确的格式，但如果使用
  PuttyGen生成的公钥格式，就需要对公钥文件处理成正确的格式。详情参照上
  面的相关内容。


* 后记
  尽管在Windows下还有很多种方法可以实现ssh的一些功能，但是使用ssh才是
  程序员的应该掌握，这才是正道，不要让Windows把程序员变成傻子，我写这
  个目的，也是希望广大程序员可以通过在Windows下使用一些*nix的工具来慢
  慢的认清Windows是多么拉圾，祝大家早日投向*nix怀抱。

    
    
    


