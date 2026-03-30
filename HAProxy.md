# HAProxy Installation and Configuration

1. Installation (RHEL / CentOS)

```bash
sudo dnf install haproxy -y
```

1.1 Ubuntu
```bash
sudo apt update
sudo apt install haproxy -y
```

2. Validate installation
```bash
haproxy -v
```

3. Start & Enable & Verify & Stop Service
```bash
sudo systemctl enable haproxy

sudo systemctl start haproxy

sudo systemctl stop haproxy

sudo systemctl status haproxy
```

4. Main config file
```bash
/etc/haproxy/haproxy.cfg
```

5. HAProxy

HAProxy (High Availability Proxy) is a fast, reliable load balancer and reverse proxy for TCP and HTTP applications.

It sits in front of your servers:
- Load balancing distributes traffic
- High availability - if one server fails traffic goes to other automatically
- Reverse proxy hide backend servers beind a single entry point
- SSL termination 
- Health Checks regularly check the health of backend

HAProxy has two main sections:

- **global** - keep configuration for HAProxy like SSL, locations, logging informaion
- **defaults** - default values for all nodes