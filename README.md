# Stock AI Agent

A powerful stock analysis platform that combines real-time market data with AI-powered insights to help you make informed investment decisions.

## 🎯 What Does This Application Do?

The Stock AI Agent is an intelligent stock analysis tool that automates the process of technical analysis and provides AI-powered investment insights. Here's what happens when you use it:

1. **Data Retrieval**: The application connects to Alpha Vantage's API and fetches the last 100 days of daily stock data for your chosen stock symbol and market.

2. **Data Processing**: Raw JSON data is converted into a structured Pandas DataFrame, with proper date parsing and timezone conversion from US/Eastern to Indian Standard Time (IST).

3. **Technical Analysis**: The system calculates key technical indicators:
   - 7-day Simple Moving Average (short-term trend)
   - 20-day Simple Moving Average (medium-term trend)
   - Daily trading volume analysis

4. **Visualization**: Three comprehensive charts are generated showing:
   - Price movement over time (identifying trends)
   - Trading volume patterns (showing market interest)
   - Moving average overlays (detecting potential buy/sell signals)

5. **AI Analysis**: The generated chart is sent to Google's Gemini AI, which performs visual analysis to:
   - Identify price patterns and trends (uptrend, downtrend, sideways)
   - Analyze volume-price relationships
   - Detect moving average crossovers (bullish/bearish signals)
   - Assess overall market sentiment
   - Provide a buy/hold/sell recommendation with reasoning

6. **Results Presentation**: All insights are displayed in an easy-to-understand format with interactive navigation.

**In essence**: You input a stock symbol, and the application automatically fetches data, creates professional charts, analyzes them using AI, and provides actionable investment insights—all in seconds!

## 🌟 Features

- **Real-time Stock Data**: Fetches daily stock data from Alpha Vantage API
- **Multi-Market Support**: Analyze stocks from BSE (Bombay Stock Exchange) and NASDAQ
- **Visual Analytics**: Interactive charts displaying:
  - Closing prices over time
  - Trading volume
  - 7-day and 20-day moving averages
- **AI-Powered Insights**: Leverages Google's Gemini AI to provide intelligent analysis and investment recommendations
- **User-Friendly Interface**: Built with Streamlit for an intuitive web-based experience

## 📋 Prerequisites

- Python 3.8 or higher
- Alpha Vantage API key ([Get free key](https://www.alphavantage.co/support/#api-key))
- Google Gemini API key ([Get key](https://makersuite.google.com/app/apikey))

## 🚀 Installation

1. **Clone or download this repository**

2. **Install required dependencies**:
```bash
pip install pandas matplotlib requests pytz streamlit google-generativeai pillow
```

Or create a `requirements.txt` file with:
```
pandas
matplotlib
requests
pytz
streamlit
google-generativeai
pillow
```

Then install:
```bash
pip install -r requirements.txt
```

## ⚙️ Configuration

### API Keys Setup

You need to configure two API keys before running the application:

1. **Alpha Vantage API Key**:
   - Sign up at [Alpha Vantage](https://www.alphavantage.co/support/#api-key)
   - Open `marketapp.py`
   - Replace `"HM26QL5T0XUYPG4T"` on line 51 with your API key:
   ```python
   stock_api_obj = StockAPI("YOUR_ALPHA_VANTAGE_API_KEY")
   ```

2. **Google Gemini API Key**:
   - Get your API key from [Google AI Studio](https://makersuite.google.com/app/apikey)
   - Open `marketapp.py`
   - Replace `"<Insert API Key>"` on line 59 with your Gemini API key:
   ```python
   ai_insights_obj = AIInsights("YOUR_GEMINI_API_KEY")
   ```

## 📖 Usage

1. **Start the application**:
```bash
streamlit run marketapp.py
```

2. **Access the web interface**:
   - The application will automatically open in your default web browser
   - If not, navigate to `http://localhost:8501`

3. **Analyze a stock**:
   - Enter the **Stock Ticker Symbol** (e.g., RELIANCE for BSE, AAPL for NASDAQ)
   - Select the **Market** (BSE or NASDAQ)
   - Click **Submit**

4. **View results**:
   - Stock performance charts with three panels:
     - Closing price trends
     - Trading volume
     - Moving averages comparison
   - AI-generated insights and investment recommendations
   - Click **Back** to analyze another stock

## 📁 Project Structure

```
stock-ai-agent/
│
├── marketapp.py                 # Main Streamlit application
├── stock_utility_handler.py     # Stock data fetching and analysis
├── ai_insights_handler.py       # AI insights generation
├── README.md                    # This file
└── requirements.txt             # Python dependencies (optional)
```

## 🔍 Detailed Code Walkthrough

### `marketapp.py` (Main Application)
This is the heart of the application that orchestrates everything:

**What it does:**
- Creates a two-page Streamlit interface (input page and results page)
- Manages session state to preserve data between page transitions
- Coordinates the workflow between data fetching, analysis, and AI insights

**Key Functions:**
- `page1()`: 
  - Displays input form for stock ticker and market selection
  - Default values: RELIANCE stock on BSE market
  - Validates input and triggers analysis on Submit
  
- `page2()`:
  - Displays "Analyzing..." spinner during data processing
  - Executes the full analysis pipeline:
    1. Calls StockAPI to fetch market data
    2. Calls StockAnalyzer to convert and visualize data
    3. Calls AIInsights to generate recommendations
  - Shows final results with chart image and AI insights
  - Provides Back button to return to input page

**Session State Variables:**
- `page`: Tracks current page (page1 or page2)
- `ticker`: Stores stock symbol entered by user
- `market`: Stores selected market (BSE or NASDAQ)
- `image_path`: Path to generated chart image
- `ai_insights`: AI-generated text analysis
- `internal_results_available`: Flag to prevent re-processing data

**Workflow:**
```
User Input → Submit → Fetch Data → Process Data → Generate Chart → 
AI Analysis → Display Results → Back (repeat)
```

### `stock_utility_handler.py` (Data & Visualization)
This module handles all data operations and chart generation:

#### **StockAPI Class**
**Purpose**: Communicates with Alpha Vantage API to fetch stock data

**Key Method - `get_stock_info(stock, market)`:**
- Constructs appropriate API URL based on market:
  - NASDAQ: Direct symbol (e.g., AAPL)
  - BSE/others: Symbol with market suffix (e.g., RELIANCE.BSE)
- Sends HTTP GET request to Alpha Vantage
- Returns JSON response containing:
  - Time Series (Daily): Dictionary of dates → price/volume data
  - Meta Data: Stock symbol, last refresh time, etc.

**API Response Structure:**
```json
{
  "Time Series (Daily)": {
    "2024-02-14": {
      "1. open": "2850.00",
      "2. high": "2875.50",
      "3. low": "2845.00",
      "4. close": "2870.25",
      "5. volume": "5234567"
    }
  }
}
```

#### **StockAnalyzer Class**
**Purpose**: Transforms raw data and creates professional visualizations

**Key Method - `json_to_dataframe(json_data, stock_symbol, market)`:**
- Parses JSON response from API
- Extracts daily data points (open, high, low, close, volume)
- Creates Pandas DataFrame for efficient data manipulation
- Performs timezone conversion:
  - Input: US/Eastern (market hours timezone)
  - Output: Asia/Kolkata IST (for Indian users)
- Adds metadata columns (stock symbol, market)
- Sets date as index for time-series operations

**Data Transformation Example:**
```
Raw: "2024-02-14" → "1. open": "2850.00"
Processed: date="2024-02-14 18:30:00 IST", open=2850.00
```

**Key Method - `plot_stock_data(df, stock_symbol, market, image_path)`:**
Creates a professional three-panel chart visualization:

**Panel 1 - Price Chart:**
- Line plot of closing prices over time
- Shows overall price trend (up/down/sideways)
- Blue color for easy identification
- Grid for precise reading

**Panel 2 - Volume Chart:**
- Bar chart of daily trading volume
- Green bars show market participation
- Higher volume = more trading activity/interest
- Helps confirm price movements

**Panel 3 - Moving Averages:**
- Overlay of price with two moving averages:
  - **7-day MA (Orange)**: Short-term trend indicator
    - Smooths out daily fluctuations
    - Responds quickly to price changes
  - **20-day MA (Red)**: Medium-term trend indicator
    - Shows longer-term direction
    - Slower to respond, filters noise
- **Crossover signals:**
  - 7-day crosses above 20-day = Bullish signal (potential buy)
  - 7-day crosses below 20-day = Bearish signal (potential sell)

**Chart Formatting:**
- Figure size: 16x10 inches (high resolution)
- X-axis: Formatted dates with monthly major ticks, weekly minor ticks
- Automatic label rotation for readability
- Interactive cursor for hover details
- Saved as PNG file (e.g., "BSE_RELIANCE.png")

### `ai_insights_handler.py` (AI Analysis Engine)
This module integrates Google's Gemini AI for intelligent analysis:

#### **AIInsights Class**
**Purpose**: Analyzes stock charts using computer vision and provides recommendations

**Initialization:**
- Configures Google's Generative AI with API key
- Loads "gemini-flash-latest" model (fast, efficient)
- This model can process both text and images

**Key Method - `get_ai_insights(image_path, stock, market)`:**

**Input Processing:**
- Opens the generated chart image using PIL (Python Imaging Library)
- Prepares a detailed prompt for the AI with context:
  - Stock symbol and market information
  - Time period (last 100 days)
  - What to analyze (volume, prices, moving averages)
  - Expected output (analysis + buy/sell recommendation)

**AI Prompt Structure:**
```
"This is an image of stock performance of stock : '{stock}' 
for the last 100 days on market : '{market}', 
on the basis of volume traded, closing prices and 
7,20 day moving averages provide some analysis and 
suggestion about this stock. 
This Stock should be purchased or not."
```

**What the AI Analyzes:**
1. **Visual Pattern Recognition:**
   - Identifies chart patterns (head & shoulders, double tops/bottoms, etc.)
   - Recognizes trend lines and support/resistance levels
   - Detects breakouts or breakdowns

2. **Technical Indicator Analysis:**
   - Moving average positioning (bullish/bearish)
   - MA crossover points (golden cross/death cross)
   - Price relationship to MAs (above = strength, below = weakness)

3. **Volume Analysis:**
   - Volume trends (increasing/decreasing)
   - Volume-price divergence (warning signals)
   - Volume confirmation of price moves

4. **Trend Assessment:**
   - Overall direction (uptrend, downtrend, sideways)
   - Trend strength (strong, weak, uncertain)
   - Potential reversal points

5. **Risk-Reward Evaluation:**
   - Current price position (near high/low)
   - Volatility assessment
   - Entry/exit point suggestions

**Output:**
- Natural language analysis explaining:
  - What the charts show
  - Technical indicators interpretation
  - Market sentiment assessment
  - Specific recommendation (Buy/Hold/Sell)
  - Reasoning behind the recommendation
  - Risk factors to consider

**Response Processing:**
- Extracts text from AI response candidates
- Combines multiple text parts if present
- Returns formatted analysis ready for display

## 🔄 Complete Application Flow

Here's how everything works together when you analyze a stock:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. USER INPUT (page1)                                       │
│    - Enter: RELIANCE                                         │
│    - Select: BSE                                             │
│    - Click: Submit                                           │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. DATA FETCHING (StockAPI)                                 │
│    - Build URL: alphavantage.co/.../RELIANCE.BSE            │
│    - HTTP GET request                                        │
│    - Receive JSON with 100 days of data                     │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. DATA PROCESSING (StockAnalyzer)                          │
│    - Parse JSON response                                     │
│    - Create DataFrame: [date, open, high, low, close, vol]  │
│    - Convert timezone: US/Eastern → IST                     │
│    - Calculate MA_7 and MA_20                                │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. VISUALIZATION (StockAnalyzer)                            │
│    - Create figure with 3 subplots                           │
│    - Plot 1: Price line chart                                │
│    - Plot 2: Volume bar chart                                │
│    - Plot 3: Price + MA_7 + MA_20                           │
│    - Format axes, add legends, grid                          │
│    - Save as: BSE_RELIANCE.png                               │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. AI ANALYSIS (AIInsights)                                 │
│    - Load chart image                                        │
│    - Send to Gemini AI with prompt                           │
│    - AI performs visual analysis:                            │
│      • Pattern recognition                                   │
│      • Trend identification                                  │
│      • Moving average analysis                               │
│      • Volume correlation                                    │
│    - Generate recommendation with reasoning                  │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. RESULTS DISPLAY (page2)                                  │
│    - Show chart image                                        │
│    - Display AI insights text                                │
│    - Provide Back button                                     │
└─────────────────────────────────────────────────────────────┘
```

**Time Estimate:** The entire process takes 5-15 seconds depending on:
- API response time (1-3 seconds)
- Data processing (1-2 seconds)
- Chart generation (1-2 seconds)
- AI analysis (2-8 seconds)

## 🔍 File Descriptions
## 🛠️ Technical Details

### Data Processing
- **Data Source**: Alpha Vantage TIME_SERIES_DAILY API
- **Time Period**: Last 100 days of trading data
- **Timezone**: Converts from US/Eastern to Asia/Kolkata (IST)
- **Technical Indicators**:
  - 7-day Simple Moving Average (SMA)
  - 20-day Simple Moving Average (SMA)

### Visualization
Three-panel chart layout:
1. **Price Chart**: Closing prices over time
2. **Volume Chart**: Daily trading volume as bar chart
3. **Moving Averages**: Comparison of price with 7-day and 20-day MAs

Chart features:
- Date formatting with monthly major ticks and weekly minor ticks
- Grid overlay for easier reading
- Legend for data identification
- Automatic date label rotation
- Saved as PNG for AI analysis

### AI Analysis
The Gemini AI model analyzes:
- Historical price movements and trends
- Trading volume patterns
- Moving average crossovers (bullish/bearish signals)
- Overall market sentiment
- Risk assessment
- Buy/Hold/Sell recommendation

## 📊 Understanding Moving Averages

Moving Averages are crucial technical indicators used in this application. Here's what they mean:

### What is a Moving Average?
A moving average smooths out price data by calculating the average price over a specific time period. As new data comes in, the oldest data point is dropped, so the average "moves" forward in time.

### The Two Moving Averages Used:

**7-Day Moving Average (Short-term)**
- Calculates average closing price of last 7 trading days
- Responds quickly to recent price changes
- Helps identify short-term trends and momentum
- More sensitive to daily fluctuations

**20-Day Moving Average (Medium-term)**
- Calculates average closing price of last 20 trading days
- Responds more slowly to price changes
- Helps identify medium-term trends
- Filters out short-term noise

### Trading Signals from Moving Averages:

**Golden Cross (Bullish Signal):**
- Occurs when 7-day MA crosses **above** 20-day MA
- Suggests upward momentum is building
- Often considered a buy signal
- Indicates short-term trend is stronger than medium-term

**Death Cross (Bearish Signal):**
- Occurs when 7-day MA crosses **below** 20-day MA
- Suggests downward momentum is building
- Often considered a sell signal
- Indicates short-term weakness

**Price Position:**
- Price **above** both MAs = Strong uptrend
- Price **between** MAs = Transitional phase
- Price **below** both MAs = Strong downtrend

**Example Interpretation:**
```
If RELIANCE is trading at ₹2,870:
- 7-day MA = ₹2,850 (short-term average)
- 20-day MA = ₹2,800 (medium-term average)

Analysis:
✓ Price above both MAs = Bullish
✓ 7-day MA > 20-day MA = Uptrend confirmed
→ Potential buy/hold signal
```

### Why Volume Matters:

**Volume Confirmation:**
- Price increase + High volume = Strong bullish signal
- Price increase + Low volume = Weak signal (may reverse)
- Price decrease + High volume = Strong bearish signal
- Price decrease + Low volume = Weak signal (may bounce)

**Volume Divergence (Warning Sign):**
- Price making new highs, but volume decreasing = Potential reversal
- Price making new lows, but volume decreasing = Potential bottom

The AI analyzes all these factors together to provide comprehensive insights!

## ⚠️ Limitations

- **API Rate Limits**:
  - Alpha Vantage free tier: 5 calls/minute, 500 calls/day
  - Gemini API: Subject to Google's usage limits
- **Data Coverage**: Last 100 days of trading data only
- **Markets**: Currently supports BSE and NASDAQ only
- **Real-time Data**: Not true real-time; data is end-of-day

## 🐛 Troubleshooting

### Common Issues

**1. API Rate Limit Exceeded**
```
Error: API rate limit reached
```
**Solution**: 
- Wait 60 seconds before making another request
- Consider upgrading to Alpha Vantage premium tier

**2. Invalid Stock Symbol**
```
KeyError: 'Time Series (Daily)'
```
**Solution**:
- Verify the ticker symbol is correct for the selected market
- BSE stocks: RELIANCE, TCS, INFY, etc.
- NASDAQ stocks: AAPL, GOOGL, MSFT, etc.

**3. Missing API Key**
```
Error: Invalid API key
```
**Solution**:
- Ensure you've replaced placeholder API keys in `marketapp.py`
- Verify your API keys are active and valid

**4. Module Not Found Error**
```
ModuleNotFoundError: No module named 'streamlit'
```
**Solution**:
- Install missing dependencies: `pip install streamlit`
- Or install all dependencies: `pip install -r requirements.txt`

**5. Chart Not Displaying**
```
FileNotFoundError or image display issues
```
**Solution**:
- Ensure the application has write permissions in its directory
- Check that matplotlib is properly installed

**6. Gemini API Configuration Error**
```
Error: API key not configured
```
**Solution**:
- Verify your Gemini API key is correct
- Ensure you have internet connectivity
- Check that google-generativeai library is installed

## 🔐 Security Best Practices

⚠️ **Important**: Never commit API keys to public repositories!

**Recommended approach**:

1. Create a `.env` file:
```
ALPHA_VANTAGE_KEY=your_alpha_vantage_key
GEMINI_API_KEY=your_gemini_key
```

2. Add `.env` to `.gitignore`

3. Modify code to use environment variables:
```python
import os
from dotenv import load_dotenv

load_dotenv()

stock_api_obj = StockAPI(os.getenv("ALPHA_VANTAGE_KEY"))
ai_insights_obj = AIInsights(os.getenv("GEMINI_API_KEY"))
```

## 📈 Example Stock Symbols

### BSE (Bombay Stock Exchange)
- RELIANCE - Reliance Industries
- TCS - Tata Consultancy Services
- INFY - Infosys
- HDFCBANK - HDFC Bank
- ICICIBANK - ICICI Bank

### NASDAQ
- AAPL - Apple Inc.
- GOOGL - Alphabet Inc.
- MSFT - Microsoft Corporation
- AMZN - Amazon.com Inc.
- TSLA - Tesla Inc.

## 💡 Example Analysis Output

Here's what a typical AI analysis might look like:

### Sample Input:
- **Stock**: RELIANCE
- **Market**: BSE
- **Time Period**: Last 100 days

### Sample AI Output:

```
RELIANCE Stock Analysis (BSE Market)

TREND ANALYSIS:
The stock shows a clear upward trend over the last 100 days, with the price 
rising from approximately ₹2,650 to ₹2,870 (8.3% gain). The overall trajectory 
indicates strong bullish momentum.

MOVING AVERAGES:
• The 7-day MA (orange line) is currently at ₹2,850, positioned above the 
  20-day MA (red line) at ₹2,800.
• This "Golden Cross" formation occurred around day 75 and has been maintained, 
  confirming the uptrend.
• The price is trading above both moving averages, indicating sustained 
  bullish sentiment.

VOLUME ANALYSIS:
• Trading volume has been consistently above average during price increases, 
  particularly noticeable in the last 30 days.
• Recent high-volume days coincide with price breakouts, confirming genuine 
  buying interest rather than manipulation.
• Volume spike on day 85 (5.2M shares) corresponds with a 3% price jump, 
  showing strong institutional participation.

SUPPORT AND RESISTANCE:
• Strong support established at ₹2,800 (20-day MA level)
• Next resistance level appears to be at ₹2,900
• Price has bounced off the 20-day MA twice, showing it as reliable support

RECOMMENDATION: BUY / ACCUMULATE

RATIONALE:
1. Strong uptrend confirmed by both moving averages
2. Price momentum supported by healthy volume
3. Recent consolidation between ₹2,850-₹2,870 suggests preparation for 
   next leg up
4. Risk-reward ratio favorable with support at ₹2,800

RISK FACTORS:
• Watch for 7-day MA crossing below 20-day MA (bearish signal)
• If price breaks below ₹2,800 with high volume, exit position
• General market conditions and sector performance should be monitored

SUGGESTED ENTRY: Current levels (₹2,865-₹2,875)
STOP LOSS: ₹2,790 (below 20-day MA)
TARGET: ₹2,950-₹3,000 (3-5% upside)
```

This is just an example - actual AI analysis will vary based on real-time data and market conditions!

## 🚀 Future Enhancements

Potential features for future versions:

- [ ] Support for additional exchanges (NSE, NYSE, LSE, etc.)
- [ ] More technical indicators (RSI, MACD, Bollinger Bands, Fibonacci)
- [ ] Portfolio tracking and management
- [ ] Historical comparison tools
- [ ] Price alerts and notifications
- [ ] Export reports to PDF/Excel
- [ ] Backtesting capabilities
- [ ] Multiple timeframe analysis (intraday, weekly, monthly)
- [ ] Fundamental analysis integration
- [ ] News sentiment analysis
- [ ] Comparison of multiple stocks
- [ ] User authentication and saved preferences

## 📄 License

This project is provided as-is for educational and personal use.

## ⚖️ Disclaimer

**IMPORTANT**: This tool is for informational and educational purposes only and should **NOT** be considered as financial advice. 

- Always conduct your own research
- Consult with qualified financial advisors before making investment decisions
- Past performance does not guarantee future results
- Stock market investments carry risk of loss
- The AI-generated insights are based on historical data and technical analysis only

The developers of this application are not responsible for any financial losses incurred from using this tool.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

To contribute:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📧 Support

For questions, issues, or suggestions:
- Open an issue in the repository
- Check existing issues for solutions
- Review the Troubleshooting section

## 🙏 Acknowledgments

- **Alpha Vantage** for providing stock market data API
- **Google Gemini** for AI-powered analysis capabilities
- **Streamlit** for the excellent web framework
- **Matplotlib** for data visualization tools

---

**Made with ❤️ for stock market enthusiasts and learners**

*Remember: Invest wisely, diversify your portfolio, and never invest more than you can afford to lose.*
