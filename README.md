Launch one VPS instance with Ubuntu 18.04 LTS

Add swapfile.

```bash
sudo dd if=/dev/zero of=/swap.img bs=4M count=256
sudo chmod 0600 /swap.img
sudo mkswap /swap.img

cat <<EOF | sudo tee -a /etc/fstab
/swap.img none swap sw 0 0
EOF

sudo swapon -a
```

Add another SSH port.

```bash
cat <<EOF | sudo tee -a /etc/ssh/sshd_config

Port 22
Port 10022  # YOUR ANOTHER PORT
EOF

sudo systemctl reload ssh
```

Modify the instance's firewall rule to allow the alternate SSH port.

Install package(s).

```bash
sudo apt update
sudo apt install ansible
```

Clone.

```bash
git clone https://github.com/nobuto-m/vps-setup.git
cd vps-setup/
```

Run

```bash
ansible-playbook -vv local.yml
```

Let's encrypt

```bash
sudo certbot certonly --apache -d '<DOMAIN>'
```

Add an user to thelounge

```
sudo -u thelounge thelounge add '<USERNAME>'
```
