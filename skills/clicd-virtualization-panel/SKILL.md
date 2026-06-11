---
name: clicd-virtualization-panel
description: Use CLICD to manage LXC/KVM virtualization with web console, NAT/IPv6 networking, WebSSH/VNC, resource controls, and security monitoring.
triggers:
  - set up CLICD virtualization panel
  - manage LXC containers with CLICD
  - configure CLICD NAT networking
  - use CLICD REST API
  - create KVM virtual machines in CLICD
  - configure CLICD security alerts
  - manage CLICD user permissions
  - integrate CLICD with billing system
---

# CLICD Virtualization Panel Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

CLICD is a lightweight virtualization management panel for LXC and KVM that provides a web console, CLI tools, REST API, NAT/IPv6 networking, WebSSH/WebVNC access, resource quotas, traffic limits, snapshots, delegated sub-user access, and security monitoring. Built with Go backend and React frontend.

## Installation

### One-Click Install

```bash
# Install CLICD
curl -fsSL https://raw.githubusercontent.com/MengMengCode/CLICD/main/install.sh | sudo sh

# Uninstall CLICD
curl -fsSL https://raw.githubusercontent.com/MengMengCode/CLICD/main/install.sh | sudo sh -s -- uninstall
```

### Prerequisites

- Linux system with systemd
- LXC or KVM/QEMU installed
- Root or sudo access
- iptables and conntrack-tools

### Post-Installation

After installation, CLICD runs as a systemd service:

```bash
# Check service status
sudo systemctl status clicd

# View logs
sudo journalctl -u clicd -f

# Restart service
sudo systemctl restart clicd
```

Access the web interface at `http://your-server-ip:8080` (default port).

## Configuration

### Main Configuration File

Configuration is typically stored in `/etc/clicd/config.yaml` or similar location:

```yaml
server:
  host: "0.0.0.0"
  port: 8080
  tls:
    enabled: false
    cert_file: ""
    key_file: ""
    letsencrypt: false
    domain: ""

database:
  type: "sqlite"
  path: "/var/lib/clicd/clicd.db"

lxc:
  enabled: true
  default_storage_pool: "default"
  default_network: "lxcbr0"

kvm:
  enabled: true
  default_storage_pool: "default"
  default_network: "virbr0"

networking:
  nat4:
    enabled: true
    public_ip: "1.2.3.4"
    port_range_start: 10000
    port_range_end: 60000
  ipv6:
    enabled: true
    prefix: "2001:db8::/48"
    auto_detect: true

security:
  alerts_enabled: true
  conntrack_monitoring: true
  
admin:
  username: "admin"
  # Set password via CLI or web interface
```

### Environment Variables

```bash
# Override config file location
export CLICD_CONFIG="/path/to/config.yaml"

# Set admin password (first run)
export CLICD_ADMIN_PASSWORD="your-secure-password"

# Database path
export CLICD_DB_PATH="/var/lib/clicd/clicd.db"

# Log level
export CLICD_LOG_LEVEL="info"
```

## CLI Usage

CLICD provides a CLI interface for common operations:

### Container Management

```bash
# List all containers
clicd container list

# Create a new LXC container
clicd container create \
  --name web-server-01 \
  --type lxc \
  --template ubuntu-22.04 \
  --cpu 2 \
  --memory 2048 \
  --disk 20 \
  --ipv4 nat \
  --ipv6 auto

# Start a container
clicd container start web-server-01

# Stop a container
clicd container stop web-server-01

# Delete a container
clicd container delete web-server-01 --force

# Reset container password
clicd container password web-server-01 --password "NewSecurePass123"

# Batch operations
clicd container batch-start --ids "1,2,3,4,5"
clicd container batch-stop --pattern "test-*"
```

### Image Management

```bash
# List available images
clicd image list

# Download an image
clicd image download ubuntu-22.04-lxc

# Enable/disable images
clicd image enable ubuntu-22.04-lxc
clicd image disable centos-7-lxc

# Clear image cache
clicd image cache-clear
```

### Networking

```bash
# List NAT port mappings
clicd nat list

# Add port mapping
clicd nat add \
  --container web-server-01 \
  --protocol tcp \
  --public-port 8080 \
  --private-port 80

# Remove port mapping
clicd nat remove --id 123

# Check IPv6 status
clicd ipv6 status

# Assign IPv6 to container
clicd ipv6 assign --container web-server-01
```

### User Management

```bash
# Create sub-user
clicd user create \
  --username client01 \
  --password "SecurePass123" \
  --containers "web-server-01,db-server-01"

# Generate delegation link
clicd user link --username client01

# List users
clicd user list

# Update user permissions
clicd user update --username client01 --add-container api-server-01
```

## REST API

All API endpoints are versioned under `/api/v1`. Authentication uses API keys or session tokens.

### Authentication

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
)

type LoginRequest struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

type LoginResponse struct {
    Token   string `json:"token"`
    Message string `json:"message"`
}

func login() (string, error) {
    apiURL := os.Getenv("CLICD_API_URL") // e.g., http://localhost:8080
    
    reqBody := LoginRequest{
        Username: os.Getenv("CLICD_USERNAME"),
        Password: os.Getenv("CLICD_PASSWORD"),
    }
    
    jsonData, _ := json.Marshal(reqBody)
    resp, err := http.Post(
        apiURL+"/api/v1/auth/login",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    var loginResp LoginResponse
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &loginResp)
    
    return loginResp.Token, nil
}
```

### Container Operations

```go
type Container struct {
    ID          int    `json:"id"`
    Name        string `json:"name"`
    Type        string `json:"type"` // "lxc" or "kvm"
    Status      string `json:"status"`
    CPU         int    `json:"cpu"`
    Memory      int    `json:"memory"` // MB
    Disk        int    `json:"disk"`   // GB
    IPv4        string `json:"ipv4"`
    IPv6        string `json:"ipv6"`
    ExpiryDate  string `json:"expiry_date"`
}

type CreateContainerRequest struct {
    Name       string `json:"name"`
    Type       string `json:"type"`
    Template   string `json:"template"`
    CPU        int    `json:"cpu"`
    Memory     int    `json:"memory"`
    Disk       int    `json:"disk"`
    Password   string `json:"password"`
    IPv4Type   string `json:"ipv4_type"`   // "nat" or "public"
    IPv6Enable bool   `json:"ipv6_enable"`
    ExpiryDays int    `json:"expiry_days"`
}

func createContainer(token string, req CreateContainerRequest) (*Container, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    jsonData, _ := json.Marshal(req)
    httpReq, _ := http.NewRequest(
        "POST",
        apiURL+"/api/v1/containers",
        bytes.NewBuffer(jsonData),
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    httpReq.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var container Container
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &container)
    
    return &container, nil
}

func listContainers(token string) ([]Container, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    httpReq, _ := http.NewRequest("GET", apiURL+"/api/v1/containers", nil)
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var containers []Container
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &containers)
    
    return containers, nil
}

func startContainer(token string, containerID int) error {
    apiURL := os.Getenv("CLICD_API_URL")
    
    httpReq, _ := http.NewRequest(
        "POST",
        fmt.Sprintf("%s/api/v1/containers/%d/start", apiURL, containerID),
        nil,
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    return nil
}

func deleteContainer(token string, containerID int, force bool) error {
    apiURL := os.Getenv("CLICD_API_URL")
    
    url := fmt.Sprintf("%s/api/v1/containers/%d", apiURL, containerID)
    if force {
        url += "?force=true"
    }
    
    httpReq, _ := http.NewRequest("DELETE", url, nil)
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    return nil
}
```

### Batch Operations

```go
type BatchActionRequest struct {
    IDs    []int  `json:"ids"`
    Action string `json:"action"` // "start", "stop", "restart", "delete"
}

type BatchCreateRequest struct {
    Count      int    `json:"count"`
    NamePrefix string `json:"name_prefix"`
    Template   string `json:"template"`
    CPU        int    `json:"cpu"`
    Memory     int    `json:"memory"`
    Disk       int    `json:"disk"`
}

func batchAction(token string, req BatchActionRequest) error {
    apiURL := os.Getenv("CLICD_API_URL")
    
    jsonData, _ := json.Marshal(req)
    httpReq, _ := http.NewRequest(
        "POST",
        apiURL+"/api/v1/containers/batch",
        bytes.NewBuffer(jsonData),
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    httpReq.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    return nil
}

func batchCreate(token string, req BatchCreateRequest) ([]Container, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    jsonData, _ := json.Marshal(req)
    httpReq, _ := http.NewRequest(
        "POST",
        apiURL+"/api/v1/containers/batch-create",
        bytes.NewBuffer(jsonData),
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    httpReq.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var containers []Container
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &containers)
    
    return containers, nil
}
```

### Networking API

```go
type NATMapping struct {
    ID           int    `json:"id"`
    ContainerID  int    `json:"container_id"`
    Protocol     string `json:"protocol"` // "tcp" or "udp"
    PublicPort   int    `json:"public_port"`
    PrivatePort  int    `json:"private_port"`
}

type CreateNATRequest struct {
    ContainerID int    `json:"container_id"`
    Protocol    string `json:"protocol"`
    PrivatePort int    `json:"private_port"`
    PublicPort  int    `json:"public_port,omitempty"` // Auto-assign if 0
}

func createNATMapping(token string, req CreateNATRequest) (*NATMapping, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    jsonData, _ := json.Marshal(req)
    httpReq, _ := http.NewRequest(
        "POST",
        apiURL+"/api/v1/nat",
        bytes.NewBuffer(jsonData),
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    httpReq.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var mapping NATMapping
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &mapping)
    
    return &mapping, nil
}

func assignIPv6(token string, containerID int) (string, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    httpReq, _ := http.NewRequest(
        "POST",
        fmt.Sprintf("%s/api/v1/containers/%d/ipv6", apiURL, containerID),
        nil,
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    var result struct {
        IPv6 string `json:"ipv6"`
    }
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &result)
    
    return result.IPv6, nil
}
```

### Security Monitoring

```go
type SecurityAlert struct {
    ID          int    `json:"id"`
    ContainerID int    `json:"container_id"`
    Type        string `json:"type"` // "port_scan", "brute_force", etc.
    Severity    string `json:"severity"`
    Description string `json:"description"`
    Timestamp   string `json:"timestamp"`
}

func getSecurityAlerts(token string, containerID int) ([]SecurityAlert, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    url := fmt.Sprintf("%s/api/v1/security/alerts", apiURL)
    if containerID > 0 {
        url += fmt.Sprintf("?container_id=%d", containerID)
    }
    
    httpReq, _ := http.NewRequest("GET", url, nil)
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var alerts []SecurityAlert
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &alerts)
    
    return alerts, nil
}
```

### Snapshots

```go
type Snapshot struct {
    ID          int    `json:"id"`
    ContainerID int    `json:"container_id"`
    Name        string `json:"name"`
    Description string `json:"description"`
    CreatedAt   string `json:"created_at"`
    Size        int64  `json:"size"` // bytes
}

type CreateSnapshotRequest struct {
    ContainerID int    `json:"container_id"`
    Name        string `json:"name"`
    Description string `json:"description"`
}

func createSnapshot(token string, req CreateSnapshotRequest) (*Snapshot, error) {
    apiURL := os.Getenv("CLICD_API_URL")
    
    jsonData, _ := json.Marshal(req)
    httpReq, _ := http.NewRequest(
        "POST",
        apiURL+"/api/v1/snapshots",
        bytes.NewBuffer(jsonData),
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    httpReq.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var snapshot Snapshot
    body, _ := io.ReadAll(resp.Body)
    json.Unmarshal(body, &snapshot)
    
    return &snapshot, nil
}

func restoreSnapshot(token string, snapshotID int) error {
    apiURL := os.Getenv("CLICD_API_URL")
    
    httpReq, _ := http.NewRequest(
        "POST",
        fmt.Sprintf("%s/api/v1/snapshots/%d/restore", apiURL, snapshotID),
        nil,
    )
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    return nil
}
```

## Common Patterns

### Complete Container Provisioning Workflow

```go
package main

import (
    "fmt"
    "log"
    "os"
)

func provisionContainer() {
    // 1. Authenticate
    token, err := login()
    if err != nil {
        log.Fatal("Login failed:", err)
    }
    
    // 2. Create container
    container, err := createContainer(token, CreateContainerRequest{
        Name:       "web-app-01",
        Type:       "lxc",
        Template:   "ubuntu-22.04",
        CPU:        4,
        Memory:     4096,
        Disk:       50,
        Password:   os.Getenv("CONTAINER_PASSWORD"),
        IPv4Type:   "nat",
        IPv6Enable: true,
        ExpiryDays: 30,
    })
    if err != nil {
        log.Fatal("Container creation failed:", err)
    }
    
    fmt.Printf("Container created: ID=%d, Name=%s\n", container.ID, container.Name)
    
    // 3. Add NAT port mappings
    httpMapping, _ := createNATMapping(token, CreateNATRequest{
        ContainerID: container.ID,
        Protocol:    "tcp",
        PrivatePort: 80,
        PublicPort:  0, // Auto-assign
    })
    fmt.Printf("HTTP port mapping: %d -> 80\n", httpMapping.PublicPort)
    
    httpsMapping, _ := createNATMapping(token, CreateNATRequest{
        ContainerID: container.ID,
        Protocol:    "tcp",
        PrivatePort: 443,
        PublicPort:  0,
    })
    fmt.Printf("HTTPS port mapping: %d -> 443\n", httpsMapping.PublicPort)
    
    // 4. Assign IPv6
    ipv6, _ := assignIPv6(token, container.ID)
    fmt.Printf("IPv6 assigned: %s\n", ipv6)
    
    // 5. Start container
    startContainer(token, container.ID)
    fmt.Println("Container started")
    
    // 6. Create initial snapshot
    snapshot, _ := createSnapshot(token, CreateSnapshotRequest{
        ContainerID: container.ID,
        Name:        "initial",
        Description: "Fresh installation",
    })
    fmt.Printf("Snapshot created: %s\n", snapshot.Name)
}
```

### Batch Container Deployment

```go
func deployMultipleContainers(count int, namePrefix string) {
    token, _ := login()
    
    containers, err := batchCreate(token, BatchCreateRequest{
        Count:      count,
        NamePrefix: namePrefix,
        Template:   "debian-12",
        CPU:        2,
        Memory:     2048,
        Disk:       20,
    })
    if err != nil {
        log.Fatal("Batch create failed:", err)
    }
    
    fmt.Printf("Created %d containers\n", len(containers))
    
    // Configure each container
    for _, container := range containers {
        // Enable IPv6
        assignIPv6(token, container.ID)
        
        // Add SSH port mapping
        createNATMapping(token, CreateNATRequest{
            ContainerID: container.ID,
            Protocol:    "tcp",
            PrivatePort: 22,
        })
        
        fmt.Printf("Configured container: %s\n", container.Name)
    }
    
    // Start all containers
    var ids []int
    for _, c := range containers {
        ids = append(ids, c.ID)
    }
    
    batchAction(token, BatchActionRequest{
        IDs:    ids,
        Action: "start",
    })
    
    fmt.Println("All containers started")
}
```

### Security Monitoring Dashboard

```go
func monitorSecurity(token string) {
    containers, _ := listContainers(token)
    
    for _, container := range containers {
        alerts, err := getSecurityAlerts(token, container.ID)
        if err != nil {
            continue
        }
        
        if len(alerts) > 0 {
            fmt.Printf("\n=== Container: %s (ID: %d) ===\n", container.Name, container.ID)
            for _, alert := range alerts {
                fmt.Printf("[%s] %s: %s\n", 
                    alert.Severity, 
                    alert.Type, 
                    alert.Description,
                )
            }
        }
    }
}
```

### Auto-Scaling Based on Load

```go
type ContainerStats struct {
    CPUUsage    float64 `json:"cpu_usage"`
    MemoryUsage int64   `json:"memory_usage"`
    DiskUsage   int64   `json:"disk_usage"`
}

func autoScale(token string, poolName string, maxInstances int) {
    containers, _ := listContainers(token)
    
    // Filter containers in this pool
    var poolContainers []Container
    for _, c := range containers {
        if c.Name[:len(poolName)] == poolName {
            poolContainers = append(poolContainers, c)
        }
    }
    
    // Check average load
    totalCPU := 0.0
    for _, c := range poolContainers {
        // Fetch stats from API
        stats := getContainerStats(token, c.ID)
        totalCPU += stats.CPUUsage
    }
    avgCPU := totalCPU / float64(len(poolContainers))
    
    // Scale up if average > 70%
    if avgCPU > 70.0 && len(poolContainers) < maxInstances {
        fmt.Println("Scaling up...")
        createContainer(token, CreateContainerRequest{
            Name:     fmt.Sprintf("%s-%d", poolName, len(poolContainers)+1),
            Type:     "lxc",
            Template: "ubuntu-22.04",
            CPU:      2,
            Memory:   2048,
            Disk:     20,
        })
    }
    
    // Scale down if average < 30%
    if avgCPU < 30.0 && len(poolContainers) > 1 {
        fmt.Println("Scaling down...")
        lastContainer := poolContainers[len(poolContainers)-1]
        stopContainer(token, lastContainer.ID)
        deleteContainer(token, lastContainer.ID, true)
    }
}
```

## Troubleshooting

### Service Won't Start

```bash
# Check systemd status
sudo systemctl status clicd

# View full logs
sudo journalctl -u clicd --no-pager

# Check config syntax
clicd config validate

# Common issues:
# - Port already in use: Change server.port in config
# - Permission denied: Ensure user has access to /var/lib/clicd
# - LXC not found: Install lxc package
```

### Container Creation Fails

```bash
# Check LXC installation
lxc-checkconfig

# Verify storage pool
lxc storage list

# Check network bridge
ip addr show lxcbr0

# Test LXC manually
sudo lxc-create -n test -t download -- -d ubuntu -r jammy -a amd64
sudo lxc-start -n test

# Enable debug logging
export CLICD_LOG_LEVEL=debug
sudo systemctl restart clicd
```

### NAT Port Forwarding Not Working

```bash
# Check iptables rules
sudo iptables -t nat -L -n -v

# Verify IP forwarding
sysctl net.ipv4.ip_forward

# Enable if disabled
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# Check public IP configuration
clicd nat check-config

# Test port manually
sudo iptables -t nat -A PREROUTING -p tcp --dport 10080 -j DNAT --to-destination 10.0.3.100:80
sudo iptables -t nat -A POSTROUTING -s 10.0.3.0/24 -j MASQUERADE
```

### IPv6 Not Working

```bash
# Check IPv6 forwarding
sysctl net.ipv6.conf.all.forwarding

# Enable IPv6 forwarding
sudo sysctl -w net.ipv6.conf.all.forwarding=1

# Check prefix detection
clicd ipv6 detect-prefix

# Manually set prefix in config
# Edit /etc/clicd/config.yaml:
# networking:
#   ipv6:
#     enabled: true
#     prefix: "2001:db8::/48"
#     auto_detect: false

# Verify container has IPv6
lxc-attach -n container-name -- ip -6 addr show
```

### API Authentication Fails

```go
// Debug authentication
func debugAuth() {
    token, err := login()
    if err != nil {
        log.Fatal("Login error:", err)
    }
    
    // Verify token format
    fmt.Printf("Token: %s\n", token)
    
    // Test authenticated request
    httpReq, _ := http.NewRequest("GET", os.Getenv("CLICD_API_URL")+"/api/v1/containers", nil)
    httpReq.Header.Set("Authorization", "Bearer "+token)
    
    client := &http.Client{}
    resp, err := client.Do(httpReq)
    if err != nil {
        log.Fatal("Request error:", err)
    }
    
    fmt.Printf("Status: %d\n", resp.StatusCode)
    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("Response: %s\n", body)
}
```

### Database Locked Errors

```bash
# Check for multiple processes
ps aux | grep clicd

# Stop service
sudo systemctl stop clicd

# Check database integrity
sqlite3 /var/lib/clicd/clicd.db "PRAGMA integrity_check;"

# Backup and recreate if corrupted
sudo cp /var/lib/clicd/clicd.db /var/lib/clicd/clicd.db.backup
sudo rm /var/lib/clicd/clicd.db
sudo systemctl start clicd
```

### High Memory Usage

```bash
# Check container limits
clicd container list --show-usage

# Set memory limits
clicd container update --id 123 --memory 1024

# Enable swap limits
# Edit /etc/default/grub:
# GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
sudo update-grub
sudo reboot

# Monitor in real-time
watch -n 1 'free -h && echo && lxc-ls -f'
```

### Security Alerts Not Working

```bash
# Check conntrack module
lsmod | grep nf_conntrack

# Load module if missing
sudo modprobe nf_conntrack

# Verify conntrack-tools installed
which conntrack

# Test conntrack manually
sudo conntrack -L

# Enable security monitoring in config
# Edit /etc/clicd/config.yaml:
# security:
#   alerts_enabled: true
#   conntrack_monitoring: true

sudo systemctl restart clicd
```

### WebSSH/VNC Not Connecting

```bash
# Check container is running
clicd container status --id 123

# Verify SSH service in container
lxc-attach -n container-name -- systemctl status ssh

# Check WebSocket connection
# In browser console:
# ws://your-server:8080/api/v1/webssh/ticket/your-ticket

# For VNC, ensure VNC server is installed in container
lxc-attach -n container-name -- apt install tigervnc-standalone-server

# Test VNC manually
vncviewer container-ip:5900
```

## Integration Examples

### Billing System Integration (Mofang)

CLICD includes a
