# pp4-valkyrie

![Build Status](https://github.com/OnyxJeff/pp4-valkyrie/actions/workflows/build.yml/badge.svg)
![Maintenance](https://img.shields.io/maintenance/yes/2026.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![GitHub release](https://img.shields.io/github/v/release/OnyxJeff/pp4-valkyrie)
![Issues](https://img.shields.io/github/issues/OnyxJeff/pp4-valkyrie)

**Valkyrie** is the internal playground server for my homelab, hosted on a Raspberry Pi 4.

## 📁 Repo Structure

```text
pp4-valkyrie/
├── .github/workflows/      # CI for YAML validation
├── backup_logs/            # Oldest logs from update script
├── dockprom/               # Docker container(s) for Prometheus Node Exporter for RPi
├── images/                 # Images for README files
├── logs/                   # Most recent update script logs
├── scripts/                # Auto-Updater script for RPi (can be associated with cronjob)
├── U6143_ssd1306/          # Python, C code, and script for UCTronics display screen
└── README.md               # You're reading it!
```

---

## 🧰 Services
- Testing docker container configs for RPi's

- Developing documentation on other RPi setups currently in my homelab.
  - This allows me to develop proper documentation for existing RPis without taking down my current services

- Current RPi Services in production
  - Odin
    - **Pi-hole**: Blocks ads, trackers, and telemetry across the network.
    - **Dockprom**: Collects Docker and system metrics for monitoring and analysis.
  - Mimir
    - **Dockprom**: Collects Docker and system metrics for monitoring and analysis.
    - **UniFi Controller**: Manages UniFi network devices and monitors network performance.
  - Loki
    - **Transmission Server**: Manages and downloads torrents with a web interface.
    - **Dockprom**: Collects Docker and system metrics for monitoring and analysis.

---

## 📝 Preparation

  - Install GIT (app used to download this repo onto your device)
  ```bash
  sudo apt install git -y
  ```

  - Download repo
  ```bash
  cd
  git clone https://github.com/OnyxJeff/pp4-valkyrie.git
  ```

---

## ⚠️ Updating the OS

- Make log folders
```bash
mkdir ~pp4-valkyrie/logs
mkdir ~pp4-valkyrie/backup_logs
```

- Update and Upgrade the System via script:
```bash
cd ~/pp4-valkyrie/scripts
chmod +x apt-get-autoupdater.sh
sudo ./apt-get-autoupdater.sh
```
> [!NOTE]
> This may take a while if using an OS other than Raspberry Pi OS

- Start CronJob (optional but recommended if doing headless/always on installation)
```bash
sudo crontab -e
```

  - add the following to the bottom of the document:
  ```bash
  # OS-Auto-Updater
    00 01 * * 0 bash $HOME/pp4-valkyrie/scripts/apt-get-autoupdater.sh
      # execute automatic update script and log every sunday at 01:00 am
    50 00 1 * * /bin/bash -c 'cp $HOME/pp4-valkyrie/logs/apt-get-autoupdater.log $HOME/pp4-valkyrie/backup_logs/apt-get-autoupdater-$(date +\%Y\%m\%d).log'
      # saves monthly version of "apt-get-autoupdater.log" on the 1st of every month at 00:50 am
    51 00 1 * * rm -f $HOME/pp4-valkyrie/logs/apt-get-autoupdater.log
      # deletes old weekly log on the 1st of every month at 00:51 am
  ```

---

## 🖥️ Installing U6143_ssd1306 Display

- Run setup_display_service.sh script
```bash
cd ~/pp4-skadi/U6143_ssd1306
chmod +x setup_display_service.sh
sudo ./setup_display_service.sh
```

- Custom display temperature type
  - Open the U6143_ssd1306/C/ssd1306_i2c.h file. You can modify the value of the TEMPERATURE_TYPE variable to change the type of temperature displayed. (The default is Fahrenheit)
  ![Select Temperature](images/select_temperature.jpg)

- Custom display IPADDRESS_TYPE type
  - Open the U6143_ssd1306/C/ssd1306_i2c.h file. You can modify the value of the IPADDRESS_TYPE variable to change the type of IP displayed. (The default is ETH0)
  ![Select IP](images/select_ip.jpg)

- Custom display information
  - Open the U6143_ssd1306/C/ssd1306_i2c.h file. You can modify the value of the IP_SWITCH variable to determine whether to display the IP address or custom information. (The custom IP address is displayed by default)
  ![Custom Display](images/custom_display.jpg)

> [!Note]
> I did rewrite the initial install script to be more repo friendly.

---

## 📦 Installing Docker Compose

- Install Docker
```bash
cd
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

- Add User to Docker Group
```bash
sudo usermod -aG docker $USER
```

> [!IMPORTANT]
> After running this command you will need to log out and log back in (or I recommend just rebooting) for the changes to take effect.

- Install Docker Compose:
```bash
sudo apt install docker-compose-plugin
```

- Verify Installation:
```bash
docker run hello-world
docker compose version
```

### 📝 Installing your first container(s)

- Installing Container Stack (via script)
```bash
cd ~/pp4-valkyrie/scripts
chmod +x docker-up-all.sh
./docker-up-all.sh
```

---

## 🧪 Installing Ansible

- Install Ansible
```bash
sudo apt install -y ansible git python3-pip
ansible --version
```

- Generate an SSH Key Pair

> [!IMPORTANT]
> Skip to "Verify your keys" section if you imported a private key from another machine

```bash
ssh-keygen -t ed25519 -C "ansible@homelab"
```
  - Explaination of flags:
    - `-t ed25519` → modern, secure key type (better than RSA)
    - `-C "ansible@homelab"` → optional comment so you know which key it is
    - You'll see promtps like:
    `Enter file in which to save the key (/home/<USER>/.ssh/id_ed25519):`
  - Press **Enter** to accept the default location.
  - When prompted for a passphrase, you can either:
    - Enter one (more secure, but you’ll type it for every run unless you use `ssh-agent`)
    - Leave empty (convenient for automated playbooks)

- Verify your keys
```bash
ls -l ~/.ssh/id_ed25519*
```

  - You should see two files:
    - `id_ed25519` → private key (keep secret!)
    - `id_ed25519.pub` → public key (what you copy to targets)

- Install SSHPass on Control Node:
```bash
sudo apt install -y sshpass
```

- Copy the public key to a target host
  Example for a Pi or Linux VM:
  ```bash
  ssh-copy-id pi@<target-host-ip>
  ```
  - Replace `pi` with the username on the target.
  - Replace `<target-host-ip>` with the IP address.
  If SSH is default port 22, this works. If custom port:
  ```bash
  ssh-copy-id -p 2222 pi@<target-host-ip>
  ```

- Test passwordless SSH
```bash
ssh pi@<target-host-ip>
```
  - You should log in without a password prompt.
  - If it asks for a password, something went wrong—check `~/.ssh/authorized_keys` on the target.

- Optional: Agent Forwarding (Handy for Ansible)
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
This lets Ansible use the key automatically

---

## Acknowledgements

This project uses or is inspired by the following repositories:

- [U6143_ssd1306](https://github.com/UCTRONICS/U6143_ssd1306) – Provides the C display code used in the systemd service setup.
- [Dockprom](https://github.com/stefanprodan/dockprom) – Used for Docker-based Prometheus monitoring and metrics collection.

---

📬 Maintained By
Jeff M. • [@OnyxJeff](https://www.github.com/onyxjeff)