# Day 01: First VPS setup (OVH)

**Date:** 2026-09-23
**Server:** OVH VPS, Ubuntu 26.04 LTS, 37 GB disk
**Hostname:** vps-be3400b8.vps.ovh.net
**IP:** 141.95.16.90
**User:** ubuntu

---

## 1. Getting access
- OVH emails the access details plus a one-time link to generate the password. The link is valid 30 days, or 7 days once opened. Don't share it.
- The **KVM** button in the OVH manager is a virtual screen and keyboard. It's the emergency way in if SSH ever breaks.

```bash
ssh ubuntu@141.95.16.90
```
The first connection asks you to trust the server's fingerprint. Type `yes` and it gets saved in `~/.ssh/known_hosts`.

## 2. SSH keys instead of passwords
Bots brute-force new servers within minutes, so passwords get turned off.

```bash
# on MY computer
ssh-keygen -t ed25519                 # creates ~/.ssh/id_ed25519 (private) + .pub (public)
ssh-copy-id ubuntu@141.95.16.90       # puts the public key in the server's ~/.ssh/authorized_keys

# test that the key alone works (no password fallback)
ssh -o PasswordAuthentication=no ubuntu@141.95.16.90
```

Then, on the server, disable password login:
```bash
echo "PasswordAuthentication no" | sudo tee /etc/ssh/sshd_config.d/00-hardening.conf
sudo systemctl restart ssh
sudo sshd -T | grep passwordauthentication   # should say: no
```
- The `00-` prefix matters. SSH uses the **first** value it reads, and Ubuntu's cloud-init config (`50-cloud-init.conf`) can switch passwords back on.
- **Rule:** keep one session open and test the login from a *second* terminal before closing anything.

## 3. Updates, reboot, firewall
```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot                      # needed after kernel updates ("System restart required")

sudo ufw allow OpenSSH           # ALWAYS allow SSH before enabling the firewall
sudo ufw enable
```

## 4. First website with Nginx
```bash
sudo apt install nginx
sudo ufw allow 'Nginx Full'      # ports 80 + 443
systemctl status nginx
```
- Web root: `/var/www/html/`
- Ubuntu's default page is `index.nginx-debian.html`. Nginx serves `index.html` first when it exists.
- Edit directly: `sudo nano /var/www/html/index.html` (Ctrl+O save, Ctrl+X exit)

Upload from my computer:
```bash
sudo chown -R ubuntu:ubuntu /var/www/html        # on server: make the folder mine
scp index.html ubuntu@141.95.16.90:/var/www/html/ # from my computer
scp -r mysite/* ubuntu@141.95.16.90:/var/www/html/
```

---

## Mistakes and lessons
1. **Always check the prompt before running a command.** `ubuntu@vps-be3400b8:~$` is the server and `dinos@InfinityGear:~$` is my machine. I ran an SSH test while already *on* the server, so it tried to connect to itself and got denied.
2. **WSL and Windows have separate `.ssh` folders.** My key is in WSL (`/home/dinos/.ssh`). `scp` from Windows CMD looked in `C:\Users\User\.ssh`, found no key, and got `Permission denied (publickey)`, which is expected once passwords are off. Fixes:
   - Run it from WSL: `cd /mnt/c/Users/User/Desktop/cloud/nosdiproject`
   - Or copy the key to Windows: `cp ~/.ssh/id_ed25519* /mnt/c/Users/User/.ssh/`
3. `Permission denied (publickey)` means a key problem. `Permission denied` while writing a file means a folder ownership problem. They are different errors.

## Useful commands
| Command | What it does |
|---|---|
| `systemctl status/restart <service>` | check or restart a service |
| `sudo ufw status` | show firewall rules |
| `sudo sshd -T` | show the SSH config actually in effect |
| `cat ~/.ssh/authorized_keys` | keys allowed to log in |
| `exit` | leave the server and go back to my machine |

---

## Roadmap
- [x] Server access with SSH keys, passwords off
- [x] Updates and firewall
- [x] Nginx serving my own page
- [ ] **Next:** domain + DNS A record → HTTPS with certbot
- [ ] Deploy a real app (React build, or Node behind an Nginx reverse proxy)
- [ ] Docker + docker compose, self-host something (Uptime Kuma / Vaultwarden)
- [ ] GitHub Actions auto-deploy, backups, reading logs (`journalctl`)
- [ ] Ansible: rebuild the whole server from a script
