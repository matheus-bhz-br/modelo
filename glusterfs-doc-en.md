# Documentation: GlusterFS Replicated Volume Configuration

## Index
1. Introduction
2. Architecture
3. Prerequisites
4. Installation Process
5. Problems Encountered and Solutions
6. Best Practices
7. Verification and Testing

## 1. Introduction

This document details the process of configuring a GlusterFS cluster with three nodes in replicated configuration. The environment was configured in January 2025 using GlusterFS 10.3 on Debian systems.

### Final Configuration
- 3 nodes in full replication
- Volume mounted via FUSE
- Synchronous data replication
- Active self-heal

## 2. Architecture

### Cluster Nodes
- Node 1 (rede): 172.29.224.106
- Node 2 (hailo): 172.29.221.194
- Node 3 (dados): 172.29.218.187

### Directory Structure
- Mount point: `/mnt/glusterfs`
- Bricks directory: `/media/matheus/rede/data/gluster/gluster_volume`

## 3. Prerequisites

### On All Nodes
```bash
# GlusterFS Installation
sudo apt install glusterfs-server

# Firewall Configuration
sudo ufw allow 24007,24008/tcp
sudo ufw allow 49152:49251/tcp
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw reload
```

### Required Ports
- 24007: GlusterFS Management
- 24008: Inter-node Communication
- 49152:49251: Dynamic ports for bricks
- Specific brick ports (identified during configuration)

## 4. Installation Process

### 4.1 Node Preparation
```bash
# On all nodes
sudo systemctl start glusterd
sudo systemctl enable glusterd
```

### 4.2 Cluster Configuration
```bash
# On the first node (rede)
sudo gluster peer probe 172.29.221.194
sudo gluster peer probe 172.29.218.187

# Verify status
sudo gluster peer status
```

### 4.3 Volume Creation
```bash
# Create directories on all nodes
sudo mkdir -p /media/matheus/rede/data/gluster/gluster_volume
sudo chown -R nobody:nogroup /media/matheus/rede/data/gluster/gluster_volume

# Create replicated volume
sudo gluster volume create volume1 replica 3 \
    172.29.224.106:/media/matheus/rede/data/gluster/gluster_volume \
    172.29.221.194:/media/matheus/rede/data/gluster/gluster_volume \
    172.29.218.187:/media/matheus/rede/data/gluster/gluster_volume

# Start the volume
sudo gluster volume start volume1
```

### 4.4 Volume Mounting
```bash
# On the node where you want to mount
sudo mkdir -p /mnt/glusterfs
sudo mount.glusterfs 172.29.224.106:volume1 /mnt/glusterfs
```

## 5. Problems Encountered and Solutions

### 5.1 Problem: Directory Already Part of a Volume
**Error:** "/media/matheus/rede/data/gluster/replica is already part of a volume"

**Solution:**
1. Rename/move existing directory
2. Create new directory with different name
3. Clean GlusterFS metadata if necessary:
```bash
sudo rm -rf /var/lib/glusterd/*
sudo systemctl restart glusterd
```

### 5.2 Problem: Mount Failure
**Error:** "Mount failed. Check the log file for more details."

**Solution:**
1. Check port connectivity
2. Open specific brick ports
3. Reinstall GlusterFS client
4. Check detailed logs

### 5.3 Problem: Connectivity
**Solution:**
```bash
# Open specific brick ports
sudo ufw allow 57066/tcp
sudo ufw allow 59763/tcp
sudo ufw allow 56225/tcp
sudo ufw reload
```

## 6. Best Practices

### 6.1 Preparation
- Plan directory structure in advance
- Document IPs and hostnames
- Verify network requirements before installation
- Configure NTP for time synchronization

### 6.2 Configuration
- Use descriptive volume names
- Implement monitoring from the start
- Configure GlusterFS metadata backup
- Document all changes

### 6.3 Maintenance
- Regularly monitor logs and status
- Maintain configuration file backups
- Test failure recovery periodically
- Keep documentation updated

## 7. Verification and Testing

### 7.1 Cluster Verification
```bash
# Peer status
sudo gluster peer status

# Volume information
sudo gluster volume info
sudo gluster volume status
```

### 7.2 Replication Testing
```bash
# Create test file
sudo touch /mnt/glusterfs/test.txt

# Verify on all nodes
ls -la /media/matheus/rede/data/gluster/gluster_volume/
```

### 7.3 Mount Verification
```bash
# Check mount point
df -h | grep glusterfs
mount | grep glusterfs
```

## Conclusion

This configuration establishes a three-node GlusterFS cluster with full replication, providing high availability and data redundancy. The main difficulties encountered were related to network connectivity and port configuration, which were resolved with proper firewall configuration and opening of necessary ports.

For future implementations, it is recommended to:
1. Detailed planning of network structure
2. Prior documentation of all requirements
3. Connectivity testing before installation
4. Implementation of monitoring from the start
5. Backup and recovery planning

This documentation should be kept updated as changes are made to the environment.