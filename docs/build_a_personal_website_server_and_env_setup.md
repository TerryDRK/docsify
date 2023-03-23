# Server & environment setup
In this part, I am going to walk you through the process of deploying the free Oracle Cloud instance, and how I set up the server environment.

## Server setup
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

Either one of these will work.

### Security group
When using a cloud server, you normally need to set up security group. This basically acts like a firewall so that you can manage your inbound / outbound traffic. We are more concerned about the **inbound** traffic, which allows you to manage **who can access my server with which port**.

The inbound rules I set for my server is:

| Source    | Protocol | Destination Port Range | Comment            |
|-----------|----------|------------------------|--------------------|
| 0.0.0.0/0 | TCP      | 22                     | for SSH connection |
| 0.0.0.0/0 | TCP      | 80                     | for HTTP traffic   |
| 0.0.0.0/0 | TCP      | 443                    | for HTTPS traffic  |

You can see that the only ports I open up for services are 80 and 443. How are these two ports going to host all the services we plan to use, you may ask? This answer will be asked in the [next chapter](/build_a_personal_website_domain_https_and_reverse_proxy), where we are going to talk about domain, HTTPS and reverse proxy. For now, we are going to leave it like this.

## Environment setup
Once we connect to the server, we can start setting up the environment need for our server.

As we are planning to run multiple services, it is important that we can have each of them run separately so that if you mis-configure one, you don't break the other services, or even the system itself. For this purpose, we use [Docker](https://www.docker.com). Docker provides easy container management, while ensures isolation between containers.  Besides, for easy container configuration, we also use [Docker Compose](https://docs.docker.com/compose/compose-file/).

To install these, we run the following:

```shell
sudo apt update
sudo apt-get update
sudo apt install docker
sudo apt install docker-compose
```

To check if you installed correctly, you can run:

```shell
docker version
```

You should see the version info of Docker without error.

**TO BE CONTINUED...**
