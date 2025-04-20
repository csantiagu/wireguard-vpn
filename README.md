
# 🔐 WireGuard VPN Setup and Client Automation using Ansible with Kill Switch

This project provides an automated and repeatable way to deploy a **WireGuard VPN server** and generate **client configuration** files using Ansible. It also includes optional tasks to **enable VPN routing/NAT** for internet access through the server.

---

## 🚀 Key Features

- **Automated WireGuard Server Setup**  
  Brings up `wg0` interface with all required settings via a Jinja2-configured template.

- **Client Configuration with Parameters**  
  Easily create client configs (like Windows `.conf` files) by passing variables at runtime.

- **VPN Routing & Internet Access**  
  Includes a playbook to enable NAT forwarding, allowing VPN clients to access the internet through the server.

- **No hardcoding required**  
  No secrets or keys are stored in the repo — everything is generated or passed dynamically.

---

## 📁 Project Structure

```
wireguard-vpn-automation/
├── playbook.yml                   # Sets up server or client config
├── enable-vpn-routing.yml         # Enables IP forwarding and NAT
├── inventory                      # Ansible inventory (target hosts)
├── clients/
│   └── client-config_windows.yml  # Parameter-driven template (used with vars)
├── templates/
│   └── wg0.conf.j2                # Jinja2 template for WireGuard server config
```

---

## ⚙️ Requirements

- A remote/local server with:
  - Ubuntu/Debian
  - OpenSSH access
  - WireGuard installed (`apt install wireguard`)

- Your control machine:
  - Ansible installed
  - SSH access to the server
  - Python on both ends
  
- 🌐 Home Network:
  - Router Port Forwarding
    - Ensure that the VPN port (e.g., 51820/UDP or your chosen port) is forwarded from your router to the internal IP address of the WireGuard server. This step is essential to allow external devices to initiate a connection to your VPN.
  - Remote Accessibility Check
    - After configuring port forwarding, verify that your VPN server is reachable from the internet. You can use tools like nmap or online port checkers to confirm that the VPN port is open and accepting connections.
 
---

## 🚧 Setup Steps

### 1. ✅ Define Inventory

Edit the `inventory` file with your VPN server's IP or hostname:
```ini
[vpn]
vpn-server ansible_host=your.server.ip ansible_user=root
```

---

### 2. 🛠️ Run Server Setup Playbook

This playbook sets up WireGuard with a dynamic keypair, port, subnet, and peerless config.

```bash
ansible-playbook -i inventory playbook.yml   -e "wg_interface=wg0 wg_port=51820 wg_address=10.0.0.1/24"
```

✅ Result: `/etc/wireguard/wg0.conf` created on server and service started.

---

### 3. 🌐 Enable VPN Routing (Optional)

Run this to allow VPN clients to access the internet via the server.

```bash
ansible-playbook -i inventory enable-vpn-routing.yml   -e "public_interface=enp3s0"
```

This will:
- Enable IP forwarding
- Set up iptables MASQUERADE rule
- Make rules persistent

---

### 4. 👤 Create a Client Configuration

To generate a client config for any OS (e.g., Windows), pass the peer details as variables:

```bash
ansible-playbook -i inventory playbook.yml   --tags client   -e "client_name=win01 client_address=10.0.0.2"
```

🗂 Output: Client `.conf` will be saved locally or printed (you can redirect or store it as needed).

Variables:
- `client_name`: name of the client file (e.g., `win01`)
- `client_address`: the private IP for the client (e.g., `10.0.0.2`)
- `client_public_key`: public key from the client device
- `client_endpoint`: public IP/port of the VPN server (for `.conf` generation)

---

## 🧠 How it Works

- Server and client keys are generated outside the playbook (for better security).
- Jinja2 templates are used to render server (`wg0.conf`) and client config files.
- Peer configs are appended to the server config when clients are added.
- You can version-control the playbooks safely (no secrets stored).

---

## 🛡️ Security Best Practices

- Generate keys using `wg genkey | tee privatekey | wg pubkey > publickey`
- Keep `privatekey` files secure and out of version control
- Use `ufw` or iptables to allow only necessary ports (e.g., 51820/UDP)

---

## ✅ Future Improvements (Ideas)

- Include QR code generation for mobile clients
- Add backup/restore of server config and keys

---

## 📝 License

MIT License. Provided as-is, no warranty.

---

Made with 💻 and 🛠️ by Charles Santiagu
