# Market Correlation Agent 📊

An advanced cryptocurrency market analysis agent that monitors cryptocurrency price movements, provides real-time correlation analysis, and automatically posts insights to Twitter using Claude and CoinGecko integrations.

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![Selenium](https://img.shields.io/badge/Selenium-4.16.0-green)
![Anthropic](https://img.shields.io/badge/Claude%20AI-3.5%20Sonnet-purple)
![License](https://img.shields.io/badge/License-MIT-yellow)

## 🌟 Features

- **Real-time Market Monitoring**: Continuously tracks cryptocurrency price movements using CoinGecko API
- **AI-Powered Analysis**: Leverages Claude 3.5 Sonnet for sophisticated market analysis
- **Automated Twitter Updates**: Posts market insights automatically with duplicate detection
- **Smart Duplicate Prevention**: Advanced algorithms to prevent redundant analyses
- **Robust Error Handling**: Comprehensive retry mechanisms and error logging
- **Configurable Parameters**: Easily adjustable market analysis sensitivity and posting frequency

## 🚀 Getting Started

### Prerequisites

```bash
python3.8+
chromedriver
```

### Installation

1. Clone the repository:
```bash
git clone https://github.com/KingRaver/zoinks.git
cd zoinks
```

2. Install required packages:
```bash
pip install -r requirements.txt
```

3. Set up environment variables in `.env`:
```env
TWITTER_USERNAME=your_twitter_username
TWITTER_PASSWORD=your_twitter_password
CLAUDE_API_KEY=your_claude_api_key
CHROME_DRIVER_PATH=/path/to/chromedriver
```

## 🔧 Configuration

The agent is highly configurable through the `Config` class. Key parameters include:

- Market Analysis Parameters:
  ```python
  MARKET_ANALYSIS_CONFIG = {
      'correlation_sensitivity': 0.7,
      'volatility_threshold': 2.0,
      'volume_significance': 100000,
      'historical_periods': [1, 4, 24]
  }
  ```

- Tweet Constraints:
  ```python
  TWEET_CONSTRAINTS = {
      'MIN_LENGTH': 220,
      'MAX_LENGTH': 270,
      'HARD_STOP_LENGTH': 280
  }
  ```

## 🏃‍♂️ Running the Agent

Start the agent with:
```bash
python3 src/bot.py
```

The agent will:
1. Initialize and authenticate with Twitter
2. Begin monitoring cryptocurrency market data
3. Generate AI-powered market analysis
4. Post insights to Twitter at configured intervals

## 📊 Analysis Methodology

The agent employs a sophisticated analysis approach:

1. **Market Data Collection**
   - Real-time price data from CoinGecko
   - Volume analysis
   - Historical trend comparison

2. **AI Analysis**
   - Short-term movement analysis
   - Market microstructure evaluation
   - Cross-pair dynamics assessment
   - Volume-weighted price trends

3. **Duplicate Prevention**
   - Price change threshold checking
   - Time-based filtering
   - Content similarity analysis

## 🛠 Error Handling

The agent includes robust error handling:
- Automatic retries for API failures
- Graceful degradation
- Comprehensive logging
- Browser session management
- Network timeout handling

## 📜 Logging

Detailed logging is implemented throughout the application:
- Market data fetching
- Analysis generation
- Twitter posting
- Error tracking
- Performance metrics

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## ⚠️ Disclaimer

This agent is for educational purposes only. Cryptocurrency trading carries significant risks. Always do your own research and never trade based solely on automated analysis.
