# Linux

# CPU Utilization

### Simple Consumption | vmstat command

---

1. **us** : Time spent running non-kernel code. (user time, including nice time)
2. **sy** : Time spent running kernel code. (system time)
3. **id** : Time spent idle.
4. **wa** : Time spent waiting for IO.
5. **st** : Time stolen from a virtual machine.

### Today’s CPU Activity | sar command

---

```sql
sar
Linux 2.6.9-42.0.3.ELsmp (dellbox.xyz.co.in)         01/13/2007

12:00:02 AM       CPU     %user     %nice   %system   %iowait     %idle
12:10:01 AM       all      1.05      0.00      0.28      0.04     98.64
12:20:01 AM       all      0.74      0.00      0.34      0.38     98.54
12:30:02 AM       all      1.09      0.00      0.28      0.10     98.53
```

You can display today’s CPU activity, with the help of sar command:

- **%user**: Percentage of CPU utilization that occurred while executing at the user level (application).
- **%nice**: Percentage of CPU utilization that occurred while executing at the user level with nice priority.
- **%system**: Percentage of CPU utilization that occurred while executing at the system level (kernel).
- **%iowait**: Percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request.
- **%idle**: Percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request.

**What is Nice?**

Nice processes have positive nice value (i.e. lower scheduling priority). If a normal (nice value 0) or high-priority (negative nice value) process comes along those programs will be kicked off the CPU.

### W**ho is Eating CPUs | ps command**

---

```sql
ps -eo pcpu,pid,user,args | sort -r -k1 | less
# or

ps -eo pcpu,pid,user,args | sort -k 1 -r | head -10
# for top 10

%CPU   PID USER     COMMAND
  96  2148 vivek    /usr/lib/vmware/bin/vmware-vmx -C /var/lib/vmware/Virtual...
 0.7  3358 mysql    /usr/libexec/mysqld --defaults-file=/etc/my.cnf ...
 0.4 29129 lighttpd /usr/bin/php
 0.4 29128 lighttpd /usr/bin/php
```

# Commands

---

## scp command

**Syntax**

```sql
scp <user>@<IP source>:<path source> <user>@<IP destination>:<path destination>
```

**Example**

```bash
# remote to local
scp navneet.saggu@10.33.194.198:/etc/apt.tar.gz ~/apt.tar.gz
# local to remote
```

## general commands

```sql
# For checking disk usage in human readable format
df -h 

# To check file sizes in human readable
ls -alh

# For clearing out a file
echo "" > filename.extentsion

# For appending something to a file
echo "message" >> filename.extentsion

# For overwriting into a file
echo "message" > filename.extentsion

# For writing to a file
cat >> filename.extension
# will open a box where you can type or paste the contents of the file
# press control c to cancel the input

```

## cut command

The line **`cut -d ',' -f $((column_index + 1))`** is a command that extracts a specific field (column) from each line of input. Let's break it down:

- **`cut`** is a command used for extracting sections from each line of input.
- **`d ','`** specifies the delimiter used in the input, in this case, a comma (**`,`**). This is common in CSV (Comma-Separated Values) files.
- **`f $((column_index + 1))`** specifies the field (column) to extract. **`$((column_index + 1))`** is an arithmetic expression that calculates the column index to extract. It adds 1 to the **`column_index`** variable because field numbering in **`cut`** starts from 1.

For example, if **`column_index`** is 2, the command will extract the second field (column) from each line of input, assuming the fields are separated by commas.

This is useful in the context of processing CSV files to focus on a specific column of interest.

## sed & awk commands

```cpp
# For row grepping
sed
# For column grepping
awk

```

## awk command

### Find and replace in file

```bash
# to replace in place
awk -i inplace '{gsub(/navneet/, "saloni")}' your_file.txt

# to do replacement and write to new file
awk '{gsub(/navneet/, "saloni"); print}' input.txt > output.txt
```

### Reading csv, using field separator (-F  flag)

This is a vaild awk command. This command finds out what are the unique values present in 2nd column, and the number of rows that have these respective values.

Keep in mind column is 1 indexed.

```bash
#!/bin/bash

# Replace 'your_file.csv' with the actual path to your CSV file
file_path="your_file.csv"

# Extract unique values and count using awk
awk -F',' -v col="2" '
    NR>1 {
        count[$col]++;
    }
    END {
        for (value in count)
            print "Value:", value, "Count:", count[value];
    }
' "$file_path"
```

```bash
awk -F',' '$3 == "AGRA" {print}' "nielsencity.csv"
```

```json
# For simply extracting elements from a row in a csv
awk -F ',' '{ print "update upswing_investment_journey set bank_id = " $2 " where journey_id = " $1 }' uuids.csv
```

## grep commands

```bash
# for finding strings in files

# to search in a given file / regex set of files
grep "redeem reward" ~/*.txt

# to search subdirectories recursively
grep -r "redeem reward" /home/tom/

# to suppress warning spam
grep -R 'getMyData()' ~/projects/ 2>/dev/null
# this redirects all unwanted output to 2>/dev/null
```

## less command

```sql
less <file name>
```

Use this to search through large files on Linux

Notes on using less

- use '/' to search for pattern
- use n and Shift+n to move forward and back through search results (pattern matches)
- use q to exit

## bash command

- The & symbol instructs commands to run in a background process and immediately returns to the command line for additional commands.
    
    ```cpp
    nohup  java ${JAVA_OPTS} -jar ${JAR_FILE} server configuration.yml 2> $LOG_PATH/syserr.log &
    
    echo $! > ${DAEMON_PID_FILE} && echo "Starting service (expected time: " ${START_WAIT_TIME} " sec) ..." && sleep ${START_WAIT_TIME}s
    ```
    
- .bashrc vs .bash_profile
    - When invoked as an interactive login shell, Bash looks for the `/etc/profile` file, and if the [file exists](https://linuxize.com/post/bash-check-if-file-exists/) , it runs the commands listed in the file. Then Bash searches for `~/.bash_profile`, `~/.bash_login`, and `~/.profile` files, in the listed order, and executes commands from the first readable file found.
    - When Bash is invoked as an interactive non-login shell, it reads and executes commands from `~/.bashrc`, if that file exists, and it is readable.
    - Explanation of interactive and login (not needed, just remember what’s written below)
        
        In simple terms, an interactive shell is a shell that reads and writes to a user’s terminal, while a non-interactive shell is a shell that is not associated with a terminal, like when executing a script.
        
        An interactive shell can be either login or non-login shell.
        
        A login shell is invoked when a user login to the terminal either remotely via ssh or locally, or when Bash is launched with the `--login` option. An interactive non-login shell is invoked from the login shell, such as when typing `bash` in the shell prompt or when opening a new Gnome terminal tab. ****
        
    
    Use `.bash_profile` to run commands that should run only once, such as customizing the `$PATH` [environment variable](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/) .
    
    Put the commands that should run every time you launch a new shell in the `.bashrc` file. This include your [aliases and functions](https://linuxize.com/post/how-to-create-bash-aliases/) , custom prompts, [history customizations](https://linuxize.com/post/history-command-in-linux/) , and so on.
    
    Typically, `~/.bash_profile` contains lines like below that source the `.bashrc` file. This means each time you log in to the terminal, both files are read and executed.
    
    **`if** [ -f ~/.bashrc ]; **then**
    	. ~/.bashrc
    **fi**`
    

## du vs df command

Will give human readable storage usage output for all files in your current directory

```cpp
sudo du -h --max-depth=1 2>/dev/null
```

`df` counts the file system allocated blocks, `du` use the size information of each files. A difference can have many cause:

1. Unlinked (deleted) files that are still open by application. `lsof +aL1 <filesystem>` will helps you to identfy the processes.
2. Files beneath mount points hidden to `du`. `debugfs` can helps you.
    
    ```bash
    $ sudo debugfs
    debugfs 1.42.12 (29-Aug-2014)
    debugfs:  open /dev/<the / file system  device>
    debugfs:  cd /boot
    debugfs:  ls -l
     1966081   40755 (2)      0      0    4096 26-May-2016 16:28 .
           2   40555 (2)      0      0    4096 11-May-2016 10:43 ..
     1974291  100644 (1)      0      0       0 26-May-2016 16:28 bob   <---<<< /boot/bob is hidden by /boot fs
    
    ```
    
3. [Sparse files](https://en.wikipedia.org/wiki/Sparse_file) that looks bigger than the reality. non allocated blocks are not counted by `df` but they are counted in the file size. Note that Hard links do not fools `du`

## tar vs gzip command

- tar in itself just bundles files together (the result is called a tarball) i.e. to package an entire folder (directory) into a single file
- zip applies compression as well. gzip is used to compress a file (it cannot compress a directory)

```bash
# tarring the directory
tar -cvf myfolder.tar myfolder
# tarring and zipping the directory
tar -zcvf myfolder.tar.gz myfolder

# unzipping a file
gunzip myfolder.tar.gz
# untarring a file
tar -xvf myfolder.tar
```

# Home Brew

---

This is a package manager used for linux and mac

```bash
# to install a particular version of a package
brew install openjdk@8

# to list all installed packages and their dependencies in a list format
brew deps --tree --installed
```

| term | description | example |
| --- | --- | --- |
| **Formula** | In Homebrew, a formula is essentially a script that tells Homebrew how to install a piece of software. It includes information like where to find the source code, how to compile it, and where to put the files. So, when you run **`brew install something`**, you're essentially installing a formula. | `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/foo.rb` |
| **Keg** | A keg is the installation prefix for a formula. In simpler terms, it's the directory where Homebrew installs all the files for a particular formula. When you run **`brew install something`**, it fetches the formula and installs it in a specific directory, which is the keg. | `/usr/local/Cellar/foo/0.1` |
| **Keg-only** | a **Formula** is **Keg-only** if it is not linked into the Homebrew prefix | the [**`openjdk`**](https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/openjdk.rb) formula |
| **Cellar** | all **Kegs** are installed here | `/usr/local/Cellar` |
| **Tap** | a Git repository of **Formulae** and/or commands | `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core` |
| **Bottle** | pre-built **Keg** used instead of building from source | `qt-4.8.4.catalina.bottle.tar.gz` |
| **Cask** | an [**extension of Homebrew**](https://github.com/Homebrew/homebrew-cask) to install macOS native apps | `/Applications/MacDown.app/Contents/SharedSupport/bin/macdown` |
| **Brew Bundle** | an [**extension of Homebrew**](https://github.com/Homebrew/homebrew-bundle) to describe dependencies | `brew 'myservice', restart_service: true` |

# Bash Basics

---

## Evaluating Expressions

1. **`$((column_index + 1))`**: `(())` Used for Arithmetic Expansion in Bash; evaluates the expression inside the double parentheses and substitutes the result.
2. **`$(column_index + 1)`**: `()` Used for Variable Substitution; would lead to error in this case.
3. **`column_index + 1`**: This is just a string containing the characters "column_index + 1". It's not an arithmetic operation or command substitution.