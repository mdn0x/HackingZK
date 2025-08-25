## Goal

Browse the Tor network securely by running Tor Browser over ProtonVPN. This creates a **VPN → Tor** chain. This guide assumes ProtonVPN is already installed and working (see the previous ProtonVPN guide → [Proton VPN on Kali XFCE Complete Guide (account → installation → free use)](./Proton%20VPN%20on%20Kali%20XFCE%20Complete%20Guide%20(account%20%E2%86%92%20installation%20%E2%86%92%20free%20use).md)).
## Prerequisites

- Kali Linux.
- ProtonVPN installed and configured (connected before using Tor).
- User with sudo privileges.
- Internet connection.

Update the system first:

```
sudo apt update && sudo apt full-upgrade -y
```
## Install Tor Browser on Kali XFCE

The Tor Project recommends installing via the official repository.
### Add Tor Project repository

```
echo "deb https://deb.torproject.org/torproject.org bookworm main" | sudo tee /etc/apt/sources.list.d/tor.list
```
### Import GPG key

```
wget -qO - https://deb.torproject.org/torproject.org/pubkey.asc | sudo gpg --dearmor -o /usr/share/keyrings/tor-archive-keyring.gpg
```
### Update and install

```
sudo apt update
sudo apt install -y torbrowser-launcher
```
### First run

```
torbrowser-launcher
```

It downloads and verifies the latest Tor Browser package from the Tor Project.
## Launching Tor over VPN

1. **Connect ProtonVPN first** using the GUI as shown in the guide.

2. Once VPN is connected, start Tor Browser:

```
torbrowser-launcher
```

3. On first launch, choose **Connect** (default). The browser will establish a Tor circuit.

Now your traffic flow is: **Your device → ProtonVPN → Tor network → Destination**.
## Security settings in Tor Browser

Open Tor Browser → top right menu → **Settings** → **Privacy & Security**.

- **Security Level**: Set to **Safer** or **Safest** to reduce attack surface.

- **HTTPS-Only Mode**: Enable.

- **Disable JavaScript** (optional, breaks some sites but increases safety).

- Avoid installing browser add-ons. They reduce anonymity.
## Legal Onion Search Engines and Directories

Use only trusted resources. Examples:

- **DuckDuckGo Onion**: `https://duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion`

- **Tor Project official onion**: `http://expyuzz4wqqyqhjn.onion`


Directories like _Hidden Wiki_ exist but may contain unsafe or illegal links. Rely instead on official or community-trusted resources.
## Best Practices for Tor + VPN

- Always start ProtonVPN **before** Tor Browser.

- Keep Tor Browser updated (it auto-updates via torbrowser-launcher).

- Do not resize the browser window (can leak screen size fingerprint).

- Avoid logging into personal accounts (Google, Facebook) inside Tor.

- Do not open documents (PDF, DOCX) downloaded over Tor outside the Tor Browser.

- Use separate sessions for sensitive browsing.
## Verification

Check your Tor exit IP:

- Visit `https://check.torproject.org/` It should confirm you are connected to Tor.

You can also check your VPN IP before opening Tor:

```
curl -4 ifconfig.me
```

It should show the ProtonVPN address, not your ISP.

---
## Uninstall Tor Browser

```
sudo apt remove --purge -y torbrowser-launcher
sudo apt autoremove -y
```