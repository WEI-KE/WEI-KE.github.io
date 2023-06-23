# Multithreaded SSH V0.1.1使用说明


## 前言

最近换了一批华为交换机，为了过等保改用SSH远程交换机，之前写的Telnet脚本用不了了，
于是最近花了点时间用Python重新写了一个多线程SSH程序 Multithreaded-SSH。

此版本还是不太灵活，但是可以执行一些基本的命令。例如可以执行dis cur配合sshlog文件做到备份设备配置，或者批量修改密码。
<!--more-->

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-

import paramiko
import getpass
import time
import datetime
import os
import threading
import logging

success_count = 0
error_count = 0
command_error = 0


class CommandTimeoutError(Exception):
    pass


def ssh(host='localhost', port=22, username='admin', password='admin@123', timeout=20, commands='dis ver',
        command_time_out=60):
    # 配置logging记录paramiko详细日志
    logging.basicConfig(level=logging.DEBUG, filename=f'log/paramiko-{datetime.date.today()}.log', filemode='w',
                        format='%(asctime)s - %(levelname)s - %(message)s')

    log_change = []

    global success_count, error_count, command_error

    # 创建日志目录和文件
    os.makedirs('log', exist_ok=True)
    os.makedirs('sshlog', exist_ok=True)
    log_file = f'log/{datetime.date.today()}.txt'
    sshlog_file = f'sshlog/{datetime.date.today()}-{host}.txt'

    # 设置密钥验证目录和文件
    # os.makedirs('sshkey', exist_ok=True)
    # known_hosts_file = './sshkey/known_hosts'
    # with open(known_hosts_file, 'w') as f:
    #     pass

    # 创建SSH客户端
    client = paramiko.SSHClient()

    # 是否验证本地密钥
    # client.set_missing_host_key_policy(paramiko.WarningPolicy())
    # client.load_host_keys(os.path.expanduser(known_hosts_file))

    # 自动添加主机密钥
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    connected = False
    break_tag = False

    for password in password:
        try:
            # 连接SSH
            print(f'{datetime.datetime.now().strftime("%H:%M:%S.%f")[:-3]} 尝试连接{host}')
            log_change.append(f'{datetime.datetime.now().strftime("%H:%M:%S.%f")[:-3]} 尝试连接{host} ')
            # with open(log_file, 'a') as f_log:
            #     f_log.write(f'{datetime.datetime.now().strftime("%H:%M:%S.%f")[:-3]} 尝试连接{host} ')
            # logging.info(f'{datetime.datetime.now().strftime("%H:%M:%S.%f")[:-3]} 尝试连接{host} ')
            client.connect(hostname=host, port=port, username=username, password=password, timeout=timeout)
            connected = True
            print(f'{host}连接成功')
            log_change.append(f'连接成功 ')
            # logging.info('连接成功')
            # with open(log_file, 'a') as f_log:
            #     f_log.write(f'连接成功 ')

            # 创建交互式shell
            shell = client.invoke_shell()
            time.sleep(0.5)
            hostname = 0
            output = ''

            while True:
                if shell.recv_ready():
                    output += shell.recv(1024).decode('utf-8')
                    # print(f'{output}')
                    # print(output)
                    if '<' in output and '>' in output:
                        hostname = output.splitlines()[-1][1:-1]
                        print(hostname)
                        log_change.append(f'{hostname} ')
                        break
                    if hostname > 6:
                        break
                    hostname += 1
                    time.sleep(0.5)

                    # with open(sshlog_file, 'a') as f_sshlog:
                    #     f_sshlog.write(f'{hostname[1:-1]} ')
                    # logging.info(f'{hostname[1:-1]}')

            if hostname.isdigit():
                print(f'获取设备名称错误跳过{host}')
                log_change.append(f'获取设备名称错误跳过{host}\n')
                # with open(log_file, 'a') as f_log:
                #     f_log.write(f'获取设备名称错误跳过{host}\n')
                # logging.error(f'获取设备名称错误跳过{host}')
                break

            shell.send(b'screen-length 0 temporary\n')  # 设置输出长度为0，临时模式
            time.sleep(1)  # 等待命令执行完成
            output += shell.recv(1024).decode('utf-8')
            # print(output)
            with open(sshlog_file, 'a') as f_sshlog:
                f_sshlog.write(output)

            # 清空output
            output = ''

            # 执行远程命令
            for command in commands:
                break_count = 0
                command = command.replace(r'\n', '\n')
                shell.send((command + '\n').encode('utf-8'))
                time.sleep(0.5)  # 等待命令执行完成

                while True:
                    if shell.recv_ready():
                        output += shell.recv(1024).decode('utf-8')
                        # print(f'{output}')
                        if '<' + hostname in output:
                            break
                        elif '[' + hostname in output:
                            break
                    if break_count >= command_time_out:
                        with open(sshlog_file, 'a') as f_sshlog:
                            f_sshlog.write(f'{output}')
                        command_error += 1
                        raise CommandTimeoutError(f'{command}')
                    break_count += 1
                    time.sleep(1)
                with open(sshlog_file, 'a') as f_sshlog:
                    f_sshlog.write(f'{output}')
                # 写入日志文件
                # with open(log_file, 'a') as f:
                #     time.sleep(1.5)
                #     # f.write(f'Command: {command}\n')
                #     f.write(output)
                #     # f.write('\n\n')

                # 清空output
                output = ''

            print(f'执行完成')
            log_change.append(f'执行完成 ')
            # with open(log_file, 'a') as f_log:
            #     f_log.write(f'执行完成\n')
            # logging.info('执行完成')
            success_count += 1

        except paramiko.AuthenticationException:
            print(f'身份验证失败：{host}')
            log_change.append(f'身份验证失败：{host}\n')
            # with open(log_file, 'a') as f_log:
            #     f_log.write(f'身份验证失败：{host} ')
            # logging.error(f'身份验证失败：{host}')

        except paramiko.SSHException as e:
            print(f'SSH连接错误：{host}, {str(e)}')
            log_change.append(f'SSH连接错误：{host}, {str(e)}\n')
            break_tag = True
            # with open(log_file, 'a') as f_log:
            #     f_log.write(f'SSH连接错误：{host}, {str(e)} ')
            # logging.error(f'SSH连接错误：{host}, {str(e)}')

        except paramiko.ssh_exception.NoValidConnectionsError as e:
            print(f'无法连接到主机：{host}, {str(e)}')
            log_change.append(f'无法连接到主机：{host}, {str(e)}\n')
            break_tag = True
            # with open(log_file, 'a') as f_log:
            #     f_log.write(f'无法连接到主机：{host}, {str(e)} ')
            # logging.error(f'无法连接到主机：{host}, {str(e)}')

        except TimeoutError as e:
            print(f'连接超时：{host}, {str(e)}')
            log_change.append(f'连接超时：{host}, {str(e)}\n')
            break_tag = True

        except CommandTimeoutError as e:
            log_change.append(f'命令执行超时，跳过主机：{host}, {str(e)}\n')

        except Exception as e:
            print(f'未知错误：{host}, {str(e)}')
            log_change.append(f'遇到未知错误：{host}, {str(e)}\n')

        finally:
            # 关闭SSH连接
            if not connected:
                with open(log_file, 'a') as f_log:
                    for log in log_change:
                        f_log.write(log)
                log_change = []
                if break_tag:
                    break
                time.sleep(0.5)
            elif connected:
                client.close()
                print(f'关闭连接：{host}')
                log_change.append(f'关闭连接：{host}\n')
                with open(log_file, 'a') as f_log:
                    for log in log_change:
                        f_log.write(log)
                log_change = []
                break
                # with open(log_file, 'a') as f_log:
                #     f_log.write(f'关闭连接')
                # logging.info('关闭连接')
    if not connected:
        error_count += 1


def main():
    # 创建日志目录和文件
    os.makedirs('log', exist_ok=True)
    os.makedirs('sshlog', exist_ok=True)
    log_file = f'log/{datetime.date.today()}.txt'
    print(f'程序启动时间：{datetime.datetime.now().strftime("%H:%M:%S.%f")[:-3]}')
    with open(log_file, 'a') as f_log:
        f_log.write(f'程序启动时间：{datetime.datetime.now().strftime("%H:%M:%S.%f")[:-3]}\n')

    username = 'admin'
    #  = getpass.getuser('请输入用户名')
    password = getpass.getpass(prompt='请输入设备密码用"空格"分隔（最多不超过3个）:', ).split(' ')
    # password = 'admin@123'
    # password = input('请输入设备密码用"空格"分隔（最多不超过3个）:').split(' ')
    # commands = [
    #     'dis version',
    #     'dis int br',
    #     'dis cur',
    # ]

    # 读取主机地址列表
    with open('host.txt', 'r') as f:
        hosts = f.read().splitlines()
    # 读取命令
    with open('commands.txt', 'r') as f:
        commands = f.read().splitlines()

    threads = []
    for host in hosts:
        thread = threading.Thread(target=ssh, args=(host,),
                                  kwargs={'username': username, 'password': password, 'commands': commands})
        threads.append(thread)
        # time.sleep(0.2)
        thread.start()

    for thread in threads:
        thread.join()

    print(f'运行结束,共{str(len(hosts))}台主机，成功{success_count}台，错误{error_count}，命令执行错误{command_error}')
    with open(log_file, 'a') as f_log:
        f_log.write(f'运行结束,共{str(len(hosts))}台主机，成功{success_count}，错误{error_count}，命令执行错误{command_error}\n')
    input("完成！按回车退出...")


if __name__ == "__main__":
    main()

```

已开源在👉[Github](https://github.com/WEI-KE/Multithreaded-SSH)

也打包了一个[exe](https://github.com/WEI-KE/Multithreaded-SSH/releases)可执行文件。有需要的自己下载。

## 使用说明

1. 确保程序同目录下有两个文件```host.txt```和```commands.txt```顾名思义，一个放置远程主机的地址（每行一个），另一个放置要执行的命令（每行一条）
2. 确保程序有创建目录和文件的权限，程序执行后会创建两个目录和数个日志文件
    - log
        - 年-月-日.log
        - Paramiko-年-月-日.log
    - sshlog
        - 年-月-日-主机地址.log

   > ```年-月-日.log```用于保存本程序输出的日志，你可以看到简单的执行情况
   >
   >  ``` Paramiko-年-月-日.log```用于储存Paramiko执行的日志用于ssh连接出现异常调试
   >
   >  ```年-月-日-主机地址.log```用于储存ssh连接到主机后每台主机输出的内容

3. 打开程序后会提示输入设备密码，密码输入时不会显示在终端。

4. 程序运行时会实时输出一些信息及总结，但是设备多的时候会比较乱，可以结束后看程序日志。

## 注意：

- 目前版本的程序```v0.1.1```命令执行完成以```<设备名```或者```[设备名```提示符作为中断判断,主要为了适配华为等品牌网络设备。
  如果某条命令执行后不会回到提示符命令会执行失败，默认超时时间60秒。例如你要执行的命令执行后有需要输入y等，目前只能将组合命令放在一行里，如```save \n y```
  用\n代表回车，如有其他需求请自行修改判断代码， 或者等后续版本（后面有更新计划）。<br>
  {{< admonition info >}}按y确认在某些设备上超时时间是30秒，30秒后会自动取消回到提示符，然后被程序捕获，此时程序会认为命令执行成功。
  所以请在程序执行后检查sshlog日志确认。{{< /admonition >}}

## 更新计划

预计在2到3周内更新：

- 增加独立定义单条命令终止符以适配更多场景和设备
- 增加独立定义单台主机命令
- 代码更加模块化 使其更易于被其他程序调用和集成
- 优化代码逻辑

不会更新的内容：

- 自动读取密码
    - 目前是程序每次执行都需要输入设备密码，以后也是。因为没有想到完美的密码存储方式。自动读取用户名也许会加

## 结语

本人是一个在编程新手村晃荡的人 代码写的很烂，欢迎在下方评论区交流，或者直接在Github上提交问题（issues）。

如果觉得本程序还不错，可以在下方点个👍也可以去Github点个⭐，感谢支持。

---

> 作者: WAKE  
> URL: https://weiqinke.com/2023/06/multithreaded-ssh/  

