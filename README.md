# Frappe-16 / ERPNext-16 Installation Guide — Ubuntu 24.04 LTS

> Source: https://github.com/alirezaemami/Frappe-ERPNext-Version-16--in-Ubuntu-24.04-LTS

---

## Prerequisites

- Ubuntu 24.04 LTS
- Python 3.14
- pip 25.3+
- Node 24
- Yarn 1.22+
- Redis 6+
- MariaDB 11.8

---

## Step 0 — Create New User

```bash
sudo adduser frappe
sudo usermod -aG sudo frappe
su - frappe
```

---

## Step 1 — Update System Packages

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Updates package list and upgrades all installed system packages.

---

## Step 2 — Add Deadsnakes PPA

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
```

Adds the Deadsnakes PPA to install newer Python versions.

---

## Step 3 — Refresh Package List

```bash
sudo apt update
```

Refreshes the package list after adding a new repository.

---

## Step 4 — Install Python 3.14

```bash
sudo apt install python3.14 python3.14-venv python3-pip
```

Installs Python 3.14 and its virtual environment module.

---

## Step 5 — Verify Python Installation

```bash
python3.14 --version
pip3.14 --version
```

Verifies Python and pip installation versions.

---

## Step 6 — Install Setuptools & pip

```bash
sudo apt-get install python3-setuptools python3-pip -y
```

Installs setuptools and pip for Python package management.

---

## Step 7 — Install pkg-config

```bash
sudo apt install pkg-config -y
```

Installs pkg-config required for compiling native dependencies.

---

## Step 8 — Install Python Dev Headers

```bash
sudo apt install python3.14-dev -y
```

Installs Python development headers for building extensions.

---

## Step 9 — Install MariaDB

```bash
sudo apt install mariadb-server -y
```

Installs MariaDB database server.

---

## Step 10 — Secure MariaDB

```bash
sudo mysql_secure_installation
```

Secures MariaDB by setting root password and removing defaults.

---

## Step 11 — Install MySQL Client Libraries

```bash
sudo apt-get install libmysqlclient-dev -y
```

Installs MySQL/MariaDB client libraries for Python connectors.

---

## Step 12 — Restart MySQL

```bash
sudo service mysql restart
```

Restarts the MySQL/MariaDB service to apply changes.

---

## Step 13 — Install Redis

```bash
sudo apt-get install redis-server -y
```

Installs Redis used by Frappe for caching and queues.

---

## Step 14 — Install curl

```bash
sudo apt install curl
```

Installs curl for downloading scripts and files.

---

## Step 15 — Install NVM

```bash
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
```

Installs NVM (Node Version Manager).

---

## Step 16 — Reload Shell Profile

```bash
source ~/.profile
```

Reloads shell profile to enable NVM.

---

## Step 17 — Install Node.js 24

```bash
nvm install 24
```

Installs Node.js version 24 using NVM.

---

## Step 18 — Install npm

```bash
sudo apt-get install npm -y
```

Installs npm package manager.

---

## Step 19 — Install Yarn

```bash
sudo npm install -g yarn
```

Installs Yarn globally for frontend asset builds.

---

## Step 20 — Install PDF Dependencies

```bash
sudo apt-get install xvfb libfontconfig -y
```

Installs dependencies required for PDF generation.

---

## Step 21 — Install Fonts for wkhtmltopdf

```bash
sudo apt-get install xfonts-75dpi
```

Installs fonts required by wkhtmltopdf.

---

## Step 22 — Download wkhtmltopdf

Go to https://wkhtmltopdf.org/downloads.html and download the `.deb` file for your architecture (amd64 or arm64).

---

## Step 23 — Install wkhtmltopdf

```bash
sudo apt install -y fontconfig
sudo apt --fix-broken install
sudo dpkg -i wkhtmltox_file.deb
```

Installs wkhtmltopdf package manually.

---

## Step 24 — Install Frappe Bench CLI

```bash
sudo -H pip3 install frappe-bench --break-system-packages
```

Installs Frappe Bench CLI tool.

---

## Step 25 — Initialize Frappe Bench

```bash
bench init frappe-bench --frappe-branch version-16 --python python3.14
```

Initializes a new Frappe bench with version 16.

---

## Step 26 — Enter Bench Directory

```bash
cd frappe-bench/
```

Moves into the Frappe bench directory.

---

## Step 27 — Start Development Server

```bash
bench start
```

Starts Frappe development server.

---

## Step 28 — Create a New Site

```bash
bench new-site local.com
```

Creates a new Frappe site.

---

## Step 29 — Set Active Site

```bash
bench use local.com
```

Sets the active site for bench commands.

---

## Step 30 — Get ERPNext App

```bash
bench get-app erpnext --branch version-16
```

Downloads ERPNext app from GitHub.

---

## Step 31 — Install ERPNext

```bash
bench install-app erpnext
```

Installs ERPNext app on the site.

---

## Step 32 — Set as Production

```bash
sudo apt update
sudo chmod -R o+rx /home/{user-name}
bench --site [site-name] enable-scheduler
bench --site [site-name] set-maintenance-mode off
sudo apt install nginx
sudo /usr/bin/python3 -m pip install --break-system-packages ansible
sudo bench setup production {user-name}
sudo supervisorctl restart all
bench setup socketio
bench setup supervisor
bench setup redis
sudo supervisorctl reload
bench use {site-name}
sudo nano /etc/supervisor/supervisord.conf
bench restart
sudo service supervisor restart
```

> **Note:** In `supervisord.conf`, change `chmod` from `0700` to `0760` under `[unix_http_server]`.

---

---

# Troubleshooting — `bench restart` Supervisor "no such group" Error

## Error

```
frappe: ERROR (no such group)
bench.exceptions.CommandFailedError: sudo supervisorctl restart frappe:
```

**Cause:** Supervisor doesn't know about the Frappe process group — the supervisor config file either wasn't linked correctly or wasn't reloaded properly.

---

## Fix — Step 1: Check if the supervisor config was generated

```bash
ls ~/frappe-bench/config/supervisor.conf
```

---

## Fix — Step 2: Check if it's linked into supervisor's conf.d

```bash
ls -la /etc/supervisor/conf.d/
```

If `frappe-bench.conf` is missing, create the symlink:

```bash
sudo ln -s /home/frappe/frappe-bench/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf
```

---

## Fix — Step 3: Reread and update supervisor

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

---

## Fix — Step 4: Verify the frappe group now exists

```bash
sudo supervisorctl status
```

You should see processes like `frappe:frappe-web`, `frappe:frappe-worker-default`, etc.

---

## Fix — Step 5: Run bench restart

```bash
bench restart
```

---

## If the config file itself is missing (Step 1 returned nothing)

Regenerate it:

```bash
bench setup supervisor
sudo ln -s /home/frappe/frappe-bench/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf
sudo supervisorctl reread
sudo supervisorctl update
```

---

## Also: Check supervisord.conf socket permission

This is critical for the `frappe` user to communicate with supervisord without sudo.

```bash
sudo nano /etc/supervisor/supervisord.conf
```

Under `[unix_http_server]`, ensure:

```ini
chmod=0760
chown=root:frappe
```

Then:

```bash
sudo service supervisor restart
bench restart
```

---

> **Root cause:** The most common cause of this error is a missing symlink in `/etc/supervisor/conf.d/` — supervisor never loaded the frappe group in the first place.
