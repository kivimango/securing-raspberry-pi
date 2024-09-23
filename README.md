# Securing the Raspberry Pi

This guide assumes you are using your Raspberry Pi in headless mode and managing it remotely over SSH. It also assumes you are running a Raspbian-based operating system.

## Steps Overview

- Adding a new user
- Changing default user credentials
- Installing security updates automatically
- Setting up a firewall
- Disabling unused services

### Adding a New User

We'll create a new user to run background processes like torrenting, web hosting, and file sharing.

Run the command:

```sh
sudo adduser new-user-name
```

Replace ```new-user-name``` with your preferred username. You'll be prompted to set a password—make sure to use a strong one with at least 8 characters, including upper and lowercase letters, numbers, and special symbols.

Next, grant the new user administrative privileges by adding it to the ```adm``` and ```sudo``` groups:

```sh
sudo gpasswd -a new-user-name adm
sudo gpasswd -a new-user-name sudo
```

Now your new user will run services instead of the default pi user.

### Changing the Default Credentials

By default, the Pi uses the ```pi``` user with the password ```raspberry,``` a common target for automated attacks. Instead of just changing this password, it's safer to lock the user:

```sh
sudo passwd -l pi
```

This prevents login attempts with the default credentials. **Do not delete the ```pi``` user**, as some packages may rely on it.

### Installing Security Updates Automatically

Keeping your Pi updated is crucial, but manually checking for updates can be tedious. First, update your system manually:

```sh
sudo apt update -y && sudo apt upgrade -y
```

To automate this, install the ```unattended-upgrades``` package:

```sh
sudo apt install unattended-upgrades
```

Next, configure it to ensure it updates Raspbian packages by editing the following file:

```sh
sudoedit /etc/apt/apt.conf.d/50unattended-upgrades
```

Add these lines under the ```Unattended-Upgrade::Origins-Pattern``` section:

```text
"origin=Raspbian,codename=${distro_codename},label=Raspbian";
"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
```

Now your system will automatically install security updates without manual intervention.

### Setting Up a Firewall

The Pi is frequently scanned for open ports, making it essential to configure a firewall. First, install 
```ufw``` (Uncomplicated Firewall):

```sh
sudo apt install ufw -y
```

Then, allow necessary services like SSH (port 22) through the firewall:

```sh
sudo ufw allow 22/tcp comment "SSH"
```

To check active rules, run:

```sh
sudo ufw status
```

Finally, enable the firewall:

```sh
sudo ufw enable
```

**Note**: Be cautious when configuring the firewall to avoid locking yourself out. If that happens, you'll need physical access to your Pi.

### Disabling Unused Services

If you’re not using the minimal version of Raspbian, there may be unnecessary services running. Disable them to save system resources and reduce security risks. List active services with:

```sh
systemctl --type=service
```

To disable a service, use:

```sh
sudo systemctl disable service-name
```

Common services to consider disabling include ```alsa-state``` (sound), ```bluetooth```, ```wpa_supplicant``` (Wi-Fi), and ```cups``` (printers).

### Hardening the SSH Connection

SSH is a common target for attacks, so it's important to lock it down. First, restrict SSH access to specific users by editing the SSH configuration file:

```sh
sudoedit /etc/ssh/sshd_config
```

Add the following line, replacing ```your-user-name``` with your new user:

```text
AllowUsers your-user-name
```

If multiple users need access, separate them with commas:

```text
AllowUsers user1,user2,user3
```

For extra security, consider using key-based authentication instead of passwords (beyond the scope of this guide).

### Blocking Malicious IPs with Fail2Ban

To block IP addresses after failed login attempts, install ```fail2ban```:

```sh
sudo apt install fail2ban
```

Create a custom configuration file to prevent it from being overwritten during updates:

```sh
sudoedit /etc/fail2ban/jail.local
```

Add the following:

```text
[DEFAULT]
bantime = 1h
banaction = ufw

[sshd]
enabled = true
```

This setup bans IP addresses after too many failed SSH login attempts by adding a deny rule to the firewall. Enable the fail2ban service:

```sh
sudo systemctl enable --now fail2ban
```

Restart the SSH service to apply the changes:

```sh
sudo systemctl restart sshd
```

### Final Steps

Test your SSH connection:

```sh
ssh user@host
```

If successful, reboot the system:

```sh
sudo systemctl reboot --now
```
