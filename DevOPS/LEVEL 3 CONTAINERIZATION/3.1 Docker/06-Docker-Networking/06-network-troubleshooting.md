# Chapter 6: Network Troubleshooting

> **Unit 6: Docker Networking** | [Course Home](../README.md)

---

## Overview

Network issues are among the most common problems in containerized environments. This chapter covers diagnostic tools, common problems, and systematic approaches to troubleshooting Docker networking.

---

## 6.1 Diagnostic Workflow

```
  Docker Network Troubleshooting Flowchart:
  
  Container can't connect
         |
         v
  Is container running?
    No --> docker start
    Yes --+
          v
  On correct network?
    No --> docker network connect
    Yes --+
          v
  Can resolve hostname?
    No --> Check DNS / network type
    Yes --+
          v
  Can ping target?
    No --> Check firewall / ICC
    Yes --+
          v
  Can connect to port?
    No --> Check service / port mapping
    Yes --> Application-level issue
```

---

## 6.2 Essential Diagnostic Commands

```bash
# 1. Inspect network configuration
docker network ls
docker network inspect bridge

# 2. Check container's network settings
docker inspect --format='{{json .NetworkSettings}}' mycontainer | python -m json.tool

# 3. Check container IP address
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mycontainer

# 4. Check which networks a container is on
docker inspect -f '{{json .NetworkSettings.Networks}}' mycontainer

# 5. Check port mappings
docker port mycontainer

# 6. Check container's DNS config
docker exec mycontainer cat /etc/resolv.conf

# 7. Check container's hosts file
docker exec mycontainer cat /etc/hosts

# 8. List all containers on a network
docker network inspect -f '{{range .Containers}}{{.Name}} {{.IPv4Address}}{{"\n"}}{{end}}' mynetwork
```

---

## 6.3 Using the Netshoot Container

```bash
# nicolaka/netshoot: Swiss Army knife for networking
# Includes: curl, nslookup, dig, ping, traceroute,
#           tcpdump, iperf, netstat, ss, ip, nmap...

# Run alongside your containers
docker run -it --rm --network mynetwork \
  nicolaka/netshoot

# Attach to a container's network namespace
docker run -it --rm --network container:mycontainer \
  nicolaka/netshoot

# Use host networking for host-level debugging
docker run -it --rm --network host \
  nicolaka/netshoot
```

```
  Netshoot Debugging Pattern:
  
  +------------------+     +------------------+
  |   mycontainer    |     |    netshoot      |
  |  (minimal image) |     | (full net tools) |
  |  No curl/ping    |     | curl, dig, ping  |
  |                  |     | tcpdump, nmap... |
  +--------+---------+     +--------+---------+
           |                        |
           +-------- bridge --------+
           
  Same network: netshoot can test 
  connectivity to mycontainer
```

---

## 6.4 DNS Troubleshooting

```bash
# Check DNS resolution inside container
docker exec mycontainer nslookup api
docker exec mycontainer dig api

# Using netshoot for DNS debugging
docker run --rm --network mynetwork nicolaka/netshoot \
  dig api

# Common DNS issues:
# 1. Using default bridge → no built-in DNS
# 2. Container name mismatch
# 3. Containers on different networks
# 4. DNS cache stale after container recreation
```

```bash
# Verify embedded DNS server
docker exec mycontainer cat /etc/resolv.conf
# Should show: nameserver 127.0.0.11
# (Docker's embedded DNS)

# If using default bridge, you'll see host DNS:
# nameserver 8.8.8.8 (or similar)
```

---

## 6.5 Connectivity Testing

```bash
# Test basic connectivity
docker exec mycontainer ping -c 3 target_container

# Test TCP port
docker exec mycontainer nc -zv target_container 5432

# Test HTTP endpoint
docker exec mycontainer wget -qO- http://api:8080/health

# Or with curl (if available)
docker exec mycontainer curl -s http://api:8080/health

# Test from host to container
curl http://localhost:8080/health

# Check listening ports inside container
docker exec mycontainer ss -tlnp
docker exec mycontainer netstat -tlnp
```

---

## 6.6 Packet Capture with tcpdump

```bash
# Capture packets on container's interface
docker run --rm -it --network container:mycontainer \
  nicolaka/netshoot tcpdump -i eth0 -n

# Capture specific port traffic
docker run --rm -it --network container:mycontainer \
  nicolaka/netshoot tcpdump -i eth0 port 80 -n

# Save capture to file
docker run --rm -it --network container:mycontainer \
  -v $(pwd):/capture \
  nicolaka/netshoot tcpdump -i eth0 -w /capture/dump.pcap

# Capture DNS queries
docker run --rm -it --network container:mycontainer \
  nicolaka/netshoot tcpdump -i eth0 port 53 -n
```

```
  Packet Capture Flow:
  
  Container A          Network          Container B
  +---------+      +----------+      +---------+
  | App     |----->| tcpdump  |----->| App     |
  | :8080   |      | captures |      | :5432   |
  +---------+      | here     |      +---------+
                   +----------+
                        |
                        v
                   dump.pcap
                   (analyze in Wireshark)
```

---

## 6.7 Common Problems and Solutions

### Port Mapping Issues

```bash
# Problem: "port is already allocated"
# Find what's using the port
docker ps --format '{{.Names}} {{.Ports}}' | grep 8080
# On host:
netstat -tlnp | grep 8080   # Linux
netstat -ano | findstr 8080  # Windows

# Solution: Use a different host port
docker run -d -p 8081:80 nginx
```

### Container Can't Reach Internet

```bash
# Check DNS resolution
docker run --rm alpine nslookup google.com

# Check IP forwarding (Linux)
sysctl net.ipv4.ip_forward
# Should be 1

# Check iptables NAT
sudo iptables -t nat -L POSTROUTING -n -v

# Check if on --internal network
docker network inspect mynetwork | grep Internal
```

### Containers Can't Find Each Other

```bash
# Verify same network
docker inspect -f '{{json .NetworkSettings.Networks}}' container1
docker inspect -f '{{json .NetworkSettings.Networks}}' container2

# Check container names (DNS uses container name)
docker ps --format '{{.Names}} {{.Networks}}'

# Test: are they on the same network?
docker network inspect mynetwork \
  -f '{{range .Containers}}{{.Name}} {{end}}'
```

---

## 6.8 Performance Troubleshooting

```bash
# Measure bandwidth between containers
# Terminal 1: Start iperf3 server
docker run -it --rm --name iperf-server \
  --network mynet nicolaka/netshoot iperf3 -s

# Terminal 2: Run iperf3 client
docker run -it --rm --network mynet \
  nicolaka/netshoot iperf3 -c iperf-server

# Measure latency
docker exec container1 ping -c 10 container2

# Check for packet loss
docker exec container1 ping -c 100 -i 0.1 container2
```

```
  Network Performance Comparison:
  
  Driver          Throughput    Latency    Use Case
  +----------+   +---------+  +--------+  +-----------+
  | host     |   | ~100%   |  | lowest |  | Max perf  |
  | macvlan  |   | ~98%    |  | low    |  | Near-bare |
  | bridge   |   | ~95%    |  | low    |  | Default   |
  | overlay  |   | ~85-90% |  | medium |  | Multi-host|
  | encrypted|   | ~70-80% |  | higher |  | Secure    |
  +----------+   +---------+  +--------+  +-----------+
  (Approximate — actual results vary by setup)
```

---

## Summary Table

| Tool / Command | Purpose | Example |
|----------------|---------|---------|
| `docker network inspect` | View network details | `docker network inspect bridge` |
| `docker inspect` | Container network config | `docker inspect --format '...'` |
| `netshoot` | Full networking toolkit | `docker run nicolaka/netshoot` |
| `nslookup / dig` | DNS resolution testing | `nslookup api` inside container |
| `ping` | Basic connectivity | `ping -c 3 target` |
| `nc (netcat)` | Port connectivity | `nc -zv target 5432` |
| `tcpdump` | Packet capture | `tcpdump -i eth0 port 80` |
| `ss / netstat` | Listening ports | `ss -tlnp` |
| `iperf3` | Bandwidth testing | `iperf3 -c server` |
| `curl / wget` | HTTP testing | `curl http://api:8080/health` |

---

## Quick Revision Questions

1. **How do you debug networking in a minimal container that has no tools?**
   - Use the netshoot container attached to the target's network namespace: `docker run --rm -it --network container:target nicolaka/netshoot`

2. **What's the first thing to check when containers can't communicate?**
   - Verify both containers are on the same user-defined network using `docker inspect` or `docker network inspect`

3. **How do you capture network packets for a container?**
   - Run tcpdump via netshoot attached to the container's network: `docker run --rm --network container:myapp nicolaka/netshoot tcpdump -i eth0`

4. **Why can't containers resolve each other by name on the default bridge?**
   - The default bridge network doesn't provide Docker's embedded DNS service. Use a user-defined bridge network instead

5. **How do you check what port a container is listening on?**
   - Use `docker exec mycontainer ss -tlnp` or `docker exec mycontainer netstat -tlnp` to see listening ports inside the container

6. **What does `docker run --network container:X` do?**
   - It shares container X's network namespace, giving the new container the same IP address, interfaces, and ports — useful for debugging with netshoot

---

[← Previous: Network Security](05-network-security.md) | [Next Unit: Docker Volumes →](../07-Docker-Volumes/01-storage-overview.md)
