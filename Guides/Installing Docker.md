# Installing Docker in an LXC Container on Proxmox 8.3.4

This guide walks you through creating a dedicated LXC container on Proxmox for running Docker (Docker in LXC), with all the required configurations to ensure it works properly.

## Step 1: Create a new LXC container

1. Log in to the Proxmox web interface
2. Select your Proxmox node in the server list
3. Click on "Create CT" to create a new container

## Step 2: Configure the container

Fill in the following settings:
- **General**
  - Node: (your Proxmox node)
  - CT ID: (choose an available ID, e.g., 100)
  - Hostname: docker-host
  - Unprivileged container: **NO** (must be privileged)
  - Password: (set a secure password)
  - SSH public key: (optional, but recommended)

- **Template**
  - Select an Ubuntu 20.04 template

- **Disks**
  - Storage: (select your preferred storage)
  - Disk size: at least 20GB recommended

- **CPU**
  - Cores: 2 (or more depending on your needs)

- **Memory**
  - Memory: 2048 MB (minimum recommended)
  - Swap: 512 MB

- **Network**
  - Configure your network settings as needed

## Step 3: Apply necessary container settings

Before starting the container, you need to modify its configuration to allow Docker to work properly:

1. In the Proxmox web UI, select your new container
2. Go to "Options"
3. Edit "Features"
   - Enable: `keyctl`, `nesting`

4. From the Proxmox shell, edit the container's configuration file:

```bash
nano /etc/pve/lxc/YOUR_CT_ID.conf
```

5. Add these lines to the file:

```
lxc.apparmor.profile: unconfined
lxc.cgroup2.devices.allow: a
lxc.cap.drop: 
```

Also ensure these are uncommented/present:
```
lxc.mount.auto: proc:rw sys:rw
```

## Step 4: Start the container and access it

1. Start the container from the Proxmox web UI
2. Click "Console" to access the container shell or SSH into it

## Step 5: Install Docker within the container

Once inside the container, install Docker:

```bash
# Update the system
apt update && apt upgrade -y

# Install dependencies
apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod 644 /etc/apt/keyrings/docker.gpg

# Add Docker repository (for Ubuntu 20.04)
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index
apt update

# Install Docker Engine
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verify installation
docker run hello-world
```

## Step 6: Create a dedicated storage location (optional)

If you want to store Docker data in a specific location:

```bash
# Stop Docker
systemctl stop docker

# Create a directory for Docker data
mkdir -p /path/to/docker/data

# Configure Docker to use the new path
nano /etc/docker/daemon.json
```

Add this content:
```json
{
  "data-root": "/path/to/docker/data"
}
```

```bash
# Start Docker again
systemctl start docker
```

## Step 7: Enable Docker to start on boot

```bash
systemctl enable docker
```

#  Optional, but recommended: # Enabling Monitoring for LXC Containers in Proxmox

Unlike virtual machines (VMs), LXC containers don't use the QEMU Guest Agent. However, you can set up proper monitoring and communication between your LXC container and Proxmox using the following methods.

## Inside the LXC Container

1. The `pve-lxc-syscalld` service typically runs on the Proxmox host, not in the container. Instead of looking for a specific agent package, ensure proper container configuration:

```bash
apt update
apt install -y iproute2 procps
```

2. For more detailed statistics in the Proxmox web interface, install these additional packages:

```bash
apt install -y lxcfs
```

3. If available in your distribution, you can also install cgroup tools:

```bash
# For Debian/Ubuntu
apt install -y cgroup-tools

# For other distributions, the package might be named differently
# apt install -y libcgroup1
```

## On the Proxmox Host

1. Edit the container's configuration file:

```bash
nano /etc/pve/lxc/YOUR_CT_ID.conf
```

2. Add or ensure these lines are present:

```
lxc.mount.entry: /sys/fs/fuse/connections sys/fs/fuse/connections none bind,optional 0 0
lxc.mount.entry: /sys/kernel/debug sys/kernel/debug none bind,optional 0 0
```

## Apply the Changes

After making these changes, restart your container to apply them:

```bash
pct restart YOUR_CT_ID
```

## Verification

To verify that monitoring is working properly:
1. In the Proxmox web UI, select your container
2. Go to the "Summary" tab
3. Check that resource usage statistics (CPU, memory, etc.) are being reported correctly
4. You should see more detailed information in the "Resources" section


## Troubleshooting

If you encounter issues:

1. **Docker daemon fails to start**: Check that all the container configuration settings from Step 3 have been applied correctly.

2. **Permission errors**: Ensure the container is running in privileged mode.

3. **Network issues**: You might need to adjust the container's network configuration to allow Docker containers to access the network properly.

4. **Resource limitations**: If Docker is running slowly, consider allocating more CPU cores and memory to the LXC container.
