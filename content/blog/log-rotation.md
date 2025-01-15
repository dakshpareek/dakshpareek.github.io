---
title: "Log Rotation on Production"
date: 2025-01-15
tags: ["logging"]
draft: false
---

Recently, I needed to check some logs for an action on production for an application which I had built few years back. I did that `ls -la` and I was shocked to see that log file was too large, in GBs. It was impacting application's speed and hindering performance.

Then I checked the log level and it was debug in production which should not happen until we are replicating some case.

So I thought of fixing log level and to also make sure that our log file is rotated when certain file size is met.

I knew we could handle this with application logging and with using some external libraries which manage logs effectively but I stumbled across `logrotate` which is an in-built linux utility used on production for rotating logs.

Below I'll be showing how I safely rotated logs on production.

1. **First we need to check if `logrotate` is installed in our system or not:**
  ```bash
  logrotate --version
  ```
  If not installed then install it.

2. **List down contents of the `logrotate.d` directory:**
   ```bash
   ls -l /etc/logrotate.d/
   ```
   - The `/etc/logrotate.d/` directory contains configuration files for `logrotate`.
   - Each file in this directory typically contains rotation rules for different services or applications.

3. **Display the contents of the main configuration file:**

   ```bash
   cat /etc/logrotate.conf
   ```
   - `/etc/logrotate.conf` contains global settings for `logrotate`.
   - It might include default settings and directives to include configurations from `/etc/logrotate.d/`.


Now I'll tell you how a configuration for logrotate for a application looks like

```plaintext
/home/ec2-user/rails-app/log/*.log {
    size 50M
    copytruncate
    compress
    delaycompress
    dateext
    dateformat -%Y%m%d-%H%M%S
    rotate 9999
    missingok
    notifempty
    create 640 ec2-user ec2-user
    olddir rotated
}
```

#### **Explanation of Each Directive**

- **`/home/ec2-user/rails-app/log/*.log`**

  - Specifies all files ending with `.log` in your application's log directory.

- **`size 50M`**

  - Rotates the log when it reaches 50 megabytes.

- **`copytruncate`**

  - Copies the current log file and truncates it after copying.
  - Allows the application to continue writing to the same log file without interruption.

- **`compress`**

  - Compresses rotated logs to save disk space (`gzip` by default).

- **`delaycompress`**

  - Delays compression of the most recent rotated log until the next rotation.
  - Ensures the most recent logs remain uncompressed for immediate access.

- **`dateext`**

  - Adds a date extension to the rotated log file names.

- **`dateformat -%Y%m%d-%H%M%S`**

  - Specifies the date format appended to the rotated log files.
  - Results in filenames like `production.log-20231013-153045`.

- **`rotate 9999`**

  - Keeps up to 9,999 rotated log files.
  - Practically, this means logs are retained indefinitely unless reaching this limit.

- **`missingok`**

  - Prevents errors if the log file is missing.
  - `logrotate` will skip the file and continue.

- **`notifempty`**

  - Does not rotate the log if it is empty.

- **`create 640 deploy deploy`**

  - After rotation, creates a new log file with permissions `640` and owned by user `deploy` and group `deploy`.
  - Ensures your application can continue writing to the new log file.

- **`olddir rotated`**

  - Rotated logs will be stored in rotated directory inside `/log` folder.


**To proceed further we need user and group details of application**
Which we can get by running the below.

  1. **List running processes for your application server.**

    **For Puma:**

    ```bash
    ps aux | grep puma | grep -v grep
    ```

  **Example Output for Puma:**

  ```bash
  deploy   1234  0.0  1.2 100000 25000 ?  S  Oct12   0:15 puma 3.12.0 (tcp://0.0.0.0:3000) [myapp]
  ```

  **Explanation:**

  - The first column (`deploy`) is the user running the process.

  2. **List files with details:**

    ```bash
    ls -l
    ```

    **Example Output:**

    ```bash
    drwxr-xr-x  7 deploy deploy  4096 Oct 12 12:34 app
    drwxr-xr-x 10 deploy deploy  4096 Oct 12 12:34 config
    drwxr-xr-x  5 deploy deploy  4096 Oct 12 12:34 log
    ```

    - The third column (`deploy`) is the owner.
    - The fourth column (`deploy`) is the group.
    - Files and directories are owned by the `deploy` user and group.


**Now let's proceed to create new configuration file for our application's logrotate**

```bash
sudo vi /etc/logrotate.d/rails_app
```

Replace `rails_app` with a name appropriate for your application.

```plaintext
/home/ec2-user/rails-app/log/*.log {
    size 50M
    copytruncate
    compress
    delaycompress
    dateext
    dateformat -%Y%m%d-%H%M%S
    rotate 9999
    missingok
    notifempty
    create 640 deploy deploy
    olddir rotated
}
```

**Action:**
Now, before actually executing `logrotate` we need to make sure that everything works correctly and doesn't break anything.

We will first run `logrotate` in debug mode, which displays what `logrotate` would do without actually performing any rotations.

```bash
sudo logrotate -d /etc/logrotate.d/rails_app
```

After going through output thoroughly, we will validate that everything works correctly. Then, we will proceed to execute `logrotate` for the app.

Running this command will process your new configuration file and attempt to rotate the logs according to the rules you've set.'

```bash
sudo logrotate -vf /etc/logrotate.d/rails_app
```

- **`-v`**: Verbose mode. Shows detailed output of the process.
- **`-f`**: Forces rotation, even if it's not necessary based on time or size.

**What to Look For:**

- **Original Log File**

  - The main log file (e.g., `production.log`) should still exist.
  - Its size may be reset to zero or reduced.

- **Rotated Log Files**

  - You should see new files with date extensions.
  - Example: `production.log-20231013-153045.gz`

**`Logrotate` usually runs daily, and we can configure that if we want to increase the frequency.**

**I was able to resolve log file issue using this and now log files are rotated frequently when they met condition**
