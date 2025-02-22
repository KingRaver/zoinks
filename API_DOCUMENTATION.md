# API Documentation

This document provides comprehensive information about the APIs used in the Market Correlation Agent, including endpoint details, authentication methods, request/response formats, error handling, and best practices.

## Table of Contents
- [CoinGecko API Integration](#coingecko-api-integration)
- [Claude AI Integration](#claude-ai-integration)
- [Twitter API Integration](#twitter-api-integration)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Authentication Management](#authentication-management)
- [Best Practices](#best-practices)
- [Testing & Development](#testing--development)

## CoinGecko API Integration

### API Overview
The bot uses CoinGecko's public API to retrieve cryptocurrency market data. This data includes current prices, trading volumes, and price changes over various time periods.

### Base URL
```
https://api.coingecko.com/api/v3
```

### Endpoints Used

#### 1. Ping Endpoint
Used to check if the API is operational.

**Request:**
```
GET /ping
```

**Response:**
```json
{
  "gecko_says": "(V3) To the Moon!"
}
```

**Implementation:**
```python
def check_coingecko_status(self) -> bool:
    """Check if CoinGecko API is operational"""
    try:
        response = self.session.get(
            f"{self.config.COINGECKO_BASE_URL}/ping",
            timeout=(5, 10)
        )
        return response.status_code == 200
    except Exception:
        return False
```

#### 2. Markets Endpoint
Used to retrieve price data for BTC and ETH.

**Request:**
```
GET /coins/markets
```

**Parameters:**
```python
params = {
    "vs_currency": "usd",
    "ids": "bitcoin,ethereum",
    "order": "market_cap_desc",
    "per_page": 2,
    "page": 1,
    "sparkline": False,
    "price_change_percentage": "1h,24h,7d"
}
```

**Response:**
```json
[
  {
    "id": "bitcoin",
    "symbol": "btc",
    "name": "Bitcoin",
    "current_price": 63421.75,
    "market_cap": 1245859608739,
    "market_cap_rank": 1,
    "total_volume": 43625862493,
    "high_24h": 64213.47,
    "low_24h": 62521.35,
    "price_change_24h": 1452.23,
    "price_change_percentage_24h": 2.34,
    "price_change_percentage_1h_in_currency": 0.12,
    "price_change_percentage_24h_in_currency": 2.34,
    "price_change_percentage_7d_in_currency": 5.67
  },
  {
    "id": "ethereum",
    "symbol": "eth",
    "name": "Ethereum",
    "current_price": 3451.89,
    "market_cap": 405732945729,
    "market_cap_rank": 2,
    "total_volume": 24152863428,
    "high_24h": 3512.15,
    "low_24h": 3405.62,
    "price_change_24h": 42.15,
    "price_change_percentage_24h": 1.23,
    "price_change_percentage_1h_in_currency": 0.05,
    "price_change_percentage_24h_in_currency": 1.23,
    "price_change_percentage_7d_in_currency": 3.45
  }
]
```

**Implementation:**
```python
def _get_crypto_data(self) -> Optional[Dict[str, Any]]:
    """Fetch BTC and ETH data from CoinGecko with retries"""
    max_retries = 3
    retry_count = 0
    
    while retry_count < max_retries:
        try:
            response = self.session.get(
                self.config.get_coingecko_markets_url(),
                params=self.config.get_coingecko_params(),
                timeout=(30, 90)
            )
            response.raise_for_status()
            logger.log_coingecko_request("/markets", success=True)
            
            data = {coin['symbol'].upper(): coin for coin in response.json()}
            
            if 'BTC' not in data or 'ETH' not in data:
                logger.log_error("Crypto Data", "Missing BTC or ETH data")
                return None
            
            logger.logger.info(f"Successfully fetched crypto data for {', '.join(data.keys())}")
            return data
                
        except requests.exceptions.Timeout:
            retry_count += 1
            wait_time = retry_count * 10
            logger.logger.warning(f"CoinGecko timeout, attempt {retry_count}, waiting {wait_time}s...")
            time.sleep(wait_time)
                
        except Exception as e:
            logger.log_coingecko_request("/markets", success=False)
            logger.log_error("CoinGecko API", str(e))
            return None
        
    logger.log_error("CoinGecko API", "Maximum retries reached")
    return None
```

### Error Codes
- `429`: Rate limit exceeded
- `5xx`: Server errors

### Rate Limits
- Free tier: 10-50 calls/minute
- No API key required for basic usage
- Exponential backoff recommended

## Claude AI Integration

### API Overview
The bot uses Anthropic's Claude API to analyze cryptocurrency market data and generate insights. Claude is a large language model that processes the market data and returns natural language analysis of market trends, correlations, and predictions.

### Base URL
```
https://api.anthropic.com
```

### Authentication
Claude API requires an API key that should be included in the request headers.
```python
headers = {
    "x-api-key": self.config.CLAUDE_API_KEY,
    "anthropic-version": "2023-06-01"
}
```

### Endpoints Used

#### Messages Endpoint
Used to generate market analysis.

**Request:**
```
POST /v1/messages
```

**Request Body:**
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "max_tokens": 1500,
  "messages": [
    {
      "role": "user",
      "content": "Analyze ETH/BTC Market Dynamics:\n\nCurrent Market Data:\nBitcoin:\n- Price: $63,421.75\n- 24h Change: 2.34%\n- Volume: $43,625,862,493\n\nEthereum:\n- Price: $3,451.89\n- 24h Change: 1.23%\n- Volume: $24,152,863,428\n\nPlease provide a concise but detailed market analysis..."
    }
  ]
}
```

**Response:**
```json
{
  "id": "msg_012345abcdef",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "ETH/BTC Market Analysis:\n\nBTC is showing stronger momentum with a 2.34% gain compared to ETH's 1.23% increase over the last 24 hours. This divergence suggests capital is favoring Bitcoin in the short term, possibly due to institutional inflows and spot ETF demand.\n\nVolume analysis shows healthy trading activity for both assets, with BTC's volume-to-market cap ratio slightly higher, indicating more active price discovery. The current price action places BTC near a key resistance level at $64,500, while ETH faces overhead resistance at $3,500.\n\nThe ETH/BTC ratio has declined by approximately 1.1% in this period, continuing a short-term trend of Bitcoin outperformance. However, the ratio remains within its monthly range, suggesting this is not yet a significant trend breakdown.\n\nShort-term traders should watch for potential consolidation at these levels before the next directional move. A break above $64,500 for BTC could trigger further FOMO buying, while ETH needs to clear $3,500 to regain momentum relative to Bitcoin."
    }
  ],
  "model": "claude-3-5-sonnet-20241022",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 143,
    "output_tokens": 1024
  }
}
```

**Implementation:**
```python
def _analyze_market_sentiment(self, crypto_data: Dict[str, Any]) -> Optional[str]:
    """Use Claude to analyze market sentiment with enhanced analysis"""
    max_retries = 3
    retry_count = 0
    
    while retry_count < max_retries:
        try:
            btc = crypto_data['BTC']
            eth = crypto_data['ETH']
            
            prompt = self.config.CLAUDE_ANALYSIS_PROMPT.format(
                btc_price=btc['current_price'],
                btc_change=btc['price_change_percentage_24h'],
                btc_volume=btc['total_volume'],
                eth_price=eth['current_price'],
                eth_change=eth['price_change_percentage_24h'],
                eth_volume=eth['total_volume']
            )
            
            response = self.claude_client.messages.create(
                model=self.config.CLAUDE_MODEL,
                max_tokens=1500,
                messages=[{"role": "user", "content": prompt}]
            )
            
            analysis = response.content[0].text
            logger.log_claude_analysis(
                btc['current_price'],
                eth['current_price'],
                analysis[:100] + "..."
            )
            return self._format_tweet_analysis(analysis, btc, eth)
                
        except Exception as e:
            retry_count += 1
            wait_time = retry_count * 10
            logger.logger.warning(f"Claude API error, attempt {retry_count}, waiting {wait_time}s...")
            time.sleep(wait_time)
            continue
        
    logger.log_error("Market Sentiment Analysis", "Maximum retries reached")
    return None
```

### Error Codes
- `400`: Bad request (invalid input)
- `401`: Unauthorized (invalid API key)
- `404`: Not found
- `429`: Rate limit exceeded
- `5xx`: Server errors

### Rate Limits
- Rate limits vary by API plan and model
- Exponential backoff on 429 responses

### Prompt Template
```
Analyze ETH/BTC Market Dynamics:

Current Market Data:
Bitcoin:
- Price: ${btc_price:,.2f}
- 24h Change: {btc_change:.2f}%
- Volume: ${btc_volume:,.0f}

Ethereum:
- Price: ${eth_price:,.2f}
- 24h Change: {eth_change:.2f}%
- Volume: ${eth_volume:,.0f}

Please provide a concise but detailed market analysis:
1. Short-term Movement: 
   - Price action in last few minutes
   - Volume profile significance
   - Immediate support/resistance levels

2. Market Microstructure:
   - Order flow analysis
   - Volume weighted price trends
   - Market depth indicators

3. Cross-Pair Dynamics:
   - ETH/BTC correlation changes
   - Relative strength shifts
   - Market maker activity signals

Focus on actionable micro-trends and real-time market behavior. Identify minimal but significant price movements.
Keep the analysis technical but concise, emphasizing key shifts in market dynamics.
```

## Twitter API Integration

### API Overview
The bot does not use the Twitter API directly, but instead uses Selenium WebDriver for browser automation to interact with Twitter's web interface. This approach avoids the need for Twitter Developer API credentials but comes with its own challenges in terms of reliability and maintenance.

### Browser Automation
The bot uses Selenium to:
1. Authenticate with Twitter
2. Compose tweets
3. Post content
4. Verify post success

### Authentication Implementation
```python
def _login_to_twitter(self) -> bool:
    """Log into Twitter using environment credentials with enhanced verification"""
    try:
        logger.logger.info("Starting Twitter login sequence")
        self.browser.driver.set_page_load_timeout(45)
        self.browser.driver.get('https://twitter.com/login')
        logger.logger.info("Navigated to Twitter login page")
        time.sleep(5)  # Wait for initial page load

        # Enter username
        username_field = WebDriverWait(self.browser.driver, 20).until(
            EC.element_to_be_clickable((By.CSS_SELECTOR, "input[autocomplete='username']"))
        )
        username_field.click()
        time.sleep(1)
        username_field.send_keys(self.config.TWITTER_USERNAME)
        
        # Click next button
        next_button = WebDriverWait(self.browser.driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, "//span[text()='Next']"))
        )
        next_button.click()
        
        # Enter password
        password_field = WebDriverWait(self.browser.driver, 20).until(
            EC.element_to_be_clickable((By.CSS_SELECTOR, "input[type='password']"))
        )
        password_field.click()
        time.sleep(1)
        password_field.send_keys(self.config.TWITTER_PASSWORD)
        
        # Click login button
        login_button = WebDriverWait(self.browser.driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, "//span[text()='Log in']"))
        )
        login_button.click()
        
        # Wait for 2FA if needed
        time.sleep(10)

        return self._verify_login()

    except Exception as e:
        logger.log_error("Twitter Login", f"Login failed: {str(e)}", exc_info=True)
        return False
```

### Posting Implementation
```python
def _post_analysis(self, tweet_text: str) -> bool:
    """Post analysis to Twitter with enhanced button detection"""
    max_retries = 3
    retry_count = 0
    
    while retry_count < max_retries:
        try:
            # Navigate to compose tweet page
            self.browser.driver.get('https://twitter.com/compose/tweet')
            time.sleep(3)
            
            # Enter tweet text
            text_area = WebDriverWait(self.browser.driver, 10).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, '[data-testid="tweetTextarea_0"]'))
            )
            text_area.click()
            time.sleep(1)
            
            # Send text in chunks to prevent hashtag issues
            text_parts = tweet_text.split('#')
            text_area.send_keys(text_parts[0])
            time.sleep(1)
            for part in text_parts[1:]:
                text_area.send_keys(f'#{part}')
                time.sleep(0.5)
            
            # Find and click post button
            post_button = None
            button_locators = [
                (By.CSS_SELECTOR, '[data-testid="tweetButton"]'),
                (By.XPATH, "//div[@role='button'][contains(., 'Post')]"),
                (By.XPATH, "//span[text()='Post']")
            ]

            for locator in button_locators:
                try:
                    post_button = WebDriverWait(self.browser.driver, 5).until(
                        EC.element_to_be_clickable(locator)
                    )
                    if post_button:
                        break
                except:
                    continue

            if post_button:
                self.browser.driver.execute_script("arguments[0].scrollIntoView(true);", post_button)
                time.sleep(1)
                self.browser.driver.execute_script("arguments[0].click();", post_button)
                time.sleep(5)
                logger.logger.info("Tweet posted successfully")
                return True
            else:
                logger.logger.error("Could not find post button")
                retry_count += 1
                time.sleep(2)
                
        except Exception as e:
            logger.logger.error(f"Tweet posting error, attempt {retry_count + 1}: {str(e)}")
            retry_count += 1
            wait_time = retry_count * 10
            logger.logger.warning(f"Waiting {wait_time}s before retry...")
            time.sleep(wait_time)
            continue
    
    logger.log_error("Tweet Creation", "Maximum retries reached")
    return False
```

### Common Challenges
1. **Selectors Changing**: Twitter frequently updates its web interface, requiring selector updates
2. **Rate Limiting**: Excessive automation can trigger rate limits or captchas
3. **Authentication Challenges**: 2FA, security checks, and session management
4. **Browser Performance**: Memory usage and stability issues during long runs

### Mitigation Strategies
1. **Robust Selectors**: Using multiple selector strategies for redundancy
2. **Throttling**: Limiting post frequency to avoid detection
3. **Session Management**: Regular session clearing and browser restarts
4. **Visual Verification**: Taking screenshots for debugging

## Error Handling

### Common Error Patterns
1. **Retry Pattern**
```python
def retry_operation(operation, max_retries=3):
    """Retry an operation with exponential backoff"""
    retry_count = 0
    last_exception = None
    
    while retry_count < max_retries:
        try:
            return operation()
        except Exception as e:
            last_exception = e
            retry_count += 1
            wait_time = (2 ** retry_count) * 5  # Exponential backoff
            logger.warning(f"Operation failed, retrying in {wait_time}s... ({retry_count}/{max_retries})")
            time.sleep(wait_time)
    
    logger.error(f"Operation failed after {max_retries} retries: {str(last_exception)}")
    raise last_exception
```

2. **API Error Handling**
```python
def handle_api_error(response, endpoint_name):
    """Handle different API error responses"""
    status_code = response.status_code
    
    if status_code == 400:
        logger.error(f"{endpoint_name} API error: Bad request")
        return "bad_request"
    elif status_code == 401:
        logger.error(f"{endpoint_name} API error: Unauthorized")
        return "unauthorized"
    elif status_code == 403:
        logger.error(f"{endpoint_name} API error: Forbidden")
        return "forbidden"
    elif status_code == 404:
        logger.error(f"{endpoint_name} API error: Not found")
        return "not_found"
    elif status_code == 429:
        retry_after = int(response.headers.get('Retry-After', 60))
        logger.warning(f"{endpoint_name} API error: Rate limited, retry after {retry_after}s")
        return "rate_limited", retry_after
    elif 500 <= status_code < 600:
        logger.error(f"{endpoint_name} API error: Server error {status_code}")
        return "server_error"
    else:
        logger.error(f"{endpoint_name} API error: Unknown error {status_code}")
        return "unknown_error"
```

3. **Browser Error Handling**
```python
def handle_browser_error(error, operation_name):
    """Handle different Selenium browser errors"""
    if isinstance(error, TimeoutException):
        logger.warning(f"Browser timeout during {operation_name}")
        return "timeout"
    elif isinstance(error, NoSuchElementException):
        logger.error(f"Element not found during {operation_name}")
        return "element_not_found"
    elif isinstance(error, WebDriverException):
        if "chrome not reachable" in str(error).lower():
            logger.error(f"Browser crashed during {operation_name}")
            return "browser_crashed"
        else:
            logger.error(f"WebDriver error during {operation_name}: {str(error)}")
            return "webdriver_error"
    else:
        logger.error(f"Unknown error during {operation_name}: {str(error)}")
        return "unknown_error"
```

### Error Logging
```python
def log_error(self, component, message, exc_info=False):
    """Log errors with structured information"""
    error_data = {
        'component': component,
        'message': message,
        'timestamp': datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
        'context': {
            'operation': self.current_operation,
            'attempt': self.attempt_count
        }
    }
    
    self.logger.error(
        f"{component} Error: {message}",
        extra={'error_data': error_data},
        exc_info=exc_info
    )
    
    # Update error metrics
    self.error_count[component] = self.error_count.get(component, 0) + 1
```

## Rate Limiting

### CoinGecko Rate Limiting
- Free tier: 10-50 calls/minute
- Implementation of rate limiting:
```python
# Simple rate limiter
class RateLimiter:
    def __init__(self, max_calls, time_frame):
        """
        Initialize rate limiter
        max_calls: Maximum calls allowed in time_frame
        time_frame: Time frame in seconds
        """
        self.max_calls = max_calls
        self.time_frame = time_frame
        self.calls = []
    
    def can_make_request(self):
        """Check if request can be made within rate limits"""
        now = time.time()
        
        # Remove calls outside the time frame
        self.calls = [call_time for call_time in self.calls if now - call_time <= self.time_frame]
        
        # Check if under the limit
        return len(self.calls) < self.max_calls
    
    def add_call(self):
        """Record a call"""
        self.calls.append(time.time())

# Usage
coingecko_limiter = RateLimiter(max_calls=45, time_frame=60)

def get_crypto_data():
    if not coingecko_limiter.can_make_request():
        time.sleep(5)  # Wait before retrying
        return get_crypto_data()
    
    coingecko_limiter.add_call()
    # Make API request
```

### Claude API Rate Limiting
- Limits vary by API tier
- Implementation:
```python
def analyze_with_rate_limit(self, data):
    """Call Claude API with rate limiting"""
    now = time.time()
    
    # Check time since last call
    elapsed = now - self.last_claude_call
    if elapsed < self.min_claude_interval:
        # Wait remaining time
        wait_time = self.min_claude_interval - elapsed
        logger.info(f"Rate limiting Claude API, waiting {wait_time:.2f}s")
        time.sleep(wait_time)
    
    # Make API call
    result = self._analyze_market_sentiment(data)
    
    # Update last call time
    self.last_claude_call = time.time()
    
    return result
```

### Twitter Rate Limiting
- Automation limits not officially documented
- Conservative approach to avoid detection:
```python
def post_with_rate_limit(self, tweet_text):
    """Post to Twitter with rate limiting"""
    now = time.time()
    
    # Check time since last post
    if self.last_post_time:
        elapsed = now - self.last_post_time
        min_interval = self.config.TWEET_MIN_INTERVAL
        
        if elapsed < min_interval:
            # Wait remaining time
            wait_time = min_interval - elapsed
            logger.info(f"Rate limiting Twitter posting, waiting {wait_time:.2f}s")
            time.sleep(wait_time)
    
    # Post tweet
    success = self._post_analysis(tweet_text)
    
    # Update last post time
    if success:
        self.last_post_time = time.time()
    
    return success
```

## Authentication Management

### API Key Security
```python
# Environment variable approach (preferred)
from os import environ
api_key = environ.get('CLAUDE_API_KEY')

# Secure storage and retrieval
def get_api_key(service_name):
    """Get API key from secure storage"""
    try:
        # Example using keyring for secure storage
        import keyring
        key = keyring.get_password("eth_btc_bot", service_name)
        if not key:
            logger.error(f"API key for {service_name} not found")
            raise ValueError(f"API key for {service_name} not found")
        return key
    except Exception as e:
        logger.error(f"Error retrieving API key: {str(e)}")
        raise
```

### Key Rotation
```python
def rotate_api_keys():
    """Rotate API keys on schedule"""
    # Example implementation
    try:
        # Get new key from secure storage or service
        new_key = get_new_api_key()
        
        # Update environment
        os.environ['CLAUDE_API_KEY'] = new_key
        
        # Update client
        bot.claude_client = anthropic.Client(api_key=new_key)
        
        logger.info("API key rotated successfully")
        return True
    except Exception as e:
        logger.error(f"API key rotation failed: {str(e)}")
        return False
```

### Twitter Credentials Security
```python
# Secure storage for Twitter credentials
def get_twitter_credentials():
    """Get Twitter credentials from secure storage"""
    credentials = {
        'username': os.environ.get('TWITTER_USERNAME'),
        'password': os.environ.get('TWITTER_PASSWORD')
    }
    
    # Validate credentials
    if not credentials['username'] or not credentials['password']:
        raise ValueError("Twitter credentials missing")
    
    # Return masked credentials for logging
    masked = {
        'username': credentials['username'],
        'password': '*' * len(credentials['password'])
    }
    logger.debug(f"Retrieved Twitter credentials: {masked}")
    
    return credentials
```

## Best Practices

### API Request Optimization
1. **Batch Requests When Possible**
```python
# Instead of multiple single-coin requests
coins = ["bitcoin", "ethereum"]
response = requests.get(f"{base_url}/coins/markets", params={
    "vs_currency": "usd",
    "ids": ",".join(coins),
    "per_page": len(coins)
})
```

2. **Request Only Needed Data**
```python
# Request only necessary fields
params = {
    "vs_currency": "usd",
    "ids": "bitcoin,ethereum",
    "price_change_percentage": "24h",  # Only get 24h change
    "sparkline": False,  # Don't need sparkline data
    "locale": "en"
}
```

3. **Implement Caching**
```python
# Simple cache implementation
class SimpleCache:
    def __init__(self, ttl=300):
        self.cache = {}
        self.ttl = ttl  # Time to live in seconds
    
    def get(self, key):
        if key in self.cache:
            data, timestamp = self.cache[key]
            if time.time() - timestamp <= self.ttl:
                return data
            # Expired
            del self.cache[key]
        return None
    
    def set(self, key, data):
        self.cache[key] = (data, time.time())
```

### Error Resilience
1. **Circuit Breaker Pattern**
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_time=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.recovery_time = recovery_time
        self.open_since = None
        self.state = "CLOSED"  # CLOSED, OPEN, HALF-OPEN
    
    def can_execute(self):
        """Check if circuit is closed and operation can execute"""
        now = time.time()
        
        if self.state == "OPEN":
            if now - self.open_since >= self.recovery_time:
                # Try half-open state
                self.state = "HALF-OPEN"
                return True
            return False
        
        return True
    
    def record_success(self):
        """Record successful execution"""
        if self.state == "HALF-OPEN":
            # Reset on successful half-open execution
            self.failure_count = 0
            self.state = "CLOSED"
            self.open_since = None
    
    def record_failure(self):
        """Record failed execution"""
        if self.state == "HALF-OPEN":
            # Failed during half-open, back to open
            self.state = "OPEN"
            self.open_since = time.time()
        elif self.state == "CLOSED":
            self.failure_count += 1
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
                self.open_since = time.time()
```

### Response Validation
```python
def validate_crypto_response(response_data):
    """Validate CoinGecko response data"""
    if not isinstance(response_data, list):
        logger.error("Invalid response format: not a list")
        return False
    
    if len(response_data) < 2:
        logger.error(f"Expected 2 coins, got {len(response_data)}")
        return False
    
    required_fields = [
        'symbol', 'current_price', 'price_change_percentage_24h', 'total_volume'
    ]
    
    for coin in response_data:
        for field in required_fields:
            if field not in coin:
                logger.error(f"Missing required field: {field}")
                return False
        
        # Validate data types
        if not isinstance(coin['current_price'], (int, float)):
            logger.error(f"Invalid price type: {type(coin['current_price'])}")
            return False
    
    # Check for expected coins
    symbols = [coin['symbol'].upper() for coin in response_data]
    if 'BTC' not in symbols or 'ETH' not in symbols:
        logger.error(f"Missing required coins. Got: {symbols}")
        return False
    
    return True
```

## Testing & Development

### API Mocking
```python
# Mock CoinGecko response
def mock_coingecko_response():
    """Return a mock CoinGecko response for testing"""
    return [
        {
            "id": "bitcoin",
            "symbol": "btc",
            "name": "Bitcoin",
            "current_price": 63421.75,
            "market_cap": 1245859608739,
            "market_cap_rank": 1,
            "total_volume": 43625862493,
            "price_change_percentage_24h": 2.34,
        },
        {
            "id": "ethereum",
            "symbol": "eth",
            "name": "Ethereum",
            "current_price": 3451.89,
            "market_cap": 405732945729,
            "market_cap_rank": 2,
            "total_volume": 24152863428,
            "price_change_percentage_24h": 1.23,
        }
    ]

# Using
