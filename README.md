# Azure NSG Rules for Twilio SIP Trunking + HTTP Access(FreePBX)

This repository provides Azure CLI scripts and guidance to configure **inbound Network Security Group (NSG) rules** for:

- ✅ Twilio Elastic SIP Trunking (Signaling + Media)
- 🌐 HTTP access to Azure Virtual Machines
- 🔐 Region-aware VoIP deployments (e.g., FreePBX, Asterisk)

---

> ⚠️ **DISCLAIMER**  
> This setup is intended for **testing and development purposes only**.  
> It is **not recommended for production environments** without further hardening, such as:
> - Source IP restrictions
> - Firewall and DDoS protection
> - Monitoring and logging
> - Rate limiting and security auditing

---

## 🚀 Use Case

For users deploying VoIP systems in Microsoft Azure (e.g., FreePBX, Asterisk, Kamailio), these NSG rules ensure:

- SIP ports (`5060`, `5061`) are open to **Twilio’s signaling gateways**
- RTP media ports (`10000–60000`) are open for **Twilio’s media traffic**
- Port `80` is publicly accessible for web-based UIs (admin panels, etc.)

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

### 🔹 HTTP Access
| Source | Protocol | Port |
|--------|----------|------|
| `*`    | TCP      | 80   |

---

## 🛠️ Azure CLI Setup

Replace `<nsg-name>` and `<resource-group>` with your own.

### 1. Twilio Media Ports
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

### 2. Twilio SIP Signaling

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

### 3. HTTP (Port 80)

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name Allow-HTTP \
  --priority 200 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-port-ranges 80 \
  --description "Allow HTTP access"
```

---

## 📌 Best Practices for Production (Not Included Here)

If deploying in production:

* Restrict access to specific trusted IPs
* Use HTTPS (port 443) and disable HTTP
* Implement Azure Firewall or WAF
* Log and monitor traffic
* Apply updates and security patches regularly

---

## 📚 Resources

* [Twilio SIP Trunking IPs](https://www.twilio.com/docs/sip-trunking/ip-addresses)
* [Azure NSG Documentation](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
* [FreePBX on Azure](https://wiki.freepbx.org/display/FPG/Azure)

---

## 🙌 Contributing

Contributions are welcome!
Submit issues or pull requests to support additional VoIP providers or improve rule management automation.
