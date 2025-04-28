# Setting Up a Mail Server on Ubuntu Server 24.04

VM Configuration:
• Name: mail-server
• OS: Ubuntu Debian 12
• CPU: 2 Cores
• RAM: 6GB
• Disk: 32GB
• Network: Bridge mode (vmbr0)
• Domain: firma.ua
• Time Zone: Asia/Yerevan
# How to Install and Configure Postfix and Dovecot on Debian 12 (Local Mail Server)

---

## Terminology Explained

- **Postfix**: A Mail Transfer Agent (MTA) responsible for sending and receiving emails.
- **Dovecot**: An IMAP/POP3 server used for receiving and storing emails securely.
- **Mailbox**: Storage location for emails; we use Maildir format (each email is a separate file).
- **Local Mail**: All emails are stored and accessed inside the server, without external internet relay.

---

## Step 1: Install Postfix

```bash
sudo apt update
sudo apt install postfix -y
```

- During installation, select "Internet Site."
- Set the system mail name (your local domain, e.g., `example.local`).

Reconfigure Postfix if needed:

```bash
sudo dpkg-reconfigure postfix
```

- **Home Mailbox**:
```bash
sudo postconf -e 'home_mailbox= mail/'
```

- **Virtual Alias Map**:
```bash
sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'
sudo nano /etc/postfix/virtual
sudo postmap /etc/postfix/virtual
```

Restart Postfix:

```bash
sudo systemctl restart postfix
sudo systemctl status postfix
```

Check open ports:

```bash
netstat -tlpn
```

You should see port `25` (SMTP) open.

### Verify Postfix Installation

```bash
postconf mail_version
```

- Displays Postfix version if installed correctly.

---

## Step 2: Create Local Email Users

```bash
sudo adduser user1
sudo adduser user2
sudo adduser user3
sudo adduser user4
```

Each user will have their mailbox under `/home/username/mail/`.

---

## Step 3: Install Dovecot

```bash
sudo apt install dovecot-imapd dovecot-pop3d -y
```

### Verify Dovecot Installation

```bash
dovecot --version
```

### Check if Dovecot is running

```bash
sudo systemctl status dovecot
```

---

## Step 4: Basic Setup for Dovecot

Edit mailbox settings:

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Find and set:

```bash
mail_location = maildir:~/Maildir
```

Restart Dovecot:

```bash
sudo systemctl restart dovecot
```

---

## Step 5: Configure Mail Domain (Local Domain)

Ensure `/etc/hosts` contains:

```bash
127.0.0.1   localhost example.local
```

Replace `example.local` with your server hostname if needed.

---

## Step 6: Setup Email Client (Thunderbird)

**Settings for Thunderbird email client:**

- **Incoming Mail Server (IMAP or POP3):**
  - Server: your-server-ip (e.g., 192.168.1.10)
  - IMAP Port: 143 (for IMAP)
  - POP3 Port: 110 (for POP3)
  - Connection Security: STARTTLS or None (for local testing)

- **Outgoing Mail Server (SMTP):**
  - Server: your-server-ip
  - SMTP Port: 25
  - Connection Security: STARTTLS or None (for local testing)

- **Authentication:**
  - Username: system username (e.g., user1)
  - Password: the password created via `adduser`

**Thunderbird Setup Steps:**

1. Open Thunderbird > Add New Account.
2. Enter Name, Email (e.g., user1@example.local), and Password.
3. Click "Manual Config."
4. Fill in the settings as shown above.
5. Click "Done."

---

# Helpful Commands

| Task | Command |
|:-----|:--------|
| Reload Postfix config | `sudo systemctl reload postfix` |
| Reload Dovecot config | `sudo systemctl restart dovecot` |
| See mail queue (Postfix) | `mailq` |
| Force flush mail queue | `postqueue -f` |

---

# Conclusion

- Postfix handles outgoing and local delivery of emails.
- Dovecot manages client access to mailboxes.
- Using Maildir format ensures better mail storage (one file per email).
- Fully local setup means emails never leave your server.

You now have a working local email server for testing and internal communication with Thunderbird!

---

# End of Documentation

