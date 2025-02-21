# Market Correlation Agent Setup Guide

This guide provides comprehensive instructions for setting up the Market Correlation Agent in both development and production environments.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Development Environment Setup](#development-environment-setup)
- [API Configuration](#api-configuration)
- [Twitter Setup](#twitter-setup)
- [Production Deployment](#production-deployment)
- [Environment Variables](#environment-variables)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### System Requirements
- Python 3.8 or higher
- Chrome browser
- 2GB RAM minimum
- 10GB free disk space
- Linux/macOS/Windows compatible

### Required Software
- Git
- Python pip
- Virtual environment tool (virtualenv or venv)
- Chrome browser
- ChromeDriver matching your Chrome version

## Development Environment Setup

### 1. Python Environment Setup
```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. ChromeDriver Installation

#### Linux
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install chromium-chromedriver

# Check installation
chromedriver --version
```

#### macOS
```bash
# Using Homebrew
brew install --cask chromedriver

# Check installation
chromedriver --version
```

#### Windows
1. Download ChromeDriver from https://sites.google.com/chromium.org/driver/
2. Add to PATH or specify location in .env file

### 3. Repository Setup
```bash
# Clone repository
git clone https://github.com/yourusername/eth-btc-correlation-bot.git
cd eth-btc-correlation-bot

# Create local config
cp .env.example .env
```

## API Configuration

### 1. Claude AI Setup
1. Sign up at https://anthropic.com/
2. Generate API key from dashboard
3. Add to .env file as `CLAUDE_API_KEY`

### 2. CoinGecko API
- No API key required for basic usage
- Rate limits: 50 calls/minute
- Consider Pro API for production use

## Twitter Setup

### 1. Twitter Developer Account
1. Apply for Twitter Developer Account at https://developer.twitter.com/
2. Create Twitter Application
3. Enable read/write permissions
4. Generate access tokens

### 2. Twitter Account Security
1. Enable 2FA on your Twitter account
2. Store backup codes securely
3. Configure app permissions

## Production Deployment

### 1. Server Setup
```bash
# Update system
sudo apt-get update
sudo apt-get upgrade

# Install required packages
sudo apt-get install python3-pip python3-venv chromium-chromedriver

# Setup application directory
sudo mkdir /opt/eth-btc-bot
sudo chown $(whoami):$(whoami) /opt/eth-btc-bot
```

### 2. Systemd Service Configuration
```ini
# /etc/systemd/system/eth-btc-bot.service
[Unit]
Description=ETH/BTC Correlation Bot
After=network.target

[Service]
Type=simple
User=bot-user
WorkingDirectory=/opt/eth-btc-bot
Environment=PATH=/opt/eth-btc-bot/venv/bin:/usr/local/bin:/usr/bin:/bin
ExecStart=/opt/eth-btc-bot/venv/bin/python main.py
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
```

### 3. Service Management
```bash
# Enable and start service
sudo systemctl enable eth-btc-bot
sudo systemctl start eth-btc-bot

# Check status
sudo systemctl status eth-btc-bot
```

## Environment Variables

### Required Variables
```bash
# Twitter Credentials
TWITTER_USERNAME=your_twitter_username
TWITTER_PASSWORD=your_twitter_password

# API Keys
CLAUDE_API_KEY=your_claude_api_key

# System Paths
CHROME_DRIVER_PATH=/usr/local/bin/chromedriver  # Adjust as needed
```

### Optional Variables
```bash
# Analysis Configuration
MAX_RETRIES=3
CORRELATION_INTERVAL=5  # minutes

# Logging
LOG_LEVEL=INFO
LOG_FILE_PATH=/var/log/eth-btc-bot/bot.log
```

## Troubleshooting

### Common Issues

1. ChromeDriver Version Mismatch
```bash
# Check Chrome version
google-chrome --version

# Download matching ChromeDriver version
# Update CHROME_DRIVER_PATH in .env
```

2. Twitter Authentication Failures
- Verify credentials in .env
- Check for 2FA requirements
- Ensure no active login sessions

3. API Rate Limits
- Implement exponential backoff
- Monitor API usage
- Consider upgrading API tier

### Logs Location
- Application logs: `/var/log/eth-btc-bot/bot.log`
- System logs: `journalctl -u eth-btc-bot`

### Support
For additional support:
1. Check GitHub Issues
2. Review error logs
3. Contact maintenance team
