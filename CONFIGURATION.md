# Configuration Guide

## Table of Contents
- [Market Analysis Parameters](#market-analysis-parameters)
- [Twitter Agent Settings](#twitter-bot-settings)
- [API Configuration](#api-configuration)
- [Logging Configuration](#logging-configuration)
- [Browser Configuration](#browser-configuration)
- [Performance Tuning](#performance-tuning)
- [Environment Variables](#environment-variables)

## Market Analysis Parameters

### Correlation Settings
```python
MARKET_ANALYSIS_CONFIG = {
    # How sensitive the correlation detection should be (0.0 to 1.0)
    'correlation_sensitivity': 0.7,
    
    # Minimum price movement to trigger analysis (percentage)
    'volatility_threshold': 2.0,
    
    # Minimum volume in USD to consider significant
    'volume_significance': 100000,
    
    # Time periods to analyze (in hours)
    'historical_periods': [1, 4, 24]
}
```

#### Tuning Guidelines:
- Increase `correlation_sensitivity` for more frequent updates
- Decrease `volatility_threshold` to catch smaller price movements
- Adjust `volume_significance` based on market conditions
- Modify `historical_periods` to change analysis timeframes

### Performance Impact:
```python
# Higher sensitivity = More API calls
if correlation_sensitivity < 0.5:
    api_calls_per_hour = 30
elif correlation_sensitivity < 0.8:
    api_calls_per_hour = 60
else:
    api_calls_per_hour = 120
```

## Twitter Agent Settings

### Posting Configuration
```python
TWEET_CONSTRAINTS = {
    # Minimum tweet length to ensure quality
    'MIN_LENGTH': 220,
    
    # Target maximum length for readability
    'MAX_LENGTH': 270,
    
    # Absolute maximum length allowed by Twitter
    'HARD_STOP_LENGTH': 280
}

# Posting frequency control
CORRELATION_INTERVAL = 5  # minutes
MAX_RETRIES = 3  # posting attempts
```

### Duplicate Detection
```python
# Price change thresholds for new posts
DUPLICATE_DETECTION = {
    'min_price_change_percentage': 0.01,
    'min_time_between_posts': 30,  # seconds
    'check_last_n_posts': 10
}
```

## API Configuration

### CoinGecko Settings
```python
COINGECKO_PARAMS = {
    "vs_currency": "usd",
    "ids": "bitcoin,ethereum",
    "order": "market_cap_desc",
    "per_page": 2,
    "page": 1,
    "sparkline": False,
    "price_change_percentage": "1h,24h,7d"
}

# API endpoint configuration
COINGECKO_BASE_URL = "https://api.coingecko.com/api/v3"
```

### Claude AI Settings
```python
CLAUDE_CONFIG = {
    'model': 'claude-3-5-sonnet-20241022',
    'max_tokens': 1500,
    'temperature': 0.7
}

# Analysis prompt template
CLAUDE_ANALYSIS_PROMPT = """
[Detailed prompt template for market analysis]
"""
```

## Logging Configuration

### Log Levels
```python
# logging.conf
{
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'bot.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 5
        },
        'console': {
            'class': 'logging.StreamHandler',
            'stream': 'ext://sys.stdout'
        }
    },
    'root': {
        'level': 'INFO',
        'handlers': ['file', 'console']
    }
}
```

### Event Categories
```python
# Event logging configuration
EVENT_LOGGING = {
    'market_data': True,
    'analysis': True,
    'twitter': True,
    'api_calls': True,
    'errors': True,
    'performance': True
}
```

## Browser Configuration

### Selenium Settings
```python
BROWSER_CONFIG = {
    'page_load_timeout': 45,
    'implicit_wait': 10,
    'explicit_wait': 20,
    'headless': True,
    'window_size': {
        'width': 1920,
        'height': 1080
    }
}

# Chrome options
CHROME_OPTIONS = [
    '--no-sandbox',
    '--disable-dev-shm-usage',
    '--disable-gpu',
    '--disable-extensions',
    '--disable-notifications'
]
```

## Performance Tuning

### Memory Management
```python
MEMORY_CONFIG = {
    'max_cached_responses': 100,
    'cache_ttl': 300,  # seconds
    'cleanup_interval': 3600  # seconds
}
```

### Rate Limiting
```python
RATE_LIMITS = {
    'coingecko': {
        'calls_per_minute': 50,
        'burst_limit': 60
    },
    'twitter': {
        'posts_per_hour': 15,
        'minimum_interval': 180  # seconds
    },
    'claude': {
        'calls_per_minute': 10,
        'tokens_per_minute': 10000
    }
}
```

## Environment Variables

### Required Variables
```bash
# API Keys and Credentials
CLAUDE_API_KEY=your_claude_api_key
TWITTER_USERNAME=your_twitter_username
TWITTER_PASSWORD=your_twitter_password

# System Paths
CHROME_DRIVER_PATH=/usr/local/bin/chromedriver
LOG_FILE_PATH=/var/log/eth-btc-bot/bot.log
```

### Optional Variables
```bash
# Feature Flags
ENABLE_DETAILED_LOGGING=true
ENABLE_PERFORMANCE_METRICS=true
ENABLE_DEBUG_MODE=false

# Timeouts
REQUEST_TIMEOUT=30
BROWSER_TIMEOUT=45
API_TIMEOUT=60

# Performance
CACHE_SIZE=1000
WORKER_THREADS=4
```

### Configuration Validation
```python
def validate_config():
    """Validate all configuration parameters"""
    required_vars = [
        'CLAUDE_API_KEY',
        'TWITTER_USERNAME',
        'TWITTER_PASSWORD',
        'CHROME_DRIVER_PATH'
    ]
    
    for var in required_vars:
        if not os.getenv(var):
            raise ValueError(f"Missing required environment variable: {var}")
            
    validate_market_config()
    validate_twitter_config()
    validate_api_config()
```

## Configuration Management Best Practices

1. **Version Control**
   - Keep template configurations in version control
   - Document all configuration changes
   - Maintain changelog for config updates

2. **Security**
   - Never commit sensitive values
   - Use environment variables for secrets
   - Encrypt sensitive configurations

3. **Deployment**
   - Use different configs per environment
   - Validate configs before deployment
   - Maintain configuration backups

4. **Monitoring**
   - Log configuration changes
   - Alert on invalid configurations
   - Track configuration impact on performance
