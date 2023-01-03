# bypassuac、administrator->system系统服务免杀提权小工具

bypassuac与administrator->system系统服务免杀提权的小工具，可过Defender、360、卡巴等

**请勿使用于任何非法用途，由此产生的后果自行承担。**

windows平台下通用（不包含内核漏洞、特定条件的提权）的提权手段：

* 提权至administrator：bypassuac，可用COM组件提权( https://github.com/0xlane/BypassUAC )，编译成exe后改一改直接执行或者用execute-assembly执行均可免杀Defender、360等。

* administrator提权至system：getsystem、父进程欺骗( https://github.com/Yihsiwei/admin2system )、偷令牌、系统服务、计划任务

## administrator->system系统服务提权

administrator提权至system的前三种提权方式由于都要与令牌/新起高权限进程相关，会被360严格监控（应该是因为360对CreateProcessWithTokenW等api进行了严格的监控）。而系统服务和计划任务并没有起子进程因此不会被监控。

关于烂土豆提权、令牌相关提权绕过360的手段可参考 https://paper.seebug.org/1953/ 。该文章用高权限的Token跑一个特权线程，再用这个特权线程来执行shellcode，不过并不是十分方便。

系统服务是通过注册服务（服务是system权限），而计划任务通过写入system权限的计划任务( https://github.com/H4de5-7/schtask-bypass )。

系统服务提权cs中也实现了，即elevate里面的svc-exe，不过上传的路径与exe本身均不免杀。

这里参考 https://github.com/pandolia/easy-service 写了个极其简单的demo。

系统服务提权的思路如下：

1、上传可注册系统服务的service.exe，要注意常规exe是无法注册的，该service.exe其实是一个外壳，只有一个作用就是在注册服务后开启新的进程，该进程为我们指定的真正需要提权的exe。

2、注册服务、开启服务、此时真正的需要被提权的exe已经执行了、停止服务、删除服务

3、删除注册系统服务的service.exe

不过系统服务提权有一个弊端就是需要落地service.exe，不像计划任务可以做成反射型dll来不落地执行。

### 使用的方法

1、编译exe（名称最好不要和常规服务名字很接近，要不360会误以为在修改系统原本的服务），p = "flag"中的flag可以替换成任意的内容（为了混淆下）

2、上传到目标主机上面，最好不要上传到temp目录下或者被360监控的系统服务路径下

3、以administrator权限顺序执行以下的命令

sc create Service名称 binpath="service.exe路径 真正需要提权的exe的路径"

sc start Service名称

（如果start成功的话需要被提权的exe已经执行成功了）

sc stop Service名称

sc delete Service名称

4、删除注册系统服务的exe

## bypassuac提权

COM组件提权：对 https://github.com/0xlane/BypassUAC 进行了一点小的修改进而武器化利用

### 使用的方法

1、编译exe

2、执行格式为

bypassuac.exe 需要提权的exe的路径 exe的相关参数

也可以execute-assembly不落地执行





