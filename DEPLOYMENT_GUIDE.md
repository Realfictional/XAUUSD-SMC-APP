# XAU-60 Trading Bot - Self-Hosted Deployment Guide

## Prerequisites

- **Windows Server** (Windows 10/11 or Server 2019+) with MT5 installed
- **Docker Desktop** (with Windows container support) or **Docker Server**
- **Git** (for cloning the repository)
- **Python 3.11+** (for local development)
- **MetaTrader 5** installed and running with demo/live account

## Quick Start

### 1. Clone & Setup

```bash
git clone https://github.com/yourusername/XAU-60-main.git
cd XAU-60-main

# Copy environment template
copy .env.example .env

# Edit .env with your MT5 credentials and webhook URLs
notepad .env
```

### 2. Configure Environment Variables

**Required for live trading:**
```env
MT5_LOGIN=123456              # Your MT5 account number
MT5_PASSWORD=YourPassword     # MT5 password
MT5_SERVER=YourBroker-Live    # Broker server name
ENCRYPTION_KEY=<generated>    # Generate: python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

**Optional for alerts:**
```env
DISCORD_WEBHOOK=https://discord.com/api/webhooks/...
TELEGRAM_TOKEN=123456:ABC...
TELEGRAM_CHAT_ID=987654321
```

### 3. Build & Run with Docker

```bash
# Build image
docker build -t xau60-bot .

# Run container
docker run -p 8501:8501 \
  --env-file .env \
  --volume C:/Program\ Files/MetaTrader\ 5:C:/MetaTrader5:ro \
  --name xau60-bot \
  xau60-bot
```

Or with Docker Compose (recommended):

```bash
docker-compose up -d
```

Access the app at: **http://localhost:8501**

## Local Development (Without Docker)

```bash
# Install dependencies
pip install -r requirements.txt

# Run directly
streamlit run streamlit_app.py
```

## Production Deployment

### Option A: Docker + Windows Service

1. **Create service batch file** (`start-bot.bat`):
```batch
@echo off
cd C:\XAU-60-main
docker-compose up
pause
```

2. **Use NSSM to create Windows Service**:
```bash
nssm install XAU60-Bot "C:\start-bot.bat"
nssm start XAU60-Bot
```

### Option B: Docker + Systemd (Linux server)

```bash
# Create systemd service
sudo nano /etc/systemd/system/xau60-bot.service
```

```ini
[Unit]
Description=XAU-60 Trading Bot
After=docker.service

[Service]
Type=simple
WorkingDirectory=/root/XAU-60-main
ExecStart=docker-compose up
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable xau60-bot
sudo systemctl start xau60-bot
```

### Option C: Kubernetes Deployment

See `k8s-deployment.yaml` for Kubernetes setup (advanced).

## Monitoring & Logs

### Docker logs:
```bash
docker-compose logs -f xau60-trading-bot
```

### Local logs:
```bash
tail -f logs/trading_bot.log
```

### Health check:
```bash
curl http://localhost:8501
```

## Security Best Practices

1. **Never commit .env file** - it contains sensitive credentials
2. **Use strong encryption key** - generate with: 
   ```python
   from cryptography.fernet import Fernet
   print(Fernet.generate_key().decode())
   ```
3. **Enable firewall rules** - expose port 8501 only internally
4. **Use HTTPS in production** - deploy behind Nginx/Traefik with SSL
5. **Rotate credentials regularly** - update MT5 passwords periodically
6. **Monitor resource usage** - set Docker memory limits (2GB default)

## Troubleshooting

### MT5 Connection Failed
```bash
# Check if MT5 is running
tasklist | findstr "terminal.exe"

# Check MT5 terminal path
docker exec xau60-bot python -c "from core.mt5_connector import check_mt5; print(check_mt5())"
```

### Port 8501 Already in Use
```bash
# Find process using port 8501
netstat -ano | findstr :8501

# Kill process or change port in docker-compose.yml
```

### Memory Issues
Increase Docker memory allocation:
```bash
# In docker-compose.yml under deploy.resources.limits
memory: 4G  # Increase from 2G
```

## Backup & Recovery

### Backup configuration:
```bash
robocopy config C:\backups\config /MIR
robocopy .streamlit C:\backups\.streamlit /MIR
```

### Backup logs:
```bash
robocopy logs C:\backups\logs /MIR
```

## Scaling

For multiple strategies/accounts:
1. Deploy multiple containers with different configs
2. Use Docker Compose to scale:
   ```bash
   docker-compose up -d --scale bot=3
   ```
3. Load balance with Nginx

## Updates

```bash
# Pull latest code
git pull origin main

# Rebuild Docker image
docker-compose build --no-cache

# Restart service
docker-compose restart
```

## Support

- Check logs: `docker-compose logs xau60-trading-bot`
- Review DEPLOYMENT_REPORT.md for architecture details
- Test with demo MT5 account first before live trading
