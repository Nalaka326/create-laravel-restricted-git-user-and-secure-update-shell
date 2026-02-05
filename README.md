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
sudo usermod -aG www-data git-wfms

# username = git-wfms
```

---

## 📁 Step 3 — Set Folder Permissions

```bash
sudo chmod -R 775 /var/www/html/wfms
sudo chown -R www-data:www-data /var/www/html/wfms
```

---

## 🛡 Step 4 — Create Restricted Shell

```bash
sudo nano /usr/local/bin/restricted-wfms.sh
```

Paste the following:

```bash
#!/bin/bash
# =========================================
# WFMS Restricted Developer Shell
# =========================================

WORKDIR="/var/www/html/wfms"
LOGFILE="/var/log/git-wfms.log"

USER_NAME=$(whoami)
CLIENT_IP=$(echo "$SSH_CONNECTION" | awk '{print $1}')

# Use safe PATH
PATH="/usr/bin:/bin:/usr/local/bin"

cd "$WORKDIR" || {
    echo "ERROR: Project directory not found"
    exit 1
}

# ---------- Logging ----------
log_action() {
    local CMD="$1"
    local STATUS="$2"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] user=$USER_NAME ip=$CLIENT_IP cmd=\"$CMD\" status=$STATUS" >> "$LOGFILE"
}

# ---------- Allowed Commands ----------
run_command() {
    local CMD="$1"
    echo "-----------------------------------------"
    echo "Running: $CMD"
    echo

    case "$CMD" in
        "npm install")
            /usr/bin/npm install
            ;;
        "composer update")
            /usr/bin/composer update
            ;;
        "git pull")
            /usr/bin/git pull
            ;;
        "git push")
            /usr/bin/git push
            ;;
        "php artisan cache:clear")
            /usr/bin/php artisan cache:clear
            ;;
        "php artisan config:clear")
            /usr/bin/php artisan config:clear
            ;;
        "php artisan config:cache")
            /usr/bin/php artisan config:cache
            ;;
        "php artisan route:clear")
            /usr/bin/php artisan route:clear
            ;;
        "php artisan route:cache")
            /usr/bin/php artisan route:cache
            ;;
        "php artisan view:clear")
            /usr/bin/php artisan view:clear
            ;;
        "php artisan view:cache")
            /usr/bin/php artisan view:cache
            ;;
        "php artisan optimize:clear")
            /usr/bin/php artisan optimize:clear
            ;;
        "php artisan storage:link")
            /usr/bin/php artisan storage:link
            ;;
        "php artisan livewire:clear")
            /usr/bin/php artisan livewire:clear
            ;;
        "php artisan filament:clear-cached-components")
            /usr/bin/php artisan filament:clear-cached-components
            ;;
        "php artisan db:seed --class=ProvinceSeeder")
            /usr/bin/php artisan db:seed --class=ProvinceSeeder
            ;;
        "php artisan db:seed --class=DistrictSeeder")
            /usr/bin/php artisan db:seed --class=DistrictSeeder
            ;;
        "php artisan db:seed --class=PermissionTableSeeder")
            /usr/bin/php artisan db:seed --class=PermissionTableSeeder
            ;;
        "php artisan db:seed --class=StatusSeeder")
            /usr/bin/php artisan db:seed --class=StatusSeeder
            ;;
        *)
            echo "ERROR: Command not allowed"
            return 127
            ;;
    esac

    STATUS=$?
    log_action "$CMD" "$STATUS"

    echo
    echo "-----------------------------------------"
}

# ---------- Direct SSH command ----------
if [ -n "$SSH_ORIGINAL_COMMAND" ]; then
    run_command "$SSH_ORIGINAL_COMMAND"
    exit
fi

# ---------- Interactive Menu ----------
while true; do
clear
cat <<EOF
=========================================
     WFMS Developer Console
=========================================

Allowed commands:
1) npm install
2) composer update
3) git pull
4) git push
5) php artisan cache:clear
6) php artisan config:clear
7) php artisan config:cache
8) php artisan route:clear
9) php artisan route:cache
10) php artisan view:clear
11) php artisan view:cache
12) php artisan optimize:clear
13) php artisan storage:link
14) php artisan livewire:clear
15) php artisan filament:clear-cached-components
16) php artisan db:seed --class=ProvinceSeeder
17) php artisan db:seed --class=DistrictSeeder
18) php artisan db:seed --class=PermissionTableSeeder
19) php artisan db:seed --class=StatusSeeder
20) Exit

EOF

read -rp "Choose an option: " CHOICE

case "$CHOICE" in
    1) CMD="npm install" ;;
    2) CMD="composer update" ;;
    3) CMD="git pull" ;;
    4) CMD="git push" ;;
    5) CMD="php artisan cache:clear" ;;
    6) CMD="php artisan config:clear" ;;
    7) CMD="php artisan config:cache" ;;
    8) CMD="php artisan route:clear" ;;
    9) CMD="php artisan route:cache" ;;
    10) CMD="php artisan view:clear" ;;
    11) CMD="php artisan view:cache" ;;
    12) CMD="php artisan optimize:clear" ;;
    13) CMD="php artisan storage:link" ;;
    14) CMD="php artisan livewire:clear" ;;
    15) CMD="php artisan filament:clear-cached-components" ;;
    16) CMD="php artisan db:seed --class=ProvinceSeeder" ;;
    17) CMD="php artisan db:seed --class=DistrictSeeder" ;;
    18) CMD="php artisan db:seed --class=PermissionTableSeeder" ;;
    19) CMD="php artisan db:seed --class=StatusSeeder" ;;
    20) echo "Goodbye"; exit 0 ;;
    *) echo "Invalid option"; sleep 1; continue ;;
esac

run_command "$CMD"
read -rp "Press Enter to continue..."
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
sudo chmod -R 777 /var/www/html/wfms/storage /var/www/html/wfms/bootstrap/cache
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
sudo chown www-data:www-data /var/log/git-wfms.log
sudo chmod 664 /var/log/git-wfms.log
```

To verify:

```bash
sudo tail /var/log/git-wfms.log
```

📜 Example output:

```
[2025-11-24 12:06:01] user=git-wfms ip=10.20.111.4 cmd="php artisan cache:clear" exit=0
[2025-11-24 12:35:19] user=git-wfms ip=10.20.111.4 cmd="php artisan cache:clear" status=0
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

# Give this permission again.
sudo chmod -R 777 /var/www/html/wfms/storage /var/www/html/wfms/bootstrap/cache
```
✅ Now you can log in using port **2223** and safely run only the allowed Git, Composer, and Artisan commands.

---
## Troubleshooting: Cannot log in via SSH port 2224 (Ubuntu 24.04)

If you cannot connect to SSH using port 2224, follow the steps below exactly.
This issue is usually caused by systemd socket activation (ssh.socket), which forces SSH to listen only on port 22.

## Final Fix (do this exactly)
1. Check SSH socket status
```bash
systemctl status ssh.socket
```
If the status shows active (listening), continue to the next step.

2. Disable SSH socket activation
```bash
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
```
Optional but recommended (prevents future overrides):
```bash
sudo systemctl mask ssh.socket
```

3. Restart the SSH service
```bash
sudo systemctl restart ssh
```

4. Verify SSH is listening on both ports
```bash
sudo ss -tlnp | grep ssh
```
Expected output:
```bash
LISTEN 0.0.0.0:22
LISTEN 0.0.0.0:2224
LISTEN [::]:22
LISTEN [::]:2224
```
Notes

* Masking ssh.socket does not reduce security
* This is required to allow multiple SSH ports
* SSH configuration is controlled by sshd_config, not ssh.socket
---
**Author:** Nalaka326     
**Last Updated:** November 2025  
**Tested on:** Ubuntu Server 24.04
