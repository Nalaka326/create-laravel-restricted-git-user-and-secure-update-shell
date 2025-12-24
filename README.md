# Secure Restricted Git User Setup for Laravel Project

This guide explains how to create a **restricted SSH user (`git-wfms`)** who can only execute predefined Git, Composer, and Laravel commands **inside** `/var/www/html/wfms`.

---

## 🧩 Step 1 — Install Git Client (Optional)

```bash
sudo apt update
sudo apt install git -y
git --version
```

---

## 👤 Step 2 — Create the Restricted User

```bash
sudo adduser git-wfms
sudo usermod -aG ftp_wfms git-wfms
```

---

## 📁 Step 3 — Set Folder Permissions

```bash
sudo chmod -R 775 /var/www/html/wfms
sudo chown -R ftp_wfms:ftp_wfms /var/www/html/wfms
# Give full permission to /storage and /bootstrap/cache
```

---

## 🛡 Step 4 — Create Restricted Shell

```bash
sudo nano /usr/local/bin/restricted-wfms.sh
```

Paste the following:

```bash
#!/bin/bash
# Restricted shell for git-wfms user

WORKDIR="/var/www/html/wfms"
LOGFILE="/var/log/git-wfms.log"
USER=$(whoami)

cd "$WORKDIR" || { echo "Project folder not found"; exit 1; }

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[1;34m'
NC='\033[0m' # No Color

# Log helper
log_action() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $USER ran: $1" >> "$LOGFILE"
}

# Non-interactive SSH command
if [ -n "$SSH_ORIGINAL_COMMAND" ]; then
    case "$SSH_ORIGINAL_COMMAND" in
        "git pull"|"git push"|"composer update"|"php artisan db:seed --class=ProvinceSeeder"|"php artisan db:seed --class=DistrictSeeder"|"php artisan db:seed --class=PermissionTableSeeder"|"php artisan db:seed --class=StatusSeeder"|"php artisan optimize:clear"|"php artisan cache:clear"|"php artisan config:cache"|"php artisan config:clear"|"php artisan route:cache"|"php artisan route:clear"|"php artisan view:cache"|"php artisan view:clear"|"php artisan storage:link"|"php artisan filament:clear-cached-components"|"php artisan livewire:clear"|"npm install")
            log_action "$SSH_ORIGINAL_COMMAND"
            eval "$SSH_ORIGINAL_COMMAND"
            ;;
        *)
            echo -e "${RED}Access denied:${NC} '$SSH_ORIGINAL_COMMAND' not allowed."
            ;;
    esac
    exit 0
fi

# Interactive Menu
while true; do
    clear
    echo -e "${BLUE}=========================================${NC}"
    echo -e "${YELLOW}     WFMS Developer Console${NC}"
    echo -e "${BLUE}=========================================${NC}"
    echo
    echo -e "${GREEN}Allowed commands:${NC}"
    echo "1) git pull"
    echo "2) git push"
    echo "3) composer update"
    echo "4) php artisan db:seed --class=ProvinceSeeder"
    echo "5) php artisan db:seed --class=DistrictSeeder"
    echo "6) php artisan db:seed --class=PermissionTableSeeder"
    echo "7) php artisan db:seed --class=StatusSeeder"
    echo "8) php artisan optimize:clear"
    echo "9) php artisan cache:clear"
    echo "10) php artisan config:cache"
    echo "11) php artisan config:clear"
    echo "12) php artisan route:cache"
    echo "13) php artisan route:clear"
    echo "14) php artisan view:cache"
    echo "15) php artisan view:clear"
    echo "16) php artisan storage:link"
    echo "17) php artisan filament:clear-cached-components"
    echo "18) php artisan livewire:clear"
    echo "19) npm install"
    echo "20) Exit"
    echo
    read -p "Choose an option: " choice

    case $choice in
        1) cmd="git pull" ;;
        2) cmd="git push" ;;
        3) cmd="composer update" ;;
        4) cmd="php artisan db:seed --class=ProvinceSeeder" ;;
        5) cmd="php artisan db:seed --class=DistrictSeeder" ;;
        6) cmd="php artisan db:seed --class=PermissionTableSeeder" ;;
        7) cmd="php artisan db:seed --class=StatusSeeder" ;;
        8) cmd="php artisan optimize:clear" ;;
        9) cmd="php artisan cache:clear" ;;
        10) cmd="php artisan config:cache" ;;
        11) cmd="php artisan config:clear" ;;
        12) cmd="php artisan route:cache" ;;
        13) cmd="php artisan route:clear" ;;
        14) cmd="php artisan view:cache" ;;
        15) cmd="php artisan view:clear" ;;
        16) cmd="php artisan storage:link" ;;
        17) cmd="php artisan filament:clear-cached-components" ;;
        18) cmd="php artisan livewire:clear" ;;
        19) cmd="npm install" ;;
        20) echo -e "${YELLOW}Goodbye!${NC}"; exit 0 ;;
        *) echo -e "${RED}Invalid option. Try again.${NC}"; continue ;;
    esac

    echo -e "${YELLOW}-----------------------------------------${NC}"
    echo -e "${GREEN}Running:${NC} $cmd"
    log_action "$cmd"
    eval "$cmd"
    echo -e "${YELLOW}-----------------------------------------${NC}"

    echo
    read -p "Press Enter to continue..." tmp
done
```

---

## ⚙ Step 5 — Make the Script Executable

```bash
sudo sed -i 's/\r$//' /usr/local/bin/restricted-wfms.sh
sudo chmod 755 /usr/local/bin/restricted-wfms.sh
echo "/usr/local/bin/restricted-wfms.sh" | sudo tee -a /etc/shells
sudo usermod -s /usr/local/bin/restricted-wfms.sh git-wfms
```

---

## 📂 Step 6 — Laravel Writable Directories

```bash
sudo chmod -R 775 /var/www/html/wfms/storage /var/www/html/wfms/bootstrap/cache
```
Setgid (g+s) permission:
```bash
sudo find /var/www/html/wfms -type d -exec chmod g+s {} \;
```
* Any new files or folders created inside it will automatically belong to the same group as the directory.       
* Useful for shared project folders where multiple users collaborate.


## 🧪 Step 7 — Test the Restricted Shell

```bash
sudo su - git-wfms
```

Or via PuTTY:

```
Host: <your server>
User: git-wfms
Port: 22
```
![Description of image](screenshots/Screenshot1.png)
---

## 🪵📜 Step 8 — Configure Logs

```bash
sudo touch /var/log/git-wfms.log
sudo chown ftp_wfms:ftp_wfms /var/log/git-wfms.log
sudo chmod 664 /var/log/git-wfms.log
```

To verify:

```bash
sudo tail /var/log/git-wfms.log
```

📜 Example output:

```
[2025-10-24 18:57:33] git-wfms ran: php artisan optimize:clear  
[2025-10-24 19:01:10] git-wfms ran: git pull
```

---

## 🪪 Step 9 — Rename the User (Optional)

```bash
sudo usermod -l git-wfms devuser
sudo usermod -d /home/git-wfms -m git-wfms
sudo usermod -s /usr/local/bin/restricted-wfms.sh git-wfms
```

---

## 🔐 Step 10 — Enable Secondary SSH Port (Optional)

```bash
sudo nano /etc/ssh/sshd_config
```

Add at the bottom:

```
Port 22
Port 2223

Match User git-wfms
    ForceCommand /usr/local/bin/restricted-wfms.sh
    AllowTcpForwarding no
    X11Forwarding no
```

> ⚠️ The `Match User` block must be **the last section** in the SSH config file.

Then verify and restart:

```bash
sudo sshd -t
sudo systemctl restart ssh
```
✅ Now you can log in using port **2223** and safely run only the allowed Git, Composer, and Artisan commands.

---
**Author:** Nalaka326     
**Last Updated:** November 2025  
**Tested on:** Ubuntu Server 24.04
