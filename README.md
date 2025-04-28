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
- Set the system mail name (your local domain, e.g., `firma.ua`).

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
```
this file our data storage for users(e.g., ` mail`user1@firma.ua`     `username`user1`)

```
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
```

Each user will have their mailbox under `/home/username/mail/`.

---

## Step 3: Install Dovecot

```bash
sudo apt install dovecot-imapd dovecot-core dovecot-pop3d -y
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

for a full working Postfix+Dovecot integration, especially when SMTP Authentication is needed, we must also configure:

Edit mailbox settings:

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Find and set:

In most modern Dovecot setups, Maildir default location is ~/Maildir (with capital M) ```mail_location = maildir:~/Maildir```.
BUT — in my Postfix setup I clearly defined this line:

 So, Dovecot must match where Postfix delivers emails. Therefore, the correct setting for me is:
 
```bash
mail_location = maildir:~/mail
```
Otherwise Dovecot would look in wrong directory and not find my mails!


```bash
sudo nano /etc/dovecot/dovecot.conf
```

Find and set: 
```  listen= *,:```➔ Makes Dovecot listen on all network interfaces (important for IMAP and SMTP authentication).

```bash
sudo nano -l /etc/dovecot/conf.d/10-auth.conf
```

Find and set:  
```  disable_plaintex_auth=no ```     ```auth_mechanisms=plain login```

➔ Allows Dovecot to accept plaintext login when STARTTLS is not configured (local testing).

```bash
sudo nano -l /etc/dovecot/conf.d/10-master.conf
```

Find and set:

```bash
unix_listner /var/spool/postfix/auth {
  mode= 0666
  user=postfix
  group=postfix
}
```
➔ So that Postfix can authenticate SMTP users via Dovecot.

Restart Dovecot:

```bash
sudo systemctl restart dovecot
sudo systemctl status dovecot
```
If you want to see open ports:

```netstat -tlpn``` 

If you want to check if firewall (ufw) is running: ```sudo ufw status``
If ufw is not installed, install it: ```sudo apt install ufw```
sudo ufw enable ```sudo ufw enable```
Open required ports: 
```
sudo ufw allow 25/tcp
sudo ufw allow 110/tcp
sudo ufw allow 143/tcp

```


🔥 Ports You Need to Open (Postfix + Dovecot + Thunderbird):
| Task | Command |Command |Command |Command |
Port | Protocol | Service | Description | Use
25 | TCP | SMTP (Postfix) | Sending emails between servers and from client (Thunderbird) if you configure SMTP | Required
110 | TCP | POP3 (Dovecot) | Receiving emails by downloading to client (Thunderbird POP3 account) | Optional if using POP3
143 | TCP | IMAP (Dovecot) | Receiving emails (Thunderbird IMAP account, emails stay on server) | Recommended instead of POP3
587 | TCP | SMTP (Submission) (Postfix) | Sending email securely from client to server (better than port 25) | Highly Recommended
993 | TCP | IMAPS (Dovecot) | Secure IMAP (IMAP over SSL/TLS) | Optional if you use SSL/TLS
995 | TCP | POP3S (Dovecot) | Secure POP3 (POP3 over SSL/TLS) | Optional if you use SSL/TLS

✅ Minimum for you (local testing without SSL):

Port 25 → for SMTP sending

Port 110 → for POP3 receiving (if you use POP3)

Port 143 → for IMAP receiving (if you use IMAP)

(Thunderbird will need either 110 or 143 depending on your choice.)

---
✅ So in my case, everything must be aligned:

Postfix writes mails to ~/mail/

Dovecot reads mails from ~/mail/

Dovecot allows authentication

Dovecot listens on all interfaces

Postfix can authenticate clients (Thunderbird) over port 25.

## Step 5: Configure Mail Domain (Local Domain)

Ensure `/etc/hosts` contains:

```bash
127.0.0.1   localhost firma.ua
```

Replace `firma.ua` with your server hostname if needed.

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

