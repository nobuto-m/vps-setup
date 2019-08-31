Launch one VPS instance with Ubuntu 18.04 LTS

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



prefetch: true
prefetchStorage: true

