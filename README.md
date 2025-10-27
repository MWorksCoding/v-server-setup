# V-Server Setup

## Description
Step-by-step instructions for setting up an Ubuntu-based V-Server, including SSH key generation, hardening SSH access, installing NGINX, serving a custom web page, and connecting the server to GitHub via SSH.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick start](#quick-start)
3. [Generate SSH Keys](#generate-ssh-keys)
4. [Copy Your SSH Key to the Server and Log In](#copy-ssh-key-id-to-your-server-and-login)
5. [Server Setup: Disable Password Authentication](#server-setup)
6. [Install NGINX and Deploy a Custom Page](#install-nginx)
7. [Git Configuration on the V-Server](#git-configuration)
8. [Project Checklist](#project-checklist)

## Prerequisites

Before you begin, ensure you have access to a virtual server (V-Server) and the necessary credentials - including its `ip_address`, `user_name`, and `password` - to establish an initial connection.

You will ll also need:
- A user account with `sudo` privileges
- A GitHub account for SSH key authentication

## Quick start

Begin by cloning this repository to your local machine, then follow the setup steps described in the README.

Open your terminal and run one of the following commands:

```
# Clone using SSH (recommended if your GitHub SSH keys are configured)

git clone git@github.com:MWorksCoding/v-server-setup.git

# Clone using HTTPS (if you have not set up SSH keys)

git clone https://github.com/MWorksCoding/v-server-setup.git
```

## Generate SSH Keys

If you have not already created an SSH key, run the following command in your terminal. Assign a clear name to your key so you can easily recognize its purpose:

```
ssh-keygen -t ed25519 -C "<your@email.com>" -f ~/.ssh/<name_of_your_key25519>
```

When asked for a passphrase, press **Enter** to skip or set one for additional security.  
Using a passphrase is strongly recommended.

This command will create:
- `~/.ssh/<name_of_your_key25519>` → private key  
- `~/.ssh/<name_of_your_key25519>.pub` → public key

## Copy Your SSH Key to the Server and Log In

After generating your SSH key, copy it to your server using the following command:

```
ssh-copy-id -i ~/.ssh/<name_of_your_key25519.pub> <your_server_user_name>@<ip_server_address>
```
- Replace <your_server_username> with the username provided for your server.
- When prompted, enter your server password (contact your administrator if you don’t have it).

Next, test your SSH connection:
```
ssh -i ~/.ssh/<name_of_your_key25519> <your_server_username>@<ip_server_address>
```
If everything is configured correctly, you should now be able to log in without entering a password.

Once logged in, verify that your SSH key was added successfully:
```
ls -al ~/.ssh/authorized_keys
```
You should see your public key listed in the authorized_keys file.

## Server Setup: Disable Password Authentication

For improved security, disable password-based login on your server by modifying the SSH configuration file.

Open the file with administrative privileges:
```
sudo nano /etc/ssh/sshd_config
```
Locate the line with `#PasswordAuthentication yes`.
Uncomment it (remove the #) and change the value to `no`:

```
-#PasswordAuthentication yes

+PasswordAuthentication no
```
- Save and exit, then restart the SSH service:
```
sudo systemctl restart ssh.service
```
Logout and log back in using your SSH key:

```
ssh -i ~/.ssh/<name_of_your_key25519> <your_user_name>@<ip_server_address>
```

You should be able to log in without entering a password.

Attempt a password-only login to confirm it is disabled:

```
ssh -o PubkeyAuthentication=no <your_user_name>@<ip_server_address>
```
This login should fail, indicating that password authentication is deactivated.

## Install NGINX and Deploy a Custom Page

Install the NGINX web server on your V-Server:
```
sudo apt update
sudo apt install nginx -y
```
After installation, check the service status:

```
systemctl status nginx.service
```
Create a new directory for your site and a custom HTML file:
```
sudo mkdir /var/www/alternatives
sudo nano /var/www/alternatives/alternate-index.html
```
- Add your custom HTML content to `alternate-index.html`
- Save and exit.

Create a new site configuration file: in `/etc/nginx/sites-enabled/alternatives` and add the following configuration:
```
sudo nano /etc/nginx/sites-enabled/alternatives
```
The content:
```
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/alternatives;
    index alternate-index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
- Save and exit.

Restart the NGINX service with
```
sudo systemctl restart nginx.service
```
Navigate to `http://<your_v_server_ip>:8081`. You should see your custom web page displayed.

## Git Configuration on the V-Server

First, install Git on your server to manage repositories (clone, pull, push, etc.):

```
sudo apt update
sudo apt install git -y
```

Next, configure your Git user and email:
```
git config --global user.name "<Your Name>"
git config --global user.email "<your@email.com>"
```

Create a new SSH key pair on the server to authenticate with GitHub. Tip: Give the key a meaningful name so you know its purpose.
```
ssh-keygen -t ed25519 -C "<your@email.com>" -f ~/.ssh/<v_server_example_name25519>
```
- When prompted for a passphrase, you can set one for extra security or press Enter to skip.
This generates:
- Private key: `~/.ssh/<v_server_example_name25519>`
- Public key: `~/.ssh/<v_server_example_name25519.pub>`

Add the key to the SSH agent:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/<meaningful_name25519>
```
Add the Key to GitHub, display your public key:

```
cat ~/.ssh/<v_server_example_name25519.pub>
```
- Copy the output.
- Go to GitHub - Settings - SSH and GPG keys - New SSH key
- Give it a title, e.g., <v_server_example_name25519>"
- Paste the key and save.

Test the connection:
```
ssh -T git@github.com
```
Finally, try cloning a repository to verify everything works:
```
git clone git@github.com:MWorksCoding/v-server-setup.git
```
## Project Checklist

You can find a detailed checklist for this project in PDF format:

- [Download the Checklist](docs/v-server-setup.pdf)
>>>>>>> feature/v-server-setup
