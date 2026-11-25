# How I set it up

## 1. Azure VM

I used my Azure for Students subscription, which doesn't need a credit card.

- Ubuntu Server 24.04 LTS (x64)
- B2as v2: 2 vCPU, 8 GB RAM. Wazuh's quickstart recommends 8 GB for a small all-in-one install, and the burstable B-series is cheaper than the general purpose D-series for a lab that's idle most of the time.
- 64 GB Standard SSD (the default 30 GB is too small for Wazuh)
- SSH key login only
- Auto-shutdown every night at 11 PM

**Problem:** my first deployment failed with `RequestDisallowedByAzure`. Student subscriptions can only deploy to a short list of regions. I found the allowed list under Policy → Assignments → "Allowed resource deployment regions" and picked North Central US, the closest one to Ohio.

## 2. Network rules

I changed every inbound rule so it only accepts my IP address:

| Port | What it's for |
| --- | --- |
| 22 | SSH into the server |
| 443 | Wazuh dashboard |
| 1514–1515 | Wazuh agent enrollment and log traffic |

My IP changes depending on where I am, so I update these rules when I switch networks.

## 3. Installing Wazuh

After updating the server, I ran Wazuh's installation assistant with the all-in-one option:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

It takes about 10–15 minutes and prints the admin password at the end.

**Problem:** the dashboard gave me `ERR_CONNECTION_RESET` the first time I tried to open it. On the server, the dashboard service was running and `curl -k -I https://localhost` returned a normal response, so Wazuh was fine. I was on a guest Wi-Fi network at the time, and it was cutting off HTTPS connections to the VM. It worked as soon as I tried from home.

**Problem:** after restarting the VM, the dashboard said "Wazuh dashboard server is not ready yet." The indexer takes a few minutes to start on an 8 GB VM. Waiting fixed it.

## 4. Mac agent

I used the "Deploy new agent" page in the dashboard to generate the install commands for macOS (Apple Silicon), then ran them on my Mac.

**Problem:** the agent showed up as "Never connected." It had registered with the server but wasn't running yet. Starting it fixed it:

```bash
sudo /Library/Ossec/bin/wazuh-control start
```
