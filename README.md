# -
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>تحلیلگر حرفه‌ای رمزارزها با داده واقعی</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: 'Segoe UI', Tahoma, sans-serif; background: #f5f7fa; padding: 20px; }
    .container { max-width: 1000px; margin: auto; background: white; padding: 25px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.05); }
    h2 { color: #2c3e50; text-align: center; margin-bottom: 25px; }
    .search-box { display: flex; gap: 10px; margin-bottom: 20px; }
    input { flex: 1; padding: 12px 15px; border-radius: 8px; border: 1px solid #ddd; font-size: 16px; }
    button { background-color: #3498db; color: white; border: none; padding: 12px 20px; border-radius: 8px; cursor: pointer; transition: all 0.3s; }
    button:hover { background-color: #2980b9; }
    .result { margin-top: 25px; }
    .price-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 20px; }
    .price-card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); border: 1px solid #eee; }
    .price-header { display: flex; align-items: center; margin-bottom: 15px; }
    .price-icon { font-size: 24px; margin-left: 10px; color: #3498db; }
    .price-value { font-size: 24px; font-weight: bold; margin: 10px 0; }
    .price-change { display: inline-block; padding: 3px 8px; border-radius: 4px; font-size: 14px; }
    .positive { background: #e8f5e9; color: #2e7d32; }
    .negative { background: #ffebee; color: #c62828; }
    .chart-container { height: 400px; margin: 30px 0; }
    .loading { text-align: center; padding: 30px; }
    .loading-spinner { border: 5px solid #f3f3f3; border-top: 5px solid #3498db; border-radius: 50%; width: 50px; height: 50px; animation: spin 1s linear infinite; margin: 20px auto; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    .time-update { text-align: left; color: #7f8c8d; margin-top: 15px; font-size: 14px; }
    .analysis-section { background: #f8f9fa; padding: 20px; border-radius: 10px; margin-top: 30px; }
  </style>
</head>
<body>
  <div class="container">
    <h2><i class="fas fa-chart-line"></i> تحلیلگر حرفه‌ای رمزارزها</h2>
    
    <div class="search-box">
      <input type="text" id="cryptoName" placeholder="نام یا نماد رمزارز (مثال: bitcoin یا BTC)" value="bitcoin">
      <button onclick="analyzeCrypto()"><i class="fas fa-search"></i> دریافت اطلاعات</button>
    </div>
    
    <div class="result" id="result">
      <div class="loading" id="loading">
        <div class="loading-spinner"></div>
        <p>در حال دریافت اطلاعات از صرافی‌های معتبر...</p>
      </div>
      <div id="priceData" class="price-grid"></div>
      <div class="chart-container">
        <canvas id="priceChart"></canvas>
      </div>
      <div class="analysis-section" id="analysisResult"></div>
      <div class="time-update" id="timeUpdate"></div>
    </div>
  </div>

  <script>
    // APIهای معتبر برای دریافت داده‌های واقعی
    const API_ENDPOINTS = {
      COINGECKO: 'https://api.coingecko.com/api/v3',
      BINANCE: 'https://api.binance.com/api/v3',
      COINBASE: 'https://api.coinbase.com/v2',
      CRYPTOCOMPARE: 'https://min-api.cryptocompare.com/data'
    };

    // نرخ تبدیل ارزها (می‌تواند از API دریافت شود)
    const exchangeRates = {
      usd: 1,
      irr: 580000, // نرخ به ریال (برای نمایش به تومان تقسیم بر 10 می‌شود)
      eur: 0.93
    };

    // تحلیل رمزارز
    async function analyzeCrypto() {
      const input = document.getElementById('cryptoName').value.trim();
      const loadingDiv = document.getElementById('loading');
      const priceDataDiv = document.getElementById('priceData');
      const analysisDiv = document.getElementById('analysisResult');
      const timeUpdateDiv = document.getElementById('timeUpdate');
      
      if (!input) {
        alert('لطفاً نام یا نماد رمزارز را وارد کنید');
        return;
      }
      
      // نمایش وضعیت بارگذاری
      loadingDiv.style.display = 'block';
      priceDataDiv.innerHTML = '';
      analysisDiv.innerHTML = '';
      
      try {
        // دریافت اطلاعات از APIهای مختلف
        const [coinData, marketData, binanceData, coinbaseData, historicalData] = await Promise.all([
          fetchCoinData(input),
          fetchMarketData(input),
          fetchBinanceData(input),
          fetchCoinbaseData(input),
          fetchHistoricalData(input)
        ]);
        
        // نمایش اطلاعات قیمت
        displayPriceData(coinData, marketData, binanceData, coinbaseData);
        
        // نمایش نمودار قیمت
        renderPriceChart(historicalData);
        
        // تحلیل فنی
        displayTechnicalAnalysis(marketData, historicalData);
        
        // به‌روزرسانی زمان آخرین به‌روزرسانی
        timeUpdateDiv.innerHTML = `<i class="fas fa-sync-alt"></i> آخرین به‌روزرسانی: ${new Date().toLocaleString('fa-IR')}`;
      } catch (error) {
        priceDataDiv.innerHTML = `<div class="error" style="color: red; grid-column: 1 / -1; text-align: center;">
          <i class="fas fa-exclamation-triangle"></i> خطا در دریافت اطلاعات: ${error.message}
        </div>`;
      } finally {
        loadingDiv.style.display = 'none';
      }
    }

    // دریافت اطلاعات پایه رمزارز از CoinGecko
    async function fetchCoinData(cryptoId) {
      const response = await fetch(`${API_ENDPOINTS.COINGECKO}/coins/${cryptoId.toLowerCase()}?localization=false&tickers=true`);
      if (!response.ok) throw new Error('رمززارز یافت نشد');
      return await response.json();
    }

    // دریافت داده‌های بازار از CoinGecko
    async function fetchMarketData(cryptoId) {
      const response = await fetch(`${API_ENDPOINTS.COINGECKO}/coins/${cryptoId.toLowerCase()}/market_chart?vs_currency=usd&days=30`);
      if (!response.ok) throw new Error('داده‌های بازار یافت نشد');
      return await response.json();
    }

    // دریافت داده از بایننس
    async function fetchBinanceData(cryptoId) {
      const symbol = `${cryptoId.toUpperCase()}USDT`;
      const response = await fetch(`${API_ENDPOINTS.BINANCE}/ticker/24hr?symbol=${symbol}`);
      if (!response.ok) throw new Error('داده‌های بایننس یافت نشد');
      return await response.json();
    }

    // دریافت داده از کوینبیس
    async function fetchCoinbaseData(cryptoId) {
      const response = await fetch(`${API_ENDPOINTS.COINBASE}/prices/${cryptoId.toUpperCase()}-USD/spot`);
      if (!response.ok) throw new Error('داده‌های کوینبیس یافت نشد');
      const data = await response.json();
      return { price: data.data.amount };
    }

    // دریافت داده‌های تاریخی از CryptoCompare
    async function fetchHistoricalData(cryptoId) {
      const response = await fetch(`${API_ENDPOINTS.CRYPTOCOMPARE}/histoday?fsym=${cryptoId.toUpperCase()}&tsym=USD&limit=30`);
      if (!response.ok) throw new Error('داده‌های تاریخی یافت نشد');
      const data = await response.json();
      return data.Data.map(item => ({
        date: new Date(item.time * 1000),
        price: item.close
      }));
    }

    // نمایش اطلاعات قیمت
    function displayPriceData(coinData, marketData, binanceData, coinbaseData) {
      const priceDataDiv = document.getElementById('priceData');
      const currentPrice = coinData.market_data.current_price.usd;
      const priceChange24h = coinData.market_data.price_change_percentage_24h;
      const priceChange7d = coinData.market_data.price_change_percentage_7d;
      
      priceDataDiv.innerHTML = `
        <div class="price-card">
          <div class="price-header">
            <h3>اطلاعات کلی</h3>
            <i class="fas fa-info-circle price-icon"></i>
          </div>
          <div class="price-value">${currentPrice.toLocaleString('en')} USD</div>
          <div>
            تغییر 24h: <span class="price-change ${priceChange24h >= 0 ? 'positive' : 'negative'}">
              ${priceChange24h >= 0 ? '+' : ''}${priceChange24h.toFixed(2)}%
            </span>
          </div>
          <div>
            تغییر 7d: <span class="price-change ${priceChange7d >= 0 ? 'positive' : 'negative'}">
              ${priceChange7d >= 0 ? '+' : ''}${priceChange7d.toFixed(2)}%
            </span>
          </div>
          <div>تومان: ${(currentPrice * exchangeRates.irr / 10).toLocaleString('fa-IR')}</div>
          <div>حجم معاملات: ${(coinData.market_data.total_volume.usd / 1000000).toFixed(2)}M</div>
        </div>
        
        <div class="price-card">
          <div class="price-header">
            <h3>CoinGecko</h3>
            <i class="fas fa-dragon price-icon"></i>
          </div>
          <div class="price-value">${currentPrice.toLocaleString('en')} USD</div>
          <div>بالاترین امروز: ${coinData.market_data.high_24h.usd.toLocaleString('en')}</div>
          <div>پایین‌ترین امروز: ${coinData.market_data.low_24h.usd.toLocaleString('en')}</div>
          <div>ارزش بازار: ${(coinData.market_data.market_cap.usd / 1000000000).toFixed(2)}B</div>
        </div>
        
        <div class="price-card">
          <div class="price-header">
            <h3>Binance</h3>
            <i class="fas fa-exchange-alt price-icon"></i>
          </div>
          <div class="price-value">${parseFloat(binanceData.lastPrice).toLocaleString('en')} USD</div>
          <div>حجم 24h: ${(binanceData.volume / 1000).toFixed(2)}K</div>
          <div>تغییر 24h: <span class="price-change ${binanceData.priceChangePercent >= 0 ? 'positive' : 'negative'}">
            ${binanceData.priceChangePercent >= 0 ? '+' : ''}${parseFloat(binanceData.priceChangePercent).toFixed(2)}%
          </span></div>
          <div>اسپرد: ${((binanceData.askPrice - binanceData.bidPrice) / binanceData.askPrice * 100).toFixed(4)}%</div>
        </div>
        
        <div class="price-card">
          <div class="price-header">
            <h3>Coinbase</h3>
            <i class="fas fa-coins price-icon"></i>
          </div>
          <div class="price-value">${parseFloat(coinbaseData.price).toLocaleString('en')} USD</div>
          <div>صرافی معتبر آمریکایی</div>
          <div>پشتیبانی از معاملات حرفه‌ای</div>
          <div>نقدشوندگی بالا</div>
        </div>
      `;
    }

    // رسم نمودار قیمت
    function renderPriceChart(historicalData) {
      const ctx = document.getElementById('priceChart').getContext('2d');
      
      const labels = historicalData.map(item => 
        item.date.toLocaleDateString('fa-IR', { month: 'short', day: 'numeric' })
      );
      
      const prices = historicalData.map(item => item.price);
      
      new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'قیمت (USD)',
            data: prices,
            borderColor: '#3498db',
            backgroundColor: 'rgba(52, 152, 219, 0.1)',
            borderWidth: 2,
            fill: true,
            tension: 0.4
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              rtl: true,
              labels: {
                font: {
                  family: 'Segoe UI'
                }
              }
            }
          },
          scales: {
            x: {
              ticks: {
                font: {
                  family: 'Segoe UI'
                }
              }
            },
            y: {
              ticks: {
                callback: function(value) {
                  return value.toLocaleString('en');
                }
              }
            }
          }
        }
      });
    }

    // نمایش تحلیل فنی
    function displayTechnicalAnalysis(marketData, historicalData) {
      const analysisDiv = document.getElementById('analysisResult');
      const prices = historicalData.map(item => item.price);
      const lastPrice = prices[prices.length - 1];
      
      // محاسبه میانگین متحرک ساده (SMA) 7 روزه
      const sma7 = prices.slice(-7).reduce((sum, price) => sum + price, 0) / 7;
      
      // محاسبه RSI (شبیه‌سازی شده)
      const priceChanges = [];
      for (let i = 1; i < prices.length; i++) {
        priceChanges.push(prices[i] - prices[i - 1]);
      }
      const avgGain = priceChanges.filter(x => x > 0).reduce((sum, gain) => sum + gain, 0) / 14;
      const avgLoss = -priceChanges.filter(x => x < 0).reduce((sum, loss) => sum + loss, 0) / 14;
      const rsi = 100 - (100 / (1 + (avgGain / avgLoss)));
      
      analysisDiv.innerHTML = `
        <h3><i class="fas fa-chart-bar"></i> تحلیل فنی</h3>
        <div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 20px; margin-top: 15px;">
          <div>
            <h4><i class="fas fa-arrow-up"></i> سیگنال‌های خرید</h4>
            <p>💰 قیمت مناسب خرید: ${(lastPrice * 0.97).toLocaleString('en')} USD (3% پایین‌تر)</p>
            <p>📌 حمایت کوتاه‌مدت: ${(lastPrice * 0.95).toLocaleString('en')} USD</p>
            <p>📌 حمایت قوی: ${(lastPrice * 0.90).toLocaleString('en')} USD</p>
          </div>
          
          <div>
            <h4><i class="fas fa-arrow-down"></i> سیگنال‌های فروش</h4>
            <p>📈 قیمت مناسب فروش: ${(lastPrice * 1.03).toLocaleString('en')} USD (3% بالاتر)</p>
            <p>📌 مقاومت کوتاه‌مدت: ${(lastPrice * 1.05).toLocaleString('en')} USD</p>
            <p>📌 مقاومت قوی: ${(lastPrice * 1.10).toLocaleString('en')} USD</p>
          </div>
          
          <div>
            <h4><i class="fas fa-calculator"></i> شاخص‌های فنی</h4>
            <p>📊 SMA 7 روزه: ${sma7.toLocaleString('en')} USD</p>
            <p>📉 RSI 14 روزه: ${rsi.toFixed(2)} ${rsi < 30 ? '(اشباع فروش)' : rsi > 70 ? '(اشباع خرید)' : ''}</p>
            <p>📈 حجم معاملات: ${(marketData.total_volumes.slice(-1)[0][1] / 1000000).toFixed(2)}M</p>
          </div>
        </div>
      `;
    }

    // شروع با تحلیل بیت‌کوین به صورت پیش‌فرض
    window.onload = function() {
      analyzeCrypto();
    };
  </script>
</body>
</html>
