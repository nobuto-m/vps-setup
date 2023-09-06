Launch one VPS instance with Ubuntu 20.04 LTS

Add swapfile.

```bash
sudo dd if=/dev/zero of=/swap.img bs=10M count=200
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
   - Prometheus Node Exporter Full - https://grafana.com/grafana/dashboards/1860-node-exporter-full/
   - Prometheus Node Exporter Full Old - https://github.com/rfrail3/grafana-dashboards/blob/master/prometheus/node-exporter-full-old.json
   - Prometheus Blackbox Exporter - https://grafana.com/grafana/dashboards/7587-prometheus-blackbox-exporter/
   - Blackbox Exporter (HTTP prober) - https://grafana.com/grafana/dashboards/13659-blackbox-exporter-http-prober/
   - Apache - https://grafana.com/grafana/dashboards/3894-apache/

Cockpit:

Set the OS user password for Cockpit.

```
sudo passwd $USER
```

Use a proper cert.

```
cat <<"EOF" | sudo tee /etc/letsencrypt/renewal-hooks/deploy/cockpit.sh
#!/bin/bash

set -e
set -u

if [ "$RENEWED_LINEAGE" = /etc/letsencrypt/live/grafana-wildcard ]; then
    cat "$RENEWED_LINEAGE/fullchain.pem" "$RENEWED_LINEAGE/privkey.pem" \
        | install -o root -g root -m 0600 /dev/stdin /etc/cockpit/ws-certs.d/90-wildcard.cert

    systemctl is-active cockpit.service >/dev/null && systemctl restart cockpit.service
fi
EOF

sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/cockpit.sh
```
