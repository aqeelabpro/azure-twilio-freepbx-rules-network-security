# Azure NSG Rules for Twilio SIP Trunking + HTTPS Access(FreePBX)

This repository provides Azure CLI scripts and guidance to configure **inbound Network Security Group (NSG) rules** for:

- ✅ Twilio Elastic SIP Trunking (Signaling + Media)
- 🔒 HTTPS access (port 443) to Azure Virtual Machines
- 🧪 Secure and region-aware **VoIP testing environments** using FreePBX, Asterisk, or other SIP-based systems

---

> ⚠️ **DISCLAIMER**  
> These configurations are intended for **testing and development use only**.  
> Do **not** use them in production without additional security measures such as:
> - IP allowlists
> - DDoS protection and rate limiting
> - HTTPS-only hardened web UIs
> - Proper logging, alerts, and audits

---

## 🔒 Inbound Rules Overview

### 🔹 Twilio SIP Signaling (5060/5061)

| Region              | IP Range             |
|---------------------|----------------------|
| North America (VA)  | 54.172.60.0/30       |
| North America (OR)  | 54.244.51.0/30       |
| Europe (Ireland)    | 54.171.127.192/30    |
| Europe (Frankfurt)  | 35.156.191.128/30    |
| APAC (Tokyo)        | 54.65.63.192/30      |
| APAC (Singapore)    | 54.169.127.128/30    |
| APAC (Sydney)       | 54.252.254.64/30     |
| South America (SP)  | 177.71.206.192/30    |

**Ports**:  
- 5060 (UDP/TCP)  
- 5061 (TCP/TLS)

---

### 🔹 Twilio Media (RTP/SRTP)

| IP Range        | Protocol | Ports         |
|-----------------|----------|---------------|
| 168.86.128.0/18 | UDP      | 10000–60000   |

---

### 🔹 HTTPS Access (Port 443)

| Source | Protocol | Port |
|--------|----------|------|
| `*`    | TCP      | 443  |

---

## 🛠️ Azure CLI Setup

Replace `<nsg-name>` and `<resource-group>` with your actual values.

### 1. Allow Twilio Media
```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name Allow-Twilio-Media \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Udp \
  --source-address-prefixes 168.86.128.0/18 \
  --destination-port-ranges 10000-60000 \
  --description "Allow Twilio RTP media"
````

### 2. Allow Twilio SIP Signaling

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name Allow-Twilio-SIP \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol '*' \
  --source-address-prefixes \
    54.172.60.0/30 \
    54.244.51.0/30 \
    54.171.127.192/30 \
    35.156.191.128/30 \
    54.65.63.192/30 \
    54.169.127.128/30 \
    54.252.254.64/30 \
    177.71.206.192/30 \
  --destination-port-ranges 5060 5061 \
  --description "Allow Twilio SIP signaling"
```

### 3. Allow HTTPS (Port 443)

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name Allow-HTTPS \
  --priority 210 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-port-ranges 443 \
  --description "Allow secure HTTPS access"
```

---

## 📌 Best Practices for Production (Not Included)

For production deployments, it's highly recommended to:

* Restrict access to specific trusted IPs
* Enable HTTPS-only admin panels and disable HTTP
* Use Azure Application Gateway or Firewall
* Configure NSG Flow Logs and alerts
* Patch your VoIP software and OS regularly

---

## 📚 References

* [Twilio SIP IP Address Guide](https://www.twilio.com/docs/sip-trunking/ip-addresses)
* [Azure NSG Docs](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
* [FreePBX on Azure Setup](https://wiki.freepbx.org/display/FPG/Azure)

---

## 🧾 License

This project is licensed under the [MIT License](LICENSE).

---

## 🙌 Contributing

Contributions and pull requests are welcome to improve rule management, support more VoIP providers, or harden production recommendations.
