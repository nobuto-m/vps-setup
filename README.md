Launch one VPS instance with Ubuntu 18.04 LTS

Add swapfile.

```bash
sudo dd if=/dev/zero of=/swap.img bs=4M count=128
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
Then restrict access to the port 22 except for "browser SSH".

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

Let's encrypt for www-home.

```bash
sudo certbot certonly --apache -d '<DOMAINS (e.g., www.example.com,example.com)>'
```

Then, enable the site.

```bash
sudo a2ensite www-home.conf
sudo systemctl reload apache2
```

Enable VPN.

```
sudo tailscale up --hostname '<visible hostname>'
```

Let's encrypt wildcard cert for Grafana.

```bash
sudo certbot certonly \
    -i nginx \
    --dns-cloudflare \
    --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
    --cert-name grafana-wildcard \
    -d '<DOMAIN (e.g., *.t.example.com)>'
```

Enable the reverse proxy with the alternate port.

```bash
sudo ln -s ../sites-available/grafana /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

Grafana:
1. change the admin password
1. set up the default data source
1. import dashboards
   - default ones under `datasources`
   - Prometheus Node Exporter Full - https://grafana.com/grafana/dashboards/1860
   - Prometheus Node Exporter Full Old - https://github.com/rfrail3/grafana-dashboards/blob/master/prometheus/node-exporter-full-old.json
