# HAProxy Active-Active Load Balancing Setup with Keepalived

This guide demonstrates how to configure HAProxy in active-active mode with load balancing between multiple backend servers using keepalived for high availability.

## Before You Begin

**Replace the following variables with your actual values:**
- `YOUR_INTERFACE`: Your network interface name (e.g., eth0, ens160, enp0s3)
- `SERVER1_IP`: IP address of your first server
- `SERVER2_IP`: IP address of your second server  
- `YOUR_VIP_IP`: Virtual IP address for failover
- `/24`: Your subnet mask (adjust as needed)

## Environment Setup

- **Server 1**: SERVER1_IP/24 (interface: YOUR_INTERFACE)
- **Server 2**: SERVER2_IP/24 (interface: YOUR_INTERFACE)  
- **Virtual IP (VIP)**: YOUR_VIP_IP
- **HAProxy port**: 8080
- **Backend**: Nginx in Docker containers (port 88)
- **Load balancing**: Round-robin between both servers

## Prerequisites

- Two Linux servers with network connectivity
- Docker installed on both servers
- Root or sudo access

## Part 1: HAProxy Installation and Configuration

### Step 1: Install HAProxy on both servers

```bash
sudo apt update
sudo apt install haproxy
```

### Step 2: Create custom content for Server 1

On Server 1 (SERVER1_IP):

```bash
mkdir -p /tmp/nginx-content
echo "<h1>Server 1 (SERVER1_IP)</h1><p>You are connected to HAProxy backend server #1.</p>" > /tmp/nginx-content/index.html
```

### Step 3: Create custom content for Server 2

On Server 2 (SERVER2_IP):

```bash
mkdir -p /tmp/nginx-content
echo "<h1>Server 2 (SERVER2_IP)</h1><p>You are connected to HAProxy backend server #2.</p>" > /tmp/nginx-content/index.html
```

### Step 4: Start Nginx backend containers with custom content

On Server 1:

```bash
sudo docker run -d --name nginx-backend -p 88:80 -v /tmp/nginx-content:/usr/share/nginx/html nginx
```

On Server 2:

```bash
sudo docker run -d --name nginx-backend -p 88:80 -v /tmp/nginx-content:/usr/share/nginx/html nginx
```

### Step 5: Configure HAProxy for load balancing

Edit the HAProxy configuration file on **both servers**:

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Add the following configuration:

```bash
global
    daemon
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend web_frontend
    bind *:8080
    default_backend web_servers

backend web_servers
    balance roundrobin
    server server1 SERVER1_IP:88 check
    server server2 SERVER2_IP:88 check
```

### Step 6: Start and enable HAProxy on both servers

```bash
sudo systemctl start haproxy
sudo systemctl enable haproxy
```

### Step 7: Verify HAProxy load balancing

Test load balancing on Server 1:

```bash
curl http://SERVER1_IP:8080
curl http://SERVER1_IP:8080
curl http://SERVER1_IP:8080
```

Test load balancing on Server 2:

```bash
curl http://SERVER2_IP:8080
curl http://SERVER2_IP:8080
curl http://SERVER2_IP:8080
```

Expected result: You should see responses alternating between "Server 1" and "Server 2".

## Part 2: Keepalived Configuration for Active-Active Setup

### Step 8: Install keepalived on both servers

```bash
sudo apt update && sudo apt install keepalived
```

### Step 9: Create keepalived configuration directory

```bash
sudo mkdir -p /etc/keepalived
```

### Step 10: Create health check script

On both servers:

```bash
sudo nano /etc/keepalived/check_nginx.sh
```

Add the following script:

```bash
#!/bin/bash
curl -f http://localhost:8080 >/dev/null 2>&1
exit $?
```

Make the script executable:

```bash
sudo chmod +x /etc/keepalived/check_nginx.sh
```

### Step 11: Configure keepalived on Server 1

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Add the following configuration:

```bash
vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -15
    fall 3
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface YOUR_INTERFACE
    virtual_router_id 51
    priority 105
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass mypass12
    }
    virtual_ipaddress {
        YOUR_VIP_IP
    }
    track_script {
        chk_nginx
    }
}
```

### Step 12: Configure keepalived on Server 2

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Add the following configuration:

```bash
vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -15
    fall 3
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface YOUR_INTERFACE
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass mypass12
    }
    virtual_ipaddress {
        YOUR_VIP_IP
    }
    track_script {
        chk_nginx
    }
}
```

### Step 13: Start and enable keepalived on both servers

```bash
sudo systemctl start keepalived
sudo systemctl enable keepalived
```

## Part 3: Testing and Verification

### Step 14: Verify VIP assignment

Check which server currently has the VIP:

```bash
# On Server 1
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP

# On Server 2  
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: One server should have the VIP assigned (likely Server 1 due to higher priority).

### Step 15: Test load balancing through VIP

```bash
curl http://YOUR_VIP_IP:8080
curl http://YOUR_VIP_IP:8080
curl http://YOUR_VIP_IP:8080
curl http://YOUR_VIP_IP:8080
```

Expected result: You should see responses alternating between "Server 1" and "Server 2", demonstrating load balancing.

### Step 16: Test failover scenario

Stop nginx on the server that currently holds the VIP:

```bash
sudo docker stop nginx-backend
```

Check VIP location:

```bash
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: VIP should switch to the other server.

Test connectivity through VIP:

```bash
curl http://YOUR_VIP_IP:8080
```

Expected result: Load balancing should continue working through the backup server.

### Step 17: Test recovery

Start nginx back on the failed server:

```bash
sudo docker start nginx-backend
```

Expected result: VIP should return to the original server, and full load balancing should resume.

## Architecture Overview

### Load Balancing Logic:

1. **Each HAProxy instance** balances between **both backend servers**
2. **Round-robin algorithm** distributes requests evenly
3. **Health checks** ensure only healthy backends receive traffic
4. **VIP failover** ensures high availability at the HAProxy level

### Failover Behavior:

1. **Normal operation**: Server with higher priority (105) holds VIP
2. **Service failure**: If nginx fails, priority drops by 15 points
3. **Server failure**: If entire server fails, VIP switches to backup
4. **Recovery**: VIP returns when original server recovers

## Configuration Parameters Explanation

### HAProxy Backend Configuration:
- **balance roundrobin**: Distributes requests evenly between servers
- **server**: Defines backend servers with health checks
- **check**: Enables health monitoring of backend servers

### keepalived Configuration:
- **state BACKUP**: Both servers in backup state for equal treatment
- **priority**: 105 vs 100 (small difference for quick failover)
- **weight -15**: Large priority reduction on health check failure
- **virtual_router_id**: Must be identical on both servers

## Monitoring and Troubleshooting

### Check HAProxy status:

```bash
sudo systemctl status haproxy
```

### Check backend server status:

```bash
echo "show stat" | sudo socat stdio /run/haproxy/admin.sock
```

### View keepalived logs:

```bash
sudo journalctl -u keepalived -f
```

### Test individual backends:

```bash
# Test Server 1 nginx directly
curl http://SERVER1_IP:88

# Test Server 2 nginx directly  
curl http://SERVER2_IP:88
```

## Benefits of Active-Active Setup

1. **Load Distribution**: Traffic is balanced between both servers
2. **High Availability**: Automatic failover if any component fails
3. **Resource Utilization**: Both servers actively handle requests
4. **Scalability**: Easy to add more backend servers
5. **Fault Tolerance**: System continues operating even with failures

## Security Considerations

1. Configure firewall rules for ports 8080 and 88
2. Use strong authentication passwords
3. Monitor logs for security violations
4. Keep system packages updated
5. Consider SSL/TLS termination at HAProxy level

## Conclusion

This active-active setup provides:

-  **Load balancing** between multiple backend servers
-  **High availability** with automatic failover
-  **Resource optimization** using both servers actively  
-  **Health monitoring** with automatic server exclusion
-  **Easy scaling** by adding more backend servers

The system is now ready for production use with full load balancing and high availability capabilities.