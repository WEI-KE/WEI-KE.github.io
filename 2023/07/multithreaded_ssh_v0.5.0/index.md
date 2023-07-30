# Multithreaded_SSH V0.5.0


Multithreaded_SSH V0.5.0 现已发布。

<!--more-->

代码已开源在👉[Github](https://github.com/WEI-KE/Multithreaded-SSH)

也打包了一个[.exe](https://github.com/WEI-KE/Multithreaded-SSH/releases)可执行文件。有需要的自己下载。

> 使用pyinstaller打包的可执行文件，不知道为什么360会报毒，如果不放心就自己打包。

## 更新说明:

1. 增加了数字菜单。
2. 增加excel读取功能。现在可以从excel文件中读取主机地址、用户名、命令、终止符。
3. 代码更加模块化。
4. 优化了代码逻辑。

## 使用说明:

1. 确保程序在当前目录有创建目录和文件的权限，程序执行时会创建两个目录和若干日志文件。
    - log
        - 年-月-日.log
        - Paramiko-年-月-日.log
    - SSH_log
        - 年-月-日-主机地址.log

   > ```年-月-日.log```用于保存本程序输出的日志，你可以在这里看到本程序的执行情况。
   >
   >  ``` Paramiko-年-月-日.log```储存Paramiko执行的日志用于ssh连接出现异常调试。
   >
   >  ```年-月-日-主机地址.log```用于储存ssh连接到主机后每台主机输出的内容。

2. 选择模式：
    - 1 txt 模式：
     
      请确保程序同目录下有两个文件```host.txt```和```commands.txt```
      ，一个放置远程主机的地址（每行一个），另一个放置要执行的命令（每行一条）。

    - 2 excel 模式：
     
      请确保程序同目录下有```SSH.xlsx```
      文件，或者手动输入文件名称。excel文件格式可以参考github上的[示例文件](https://github.com/WEI-KE/Multithreaded-SSH/blob/main/SSH.xlsx)
      ,详细要求如下：

        - 标题需要包含```Hostname```、```Address```、```Username```、```Multi_SSH_Enable```、```cmd1```，可选：```end*```。
          标题无顺序要求。

        - 命令序号连续增加，不要跳过数字。如```cmd1```,```cmd2```,```cmd3```,```cmd5```，在执行时cmd5及以后的内容将被抛弃
          ，end序号可以不连续，end序号是对应相同命令序号的执行终止符。作为标题同样无排列顺序要求。

      | Hostname    | Address | Username | Multi_SSH_Enable                        | cmd1 | end1    |
      |-------------|---------|----------|-----------------------------------------|------|---------|
      | 主机名<br>可以留空 | 主机地址    | 用户名      | 启用标记<br/>True 启用<br/>False 禁用<br/>留空为启用 | 命令   | 命令执行终止符 |

   > 如果你是使用源代码请确保目录下有```excel_read.py```。

3. 选择模式后会提示输入设备密码，密码输入时不会显示在终端。

4. 程序运行时会实时输出一些信息及总结，但是设备多的时候会比较乱，可以结束后看程序日志。

## 注意

1. 此版本程序虽然解决了[```v0.1.1```](https://weike.club/2023/06/multithreaded-ssh-v0.1.1/)
   中[不能自定义结束符](https://weike.club/2023/06/multithreaded-ssh-v0.1.1/#%E6%B3%A8%E6%84%8F)
   的问题，但依然保留了在命令中用```\n```代表回车的能力。
2. 在txt模式下，或者excel中没有填写主机名，连接到主机后会尝试用```<```和```>```中间的字符定义为主机名。获取失败将跳过主机。
3. 在txt模式下，或者excel中没有填写end的情况下默认以```<设备名```或者```[设备名```提示符作为中断判断。

## 更新计划

代码进入稳定期， 近期不会有更新，除非发现问题，或有小的修改。

可能会增加配置文件方便使用。

## 结语

写代码的时候发现python支持用中文作为变量所以便有了这个[整活的版本](https://github.com/WEI-KE/Multithreaded-SSH/tree/main/%E4%B8%AD%E6%96%87%E7%89%88)

欢迎在下方评论区交流，或者直接在Github上提交问题（issues）。

如果觉得本程序还不错，可以在下方点个👍也可以去[Github](https://github.com/WEI-KE/Multithreaded-SSH)点个⭐，感谢支持。

---

> 作者: WAKE  
> URL: https://weike.club/2023/07/multithreaded_ssh_v0.5.0/  

