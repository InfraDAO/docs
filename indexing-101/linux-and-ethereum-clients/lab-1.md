# Lab 1: Sync a Gnosis blockchain node

In this lab, you will apply your knowledge of Linux and Ethereum clients to set up a local development environment for Ethereum. You will learn how to install and configure the Nethermind client, connect to the Gnosis network, and interact with the Gnosis blockchain. By the end of this lab, you will have a functional Gnosis node running on your machine and will be ready to start exploring The Graph ecosystem.

This lab exercise assumes you are configuring a [Hetzner AX102 dedicated root server](https://www.hetzner.com/dedicated-rootserver/ax102) for your Gnosis blockchain node.

## Configure the server

### Put your Hetzner AX102 server into RescueMode

1. Log in to your Hetzner account and go to the "Server" tab.
2. Select the server you want to put into RescueMode.
3. Click on the "Rescue" tab in the server details.
4. Choose the Rescue system you want to use and click on "Activate".
5. Wait for the system to activate, which may take a few minutes.
6. Once the Rescue system is active, you can log in using the provided login credentials.

### Install the [Ubuntu 22.04](https://releases.ubuntu.com/22.04/) Linux distribution with the [`installimage`](https://docs.hetzner.com/robot/dedicated-server/operating-systems/installimage/) script

1. Use the password displayed on Hetzner Robot to log into the Rescue System as "root".
2.  Run the `installimage` script by typing the following command

    ```bash
    installimage -n GnosisNode -r yes -i images/Ubuntu-2204-jammy-amd64-base.tar.gz -d sda -p /boot:ext3:1024M,lvm:vg0:all -v vg0:root:/:ext4:all
    ```

    Overall, this command will perform an installation of Ubuntu 22.04 on the "sda" device using the specified partition and logical volume layout, with the hostname set to "GnosisNode" and the server rebooting automatically after the installation is complete.

    1. `installimage` - This is the command to execute the installimage script.
    2. `-n GnosisNode` - This sets the hostname for the server to "GnosisNode".
    3. `-r yes` - This specifies that the server should automatically reboot after the installation process is complete.
    4. `-i images/Ubuntu-2204-jammy-amd64-base.tar.gz` - This specifies the path to the base installation image for Ubuntu 22.04. In this case, the file is located in the "images" directory and is named "Ubuntu-2204-jammy-amd64-base.tar.gz".
    5. `-d sda` - This specifies the device that will be used for the installation. In this case, the installation will be performed on the "sda" device.
    6. `-p /boot:ext3:1024M,lvm:vg0:all` - This specifies the partition layout for the installation. In this case, there will be two partitions created: one for the "/boot" directory, which will be formatted with the ext3 file system and will have a size of 1024MB, and one for LVM, which will contain all remaining space on the disk and will be used for logical volume management.
    7. `-v vg0:root:/:ext4:all` - This specifies the logical volume layout for the installation. In this case, one logical volume named "root" will be created in the volume group "vg0", which will be mounted at the root directory ("/") and formatted with the ext4 file system.

### Install Linux packages

1.  Update packages

    ```bash
    apt update && apt upgrade 
    ```
2.  Install build-essential

    ```bash
    apt install build-essential 
    ```
3.  Install `git`

    ```bash
    apt install git
    ```
4.  Install `unzip`

    ```bash
    apt install unzip
    ```
5.  Install `ufw` firewall

    ```bash
    apt install ufw 
    ```

### User management

1.  Add a new user account named `dev` to a Linux system

    ```bash
    adduser dev 
    ```
2.  Adds the `dev` user to the `sudo` group, giving them the ability to execute commands with administrative privileges. Note that after running this command, the user will need to log out and log back in for the changes to take effect. This type of user account is typically used for system services or applications that need to run with a specific set of permissions, but do not require direct access to the system.

    ```bash
    usermod -aG sudo dev
    ```

    1. `usermod` - This is the command to modify user account properties.
    2. `-aG sudo` - The `-a` option appends the specified group to the user's list of groups, while the `-G` option specifies the groups to which the user should be added. In this case, the sudo group is added to the user's list of groups.
    3. `dev` - This is the username of the user account that will be modified.
3.  Add a new user account named `nethermind` to a Linux system

    ```bash
    sudo useradd --no-create-home --shell /bin/false nethermind
    ```

    1. `sudo` - This command is used to run the useradd command with elevated privileges. This is necessary because creating a new user account requires administrative privileges.
    2. `useradd` - This is the command to create a new user account.
    3. `--no-create-home` - This option specifies that a home directory should not be created for the new user account.
    4. `--shell /bin/false` - This option sets the login shell for the new user account to `/bin/false`, which means that the user will not be able to log in to the system.
    5. `nethermind` - This is the username of the new user account that will be created.

### SSH

1.  Switch to `dev` user

    ```bash
    su dev
    ```
2.  Change the current working directory to the user's home directory.

    ```bash
    cd ~
    ```
3.  Create a directory called `.ssh`

    ```bash
    mkdir .ssh
    ```
4.  Create a file to store your public `ssh` keys

    ```bash
    touch .ssh/authorized_keys
    ```
5.  Change the file permissions of the `.ssh` directory to `rwx------`, which means that only the owner of the directory can read, write, and execute files within it. This is useful for ensuring that sensitive files within the directory, such as private SSH keys, are only accessible to the owner of the directory.

    ```bash
    chmod 700 .ssh 
    ```

    1. `chmod` - This is the command to change the file permissions of a file or directory.
    2. `700` - This is the numerical representation of the file permissions. In this case, `7` sets the owner's permissions to `rwx` (read, write, execute), while `0` sets the permissions for the group and others to `---` (no permissions).
    3. `.ssh` - This is the name of the directory whose file permissions are being changed.
6.  Change the file permissions of the `authorized_keys` file within the `.ssh` directory to `rw-------`, which means that only the owner of the file can read and write to it. This is an important security measure, as the authorized\_keys file is used to authenticate SSH connections, and granting unauthorized access to it could allow an attacker to gain access to the system.

    ```bash
    chmod 600 .ssh/authorized_keys
    ```

    1. `chmod` - This is the command to change the file permissions of a file or directory.
    2. `600` - This is the numerical representation of the file permissions. In this case, `6` sets the owner's permissions to `rw` (read, write), while `0` sets the permissions for the group and others to `---` (no permissions).
    3. `.ssh/authorized_keys` - This is the name and path of the file whose file permissions are being changed.
7.  Change the current working directory to the .ssh directory within the user's home directory

    ```bash
    cd .ssh
    ```
8. Open the `authorized_keys` file in the Nano text editor for editing and paste in your public `ssh` keys. Once the user has finished editing the file, they can save their changes and exit the editor by pressing `Ctrl+X`, then `Y` to confirm the changes and `Enter` to save the file with the same name.
9.  Check that you can login to your server as `dev` user with `ssh` public key authentication. Replace server\_ip\_address with the actual IP address of the server.

    ```bash
    ssh dev@server_ip_address
    ```
10. Disable `root` and password-based logins to your server

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    1. Set `PermitRootLogin no`: This is a security best practice because the root user has unrestricted access to the entire system, so allowing remote login as root poses a significant security risk. Instead, it's recommended to log in as a non-root user with sudo privileges, and use the sudo command to perform administrative tasks.
    2. Set `PasswordAuthentication no`: This is a security best practice because it reduces the risk of brute force attacks against SSH login credentials. Public key authentication requires the use of a private key on the client machine and a corresponding public key on the server. This method of authentication is much more secure than using passwords, which can be guessed or cracked through brute force attacks.
    3. Set `UsePAM no`: The server will not use the PAM framework for authentication, and will instead rely on its own built-in authentication mechanisms. PAM is a modular system that allows for different authentication methods to be used, such as LDAP or Kerberos. However, disabling PAM can provide a more secure environment since it reduces the attack surface of the system.

### Firewall

1.  Allow incoming SSH traffic on port 22 through the system's firewall. This is necessary in order to establish SSH connections to the system from remote clients, such as other computers or mobile devices.

    ```bash
    sudo ufw allow 22
    ```

    1. `sudo` - This command is used to run the `ufw` command with elevated privileges. This is necessary because modifying firewall rules requires administrative privileges.
    2. `ufw` - This is the command to manage the system's firewall rules using the `ufw` utility.
    3. `allow` - This option is used to add a new rule to allow incoming traffic through the firewall.
    4. `22` - This specifies the port number for the incoming traffic that should be allowed. In this case, port 22 is used, which is the default port for SSH traffic.
2.  Allow incoming traffic on port 40403 through the system's firewall. This is useful when running a service or application that requires incoming traffic on that particular port, such as a web server or a database server.

    ```bash
    sudo ufw allow 40403
    ```
3.  Enable the `ufw` (Uncomplicated Firewall) utility on your system, with administrative privileges. All incoming and outgoing traffic is blocked by default, except for ports with an `allow` rule. This is a good security measure, as it prevents unauthorized access to the system.

    ```bash
    sudo ufw enable
    ```

## Configure the blockchain client

### Nethermind

In this section, we will learn how to run Nethermind, a client implementation of the Ethereum blockchain. We will:

* Download the latest version of Nethermind from the official website
* Configure the client by editing the nethermind.cfg file to set the appropriate network ID and parameters for syncing the blockchain
* Run the Nethermind client using the appropriate command for your operating system (e.g., `nethermind.Run` for Windows or `./nethermind` for Linux/Mac)
* Monitor the client's progress using the logs and various tools available in the Nethermind interface
* Once fully synced, you can interact with the Ethereum network using the Nethermind client and start using various Ethereum-based applications and services.

1.  **Installing Nethermind**

    `sudo add-apt-repository ppa:nethermindeth/nethermind`

    * Adds the Nethermind PPA to the system's software sources.
    * Allows you to install and receive updates for Nethermind using the apt package manager.

    `sudo apt install nethermind`

    * Installs the Nethermind package from the system's package repositories.
    * Downloads and installs the necessary files and dependencies for Nethermind to run.
    * Enables you to run Nethermind using the nethermind command in the terminal.

    `docker pull nethermind/nethermind`

    * docker pull is a command used to download a Docker image from a container registry.
    * nethermind/nethermind is the name of the image being downloaded from Docker Hub.
    * This command downloads the latest version of the Nethermind image from Docker Hub.
    * Once the image is downloaded, it can be used to run Nethermind in a Docker container.
2.  **Configure JSON-RPC API**

    JWT Secrets - JSON Web Token authentication was added to the JSON-RPC API for security reasons to ensure that nothing interferes with the communication between the Execution Client (Nethermind in this case) and the Consensus Client. This requires you to create a file containing a hexadecimal “secret” that will be passed to each.

    To create this “Secret File” use the following command: `openssl rand -hex 32 | tr -d "\n" > "/tmp/jwtsecret"` where "/tmp/jwtsecret" will be the file path and name when created.

    Engine module needs to be explicitly switched on in the Netherming config file:

    ```
    "JsonRpc": {
        "Enabled": true,
        "Timeout": 20000,
        "Host": "127.0.0.1",
        "Port": 8545,
        "EnabledModules": ["Eth", "Subscribe", "Trace", "TxPool", "Web3", "Personal", "Proof", "Net", "Parity", "Health"],
        "EnginePort": 8551,
        "EngineHost": "127.0.0.1",
        "JwtSecretFile": "keystore/jwt-secret"
    },
    ```
3.  **Run Nethermind**

    Ensure you have:

    * Installed Nethermind
    * Created a JWT secret file
    * Engine module is enabled with authenticated port

    **Running Nethermind from docker:**

    `docker run -it -v /home/user/data:/nethermind/data nethermind/nethermind --config ropsten --JsonRpc.Enabled true --JsonRpc.JwtSecretFile=PATH --datadir data --JsonRpc.EngineHost=0.0.0.0 --JsonRpc.EnginePort=8551`

    * `--config` flag \*\*\*\* is the network.
    * `v /home/user/data:/nethermind/data` sets local directory we will be storing our data to
    * `--JsonRpc.JwtSecretFile=PATH` where PATH is the location of your JWT secret ex. /tmp/jwtsecret
    * `--datadir` data maps the database, keystore, and logs all at once
4.  **Run Consensus Clients**

    Once Nethermind has started you can start the CL client. See the next section for commands to install and run the CL client you installed.

To learn more about running nethermind, refer to the official docs [here](https://docs.nethermind.io/nethermind/first-steps-with-nethermind/running-nethermind-post-merge)

## Claim your Unit 1 POAP

### Create a new issue in the Indexing 101 tutorial repository using the Unit 1 POAP Form template

1. Navigate to the [Unit 1 POAP Form template](https://github.com/IndexerDAO/docs/issues/new?assignees=\&labels=\&template=unit-1-poap-form.md\&title=%5BUnit+1+Submission%5D%3A+)
2. Update the issue with a screenshot of your `journalctl` logs and Ethereum address
3. Click `Submit new issue`
