# Class 1 — The Complete Beginner Walkthrough AWS

This guide assumes you have never used a terminal, a cloud server, Git, Docker, or a database before. Nothing is skipped. At each step I'll tell you: what to type, why, and what you should see happen afterward, so you always know if it worked.

**Read this first — the one idea that makes everything else make sense:**

> You will be working with **two different computers**: your own laptop, and a cloud server you create on AWS. Your laptop's terminal can "remote control" the server once you connect to it with a command called `ssh`. Once connected, everything you type happens **on the server**, not your laptop — like picking up a phone that controls a computer far away. Typing `exit` disconnects you and puts you back in control of your own laptop.

---

## Part 1: Create your cloud server

### Step 1.1 — Create an AWS account

1. Go to **aws.amazon.com** in your web browser.
2. Click **Create an AWS Account**.
3. Follow the signup steps (you'll need an email, a credit card for verification, and a phone number). New accounts get a free-tier allowance, so this won't cost anything if you stay within it.

### Step 1.2 — Open the EC2 service

1. Once logged into the AWS Console, look at the top-left search bar.
2. Type **EC2** and click on it in the results.
3. You'll land on the EC2 Dashboard — this is the control panel for creating virtual servers.

### Step 1.3 — Launch your server

1. Click the orange **Launch instance** button.
2. Under **Name and tags**, type a name, e.g. `my-data-server`.
3. Under **Application and OS Images**, click **Ubuntu**, then make sure the version dropdown says **Ubuntu 24.04 LTS**.
4. Under **Instance type**, select **t2.small**.
5. Under **Key pair (login)**:
   - Click the dropdown → **Create new key pair**
   - Name it e.g. `my-server-key`
   - Key pair type: **RSA**
   - Private key file format: **.pem** (Mac/Linux/Windows Terminal) or **.ppk** (only if you plan to use PuTTY)
   - Click **Create key pair** — a file will download to your computer automatically. **Do not lose this file.** Move it somewhere memorable, like a folder called `Documents/keys`.
6. Under **Network settings**, make sure **Allow SSH traffic from** is checked (it usually is by default).
7. Leave **Configure storage** as the default.
8. Click the orange **Launch instance** button at the bottom.
9. You'll see a green success message. Click **View all instances**.

### Step 1.4 — Find your server's address

1. On the Instances page, you'll see your new server with a status that starts as "Pending" and changes to "Running" after about a minute. Wait for "Running."
2. Click on the instance to open its details.
3. Find the field labeled **Public IPv4 address** — it'll look like `13.214.179.228`. **Copy this number down somewhere** — you'll need it constantly. I'll refer to it below as `YOUR_SERVER_IP`.

---

## Part 2: Connect to your server for the first time

### Step 2.1 — Open a terminal on your own laptop

- **On Mac:** open the app called **Terminal** (search for it with Spotlight, Cmd+Space).
- **On Windows:** open **Windows Terminal** or **PowerShell** (search for either in the Start menu).

A terminal is just a window where you type text commands instead of clicking buttons. A blinking cursor waiting for input means it's ready.

### Step 2.2 — Locate your downloaded key file

You need to know the exact location of the `.pem` file you downloaded in Step 1.3. If it's in your Downloads folder, its path is probably:
- Mac: `/Users/YourName/Downloads/my-server-key.pem`
- Windows: `C:\Users\YourName\Downloads\my-server-key.pem`

### Step 2.3 — (Mac/Linux only) Set the correct permissions on the key file

```bash
chmod 400 /Users/YourName/Downloads/my-server-key.pem
```
**What this does:** SSH refuses to use a key file if it's too "open" (readable by others). This command locks it down to only you. You'll see no output at all if it works — no news is good news.

### Step 2.4 — Connect

Type this, replacing the path and IP with your own:
```bash
ssh -i /Users/YourName/Downloads/my-server-key.pem ubuntu@YOUR_SERVER_IP
```

**What you should see:**
```
The authenticity of host 'YOUR_SERVER_IP' can't be established.
Are you sure you want to continue connecting (yes/no)?
```
Type `yes` and press Enter. This only happens once per server.

Then you should see a welcome message ending in a prompt that looks like:
```
ubuntu@ip-172-31-9-7:~$
```

**This means it worked.** That `$` at the end is where your typing goes from now on — and everything you type now happens **on the server**, not your laptop.

---

## Part 3: Terminal basics — practice before building anything

Try these one at a time, right now, to build muscle memory. Type each command, press Enter, and read what I say you should see.

### `pwd` — "where am I?"
```bash
pwd
```
**You'll see:**
```
/home/ubuntu
```
This tells you which folder you're currently "standing in." When you first connect, you always start in `/home/ubuntu`.

### `ls -la` — "what's in this folder?"
```bash
ls -la
```
**You'll see** a list of files and folders, something like:
```
drwxr-xr-x  4 ubuntu ubuntu 4096 Aug 22 02:13 .
drwxr-xr-x  3 root   root   4096 Aug 22 01:40 ..
-rw-r--r--  1 ubuntu ubuntu  220 Mar 31  2024 .bash_logout
drwx------  2 ubuntu ubuntu 4096 Aug 22 02:13 .ssh
```
Each line is one file or folder. Names starting with a `.` are "hidden" files, normally not shown unless you use `-la`.

### `mkdir` — "make a new folder"
```bash
mkdir test-folder
```
**You'll see nothing** — no output means it succeeded. Confirm it by listing again:
```bash
ls -la
```
Now you'll see `test-folder` in the list, because you just created it.

### `cd` — "move into a folder"
```bash
cd test-folder
pwd
```
**You'll see:**
```
/home/ubuntu/test-folder
```
Notice `pwd` now shows you're standing inside the folder you made.

### `cd ..` — "go back up one level"
```bash
cd ..
pwd
```
**You'll see:**
```
/home/ubuntu
```
You're back where you started.

### Clean up your test folder (optional)
```bash
rmdir test-folder
```
This removes the empty folder you made for practice.

---

## Part 4: Set up a home for your project

### Step 4.1 — Go to `/opt`, the standard place for installed software

```bash
cd /opt
ls -la
```
`/opt` is a folder that already exists on every Ubuntu server, meant for extra software you install yourself.

### Step 4.2 — Try to make a folder here

```bash
mkdir services
```
**You will very likely see an error:**
```
mkdir: cannot create directory 'services': Permission denied
```
**Why:** `/opt` is owned by the server's administrator (`root`), and you're logged in as a regular user (`ubuntu`). Regular users can't create things here without special permission.

### Step 4.3 — Do it with administrator permission

```bash
sudo mkdir services
```
`sudo` means "do this as the administrator, just this once." You may be asked to type your password — this is the same as your SSH login, but often there's no separate password needed for `ubuntu` on AWS, so it may just work silently.

**Confirm it worked:**
```bash
ls -la
```
You should now see `services` listed.

### Step 4.4 — Fix the folder's ownership so you can use it normally

Even though the folder now exists, it's still owned by `root`, not you. Fix that:
```bash
sudo chown -R ubuntu:ubuntu services
```
**What this does:** `chown` = "change owner." This says "make `ubuntu` the owner of `services`, and everything inside it (`-R` = recursively)." Now you can create, edit, and delete things inside `services` without needing `sudo` anymore.

### Step 4.5 — Move into it

```bash
cd services
pwd
```
**You'll see:**
```
/opt/services
```
This is now your project workspace. Remember this path — it's where "nadibaby" (or whatever you name your project) will live.

---

## Part 5: Connect your server to GitHub

### Step 5.1 — Create a GitHub account (skip if you have one)

Go to **github.com** and sign up if you haven't already.

### Step 5.2 — Generate a new SSH key, specifically for this server

While standing in `/opt/services` (or really, anywhere — it doesn't matter for this step):
```bash
ssh-keygen -t ed25519
```
**You'll be asked:**
```
Enter file in which to save the key (/home/ubuntu/.ssh/id_ed25519):
```
Just press **Enter** to accept the default location.
```
Enter passphrase (empty for no passphrase):
```
Press **Enter** again to skip (or type one if you want extra security — just remember it).
```
Enter same passphrase again:
```
Press **Enter** again.

**You'll see** a block of text ending in a little art image made of ASCII characters — this confirms the key was created.

### Step 5.3 — Display and copy your public key

```bash
cat ~/.ssh/id_ed25519.pub
```
**You'll see one long line**, starting with `ssh-ed25519` and ending with something like `ubuntu@ip-172-31-9-7`. **Select and copy this entire line** (in most terminals: click and drag to highlight, then Cmd+C or Ctrl+C).

### Step 5.4 — Add it to GitHub

1. In your browser, go to **github.com/settings/keys** (or: click your profile picture → Settings → SSH and GPG keys).
2. Click **New SSH key**.
3. Title: something like `my-ec2-server`.
4. Key: paste the line you copied.
5. Click **Add SSH key**.

### Step 5.5 — Test the connection

Back in your terminal:
```bash
ssh -T git@github.com
```
The first time, you'll see the "authenticity of host" question again — type `yes`.

**You should then see:**
```
Hi <your-github-username>! You've successfully authenticated, but GitHub does not provide shell access.
```
This message is exactly what you want to see — it confirms the connection works.

### Step 5.6 — Tell Git who you are

```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@example.com"
```
No output means success. This information gets attached to every change you save from now on.

---

## Part 6: Get your project onto the server

### Option A — You already have a GitHub repository to clone

Make sure you're standing in the right place first:
```bash
cd /opt/services
pwd
```
Should show `/opt/services`.

Now clone it (replace with your actual repository URL — find this on GitHub by clicking the green **Code** button → **SSH**):
```bash
git clone git@github.com:your-username/your-repo-name.git
```
**You'll see** progress messages like:
```
Cloning into 'your-repo-name'...
remote: Enumerating objects: 6, done.
...
```
When it finishes, check it's there:
```bash
ls -la
cd your-repo-name
pwd
```
You should now see `/opt/services/your-repo-name` — **this is your project folder.**

### Option B — You're starting completely fresh (no repo yet)

1. On GitHub, click the **+** icon (top right) → **New repository**.
2. Name it (e.g. `nadibaby-data-analytics`), keep it **Public** or **Private** as you prefer, and check **Add a README**.
3. Click **Create repository**.
4. Now follow **Option A** above using this new repository's URL.

### If `find` says a folder doesn't exist anywhere

If you're ever unsure where something ended up, this command searches your **entire server**:
```bash
find / -iname "*your-project-name*" 2>/dev/null
```
It will print the full path to anything matching that name, wherever it actually is.

---

## Part 7: Saving your work with Git (do this every time you make a change)

This is the cycle you'll repeat constantly. Make sure you're inside your project folder first (`cd /opt/services/your-repo-name`).

### Step 7.1 — Make a change (example: create a test file)

```bash
touch notes.txt
```
`touch` creates an empty file. Confirm it exists:
```bash
ls -la
```

### Step 7.2 — Check what Git sees

```bash
git status
```
**You'll see:**
```
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        notes.txt
```
This means Git has noticed a new file, but isn't tracking it yet.

### Step 7.3 — Stage the file

```bash
git add notes.txt
git status
```
**You'll now see:**
```
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   notes.txt
```

### Step 7.4 — Commit (save a snapshot)

```bash
git commit -m "add notes file"
```
**You'll see** a short summary confirming the commit was made.

### Step 7.5 — Push (upload it to GitHub)

```bash
git push
```
**You'll see** upload progress, ending with something like `main -> main`.

### Step 7.6 — Confirm everything is saved

```bash
git status
```
**You should see:**
```
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```
**"Nothing to commit, working tree clean" is your green light** — everything is safely backed up on GitHub.

---

## Part 8: Install Docker

Still inside your SSH session on the server:

```bash
sudo apt-get update
```
This refreshes the server's list of available software. You'll see a bunch of "Get" and "Fetched" lines — normal.

```bash
sudo apt-get install ca-certificates curl
```
Type `Y` and press Enter if asked to confirm.

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
No visible output for these — that's expected.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
This is one single command even though it spans multiple lines — copy and paste the whole block at once.

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
This is the actual Docker installation — it'll take a minute or two, with lots of scrolling text. Type `Y` if prompted.

### Verify Docker works

```bash
sudo docker run hello-world
```
**You should see** a friendly message starting with:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Let Docker run without typing `sudo` every time

```bash
sudo groupadd docker
```
(If you see "group already exists," that's fine — ignore it and continue.)
```bash
sudo usermod -aG docker $USER
newgrp docker
```
Now test again, **without** `sudo` this time:
```bash
docker run hello-world
```
If you see the same friendly message as before, it worked.

### Make Docker start automatically after a reboot

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```
You'll see two lines confirming symlinks were created — that's success.

---

## Part 9: Run PostgreSQL in Docker

### Step 9.1 — Generate a strong password

```bash
openssl rand -base64 32
```
**You'll see** a random string of letters, numbers, and symbols. **Copy this somewhere safe** — this will be your database password. I'll refer to it as `YOUR_DB_PASSWORD` below.

### Step 9.2 — Start the database

```bash
docker run --rm --name postgres-dev \
  -e POSTGRES_PASSWORD=YOUR_DB_PASSWORD \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  -d postgres:17
```
Replace `YOUR_DB_PASSWORD` with the password you generated. **Note the `:17`** — this pins a specific version so it never silently changes on you later.

**You'll see** a long string of letters and numbers printed — that's the new container's ID, confirming it started.

### Step 9.3 — Confirm it's running

```bash
docker ps
```
**You should see** one line listing `postgres-dev`, with a status like `Up 5 seconds`.

---

## Part 10: Connect visually with DBeaver

### Step 10.1 — Install DBeaver on your own laptop (not the server)

Go to **dbeaver.io** in your browser, download the **Community Edition** for your operating system, and install it like any normal application.

### Step 10.2 — Open the PostgreSQL port on your server

1. Go back to the AWS Console → EC2 → **Security Groups** (in the left sidebar, under Network & Security).
2. Click on the security group attached to your instance.
3. Click **Inbound rules** → **Edit inbound rules** → **Add rule**.
4. Type: select **PostgreSQL** from the dropdown (this fills in port 5432 automatically).
5. Source: choose **My IP** (safer — only your current internet connection can reach it).
6. Click **Save rules**.

### Step 10.3 — Create the connection in DBeaver

1. Open DBeaver. Click the plug-with-a-plus icon (**New Database Connection**).
2. Choose **PostgreSQL** → **Next**.
3. Fill in:
   - **Host:** `YOUR_SERVER_IP`
   - **Port:** `5432`
   - **Database:** `mydb`
   - **Username:** `postgres`
   - **Password:** `YOUR_DB_PASSWORD`
4. Click **Test Connection...** — you should see a success message.
5. Click **Finish**.

You'll now see your database in DBeaver's left sidebar. Double-click it to browse.

---

## Part 11: Connect from Python

### Step 11.1 — Create your `.env` file

Back in your SSH terminal, make sure you're in your project folder:
```bash
cd /opt/services/your-repo-name
```
Create the file using a simple text editor called `nano`:
```bash
nano .env
```
This opens a text editor inside your terminal. Type (replacing with your real values):
```
PGHOST=localhost
PGPORT=5432
PGDATABASE=mydb
PGUSER=postgres
PGPASSWORD=YOUR_DB_PASSWORD
```
To save and exit `nano`: press **Ctrl+O** (that's the letter O, meaning "output/save"), then **Enter** to confirm the filename, then **Ctrl+X** to exit.

Confirm the file exists:
```bash
cat .env
```
You should see exactly what you typed.

### Step 11.2 — Make sure this file never gets uploaded to GitHub

```bash
nano .gitignore
```
Add a line that says:
```
.env
```
Save and exit the same way (Ctrl+O, Enter, Ctrl+X).

### Step 11.3 — Set up a Python virtual environment

```bash
sudo apt update
sudo apt install python3-venv python3-pip
python3 -m venv .venv
source .venv/bin/activate
```
**You'll notice** your terminal prompt now starts with `(.venv)` — this confirms the virtual environment is active.

### Step 11.4 — Create your requirements file

```bash
nano requirements.txt
```
Type:
```
psycopg2-binary
python-dotenv
```
Save and exit (Ctrl+O, Enter, Ctrl+X).

Install them:
```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```
You'll see download and install progress, ending in a success message.

### Step 11.5 — Write the connection script

```bash
nano db_connect.py
```
Type (or paste) exactly this:
```python
import os
import psycopg2
from dotenv import load_dotenv

load_dotenv()

DB_CONFIG = {
    "host": os.environ.get("PGHOST", "localhost"),
    "port": os.environ.get("PGPORT", "5432"),
    "dbname": os.environ.get("PGDATABASE", "postgres"),
    "user": os.environ.get("PGUSER", "postgres"),
    "password": os.environ.get("PGPASSWORD", ""),
}

def get_connection():
    return psycopg2.connect(**DB_CONFIG)

def main():
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT version();")
            row = cur.fetchone()
            print("Connected to:", row[0])
    finally:
        conn.close()

if __name__ == "__main__":
    main()
```
Save and exit (Ctrl+O, Enter, Ctrl+X).

### Step 11.6 — Run it

```bash
python db_connect.py
```
**You should see:**
```
Connected to: PostgreSQL 17.x ...
```
This one line confirms your Python code successfully talked to your database.

---

## Part 12: Design your first tables

### Step 12.1 — Open a SQL editor

In DBeaver, with your connection open, click **SQL Editor** → **New SQL Script**.

### Step 12.2 — Create a schema

Type into the editor:
```sql
CREATE SCHEMA IF NOT EXISTS operational;
```
Click the ▶ (execute) button, or press Ctrl+Enter / Cmd+Enter. You should see "Success" or similar at the bottom.

### Step 12.3 — Create your first table

```sql
CREATE TABLE IF NOT EXISTS operational.customers (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(150) NOT NULL,
    email VARCHAR(150),
    registration_date DATE
);
```
Run it the same way. In the left sidebar, expand your database → Schemas → operational → Tables — you should now see `customers` listed.

### Step 12.4 — Add a second, related table

```sql
CREATE TABLE IF NOT EXISTS operational.orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date TIMESTAMP NOT NULL
);
```

### Step 12.5 — Link them with a foreign key

```sql
ALTER TABLE operational.orders
    ADD CONSTRAINT fk_orders_customer
    FOREIGN KEY (customer_id) REFERENCES operational.customers(customer_id);
```
This tells PostgreSQL: "every `customer_id` in `orders` must actually exist in `customers`." Try inserting an order with a fake `customer_id` later and you'll see PostgreSQL refuse it — that's this rule protecting your data.

---

## Quick reference: commands you'll use constantly

| I want to... | Command |
|---|---|
| Connect to my server | `ssh -i mykey.pem ubuntu@YOUR_SERVER_IP` |
| See where I am | `pwd` |
| List files here | `ls -la` |
| Make a folder | `mkdir foldername` |
| Move into a folder | `cd foldername` |
| Go back up one level | `cd ..` |
| Create/edit a text file | `nano filename` |
| See what Git has noticed | `git status` |
| Save a change | `git add .` then `git commit -m "message"` then `git push` |
| See running Docker containers | `docker ps` |
| Turn on my Python environment | `source .venv/bin/activate` |
| Disconnect from the server | `exit` |

---


That's completely normal and not a sign you've broken anything — just tell me exactly what command you ran and exactly what appeared on your screen (copy-paste it if you can), and I'll help you figure out the next step from there.
