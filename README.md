# OpenCanary Honeypot — Kali Linux
## Project Overview

This project implements OpenCanary on Kali Linux to detect unauthorized network service interactions and send automated email notifications through Gmail SMTP.

OpenCanary simulates network services and records suspicious activity, allowing users to monitor attempted connections and authentication attempts.

## Features

- SSH honeypot monitoring on port 22
- FTP honeypot monitoring on port 21
- Telnet honeypot monitoring on port 23
- MySQL honeypot monitoring on port 3306
- Automated Gmail email notifications
- Local JSON event logging
- Windows-to-Kali connectivity testing
- VirtualBox NAT and Host-only networking

## 1. System Requirements

- Kali Linux installed in VirtualBox
- Python 3, pip, and Python virtual environment
- OpenCanary
- Gmail account with 2-Step Verification enabled
- Google App Password
- Windows host for testing

## 2. Install OpenCanary on Kali Linux

Update Kali Linux and install the required Python packages.

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv -y
```

Create and activate a Python virtual environment.

```bash
cd /home/kali
python3 -m venv env
source env/bin/activate
```

Install OpenCanary.

```bash
pip install opencanary
```

Verify the installation.

```bash
which opencanaryd
pip show opencanary
```

Create the default configuration.

```bash
opencanaryd --copyconfig
```

If needed, create the system configuration directory and copy the generated configuration into it.

```bash
sudo mkdir -p /etc/opencanaryd
sudo cp ~/.opencanary.conf /etc/opencanaryd/opencanary.conf
```

If a working configuration already exists, do not overwrite it.

## 3. Configure OpenCanary Services

Open the configuration file.

```bash
sudo nano /etc/opencanaryd/opencanary.conf
```

Enable the following services within the existing JSON configuration.

```json
"ftp.enabled": true,
"ftp.port": 21,
"ssh.enabled": true,
"ssh.port": 22,
"telnet.enabled": true,
"telnet.port": 23,
"mysql.enabled": true,
"mysql.port": 3306
```

Save using `Ctrl + O`, `Enter`, and `Ctrl + X`.

## 4. Configure Gmail Email Notifications

### Generate a Google App Password

1. Enable 2-Step Verification on your Google account.
2. Visit https://myaccount.google.com/apppasswords.
3. Generate a 16-character App Password for OpenCanary.

### Modify the OpenCanary Logger

Open the configuration file.

```bash
sudo nano /etc/opencanaryd/opencanary.conf
```

Locate the `logger` section and configure its `handlers` object.

```json
"handlers": {
    "console": {
        "class": "logging.StreamHandler",
        "stream": "ext://sys.stdout"
    },
    "file": {
        "class": "logging.FileHandler",
        "filename": "/var/tmp/opencanary.log"
    },
    "SMTP": {
        "class": "logging.handlers.SMTPHandler",
        "mailhost": ["smtp.gmail.com", 587],
        "fromaddr": "YOUR_EMAIL@gmail.com",
        "toaddrs": ["RECIPIENT_EMAIL@example.com"],
        "subject": "OpenCanary Security Alert",
        "credentials": [
            "YOUR_EMAIL@gmail.com",
            "YOUR_GOOGLE_APP_PASSWORD"
        ],
        "secure": []
    }
}
```

Replace the placeholders with your email addresses and Google App Password.

Ensure the existing logger configuration includes `"SMTP"` in its enabled handler list.

**Security:** Never upload a configuration containing your actual App Password to GitHub.

## 5. Validate and Start OpenCanary

Check that the configuration contains valid JSON.

```bash
python3 -m json.tool /etc/opencanaryd/opencanary.conf
```

Start OpenCanary using the Python virtual environment.

```bash
cd /home/kali
source env/bin/activate
sudo /home/kali/env/bin/opencanaryd --dev
```

Verify the listening ports.

```bash
sudo ss -ltnp
```

## 6. Test Email Notifications

From Windows PowerShell, replace `<KALI-IP>` with the Kali VM's Host-only IP address.

```powershell
ssh admin@<KALI-IP>
```

Enter a fake password to generate an authentication attempt.

On Kali, monitor the OpenCanary logs.

```bash
sudo tail -f /var/tmp/opencanary.log
```

When OpenCanary detects the interaction, it records the event and sends an email through the configured Gmail SMTP handler.

The notification subject is:

```text
OpenCanary Security Alert
```

## Project Outcome

OpenCanary was successfully installed on Kali Linux and configured to monitor SSH, FTP, Telnet, and MySQL interactions. Gmail SMTP integration enabled automated security alerts, while local logging provided a record of detected activity.

This setup provides a foundation for future integration with web honeypots and security monitoring tools.
