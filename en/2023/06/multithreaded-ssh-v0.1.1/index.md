# Multithreaded SSH V0.1.1


## Introduction

Recently, I replaced a batch of Huawei switches and switched to using SSH for remote access, which rendered my previous
Telnet script unusable. Therefore, I spent some time rewriting a multi-threaded SSH program called Multithreaded-SSH
using Python.

This version is still not very flexible, but it can execute some basic commands. For example, it can execute "dis cur"
command combined with the "sshlog" file to backup device configurations, or perform batch password modifications.

<!--more-->


The code is open source on 👉 [GitHub](https://github.com/WEI-KE/Multithreaded-SSH).

I have also packaged it into an [.exe](https://github.com/WEI-KE/Multithreaded-SSH/releases) executable file. Feel free
to download it if you need it.

Currently, the program is not available in English, but there might be an English version in the future.

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
    - Currently, the program requires entering the device password each time it .is executed, and it will remain so in
      the future. I haven't found a perfect way to store passwords. Automatic retrieval of usernames may be considered.

## Conclusion

I'm still a beginner in the programming world, and my code may not be great. I welcome discussions in the comments
section below or direct problem submissions (issues) on GitHub.

If you find this program useful, you can give it a 👍 below or give it a ⭐ on GitHub. Thank you for your
support.

This article was translated by ChatGPT.

---

> Author: WAKE  
> URL: https://weike.club/en/2023/06/multithreaded-ssh-v0.1.1/  

