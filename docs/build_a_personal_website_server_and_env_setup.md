# Server & environment setup
In this part, I am going to walk you through the process of deploying the free Oracle Cloud instance, and how I set up the server environment.

## Cloud server setup
As I mentioned in [Intro](/build_a_personal_website_intro) part, I am using [Oracle Cloud](https://www.oracle.com/cloud/) server because they are free lol. You can use [AWS](https://aws.amazon.com), [Azure](https://azure.microsoft.com/en-us), etc., whatever provider you prefer, or, even your own physical server.

The server I use has only 1 CPU core, 1GB of memory, and 47GB of disc space. Thus, I want to make everything light-weighted, so that I can maximize the number of services I can run on it.

### System OS
For system OS, I use [Ubuntu](https://ubuntu.com) 22.04, because I am more familiar with it. You can use other OS like [CentOS](https://www.centos.org), [Windows Server](https://www.microsoft.com/en-us/windows-server), etc., as long as it can run [Docker](https://www.docker.com). All the following commands will base on Ubuntu.

Luckily, if you choose to use cloud server providers, The system will be installed automatically, so I will skip the system installation process (because there is none 😂). However, if you are using your own physical server, you can always get help from the OS's official documents when it comes to how to install it. We will focus on cloud servers in the following configuration steps.

### SSH connection
Once the system is up running, you can use SSH to access your new server. Remember to get a public IPv4 address for the server. In Oracle Cloud, you will normally have it. In AWS, you probably need to use elastic IP. We will assume the server public IPv4 address is `1.2.3.4`.

When configuring the server instance, you will normally have 3 options:

- **Username + password:** This is the one that is the easiest to understand - use a username + password pair to log into your server. Use `ssh username@1.2.3.4`, and you will be asked to enter the password.
- **PEM:** This will generate a pem file (or text content of it) for you to connect to the server. In this case, use `ssh -i /path/to/your.pem username@1.2.3.4` to connect to the server.
- **Public key:** In this case, your ssh public key will be added to the server's authorized users, allowing you to access the server without password or PEM. To use this method, you need to provide your `~/.ssh/id_rsa.pub` when setting up the server. If you find yourself not having this file (or the entire directory), you can always generate it with `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`. Refer to [this page](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for more detailed instructions.

**TO BE CONTINUED...**
