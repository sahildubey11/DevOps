# Networking Fundamentals

## OSI Model

The Open Systems Interconnection (OSI) model is a conceptual framework with 7 layers:

| Layer | Name | Description | Protocols/Examples |
|-------|------|-------------|-------------------|
| 7 | Application | User interface | HTTP, FTP, DNS, SMTP |
| 6 | Presentation | Data format | SSL/TLS, JPEG, ASCII |
| 5 | Session | Connection management | NetBIOS, RPC |
| 4 | Transport | End-to-end delivery | TCP, UDP |
| 3 | Network | Routing and addressing | IP, ICMP, OSPF |
| 2 | Data Link | MAC addressing | Ethernet, Wi-Fi |
| 1 | Physical | Physical connection | Cables, hubs |

## TCP/IP Model

Simplified 4-layer model:

1. **Application Layer**: HTTP, DNS, FTP, SSH
2. **Transport Layer**: TCP, UDP
3. **Internet Layer**: IP, ICMP, ARP
4. **Network Access Layer**: Ethernet, Wi-Fi

## IP Addressing

### IPv4
- 32-bit address
- Format: xxx.xxx.xxx.xxx (e.g., 192.168.1.1)
- Classes: A, B, C, D, E
- Private ranges:
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16

### IPv6
- 128-bit address
- Format: xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
- Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

### CIDR Notation

```
192.168.1.0/24
- 192.168.1.0: Network address
- /24: Subnet mask (255.255.255.0)
- Usable hosts: 254 (192.168.1.1 to 192.168.1.254)

10.0.0.0/16
- /16: Subnet mask (255.255.0.0)
- Usable hosts: 65,534
```

### Subnetting Example

```
Network: 192.168.1.0/24
Split into 4 subnets (/26):

Subnet 1: 192.168.1.0/26   (192.168.1.1 - 192.168.1.62)
Subnet 2: 192.168.1.64/26  (192.168.1.65 - 192.168.1.126)
Subnet 3: 192.168.1.128/26 (192.168.1.129 - 192.168.1.190)
Subnet 4: 192.168.1.192/26 (192.168.1.193 - 192.168.1.254)
```

## Common Protocols

### HTTP/HTTPS
- **Port**: 80 (HTTP), 443 (HTTPS)
- **Purpose**: Web traffic
- **Protocol**: TCP

### SSH
- **Port**: 22
- **Purpose**: Secure remote access
- **Protocol**: TCP

### DNS
- **Port**: 53
- **Purpose**: Domain name resolution
- **Protocol**: UDP (usually), TCP (for large responses)

### FTP
- **Port**: 20 (data), 21 (control)
- **Purpose**: File transfer
- **Protocol**: TCP

### SMTP/IMAP/POP3
- **Ports**: 25 (SMTP), 143 (IMAP), 110 (POP3)
- **Purpose**: Email
- **Protocol**: TCP

## Networking Commands

### Linux Networking Commands

```bash
# View network interfaces
ip addr show
ifconfig

# View routing table
ip route show
route -n

# Test connectivity
ping google.com
ping -c 4 8.8.8.8

# Trace route
traceroute google.com
tracepath google.com

# DNS lookup
nslookup google.com
dig google.com
host google.com

# View active connections
netstat -tuln
ss -tuln

# View network statistics
netstat -s
ss -s

# Check listening ports
netstat -tuln | grep LISTEN
ss -tuln | grep LISTEN

# Test port connectivity
telnet example.com 80
nc -zv example.com 80

# Network interface statistics
ip -s link

# ARP table
arp -a
ip neighbor show

# Download file
wget https://example.com/file.txt
curl -O https://example.com/file.txt
```

### Advanced Network Debugging

```bash
# Packet capture
tcpdump -i eth0
tcpdump -i eth0 port 80
tcpdump -i eth0 -w capture.pcap

# HTTP requests
curl -v https://example.com
curl -I https://example.com  # Headers only

# Speed test
speedtest-cli

# Bandwidth usage
iftop
nethogs

# MTU discovery
ping -M do -s 1472 google.com

# Network performance
iperf3 -c server_ip
```

## Load Balancing

### Types

1. **Round Robin**: Distribute requests equally
2. **Least Connections**: Send to server with fewest connections
3. **IP Hash**: Route based on client IP
4. **Weighted**: Distribute based on server capacity

### Nginx Load Balancer

```nginx
http {
    upstream backend {
        # Round robin (default)
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
        
        # With weights
        # server backend1.example.com weight=3;
        # server backend2.example.com weight=2;
        # server backend3.example.com weight=1;
        
        # Least connections
        # least_conn;
        
        # IP hash
        # ip_hash;
        
        # Health checks
        server backend4.example.com max_fails=3 fail_timeout=30s;
    }

    server {
        listen 80;
        
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### HAProxy Load Balancer

```
global
    log /dev/log local0
    maxconn 4096

defaults
    log     global
    mode    http
    option  httplog
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http-in
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    option httpchk GET /health
    server server1 192.168.1.10:8080 check
    server server2 192.168.1.11:8080 check
    server server3 192.168.1.12:8080 check
```

## DNS

### DNS Record Types

- **A**: Maps domain to IPv4
- **AAAA**: Maps domain to IPv6
- **CNAME**: Alias to another domain
- **MX**: Mail server
- **TXT**: Text records (SPF, DKIM)
- **NS**: Nameserver
- **SOA**: Start of Authority

### DNS Configuration Example

```
example.com.        IN  A       192.0.2.1
www.example.com.    IN  A       192.0.2.1
mail.example.com.   IN  A       192.0.2.10
example.com.        IN  MX  10  mail.example.com.
api.example.com.    IN  CNAME   example.com.
example.com.        IN  TXT     "v=spf1 mx ~all"
```

### DNS Lookup

```bash
# A record
dig example.com A

# MX records
dig example.com MX

# All records
dig example.com ANY

# Reverse DNS
dig -x 192.0.2.1

# Query specific DNS server
dig @8.8.8.8 example.com

# Trace DNS resolution
dig +trace example.com
```

## Firewalls

### iptables (Linux)

```bash
# View rules
iptables -L -n -v

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Drop all other incoming
iptables -P INPUT DROP

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### UFW (Uncomplicated Firewall)

```bash
# Enable UFW
ufw enable

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow SSH
ufw allow 22/tcp
ufw allow ssh

# Allow HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Allow from specific IP
ufw allow from 192.168.1.100

# Allow specific port from specific IP
ufw allow from 192.168.1.100 to any port 22

# Deny
ufw deny 23

# Delete rule
ufw delete allow 80

# View status
ufw status
ufw status numbered
```

### AWS Security Groups

```hcl
# Terraform example
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description     = "SSH from bastion"
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}
```

## VPN

### OpenVPN Setup

```bash
# Install OpenVPN
apt-get install openvpn

# Generate server configuration
# Configure client

# Start server
systemctl start openvpn@server

# Connect as client
openvpn --config client.ovpn
```

### WireGuard

```bash
# Install WireGuard
apt-get install wireguard

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Server configuration (/etc/wireguard/wg0.conf)
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <server-private-key>

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32

# Start WireGuard
wg-quick up wg0
```

## Network Troubleshooting

### Common Issues and Solutions

```bash
# Can't reach host
ping <host>  # Check connectivity
traceroute <host>  # Find where connection fails
nslookup <host>  # Check DNS resolution

# Port not accessible
telnet <host> <port>
nc -zv <host> <port>

# Slow network
iperf3 -c <server>  # Test bandwidth
ping -i 0.2 <host>  # Check latency

# DNS issues
dig <domain>  # Check DNS resolution
cat /etc/resolv.conf  # Check DNS servers

# Check if port is in use
lsof -i :<port>
netstat -tuln | grep <port>

# Check routing
ip route get <destination>
traceroute <destination>
```

## CDN (Content Delivery Network)

### Benefits
- Reduced latency
- Improved performance
- Better availability
- DDoS protection
- Reduced bandwidth costs

### Popular CDNs
- Cloudflare
- Amazon CloudFront
- Akamai
- Fastly
- Azure CDN

### Cloudflare Example

```bash
# Set up Cloudflare
1. Add your domain
2. Update nameservers
3. Configure DNS records
4. Enable CDN (proxy)
5. Configure caching rules
6. Enable SSL/TLS
```

## Interview Questions

1. Explain the OSI model
2. What is the difference between TCP and UDP?
3. What is a subnet mask?
4. How does DNS work?
5. Explain the three-way handshake
6. What is NAT?
7. Difference between hub, switch, and router?
8. What is a load balancer?
9. How do you troubleshoot network connectivity issues?
10. What is the difference between IPv4 and IPv6?

## Practical Exercises

1. Configure a load balancer with Nginx
2. Set up a VPN connection
3. Configure firewall rules
4. Troubleshoot network connectivity issues
5. Set up DNS records
6. Configure network monitoring
7. Implement network security best practices
8. Set up a reverse proxy

## Resources

- [Networking Fundamentals (Cisco)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
- [TCP/IP Guide](http://www.tcpipguide.com/)
- [DNS Documentation](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Network troubleshooting tools](https://www.tecmint.com/linux-network-configuration-and-troubleshooting-commands/)
