## Goal

Install the Proton VPN GUI on Kali with XFCE, log in with a Proton account, and use it in Free mode safely and reliably.
## Prerequisites

- Kali Linux updated with XFCE environment.
   
- User with sudo privileges.

- Working internet connection.

Update the system:

```bash
sudo apt update && sudo apt full-upgrade -y
```

## Create a Proton account (Free plan)

1. Go to proton.me and select **Create account**.

2. Choose **Proton Free**.

3. Complete registration and verify email/phone if required.

4. (Optional but recommended) Enable 2FA:
    - Account settings → **Security** → **Two-factor authentication** → **Authenticator app (TOTP)**.
    - Save the **recovery codes** offline.

> Note: the Proton account is shared across Mail, Drive, Calendar, and VPN. The same credentials are used for the VPN app.
## Install Proton VPN GUI on Kali XFCE

Download the .deb package and install it. Version referenced: **1.0.8**. https://protonvpn.com/support/official-linux-vpn-debian/

```bash
wget https://repo.protonvpn.com/debian/dists/stable/main/binary-all/protonvpn-stable-release_1.0.8_all.deb
sudo dpkg -i ./protonvpn-stable-release_1.0.8_all.deb && sudo apt update
sudo apt install proton-vpn-gnome-desktop
sudo apt --fix-broken install -y
```

Verify the installation:

```bash
apt list --installed | grep -i proton-vpn
```
## Launch the app and login

- Start from menu: **Internet → Proton VPN** (or search “Proton VPN”).

- Or via terminal:

```
proton-vpn-gnome-desktop &
```

- Select **Sign in** with the Proton account created earlier.

> On XFCE the GUI requires GNOME libraries. `apt --fix-broken install` handles them automatically.
## Use in Free mode

1. After login, the main screen shows **Quick Connect** and the list of **Countries/Servers**.

2. For immediate connection: **Quick Connect**.

3. To select manually: go to **Countries** and pick a **Free** server location.

4. Wait until status shows **Connected**. The icon changes and your public VPN IP appears.

5. To disconnect: **Disconnect**.

Free plan limitations:

- Fewer locations available.
- No streaming servers, Secure Core, or port forwarding.
- Lower bandwidth priority than paid plans.
## Recommended settings

Open **Settings** in the app.

- **Protocol**: **Automatic**. App chooses WireGuard/OpenVPN depending on the network.

- **Kill Switch**: **On**. Blocks traffic if VPN drops.

- **Auto‑connect on startup**: **Fastest** if you want VPN at boot.
## Verify it works

Check public IP and DNS.

Public IP:

```bash
curl -4 ifconfig.me; echo
curl -6 ifconfig.me; echo  # if your ISP/network supports IPv6
```

DNS leak test: visit any “DNS leak test” website while connected. You should see Proton resolvers or the VPN server.
## Troubleshooting

**A. “dpkg: dependency problems prevent configuration”**

```bash
sudo apt --fix-broken install -y
```

**B. GUI doesn’t start**

```bash
proton-vpn-gnome-desktop
```

Read terminal output. Update packages and retry.

**C. No internet after crash or Kill Switch**

1. Disconnect and temporarily disable **Kill Switch** in the app.
2. Restart NetworkManager:
3. 
```bash
sudo systemctl restart NetworkManager
```

3. Reset DNS if needed:

```bash
sudo resolvectl flush-caches 2>/dev/null || true
```

4. Restart the app and reconnect.

**D. Slowness or instability**

- Switch server or protocol (Automatic recommended).

- Avoid congested Wi‑Fi. Test with ethernet.
## Autostart on XFCE login

Enable via app (**Auto‑connect on startup**) or XFCE settings:

Settings → **Session and Startup** → **Application Autostart** → **Add** → Name “Proton VPN” → Command:

```
proton-vpn-gnome-desktop
```

---
## Clean uninstall

```bash
sudo apt remove --purge -y proton-vpn-gnome-desktop proton-vpn-gtk-app
sudo apt autoremove -y
```

Remove installer if needed:

```bash
rm -f ~/proton-vpn-gnome-desktop_1.0.8_all.deb /tmp/pv.deb
```
## Best practices

- Keep Kali updated.

- Keep Kill Switch enabled to avoid leaks if connection drops.

- Regularly verify IP and DNS.

- Re-test when changing ISP or network.