# Multithreaded SSH V0.1.1


## Introduction

Recently, I replaced a batch of Huawei switches and switched to using SSH for remote access, which rendered my previous
Telnet script unusable. Therefore, I spent some time rewriting a multi-threaded SSH program called Multithreaded-SSH
using Python.

This version is still not very flexible, but it can execute some basic commands. For example, it can execute "dis cur"
command combined with the "sshlog" file to backup device configurations, or perform batch password modifications.

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
        f_log.write(
            f'运行结束,共{str(len(hosts))}台主机，成功{success_count}，错误{error_count}，命令执行错误{command_error}\n')
    input("完成！按回车退出...")


if __name__ == "__main__":
    main()

```

The code is too long and is collapsed by default. 👆Click above to expand.

The code is open source on [GitHub](https://github.com/WEI-KE/Multithreaded-SSH).

I have also packaged it into an [.exe](https://github.com/WEI-KE/Multithreaded-SSH/releases) executable file. Feel free
to download it if you need it.

> The executable file packaged with PyInstaller may be flagged as malicious by security software for unknown
> reasons. If you have concerns, you can choose to package it yourself.

## Usage Instructions

1. Make sure there are two files named `host.txt` and `commands.txt` in the same directory as the program. As the names
   suggest, `host.txt` contains the addresses of remote hosts (one per line), and `commands.txt` contains the commands
   to be executed (one command per line).

2. Make sure the program has permissions to create directories and files. After execution, the program will create two
   directories and several log files:
    - log
        - year-month-day.log
        - Paramiko-year-month-day.log
    - sshlog
        - year-month-day-host-address.log

   > `year-month-day.log` is used to store the program's output logs, where you can see the execution status of the
   program.
   >
   > `Paramiko-year-month-day.log` stores the logs of Paramiko for troubleshooting SSH connections.
   >
   > `year-month-day-host-address.log` is used to store the output of each host after establishing an SSH connection.

3. When prompted, enter the device password. The password will not be displayed in the terminal.

4. The program will provide real-time output and summary while running, but it can be messy when dealing with multiple
   devices. You can check the program logs after it finishes.

## Note

- In the current version of the program (`v0.1.1`), the completion of a command is determined by the
  prompt `<device-name` or `[device-name`. This is mainly to adapt to network devices of brands like Huawei. If a
  command does not return to the prompt within 60 seconds (default time), the program will consider it as a command
  execution failure. For example, if the command requires inputting 'y', currently you can only put the combined command
  in one line, like `save \n y`. Use `\n` for line breaks. If you have other requirements, you can modify the code for
  handling command completion or wait for future updates.
  {{< admonition info >}}On some devices, the confirmation prompt times out after 30 seconds, and it will automatically
  cancel and return to the prompt. The program will capture this and consider the command execution successful. So,
  please check the sshlog logs for confirmation after the program finishes.{{< /admonition >}}

## Update Plans

Expected updates within 2 to 3 weeks:

- Add the ability to define commands for individual hosts.
- Add the ability to define termination characters for each command to adapt to different scenarios and devices.
- Improve code modularity for easier integration and use by other programs.
- Optimize code logic.
- Enhance code comments.

Not planned for updates:

- Automatic password retrieval:
    - Currently, the program requires entering the device password each time it

is executed, and it will remain so in the future. I haven't found a perfect way to store passwords. Automatic retrieval
of usernames may be considered.

## Conclusion

I'm still a beginner in the programming world, and my code may not be great. I welcome discussions in the comments
section below or direct problem submissions (issues) on GitHub.

If you find this program useful, you can give it a thumbs up below or give it a star on GitHub. Thank you for your
support.

This article was translated by ChatGPT.

---

> Author: WAKE  
> URL: https://weike.club/en/2023/06/multithreaded-ssh-v0.1.1/  

