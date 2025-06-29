# n8n Docker Setup - Persistent Deployment

This guide covers running n8n persistently with Docker Compose using your existing workflows and data.

## Prerequisites

- Docker and Docker Compose installed
- Existing `n8n_data` volume with your workflows (already created)

## Configuration

### Docker Compose File (`docker-compose.yml`)

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=Pacific/Honolulu
      - TZ=Pacific/Honolulu
      # Add any other environment variables from your README examples
    volumes:
      - n8n_data:/home/node/.n8n
    # Uncomment below if you want tunnel mode
    # command: start --tunnel

volumes:
  n8n_data:
    external: true  # Uses your existing volume 
```

## Basic Commands

### Starting n8n

```bash
# Start n8n in the background (detached mode)
docker compose up -d

# Start with logs visible (foreground mode)
docker compose up
```

### Checking Status

```bash
# Check if containers are running
docker compose ps

# View current logs
docker compose logs n8n

# Follow logs in real-time
docker compose logs -f n8n
```

### Stopping n8n

```bash
# Stop n8n (can be restarted)
docker compose down

# Stop only the n8n service
docker compose stop n8n
```

### Restarting n8n

```bash
# Restart the n8n service
docker compose restart n8n

# Stop and start again
docker compose down && docker compose up -d
```

## Management Commands

### Updating n8n

```bash
# Pull latest image
docker compose pull

# Stop current version and start with new image
docker compose down
docker compose up -d

# One-liner update
docker compose pull && docker compose up -d --force-recreate
```

### Monitoring

```bash
# Check resource usage
docker stats n8n

# View container details
docker inspect n8n

# Check volume usage
docker volume inspect n8n_data
```

### Troubleshooting

```bash
# View detailed logs
docker compose logs --details n8n

# Check if volume exists
docker volume ls | grep n8n_data

# Inspect container configuration
docker compose config

# Force recreate container
docker compose up -d --force-recreate
```

## Full Deployment Workflow

1. **Navigate to project directory:**
   ```bash
   cd /path/to/your/project
   pwd  # Verify you're in the right location
   ```

2. **Verify your volume exists:**
   ```bash
   docker volume ls | grep n8n_data
   ```

3. **Start n8n:**
   ```bash
   docker compose up -d
   ```

4. **Verify it's running:**
   ```bash
   docker compose ps
   docker compose logs n8n
   ```

5. **Access n8n:**
   - Open browser to: http://localhost:5678
   - Your existing workflows should be available

## Environment Configuration

### Timezone Settings
- **Current timezone:** Pacific/Honolulu (HST)
- **GENERIC_TIMEZONE:** Controls n8n scheduling (Schedule node)
- **TZ:** Controls system timezone for scripts and commands

### Optional Environment Variables

Add these to the `environment:` section if needed:

```yaml
environment:
  - GENERIC_TIMEZONE=Pacific/Honolulu
  - TZ=Pacific/Honolulu
  - N8N_BASIC_AUTH_ACTIVE=true
  - N8N_BASIC_AUTH_USER=admin
  - N8N_BASIC_AUTH_PASSWORD=your_password_here
  - N8N_HOST=0.0.0.0
  - N8N_PORT=5678
  - NODE_OPTIONS=--max-old-space-size=2048
```

## Webhook/Tunnel Mode

To enable external webhook access, uncomment the command line:

```yaml
services:
  n8n:
    # ... other settings ...
    command: start --tunnel
```

Then restart:
```bash
docker compose down && docker compose up -d
```

## Backup and Maintenance

### Backup Your Data
```bash
# Create backup of the entire volume
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine tar czf /backup/n8n_backup_$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# List backups
ls -la n8n_backup_*.tar.gz
```

### Restore from Backup
```bash
# Stop n8n first
docker compose down

# Restore backup (replace with your backup filename)
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine tar xzf /backup/n8n_backup_YYYYMMDD_HHMMSS.tar.gz -C /data

# Start n8n
docker compose up -d
```

## Quick Reference

| Action | Command |
|--------|---------|
| Start | `docker compose up -d` |
| Stop | `docker compose down` |
| Restart | `docker compose restart` |
| Logs | `docker compose logs -f n8n` |
| Status | `docker compose ps` |
| Update | `docker compose pull && docker compose up -d` |
| Access | http://localhost:5678 |

## Features of This Setup

- ✅ **Persistent:** Survives container restarts and system reboots
- ✅ **Auto-restart:** Automatically restarts if it crashes
- ✅ **Data preservation:** Uses your existing `n8n_data` volume
- ✅ **Easy management:** Simple Docker Compose commands
- ✅ **Honolulu timezone:** Properly configured for HST
- ✅ **Background operation:** Runs as a daemon service

## Support

If you encounter issues:

1. Check logs: `docker compose logs n8n`
2. Verify volume: `docker volume inspect n8n_data`
3. Check Docker status: `docker system info`
4. Restart services: `docker compose restart`

Your workflows and data in the `n8n_data` volume are always preserved across container operations. 