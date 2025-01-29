## Step-by-Step Instructions

### 1. Update System Packages
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### 2. Install Required Packages
#### For NFS Shares:
```bash
sudo apt-get install nfs-common -y
```

#### For CIFS/SMB Shares:
```bash
sudo apt-get install cifs-utils -y
```

### 3. Configure Storage Mounts

#### Option A: NFS Configuration
Edit the `fstab` file to add NFS mounts:
```bash
sudo nano /etc/fstab
```

Add the following entries (replace `SERVER_IP` and `SHARE_PATH`):
```text
SERVER_IP:/SHARE_PATH/docker     /mnt/nas/docker    nfs    rw,hard,intr,rsize=8192,wsize=8192,timeo=14,_netdev,auto,nofail    0 0
SERVER_IP:/SHARE_PATH/data       /mnt/nas/data      nfs    rw,hard,intr,rsize=8192,wsize=8192,timeo=14,_netdev,auto,nofail    0 0
```

#### Option B: CIFS/SMB Configuration
Create a credentials file:
```bash
sudo nano /root/.smbcredentials
```

Add your credentials:
```text
username=your_username
password=your_password
```

Secure the credentials file:
```bash
sudo chmod 600 /root/.smbcredentials
```

Edit the `fstab` file to add CIFS mounts:
```bash
sudo nano /etc/fstab
```

Add the following entries (replace `SERVER_IP`):
```text
//SERVER_IP/docker     /mnt/nas/docker    cifs    credentials=/root/.smbcredentials,iocharset=utf8,vers=3.0,cache=strict,file_mode=0777,dir_mode=0777,_netdev,auto,nofail    0 0
//SERVER_IP/data       /mnt/nas/data      cifs    credentials=/root/.smbcredentials,iocharset=utf8,vers=3.0,cache=strict,file_mode=0777,dir_mode=0777,_netdev,auto,nofail    0 0
```

### 4. Test and Verify Mounts
Mount all volumes:
```bash
sudo mount -a
```

Verify mounts:
```bash
df -h
```

### 5. Set Permissions
```bash
sudo chown -R 1000:100 /mnt/nas/docker
sudo chown -R 1000:100 /mnt/nas/data
```

---

## Important Notes
- **Permission Settings:**  
  Ensure `PUID=1000` and `PGID=100` in your Docker Compose file match the ownership of the mounted directories.
- Adjust the `PUID` and `PGID` values as needed based on your user/group IDs.

---

## Troubleshooting Tips
- **Network Connectivity:**  
  Verify that the VM can reach the network storage server using `ping` or `nc`.
- **Mount Issues:**  
  Check the system logs for mount errors:
  ```bash
  journalctl -u systemd-fsck* -b
  ```
- **Permissions:**  
  Ensure the mounted directories have the correct permissions and ownership.

---

## Docker Compose Configuration Example

Here's an example of how to map these volumes in your `docker-compose.yml` file:

```yaml
version: '3'
services:
  my-container:
    image: my-image:latest
    volumes:
      - /mnt/nas/docker:/app/data
      - /mnt/nas/data:/var/lib/mysql
```

---

## Verification Steps

After setting up the mounts, verify that your Docker containers can access the shared storage:

1. Check mounted directories inside a running container:
   ```bash
   docker exec my-container ls -la /app/data
   ```

2. Verify permissions and ownership:
   ```bash
   docker exec my-container stat /app/data
   ```

3. Check system logs for any issues:
   ```bash
   journalctl -u systemd-fsck* -b
   ```
```
