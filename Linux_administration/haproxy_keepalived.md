# HAProxy High Availability Setup with Keepalived and VIP

This guide demonstrates how to configure HAProxy in primary/backup mode with automatic failover using keepalived and a Virtual IP (VIP).

## Before You Begin

**Replace the following variables with your actual values:**
- `YOUR_INTERFACE`: Your network interface name (e.g., eth0, ens160, enp0s3)
- `SERVER1_IP`: IP address of your primary server
- `SERVER2_IP`: IP address of your backup server  
- `YOUR_VIP_IP`: Virtual IP address for failover
- `/24`: Your subnet mask (adjust as needed)

## Environment Setup

- **Primary server**: SERVER1_IP/24 (interface: YOUR_INTERFACE)
- **Backup server**: SERVER2_IP/24 (interface: YOUR_INTERFACE)  
- **Virtual IP (VIP)**: YOUR_VIP_IP
- **HAProxy port**: 8080
- **Backend**: Nginx in Docker container (port 88)

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

### Step 2: Start Nginx backend in Docker container

```bash
# On both servers
sudo docker run -d --name nginx-backend -p 88:80 nginx
```

### Step 3: Configure HAProxy

Edit the HAProxy configuration file:

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
    server nginx-backend 127.0.0.1:88 check
```

### Step 4: Start and enable HAProxy

```bash
sudo systemctl start haproxy
sudo systemctl enable haproxy
```

### Step 5: Verify HAProxy is working

```bash
# Check if HAProxy is listening on port 8080
sudo netstat -tlnp | grep :8080

# Test the connection
curl -I localhost:8080
```

## Part 2: Keepalived Installation and Configuration

### Step 6: Install keepalived on both servers

```bash
sudo apt update && sudo apt install keepalived
```

### Step 7: Create keepalived configuration directory

```bash
sudo mkdir -p /etc/keepalived
```

### Step 8: Create health check script

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

### Step 9: Configure keepalived on PRIMARY server (SERVER1_IP)

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
    state MASTER
    interface YOUR_INTERFACE
    virtual_router_id 51
    priority 110
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

### Step 10: Configure keepalived on BACKUP server (SERVER2_IP)

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Add the following configuration (note the differences):

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

### Step 11: Start and enable keepalived on both servers

```bash
sudo systemctl start keepalived
sudo systemctl enable keepalived
```

## Part 3: Testing and Verification

### Step 12: Verify VIP assignment

On PRIMARY server:

```bash
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: VIP should be assigned to PRIMARY server.

On BACKUP server:

```bash
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: No VIP should be present.

### Step 13: Test HAProxy through VIP

```bash
curl -I YOUR_VIP_IP:8080
```

Expected result: HTTP 200 OK response from nginx.

### Step 14: Test nginx failure scenario

Stop nginx on PRIMARY server:

```bash
sudo docker stop nginx-backend
```

Check VIP location:

```bash
# On BACKUP server
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: VIP should now be assigned to BACKUP server.

### Step 15: Test nginx recovery

Start nginx on PRIMARY server:

```bash
sudo docker start nginx-backend
```

Check VIP location:

```bash
# On PRIMARY server
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: VIP should return to PRIMARY server.

### Step 16: Test complete server failure

Stop keepalived on PRIMARY server:

```bash
sudo systemctl stop keepalived
```

Check VIP location on BACKUP server:

```bash
ip addr show YOUR_INTERFACE | grep YOUR_VIP_IP
```

Expected result: VIP should be assigned to BACKUP server.

### Step 17: Test server recovery

Start keepalived on PRIMARY server:

```bash
sudo systemctl start keepalived
```

Expected result: VIP should return to PRIMARY server.

## Configuration Parameters Explanation

### keepalived Configuration Parameters:

- **state**: MASTER for primary, BACKUP for backup server
- **priority**: Higher priority server becomes MASTER (110 > 100)
- **weight**: Priority reduction when health check fails (-15)
- **interval**: Health check frequency (2 seconds)
- **fall**: Number of consecutive failures before considering service down (3)
- **rise**: Number of consecutive successes before considering service up (2)
- **virtual_router_id**: Must be same on both servers (51)

### Failover Logic:

1. PRIMARY server priority: 110
2. BACKUP server priority: 100
3. When nginx fails: PRIMARY priority becomes 110 - 15 = 95
4. Since 95 < 100, VIP switches to BACKUP server
5. When nginx recovers: PRIMARY priority returns to 110, VIP switches back

## Monitoring and Troubleshooting

### Check keepalived status:

```bash
sudo systemctl status keepalived
```

### View keepalived logs:

```bash
sudo journalctl -u keepalived -f
```

### Test health check script manually:

```bash
sudo /etc/keepalived/check_nginx.sh
echo "Exit code: $?"
```

### Check current VIP assignment:

```bash
ip addr show YOUR_INTERFACE
```

## Security Considerations

1. Use firewall rules to restrict access to HAProxy port
2. Consider using stronger authentication methods
3. Monitor logs for security violations
4. Regularly update system packages

## Conclusion

This setup provides:

-  Automatic failover when nginx service fails
-  Automatic failover when primary server fails completely  
-  Automatic recovery when services/servers are restored
-  Load balancing capabilities through HAProxy
-  High availability with minimal downtime

The system is now ready for production use with automatic failover capabilities.