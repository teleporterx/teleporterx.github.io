---
title: "Linux: Running processes in the background"
description: "A quick guide on how to run processes in the background in Linux."
date: 2025-01-07
categories: [
    Linux,
    Process Management,
]
tags: [
    _linux_process-management,
    background-processes,
]
# layout: default
---

## Manual Method
- Run the desired command that spawns a long lived process
```bash
uvicorn app:app --host 0.0.0.0 --port 5000
```
- Suspend process using SIGSTOP ⇒ CTRL + Z
```bash
^Z
[1]+  Stopped                 uvicorn srvr.srvr:app --host 0.0.0.0 --port 5000
```
- The job is now suspended and can be viewed using `jobs`
```bash
jobs
[1]+  Stopped                 uvicorn srvr.srvr:app --host 0.0.0.0 --port 5000
```
- Resume the process in the background allowing it to continue running without occupying the terminal. (As a child). `bg` moves the last job to the background `bg %1`, `bg %2` etc. can be used to move specific jobs to the background.
```bash
bg
[1]+ uvicorn srvr.srvr:app --host 0.0.0.0 --port 5000 &
```
- To bring the process back to the foreground use `fg`. `fg %1`, `fg %2` etc. can be used to bring specific jobs to the foreground.
```bash
fg
uvicorn srvr.srvr:app --host 0.0.0.0 --port 5000
```
- disown the process to prevent it from being killed when the terminal is closed. (detach the process from the terminal); `disown %1`, `disown %2` etc. can be used to disown specific jobs. This removes the last process from the shell's job table, which means it will no longer be associated with the terminal. Even if you close the terminal, the process will keep running.
```bash
disown
```
- If you want to stop that process after it is disowned, then you’d need to send SIGTERM to it. This is done by using kill.
```bash
kill <pid>
```
- To find the PID of the process, you can use the following:
```bash
ps aux | grep <name/command pattern>
```
- In case you want to re-attach a disowned process to the terminal, you can use reptyr or similar packages (note that reptyr is not installed by default on most systems, so you may need to install it first.): 
```bash
reptyr <PID>
```

### Redirecting output
- This is an essential step when running a process in the background. If you don't redirect the output, the process will still write to the terminal, which can be annoying.
- To redirect the output to a file, you can use the following (This will redirect both stdout and stderr to the file output.log):
```bash
command > output.log 2>&1 &
```
- If you want to redirect only stdout, you can use:
```bash
command > output.log &
```
- If you want to redirect only stderr, you can use:
```bash
command 2> output.log &
``` 

### Redirecting output to /dev/null
- If you don't want to save the output, you can redirect it to /dev/null. This is useful when you don't care about the output, or you want to suppress it.
```bash
command > /dev/null 2>&1 &
```

### One-liner to run a command in the background, suppress output and disown it
> You might, in most cases need only this. This one-liner is noteworthy as it combines all the steps mentioned above into a single command.
{: .prompt-info}
```bash
command > /dev/null 2>&1 & disown
```
---

## Using nohup
- `nohup` is a POSIX command that is used to ignore the HUP (hangup) signal. The HUP signal is, by convention, the way a terminal warns dependent processes of logout. Output that would normally go to the terminal goes to a file called nohup.out if it is not redirected.
- To run a command in the background using nohup, simply prepend the command with nohup.
```bash
nohup command &
```
- jobs, bg, fg, disown can be used with nohup as well.

---