# 🚀 Production Deployment Guide

Simple and effective guide to deploy File Manager in production with Docker.

## 📋 Prerequisites

- **Server**: Ubuntu 20.04+ with root access
- **Domain**: Domain name pointed to your server
- **Resources**: Minimum 2GB RAM, 20GB storage
- **Open ports**: 22 (SSH), 80 (HTTP), 443 (HTTPS)

## 🐳 Docker Method (Recommended)

### Step 1: Server Preparation

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget git

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Docker Compose v2 is included with Docker installation

# Reboot to apply permissions
sudo reboot
```

### Step 2: Application Deployment

```bash
# Clone repository
git clone https://github.com/LOSS98/files-management-app-with-api.git
cd files-management-app-with-api

# Configure environment
cp .env.example .env
nano .env
```

**Environment Configuration (.env):**
```env
NODE_ENV=production
JWT_SECRET=your-super-secure-256-bit-secret-key-change-this-now
ADMIN_PASSWORD=YourSecureAdminPassword123!
```

**Start with Docker:**
```bash
# Build and start containers
docker compose up -d --build

# Check status
docker compose ps
docker compose logs -f
```


### Step 3: DNS Configuration

**Configure your domain's DNS records:**

1. **A Record**: `your-domain.com` → `your-server-ip`
2. **CNAME Record**: `www.your-domain.com` → `your-domain.com`

**Verify DNS propagation:**
```bash
# Check A record
nslookup your-domain.com

# Test HTTP connection
curl -I http://your-domain.com
```

**⚠️ Important:** Wait for DNS propagation before proceeding to the next step.


### Step 4: Firewall Configuration

```bash
# Configure UFW
sudo ufw --force reset

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH, HTTP and HTTPS
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw --force enable

# Check status
sudo ufw status verbose
```

### Step 5: Deployment Verification

```bash
# Check services
docker compose ps

# Test application
curl -I http://your-domain.com:3000

# Check logs
docker compose logs -f
```

**Access your application:**
- **URL**: http://your-domain.com:3000
- **Admin login**: `admin` / `YourSecurePassword123!`

## 🔧 Maintenance and Monitoring

### Automatic monitoring

**Monitoring script (`/usr/local/bin/file-manager-health.sh`):**
```bash
#!/bin/bash
# File Manager monitoring script

cd /path/to/files-management-app-with-api

# Check Docker containers
if ! docker compose ps | grep -q "Up"; then
    echo "Docker containers stopped, restarting..."
    docker compose restart
fi

```

```bash
# Make executable
sudo chmod +x /usr/local/bin/file-manager-health.sh

# Add to crontab (check every 5 minutes)
sudo crontab -e
*/5 * * * * /usr/local/bin/file-manager-health.sh
```

### Automatic backup

**Backup script (`/usr/local/bin/file-manager-backup.sh`):**
```bash
#!/bin/bash
# File Manager backup script

BACKUP_DIR="/backup/file-manager"
DATE=$(date +%Y%m%d_%H%M%S)
APP_DIR="/path/to/files-management-app-with-api"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
docker compose exec -T backend cat /app/data/database.sqlite > $BACKUP_DIR/database_$DATE.sqlite

# Backup uploaded files
tar -czf $BACKUP_DIR/uploads_$DATE.tar.gz -C $APP_DIR uploads/

# Backup configuration
cp $APP_DIR/.env $BACKUP_DIR/env_$DATE.backup

# Remove backups older than 30 days
find $BACKUP_DIR -name "*" -type f -mtime +30 -delete

echo "Backup completed: $DATE"
```

```bash
# Make executable
sudo chmod +x /usr/local/bin/file-manager-backup.sh

# Add to crontab (daily backup at 2 AM)
sudo crontab -e
0 2 * * * /usr/local/bin/file-manager-backup.sh
```

## 🔄 Updates

### Application update

```bash
# Go to directory
cd /path/to/files-management-app-with-api

# Pull latest changes
git pull origin main

# Rebuild and restart containers
docker compose down
docker compose up -d --build

# Check status
docker compose ps
```


## 🚨 Troubleshooting

### Common issues

**Containers won't start:**
```bash
# Check logs
docker compose logs

# Check disk space
df -h

# Check configuration
docker compose config
```


**Performance issues:**
```bash
# Monitor resources
htop
docker stats

# Check error logs
docker compose logs --tail=100
```

**DNS issues:**
```bash
# Check DNS propagation
nslookup your-domain.com
dig your-domain.com

# Test connectivity
ping your-domain.com
```

Your File Manager application is now securely deployed in production! 🎉

---

**Steps Summary:**
1. ✅ Server preparation + Docker
2. ✅ Application deployment with Docker Compose  
3. ✅ Domain DNS configuration
4. ✅ UFW firewall configuration
5. ✅ Automatic monitoring and backup

**Advantages of this method:**
- 🚀 **Simple**: Direct Docker deployment
- ⚡ **Fast**: Fewer manual steps  
- 🐳 **Containerized**: Everything runs in Docker