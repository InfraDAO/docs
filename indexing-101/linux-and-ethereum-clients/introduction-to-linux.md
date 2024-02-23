# Introduction to Linux

Linux is a free and open-source operating system widely used in modern computing systems, including servers, supercomputers, mobile devices, and embedded devices. Compared to proprietary alternatives, Linux provides a more customizable and flexible computing environment, allowing users to modify and optimize the system to suit their specific needs. It also offers robust security features, making it a popular choice for web servers and other critical infrastructure.

Linux is also widely used in web3 applications due to its ability to support decentralized and distributed computing systems. With its modular and scalable design, Linux allows developers to build decentralized applications that can run on a variety of devices and architectures, making it a key component in the web3 technology stack.

This lesson will cover fundamental skills and commands for working with Linux, including creating and manipulating files and directories, managing permissions and security, and using key commands to navigate and manage the system.

## Lesson objectives

* Create and manipulate folders and files
* Create and manage permissions, security, and services
* Some helpful commands

## Create and manipulate folders and files

### ls

`ls` is a command used in Unix-like operating systems to list the files and directories in a given directory.

1.  To list the contents of the current directory:

    ```bash
    ls
    ```
2.  To list the contents of a specific directory:

    ```bash
    ls /path/to/directory
    ```
3.  To list the contents of a directory in long format (displays more information about each file):

    ```bash
    ls -l /path/to/directory
    ```
4.  To list the contents of a directory including hidden files:

    ```bash
    ls -a /path/to/directory
    ```

### mkdir

`mkdir` is a command used in Unix-like operating systems to create a new directory. It stands for "make directory." You can use it to create a new directory in the current directory or to create a new directory with a specified path.

1.  To create a new directory in the current directory:

    ```bash
    mkdir new_directory
    ```
2.  To create a new directory with a specified path:

    ```bash
    mkdir /path/to/new_directory
    ```
3.  To create a new directory with multiple nested directories:

    ```bash
    mkdir -p /path/to/new_directory/nested_directory/subdirectory
    ```

    > Note: The -p option creates all the directories in the specified path, even if some of the parent directories do not exist.
4.  To create multiple directories at once:

    ```bash
    mkdir directory1 directory2 directory3
    ```

    > Note: This will create three directories named "directory1," "directory2," and "directory3" in the current directory.

### cd

`cd` is a command used in Unix-like operating systems to change the current working directory. It stands for "change directory." You can use it to navigate to a different directory in the file system.

1.  To change to the home directory:

    ```bash
    cd
    ```
2.  To change to a specific directory:

    ```bash
    cd /path/to/directory
    ```
3.  To change to the parent directory:

    ```bash
    cd ..
    ```
4.  To change to the previous directory:

    ```bash
    cd -
    ```

    > Note: This will switch to the previous directory you were in.
5.  To change to a directory using a relative path:

    ```bash
    cd path/to/directory
    ```

    > Note: This will change to a directory relative to the current working directory.

### touch

`touch` is a command used in Unix-like operating systems to create an empty file or update the timestamp of an existing file. It stands for "touch file." You can use it to create a new file or update the modification time of an existing file without changing its content.

1.  To create a new empty file:

    ```bash
    touch filename.txt
    ```

    > Note: This will create an empty file named "filename.txt" in the current directory.
2.  To create multiple files at once:

    ```bash
    touch file1.txt file2.txt file3.txt
    ```

    > Note: This will create three empty files named "file1.txt," "file2.txt," and "file3.txt" in the current directory.
3.  To update the modification time of a file to a specific time:

    ```bash
    touch -t 202201010101.01 filename.txt
    ```

    > Note: This will update the modification time of the file "filename.txt" to January 1, 2022, at 1:01:01.01 AM.

### git

`git` is a command-line tool used for version control in software development. It allows developers to track changes to source code over time, collaborate with others on the same project, and revert to earlier versions of the code if necessary. With Git, you can create branches to experiment with new features, merge changes from different branches, and manage the history of the codebase.

1.  To initialize a new Git repository:

    ```bash
    git init
    ```

    > Note: This will create a new Git repository in the current directory.
2.  To add files to the staging area:

    ```bash
    git add filename.txt
    ```

    > Note: This will add the file "filename.txt" to the staging area, which is where you prepare changes to be committed.
3.  To commit changes to the repository:

    ```bash
    git commit -m "Commit message"
    ```

    > Note: This will commit the changes in the staging area to the repository with a message describing the changes.
4.  To create a new branch:

    ```bash
    git branch new-branch
    ```

    > Note: This will create a new branch named "new-branch" based on the current branch.
5.  To switch to a different branch:

    ```bash
    git checkout branch-name
    ```

    > Note: This will switch to the branch named "branch-name."
6.  To merge changes from one branch into another:

    ```bash
    git merge branch-name
    ```

    > Note: This will merge the changes from the branch named "branch-name" into the current branch.

### nano

`nano` is a command-line text editor for Unix-like operating systems. It allows users to create and edit text files in the terminal. With nano, you can create new files, open existing files, make changes to text, and save the changes. It is a beginner-friendly editor that is easy to use and learn.

1.  To create a new file:

    ```bash
    nano filename.txt
    ```

    > Note: This will open a new file named "filename.txt" in the nano editor.
2.  This will open a new file named "filename.txt" in the nano editor.

    ```bash
    nano existing_file.txt
    ```

    > Note: This will open the existing file named "existing\_file.txt" in the nano editor.
3.  To make changes to the text:

    ```
    Use the arrow keys to move the cursor to the location where you want to make changes. 
    Type in the text that you want to add or delete.
    ```
4.  To save changes:

    ```
    Press `Ctrl+O` to save changes to the file. Press `Ctrl+X` to exit nano.
    ```
5.  To search for text:

    ```
    Press `Ctrl+W` to search for text within the file. 
    Type in the text you want to search for and press `Enter`.
    ```
6.  To cut, copy, and paste text:

    ```bash
    Use the `Alt+6` shortcut to cut text, 
    `Alt+u` to undo, 
    `Alt+0` to copy text, 
    and `Alt+e` to paste text.
    ```

### vim

`vim` is a command-line text editor for Unix-like operating systems. It allows users to create and edit text files in the terminal. With vim, you can create new files, open existing files, make changes to text, and save the changes. It is a powerful and customizable editor that can be used efficiently with keyboard shortcuts. It is widely used by developers and system administrators.

1.  To create a new file:

    ```bash
    vim filename.txt
    ```

    > Note: This will open a new file named "filename.txt" in the vim editor.
2.  To open an existing file:

    ```bash
    vim existing_file.txt
    ```

    > Note: This will open the existing file named "existing\_file.txt" in the vim editor.
3.  To make changes to the text:

    ```
    Use the arrow keys to move the cursor to the location where you want to make changes. 
    Type in the text that you want to add or delete.
    ```
4.  To save changes:

    ```
    Press `Esc` to enter command mode, then type `:w` and press `Enter` to save changes to the file.
    Type `:q` and press `Enter` to quit vim.
    ```
5.  To search for text:

    ```
    Press `/` to enter search mode. 
    Type in the text you want to search for and press `Enter`. 
    Use `n` to go to the next match, and `N` to go to the previous match.
    ```
6.  To cut, copy, and paste text:

    ```
    Use `dd` to cut a line, `yy` to copy a line, and `p` to paste a line.
    ```

### cp

`cp` is a command used in Unix-like operating systems to copy files or directories. It allows users to create a duplicate copy of an existing file or directory. The basic syntax of the `cp` command is `cp source destination`. The `source` parameter is the file or directory that you want to copy, and the `destination` parameter is where you want to copy the file or directory to.

1.  To copy a file to another location:

    ```bash
    cp file.txt /path/to/destination/
    ```

    > Note: This will copy the file "file.txt" to the specified destination directory.
2.  To copy a file and rename it:

    ```bash
    cp file.txt newfile.txt
    ```

    > Note: This will create a copy of "file.txt" with the new name "newfile.txt" in the same directory.
3.  To copy a directory and its contents:

    ```bash
    cp -r directory/ /path/to/destination/
    ```

    > Note: This will copy the directory "directory" and its contents to the specified destination directory.
4.  To overwrite an existing file:

    ```bash
    cp -f source_file.txt destination_file.txt
    ```

    > Note: This will copy the file "source\_file.txt" to "destination\_file.txt", overwriting it if it already exists. The -r option is used to copy directories recursively, and the -f option is used to force overwriting of existing files without prompting for confirmation.

### mv

`mv` is a command used in Unix-like operating systems to move or rename files and directories. It allows users to change the location or name of an existing file or directory. The basic syntax of the `mv` command is `mv source destination`. The `source` parameter is the file or directory that you want to move or rename, and the `destination` parameter is where you want to move or rename the file or directory to.

1.  To move a file to another location:

    ```bash
    mv file.txt /path/to/destination/
    ```

    > Note: This will move the file "file.txt" to the specified destination directory.
2.  To rename a file:

    ```bash
    mv oldname.txt newname.txt
    ```

    > Note: This will rename the file "oldname.txt" to "newname.txt" in the same directory.
3.  To move a directory and its contents:

    ```bash
    mv directory/ /path/to/destination/
    ```

    > Note: This will move the directory "directory" and its contents to the specified destination directory.
4.  To overwrite an existing file:

    ```bash
    mv -f source_file.txt destination_file.txt
    ```

    Note: This will move the file "source\_file.txt" to "destination\_file.txt", overwriting it if it already exists. The `-f` option is used to force overwriting of existing files without prompting for confirmation.

### cat

`cat` is a command used in Unix-like operating systems to display the contents of a file on the terminal. It allows users to view the content of one or more files in their entirety. The basic syntax of the cat command is cat file1 file2 file3. The file1, file2, and file3 parameters represent the names of the files that you want to view the contents of.

1.  To display the contents of a single file:

    ```bash
    cat file.txt
    ```

    Note: This will display the contents of the file "file.txt" on the terminal.
2.  To concatenate multiple files into one file:

    ```bash
    cat file1.txt file2.txt > combined.txt
    ```

    Note: This will concatenate the contents of "file1.txt" and "file2.txt" into a new file named "combined.txt".
3.  To display the contents of a file with line numbers:

    ```bash
    cat -n file.txt
    ```

    Note: This will display the contents of the file "file.txt" on the terminal with line numbers added to each line.
4.  To display the contents of a file and scroll through it:

    ```bash
    cat file.txt | less
    ```

    Note: This will display the contents of the file "file.txt" on the terminal using the less command, which allows for scrolling through the file. The > operator is used to redirect the output of the cat command to a new file, and the | operator is used to pipe the output of one command to another command.

### echo

`echo` is a command used in Unix-like operating systems to print text to the terminal or redirect it to a file. It allows users to output text or variables as text strings. The basic syntax of the `echo` command is `echo [option] [string]`. The `[option]` parameter is optional and can be used to modify the behavior of the `echo` command, while the `[string]` parameter is the text that you want to print or redirect.

1.  To display a message on the terminal:

    ```bash
    echo "Hello, World!"
    ```

    > Note: This will print the message "Hello, World!" on the terminal.
2.  To redirect text to a file:

    ```bash
    echo "This is a text file." > text.txt
    ```

    > Note: This will create a new file named "text.txt" and write the text "This is a text file." to it.
3.  To append text to an existing file:

    ```bash
    echo "This is more text." >> text.txt
    ```

    > Note: This will append the text "This is more text." to the end of the existing "text.txt" file.
4.  To use a variable in an echo statement:

    ```bash
    NAME="John"
    echo "My name is $NAME."
    ```

    > Note: This will print the message "My name is John." on the terminal, using the variable $NAME in the text string. The > operator is used to redirect the output of the echo command to a new file, and the >> operator is used to append the output to an existing file.

### >

The `>` operator is used in Unix-like operating systems to redirect the output of a command to a file. It allows users to create a new file or overwrite the contents of an existing file with the output of a command. The basic syntax of the `>` operator is `command > file`. The `command` parameter represents the command whose output you want to redirect, and the `file` parameter represents the file that you want to redirect the output to.

1.  To redirect the output of a command to a file instead of displaying it on the terminal:

    ```bash
    ls -la > file_list.txt
    ```

    > Note: This will run the ls command with the -la option, which lists all files in the current directory, including hidden files, and redirects the output to a new file named "file\_list.txt".

### >>

The `>>` operator is used in Unix-like operating systems to append the output of a command to an existing file. It allows users to add new content to the end of a file without overwriting any existing content. The basic syntax of the `>>` operator is `command >> file`. The `command` parameter represents the command whose output you want to append, and the `file` parameter represents the file that you want to append the output to.

1.  To append the output of a command to an existing file:

    ```bash
    ls -la >> file_list.txt
    ```

    > Note: This will run the ls command with the -la option, which lists all files in the current directory, including hidden files, and append the output to an existing file named "file\_list.txt".

### |

The `|` (pipe) operator in Unix-like operating systems is used to redirect the output of one command to the input of another command. It allows users to chain together multiple commands to perform more complex operations without having to save intermediate results to files. The basic syntax of the `|` operator is `command1 | command2`. The `command1` parameter represents the command whose output you want to redirect, and the `command2` parameter represents the command that will receive the redirected output as input.

1.  To list all files in the current directory and then search for a specific file within that list:

    ```bash
    ls | grep specific_file
    ```

    > Note: This will list all files in the current directory and then search for the string "specific\_file" within that list.
2.  To count the number of files in the current directory:

    ```bash
    ls | wc -l
    ```

    > Note: This will list all files in the current directory and then count the number of lines in the output, which is equivalent to the number of files in the directory.
3.  To sort the output of a command alphabetically:

    ```bash
    command | sort
    ```

    > Note: This will run the command and sort the output alphabetically. The | operator can be used with any command that produces output to the console. It is a powerful tool for chaining together commands to perform more complex operations.

### rm

The `rm` command in Unix-like operating systems is used to remove or delete files and directories. The basic syntax of the `rm` command is `rm [options] [file(s)]`. The `options` parameter represents any additional options you want to use with the command, and the `file(s)` parameter represents the name(s) of the file(s) you want to remove. By default, `rm` does not prompt for confirmation before deleting files, so it's important to be careful when using this command.

1.  To remove a single file:

    ```bash
    rm filename.txt
    ```

    > Note: This will remove the file named "filename.txt" from the current directory.
2.  To remove multiple files at once:

    ```bash
    rm file1.txt file2.txt file3.txt
    ```

    > Note: This will remove the files named "file1.txt", "file2.txt", and "file3.txt" from the current directory.
3.  To remove a directory and all its contents:

    ```bash
    rm -r directory_name
    ```

    > Note: This will remove the directory named "directory\_name" and all its contents, including any files and subdirectories. The -r option is required to remove directories with rm, as directories are not removable by default. It's important to be careful when using the rm command, as it permanently deletes files and directories without any way to recover them.

### curl

`curl` is a tool used to transfer data to or from a server, using one of several supported protocols, including HTTP, FTP, and SMTP. The basic syntax of the `curl` command is `curl [options] [URL]`. The `options` parameter represents any additional options you want to use with the command, and the `URL` parameter represents the URL of the server you want to communicate with.

1.  To retrieve the contents of a web page:

    ```bash
    curl https://example.com
    ```

    > Note: This will retrieve the contents of the web page at https://example.com and display it in the terminal.
2.  To download a file from a server:

    ```bash
    curl -O https://example.com/file.txt
    ```

    > Note: This will download the file named file.txt from the server at https://example.com and save it to the current directory.
3.  To upload a file to a server:

    ```bash
    curl -F "file=@/path/to/file.txt" https://example.com/upload.php
    ```

    > Note: This will upload the file located at /path/to/file.txt to the server at https://example.com/upload.php using a POST request.

### wget

`wget` is a tool used to download files from the internet. It supports various protocols such as HTTP, HTTPS, and FTP. Its basic syntax is `wget [options] [URL]`. The `options` parameter represents any additional options you want to use with the command, and the `URL` parameter represents the URL of the file you want to download. `wget` is a powerful tool that can be used to download large files, recursively download entire directories, and continue interrupted downloads.

1.  Download a file from a URL:

    ```bash
    wget https://example.com/file.zip
    ```
2.  Download a file and save it with a different name:

    ```bash
    wget -O newfile.zip https://example.com/file.zip
    ```
3.  Download a file and continue the download if it is interrupted:

    ```bash
    wget -c https://example.com/largefile.iso
    ```
4.  Download multiple files from a URL:

    ```bash
    wget -i urls.txt
    ```

    > Note: where urls.txt is a text file containing one URL per line.
5.  Limit the download speed:

    ```bash
    wget --limit-rate=500k https://example.com/largefile.iso
    ```
6.  Download a file over FTP:

    ```bash
    wget ftp://ftp.example.com/file.zip
    ```
7.  Download a file using a proxy server:

    ```bash
    wget --proxy=on --proxy-user=user --proxy-password=pass https://example.com/file.zip
    ```

### screen

`screen` is a terminal multiplexer that allows you to create and manage multiple terminal sessions within a single window. With `screen`, you can run multiple shell instances, detach from them and reattach to them later, and share sessions with other users. Once inside a screen session, you can use various keyboard shortcuts to navigate between multiple windows, split screens, and perform other tasks. `screen` is particularly useful for remote server management or for running long-running processes that you want to keep running even if you log out of your shell.

1.  Start a new screen session with a specific name:

    ```bash
    screen -S mysession
    ```
2.  Detach from a screen session:

    ```
    Press `Ctrl-a` followed by the `d` key.
    ```
3.  Reattach to a detached screen session:

    ```bash
    screen -r mysession
    ```
4.  Create a new window within a screen session:

    ```
    Press `Ctrl-a` followed by the `c` key.
    ```
5.  Switch between windows in a screen session:

    ```
    Press `Ctrl-a` followed by the `n` key to move to the next window
    or `Ctrl-a` followed by the `p` key to move to the previous window.
    ```
6.  Split the screen horizontally:

    ```
    Press `Ctrl-a` followed by the `S` key.
    ```
7.  Split the screen vertically:

    ```
    Press `Ctrl-a` followed by the `|` key.
    ```
8.  List available screen sessions:

    ```bash
    screen -ls
    ```

### tar

The `tar` command is used for archiving and compressing files in a single file, often called a "tarball". It is a command-line utility commonly used in Linux and Unix operating systems.

1.  To create a compressed tarball of a directory and its contents:

    ```bash
    tar -czvf myfiles.tar.gz /path/to/my/directory/
    ```
2.  To extract a compressed tarball to a specific directory:

    ```bash
    tar -xzvf myfiles.tar.gz -C /path/to/extracted/directory/
    ```
3.  To add a file to an existing tarball:

    ```bash
    tar -rvf myfiles.tar /path/to/newfile.txt
    ```
4.  To extract a specific file from an existing tarball:

    ```bash
    tar -xvf myfiles.tar /path/to/file.txt
    ```
5.  To create an incremental backup of a directory using a dated tarball filename:

    ```bash
    tar -czvf myfiles-$(date +%Y%m%d).tar.gz --listed-incremental=/path/to/backup.snar /path/to/my/directory/
    ```

### unzip

The `unzip` command is used to extract files from a compressed archive in the ZIP format. Here's a short explanation:

1.  To extract all files from an archive named archive.zip:

    ```bash
    unzip archive.zip
    ```
2.  To extract a specific file named example.txt from an archive named archive.zip:

    ```bash
    unzip archive.zip example.txt
    ```
3.  To extract an archive named archive.zip into a directory named extracted\_files:

    ```bash
    unzip archive.zip -d extracted_files/
    ```
4.  To list the contents of an archive named archive.zip without extracting them:

    ```bash
    unzip -l archive.zip
    ```

### dtrx

The `dtrx` command is a tool used for extracting various archive file formats. It can automatically detect the archive type and use the appropriate extraction tool. It can also handle nested archives, and can extract files to a specific directory.

1.  To extract a single archive file named `archive.tar.gz` to the current directory:

    ```bash
    dtrx archive.tar.gz
    ```
2.  To extract a nested archive file named `nested_archive.zip` that is located within another archive file named `archive.tar.gz` to the current directory:

    ```bash
    dtrx archive.tar.gz
    cd archive
    dtrx nested_archive.zip
    ```
3.  To extract a single archive file named `archive.tar.bz2` to a specific directory named `my_folder`:

    ```bash
    dtrx -n my_folder archive.tar.bz2
    ```
4.  To list the contents of an archive file named archive.tar.gz without extracting them:

    ```bash
    dtrx -l archive.tar.gz
    ```

## Create and manage permissions, security, and services

### chmod

`chmod` is a command used in Linux/Unix to change the permissions of files or directories. It allows the user to grant or revoke read, write, and execute permissions to themselves, their group, or others. The permissions can be set using numerical values or symbols such as u (user), g (group), o (others), and a (all).

1.  Set read, write, and execute permissions for the owner, and only read and execute permissions for the group and others:

    ```bash
    chmod 755 filename
    ```
2.  Remove write permission for the group and others:

    ```bash
    chmod go-w filename
    ```
3.  Add execute permission for the owner and group:

    ```bash
    chmod ug+x filename
    ```
4.  Set read and write permissions for the owner and group, but no permissions for others:

    ```bash
    chmod ug=rw,o= filename
    ```
5.  Recursively change the permissions for all files and directories in a directory:

    ```bash
    chmod -R 755 directory
    ```

### chown

The `chown` command is used in Linux/Unix to change the owner and/or group ownership of a file or directory. This command is particularly useful when transferring files or granting permissions. It allows the administrator to specify who can read, write, or execute the file/directory.

1.  Change the ownership of a file to a specific user and group:

    ```bash
    chown user:group file.txt
    ```

    > Note: This command changes the ownership of file.txt to the user user and the group group.
2.  Change the ownership of a directory and all of its contents:

    ```bash
    chown -R user:group /path/to/directory
    ```

    > Note: This command recursively changes the ownership of the directory /path/to/directory and all of its contents to the user user and the group group.
3.  Change the ownership of a file to the current user:

    ```bash
    chown $USER file.txt
    ```

    > Note: This command changes the ownership of file.txt to the current user.
4.  Change the ownership of a file to the root user:

    ```bash
    sudo chown root file.txt
    ```

    > Note: This command changes the ownership of file.txt to the root user.

### whoami

The `whoami` command in Linux displays the current username of the user who is logged in to the system. It is a simple and useful command that can be used to verify the identity of the current user.

1.  To display the current user name:

    ```bash
    whoami
    ```
2.  To use the whoami output in a script:

    ```bash
    user=$(whoami)
    echo "Hello, $user!"
    ```

    > Note: This script will output a greeting message that includes the current user name.
3.  To check if the current user has sudo privileges:

    ```bash
    if [[ $(whoami) == "root" ]]; then echo "I have sudo privileges"; else echo "I don't have sudo privileges"; fi
    ```

    > Note: This command will check if the current user is "root" and print a message accordingly.

### adduser

The `adduser` command in Linux is used to create a new user account on the system. It prompts for the user's name and password, creates a home directory, and adds the user to the system. The command also allows for additional options such as specifying the user's group or setting a custom home directory.

1.  To add a new user account with default settings:

    ```bash
    sudo adduser username
    ```
2.  To add a new user account and set the home directory:

    ```bash
    sudo adduser --home /home/newuser username
    ```
3.  To add a new user account with a specific user ID (UID):

    ```bash
    sudo adduser --uid 1001 username
    ```
4.  To add a new user account with a specific group ID (GID):

    ```bash
    sudo adduser --gid 1001 username
    ```
5.  To add a new user account and specify additional groups:

    ```bash
    sudo adduser --groups group1,group2 username
    ```

Note: The adduser command may vary depending on the Linux distribution and version being used. It is recommended to refer to the documentation for the specific system to ensure proper usage.

### usermod

The `usermod` command in Linux is used to modify user account attributes such as username, home directory, login shell, and group membership. It is used to make changes to an existing user account.

1.  Change the username of an existing user:

    ```bash
    sudo usermod -l newusername oldusername
    ```
2.  Add a user to a secondary group:

    ```bash
    sudo usermod -a -G groupname username
    ```
3.  Remove a user from a secondary group:

    ```bash
    sudo usermod -G groupname1,groupname2 username
    ```
4.  Lock or unlock a user account:

    ```bash
    sudo usermod -L username
    sudo usermod -U username
    ```
5.  Change the default shell for a user:

    ```bash
    sudo usermod -s /bin/zsh username
    ```

### sudo

`sudo` is a command used in Linux to grant temporary root privileges to a normal user account. It allows users to execute commands with elevated privileges without logging in as the root user. The user running the `sudo` command will need to enter their own password to confirm their identity. This helps to improve security by allowing administrators to grant certain privileges to users without giving them full administrative access.

1.  Updates the list of available packages and their versions from the server:

    ```bash
    sudo apt-get update
    ```
2.  Installs a package with administrative privileges:

    ```bash
    sudo apt-get install <package_name>
    ```
3.  Restarts a system service:

    ```bash
    sudo systemctl restart <service_name>
    ```
4.  Adds a new user to the system:

    ```bash
    sudo useradd <username>
    ```
5.  Changes the file permission to allow read, write, and execute for the owner, and only read and execute for others.

    ```bash
    sudo chmod 755 <file_name>
    ```

### su

The `su` (short for "substitute user") command is used to switch to another user account or to run commands with a different user's privileges. When executed without specifying a username, it will switch to the root user. The `su` command is often used by system administrators to perform administrative tasks that require elevated privileges. It is also commonly used to temporarily switch to another user's account in order to run commands or perform tasks as that user.

1.  To switch to the root user:

    ```bash
    su -
    ```
2.  To switch to another user, such as "jdoe":

    ```bash
    su - jdoe
    ```
3.  To switch to a user without changing the current working directory:

    ```bash
    su jdoe
    ```
4.  To switch to a user and run a specific command as that user:

    ```bash
    su - jdoe -c "ls -l"
    ```
5.  To switch to a user and start a new shell session as that user:

    ```bash
    su - jdoe -s /bin/bash
    ```

### ssh

The `ssh` command is used to establish a secure shell connection to a remote server. It allows a user to log in to a remote machine and execute commands on it securely. The command uses encryption to protect the communication between the client and the server, preventing unauthorized access to sensitive information.

1.  To connect to a remote server with username and IP address:

    ```bash
    ssh username@ip_address
    ```
2.  To connect to a remote server with a specific port:

    ```bash
    ssh -p port_number username@ip_address
    ```
3.  To specify a private key file for authentication:

    ```bash
    ssh -i /path/to/private_key username@ip_address
    ```
4.  To run a command on a remote server:

    ```bash
    ssh username@ip_address command
    ```
5.  To copy a file from a remote server to a local machine:

    ```bash
    scp username@ip_address:/path/to/remote/file /path/to/local/file
    ```

### ssh-keygen

The `ssh-keygen` command is used to generate SSH keys for secure authentication between remote systems. It creates a public and private key pair that can be used to authenticate with a remote system without requiring a password. The private key is kept on the local system and the public key is added to the remote system's authorized keys file.

1.  Generate a new RSA key pair with the default settings:

    ```bash
    ssh-keygen
    ```
2.  Generate a new Ed25519 key pair:

    ```bash
    ssh-keygen -t ed25519
    ```
3.  Generate a new RSA key pair with a specific name and location:

    ```bash
    ssh-keygen -t rsa -b 4096 -C "my_key" -f /path/to/my_key
    ```
4.  Generate a new key pair and set a passphrase:

    ```bash
    ssh-keygen -t rsa -b 2048 -N "my_passphrase" -f ~/.ssh/my_key
    ```
5.  Generate a new key pair and save it to a different directory:

    ```bash
    ssh-keygen -t rsa -b 2048 -f ~/my_key -P "" -m PEM -q -O force-command="echo 'This account can only be used for SFTP'"
    ```

### ufw

`ufw` stands for Uncomplicated Firewall, which is a user-friendly command-line tool for managing firewall rules in Ubuntu and other Linux distributions. It allows users to easily configure rules to block or allow incoming and outgoing network traffic on their system.

1.  Enable the UFW firewall:

    ```bash
    sudo ufw enable
    ```
2.  Allow incoming SSH traffic:

    ```bash
    sudo ufw allow ssh
    ```
3.  Allow incoming traffic on port 80 for HTTP:

    ```bash
    sudo ufw allow 80/tcp
    ```
4.  Allow incoming traffic on port 443 for HTTPS:

    ```bash
    sudo ufw allow 443/tcp
    ```
5.  Deny incoming traffic on a specific port:

    ```bash
    sudo ufw deny 22/tcp
    ```
6.  Allow outgoing traffic to a specific IP address:

    ```bash
    sudo ufw allow out to 203.0.113.0/24
    ```
7.  Delete a specific firewall rule:

    ```bash
    sudo ufw delete allow 80/tcp
    ```
8.  Disable the UFW firewall:

    ```bash
    sudo ufw disable
    ```

### systemctl

`systemctl` is a command-line tool for controlling the `systemd` system and service manager, which is used in most modern Linux distributions. It allows the user to manage and control various system services, daemons, and targets, such as starting, stopping, enabling, or disabling them.

1.  To start a service:

    ```bash
    sudo systemctl start service-name
    ```
2.  To stop a service:

    ```bash
    sudo systemctl stop service-name
    ```
3.  To restart a service:

    ```bash
    sudo systemctl restart service-name
    ```
4.  To check the status of a service:

    ```bash
    sudo systemctl status service-name
    ```
5.  To enable a service to start at boot:

    ```bash
    sudo systemctl enable service-name
    ```
6.  To disable a service from starting at boot:

    ```bash
    sudo systemctl disable service-name
    ```
7.  To reload the configuration of a service:

    ```bash
    sudo systemctl reload service-name
    ```
8.  To list all active services:

    ```bash
    sudo systemctl list-units --type=service
    ```
9.  To check if a service is enabled:

    ```bash
    sudo systemctl is-enabled service-name
    ```
10. To check if a service is running:

    ```bash
    sudo systemctl is-active service-name
    ```

## Some helpful commands

### !!

The `!!` command is a shell shortcut that allows you to repeat the previous command. It simply executes the previous command in the command history. It is a quick way to repeat a command without having to type it out again.

1. Suppose you mistakenly entered a command as a non-root user that required root permissions. You can use `sudo !!` to quickly execute the same command as root.
2. If you want to repeat a command with slight modifications, you can use `!!` and then edit the command to suit your needs.
3. If you accidentally typed a command incorrectly, you can use `!!` to repeat the corrected version of the command.

### top

The `top` command is a Linux utility that provides real-time information about the processes that are currently running on the system, as well as information about the system's overall resource usage. It displays a dynamic, ordered list of running processes and updates it periodically. The `top` command is often used by system administrators to monitor the system's performance and identify processes that are consuming too many resources.

1. Simply run `top` to view a real-time dynamic display of the system's processes.
2. Use `top -u username` to display all processes that belong to a particular user.
3. To sort processes by CPU usage, press the "P" key. To sort by memory usage, press "M". To sort by process ID, press "N". To switch between ascending and descending order, press "R".
4. Use the `top -H` command to display individual threads of processes.
5. Use `top -p PID` to view the information of a particular process using its PID.

### htop

`htop` is a command-line utility for monitoring system resources on Linux. It is an improved version of the `top` command and provides a more user-friendly and interactive way of viewing and managing processes and system resource usage.

1. To start `htop` in interactive mode, simply run the command `htop` in the terminal.
2. To sort the processes by CPU usage, press the F6 key, select the "PERCENT\_CPU" option, and press Enter.
3. To search for a specific process, press the F3 key and enter the name of the process in the search field.
4. To kill a process, select it using the arrow keys and press the F9 key. Then, select the signal you want to send to the process (e.g. SIGTERM or SIGKILL) and press Enter.
5. To monitor a specific user's processes, press the F4 key and enter the username in the "Setup filter" field.
6. To change the color scheme, press the F2 key and select "Colors" from the menu. You can then select a pre-defined color scheme or customize your own.

### ps

The `ps` command in Linux is used to display information about the running processes on a system. It can be used to view the currently running processes, their process IDs (PIDs), and other information such as their CPU and memory usage.

1.  Display a list of all running processes with additional information:

    ```bash
    ps aux
    ```
2.  Display a process tree for all running processes:

    ```bash
    ps ef
    ```
3.  Display information for a specific process ID:

    ```bash
    ps -p 1234
    ```
4.  Display all processes owned by a specific user:

    ```bash
    ps -u username
    ```

### grep

The `grep` command is a command-line utility for searching for a specific pattern or regular expression in a file or output. It can be used to search for specific strings in a single file or across multiple files. The command returns the lines containing the search string and is commonly used in conjunction with other commands in a pipeline.

1.  Search for the pattern in the specified file and displays all lines that contain the pattern.

    ```bash
    grep pattern file.txt
    ```
2.  Search for the pattern in the specified file, ignoring case sensitivity:

    ```bash
    grep -i pattern file.txt
    ```
3.  Search for the pattern recursively in all files within the specified directory and its subdirectories.

    ```bash
    grep -r pattern /directory
    ```
4.  List all running processes and filters the output to display only the process with the specified name.

    ```bash
    ps -ef | grep process_name
    ```
5.  Search through the command history and displays only the commands that contain the specified name.

    ```bash
    history | grep command_name
    ```

### netstat

`netstat` (network statistics) is a command-line tool used to display various information about network connections, routing tables, and network interfaces on a Linux system. It can be used to view active network connections, check listening ports, display network statistics, and monitor network activity. This information can be helpful in troubleshooting network issues, monitoring network traffic, and identifying network security threats.

1.  Show all active network connections and the processes that are listening on those connections.

    ```bash
    netstat -tulpn
    ```
2.  Display the kernel routing table.

    ```bash
    netstat -r
    ```
3.  Display statistics for each protocol.

    ```bash
    netstat -s
    ```
4.  Display all active network connections, both listening and non-listening.

    ```bash
    netstat -a
    ```
5.  Display all listening network connections.

    ```bash
    netstat -l
    ```
6.  Display network addresses as numbers rather than host and domain names

    ```bash
    netstat -n
    ```
7.  Display the process ID and name of the program using each network connection.

    ```bash
    netstat -p
    ```
8.  Display a continuous stream of network connection information.

    ```bash
    netstat -c
    ```

### ifconfig

The `ifconfig` command is a network configuration tool used to display and manage network interface configuration. It shows the status of the network interfaces available on a system and can be used to configure and troubleshoot network-related problems. With the `ifconfig command`, you can view the IP address, netmask, and other network-related information about each interface on the system. You can also use it to assign an IP address, enable or disable an interface, and configure other network-related settings.

1.  To display the configuration of all network interfaces:

    ```bash
    ifconfig -a
    ```
2.  To display only the IP addresses of all network interfaces:

    ```bash
    ifconfig | grep 'inet '
    ```
3.  To configure a network interface with a specific IP address:

    ```bash
    sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0
    ```
4.  To bring up a network interface that is currently down:

    ```bash
    sudo ifconfig eth0 up
    ```
5.  To bring down a network interface:

    ```bash
    sudo ifconfig eth0 down
    ```

### ip

The `ip` command is used for configuring and displaying network interfaces and routing tables in Linux. It is a replacement for the older `ifconfig` command and provides more advanced and detailed information about network configurations. It can be used for assigning IP addresses to interfaces, setting up routes, configuring VLANs, tunnels, and bridges, and managing network namespaces.

1.  To show the IP address of a network interface:

    ```bash
    ip address show eth0
    ```
2.  To set the IP address of a network interface:

    ```bash
    ip address add 192.168.1.10/24 dev eth0
    ```
3.  To delete an IP address from a network interface:

    ```bash
    ip address del 192.168.1.10/24 dev eth0
    ```
4.  To show the routing table:

    ```bash
    ip route show
    ```
5.  To add a route to the routing table:

    ```bash
    ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0
    ```
6.  To delete a route from the routing table:

    ```bash
    ip route del 192.168.2.0/24 via 192.168.1.1 dev eth0
    ```

### iptables

`iptables` is a powerful command-line tool used to manage the Linux firewall. It allows you to set up rules and filters to control network traffic coming into and out of your system. Using `iptables`, you can create rules to accept or reject traffic based on various criteria such as the source IP address, destination IP address, protocol, and port number.

1.  Allow all incoming traffic from a specific IP address:

    ```bash
    iptables -A INPUT -s 192.168.1.100 -j ACCEPT
    ```
2.  Block all incoming traffic from a specific IP address:

    ```bash
    iptables -A INPUT -s 192.168.1.100 -j DROP
    ```
3.  Allow all traffic to a specific port:

    ```bash
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    ```
4.  Block all traffic to a specific port:

    ```bash
    iptables -A INPUT -p tcp --dport 22 -j DROP
    ```
5.  Allow all traffic on the loopback interface:

    ```bash
    iptables -A INPUT -i lo -j ACCEPT
    ```
6.  Block all traffic from a specific network interface: \`\` bash iptables -A INPUT -i eth0 -j DROP

    ```
    ```

### nftables

`nftables` is a command-line utility in Linux used to manage the Netfilter firewall subsystem. It is designed to replace the previous iptables tool. It provides a simpler and more expressive syntax, improved performance, and enhanced features for managing network traffic. Using `nftables`, you can configure firewall rules and filters, create network address translation (NAT) rules, and perform packet mangling. It allows you to filter and modify network packets based on various criteria such as IP addresses, ports, protocols, and packet content.

1.  Add a rule to the filter table of nftables that allows incoming TCP traffic on port 22 (SSH) to be accepted.

    ```bash
    nft add rule filter input tcp dport 22 accept
    ```
2.  List all the rules that are currently configured in the nftables ruleset.

    ```bash
    nft list ruleset
    ```
3.  Create a new table named nat in the nftables ruleset.

    ```bash
    nft add table nat
    ```
4.  Add a new chain named prerouting to the nat table that is triggered before the routing decision is made

    ```bash
    nft add chain nat prerouting { type nat hook prerouting priority 0; }
    ```
5.  Add a rule to the prerouting chain that redirects incoming TCP traffic on port 80 to the IP address 192.168.1.2.

    ```bash
    nft add rule nat prerouting tcp dport 80 dnat 192.168.1.2
    ```

### df

`df` (Disk Free) is a command in Linux/Unix systems used to display the amount of available and used disk space on a file system. It provides information about the total size of a filesystem, the amount of free space available, and the amount of space used by each directory and file within that filesystem. It is often used to identify file systems that are running low on disk space.

1.  Show the disk space usage information for all mounted file systems

    ```bash
    df
    ```
2.  Show the disk space usage information in a human-readable format (e.g. in gigabytes, megabytes, etc.)

    ```bash
    df -h
    ```
3.  Show the disk space usage information along with the file system type

    ```bash
    df -T
    ```
4.  Show the inode usage information for file systems

    ```bash
    df -i
    ```
5.  Shoq the disk space usage information for all file systems, including those that are not mounted yet

    ```bash
    df -a
    ```
6.  Shows the disk space usage information for a specific file system (in this example, /dev/sda1)

    ```bash
    df /dev/sda1
    ```

### openssl

`openssl` is a command-line tool used for working with secure communications over networks using SSL/TLS protocols. It provides various cryptographic functions such as generating and verifying digital signatures, creating and verifying SSL/TLS connections, generating and encrypting private keys, and more.

1.  Generate a new private key

    ```bash
    openssl genpkey -algorithm RSA -out private_key.pem -aes256
    ```
2.  Generate a self-signed certificate:

    ```bash
    openssl req -new -x509 -key private_key.pem -out public_cert.pem -days 365
    ```
3.  Verify a certificate

    ```bash
    openssl verify public_cert.pem
    ```
4.  Encrypt a file with a password:

    ```bash
    openssl enc -aes-256-cbc -salt -in file.txt -out file.txt.enc
    ```
5.  Decrypt a file:

    ```bash
    openssl enc -d -aes-256-cbc -in file.txt.enc -out file.txt
    ```

### rsync

The `rsync` command is used for file synchronization and transfer between multiple systems. It can transfer files over SSH or a direct network connection and is designed to be fast and efficient. The command can also perform incremental backups and only transfer the changed parts of a file, reducing network traffic and improving transfer speed.

1.  Copy files from local to remote server:

    ```bash
    rsync -avz /path/to/local/folder/ user@remote:/path/to/remote/folder/
    ```
2.  Copy files from remote to local server:

    ```bash
    rsync -avz user@remote:/path/to/remote/folder/ /path/to/local/folder/
    ```
3.  Sync two folders on the same server:

    ```bash
    rsync -avz /path/to/source/folder/ /path/to/destination/folder/
    ```
4.  Exclude certain files or directories:

    ```bash
    rsync -avz --exclude='*.log' /path/to/source/folder/ /path/to/destination/folder/
    ```
5.  Copy files in archive mode:

    ```bash
    rsync -avz --archive /path/to/source/folder/ /path/to/destination/folder/
    ```
6.  Delete files on the destination that do not exist on the source:

    ```bash
    rsync -avz --delete /path/to/source/folder/ /path/to/destination/folder/
    ```

### lsb\_release

The `lsb_release` command is used to obtain information about the Linux distribution release on which the system is running. It prints the LSB (Linux Standard Base) and distribution-specific information. It can be used to determine the distribution name, release number, and codename.

1.  To display the distribution-specific information:

    ```bash
    lsb_release -a
    ```
2.  To display only the distribution ID:

    ```bash
    lsb_release -i
    ```
3.  To display only the distribution release number:

    ```bash
    lsb_release -r
    ```
4.  To display only the description of the distribution:

    ```bash
    lsb_release -d
    ```
