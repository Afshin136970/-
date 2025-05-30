# -
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>تحلیلگر پیشرفته رمزارزها</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f2f5; padding: 20px; color: #333; }
    .container { max-width: 900px; margin: auto; background: white; padding: 25px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
    h2 { color: #2c3e50; text-align: center; margin-bottom: 25px; border-bottom: 2px solid #eee; padding-bottom: 10px; }
    .search-box { display: flex; gap: 10px; margin-bottom: 20px; }
    input { flex: 1; padding: 12px 15px; border-radius: 8px; border: 1px solid #ddd; font-size: 16px; }
    button { background-color: #3498db; color: white; border: none; padding: 12px 20px; border-radius: 8px; cursor: pointer; font-weight: bold; transition: all 0.3s; }
    button:hover { background-color: #2980b9; }
    .result { margin-top: 25px; background: #f9f9f9; padding: 20px; border-radius: 10px; direction: ltr; }
    .price-section { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 15px; margin-bottom: 20px; }
    .price-card { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
    .price-card h3 { margin-top: 0; color: #2c3e50; border-bottom: 1px solid #eee; padding-bottom: 8px; }
    .price-value { font-size: 18px; font-weight: bold; color: #27ae60; }
    .negative { color: #e74c3c; }
    .exchange-tabs { display: flex; gap: 10px; margin-bottom: 15px; }
    .exchange-tab { padding: 8px 15px; background: #eee; border-radius: 5px; cursor: pointer; }
    .exchange-tab.active { background: #3498db; color: white; }
    .loading { text-align: center; padding: 20px; }
    .loading-spinner { border: 4px solid #f3f3f3; border-top: 4px solid #3498db; border-radius: 50%; width: 30px; height: 30px; animation: spin 1s linear infinite; margin: 0 auto; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    .time-update { font-size: 12px; color: #7f8c8d; text-align: left; margin-top: 10px; }
    .news-section { margin-top: 30px; border-top: 1px solid #eee; padding-top: 20px; }
    .news-item { margin-bottom: 15px; padding-bottom: 15px; border-bottom: 1px dashed #eee; }
    .chart-container { height: 300px; margin: 20px 0; background: #f9f9f9; border-radius: 8px; display: flex; align-items: center; justify-content: center; }
  </style>
</head>
<body>
  <div class="container">
    <h2><i class="fas fa-chart-line"></i> تحلیلگر پیشرفته رمزارزها</h2>
    
    <div class="search-box">
      <input type="text" id="cryptoName" placeholder="نام یا نماد رمزارز (مثال: بیت‌کوین، BTC، اتریوم)">
      <button onclick="analyzeCrypto()"><i class="fas fa-search"></i> تحلیل و مقایسه</button>
    </div>
    
    <div class="exchange-tabs" id="exchangeTabs">
      <!-- تب های صرافی ها به صورت دینامیک اضافه می شوند -->
    </div>
    
    <div class="result" id="result">
      <div class="loading" id="loading">
        <div class="loading-spinner"></div>
        <p>در حال دریافت اطلاعات از صرافی‌ها و شبکه‌های مختلف...</p>
      </div>
      <div id="analysisResult"></div>
      <div class="time-update" id="timeUpdate"></div>
    </div>
    
    <div class="chart-container" id="priceChart">
      <p>نمودار قیمت پس از انتخاب رمزارز نمایش داده می‌شود</p>
    </div>
    
    <div class="news-section" id="newsSection">
      <h3><i class="fas fa-newspaper"></i> اخبار و تحلیلات مرتبط</h3>
      <div id="newsContent"></div>
    </div>
  </div>

  <script>
    // لیست صرافی‌های معتبر
    const exchanges = [
      { id: 'binance', name: 'بایننس', icon: 'fas fa-exchange-alt' },
      { id: 'coinbase', name: 'کوینبیس', icon: 'fas fa-coins' },
      { id: 'kucoin', name: 'کوکوین', icon: 'fas fa-chart-pie' },
      { id: 'okx', name: 'OKX', icon: 'fas fa-project-diagram' },
      { id: 'bybit', name: 'بایبیت', icon: 'fas fa-robot' },
      { id: 'dex', name: 'صرافی غیرمتمرکز', icon: 'fas fa-link' }
    ];

    // لیست شبکه‌های ال بانک
    const lendingPlatforms = [
      { id: 'aave', name: 'آوه', icon: 'fas fa-hand-holding-usd' },
      { id: 'compound', name: 'کامپوند', icon: 'fas fa-piggy-bank' },
      { id: 'makerdao', name: 'میکر دائو', icon: 'fas fa-landmark' }
    ];

    // نرخ‌های تبدیل (می‌تواند از API دریافت شود)
    const exchangeRates = {
      usd: 1,
      usdt: 1,
      irr: 58000,
      eur: 0.92
    };

    // مقدار دهی اولیه تب‌های صرافی‌ها
    function initExchangeTabs() {
      const tabsContainer = document.getElementById('exchangeTabs');
      
      // افزودن تب "همه"
      const allTab = document.createElement('div');
      allTab.className = 'exchange-tab active';
      allTab.innerHTML = '<i class="fas fa-globe"></i> همه';
      allTab.onclick = () => filterResults('all');
      tabsContainer.appendChild(allTab);
      
      // افزودن تب‌های صرافی‌ها
      exchanges.forEach(exchange => {
        const tab = document.createElement('div');
        tab.className = 'exchange-tab';
        tab.innerHTML = `<i class="${exchange.icon}"></i> ${exchange.name}`;
        tab.onclick = () => filterResults(exchange.id);
        tabsContainer.appendChild(tab);
      });
      
      // افزودن تب‌های ال بانک‌ها
      lendingPlatforms.forEach(platform => {
        const tab = document.createElement('div');
        tab.className = 'exchange-tab';
        tab.innerHTML = `<i class="${platform.icon}"></i> ${platform.name}`;
        tab.onclick = () => filterResults(platform.id);
        tabsContainer.appendChild(tab);
      });
    }

    // فیلتر کردن نتایج بر اساس صرافی انتخاب شده
    function filterResults(exchangeId) {
      const tabs = document.querySelectorAll('.exchange-tab');
      tabs.forEach(tab => tab.classList.remove('active'));
      event.target.classList.add('active');
      
      // اینجا می‌توانید منطق فیلتر کردن را پیاده‌سازی کنید
      console.log(`فیلتر بر اساس: ${exchangeId}`);
    }

    // تحلیل رمزارز
    async function analyzeCrypto() {
      const input = document.getElementById('cryptoName').value.trim();
      const resultDiv = document.getElementById('analysisResult');
      const loadingDiv = document.getElementById('loading');
      const timeUpdateDiv = document.getElementById('timeUpdate');
      
      if (!input) {
        alert('لطفا نام یا نماد رمزارز را وارد کنید');
        return;
      }
      
      // نمایش وضعیت بارگذاری
      loadingDiv.style.display = 'block';
      resultDiv.innerHTML = '';
      
      try {
        // شبیه‌سازی دریافت داده از چندین منبع
        const analysisData = await fetchMultiSourceData(input);
        
        // نمایش نتایج
        displayAnalysisResults(analysisData);
        
        // به‌روزرسانی زمان آخرین به‌روزرسانی
        timeUpdateDiv.innerHTML = `<i class="fas fa-sync-alt"></i> آخرین به‌روزرسانی: ${new Date().toLocaleString('fa-IR')}`;
      } catch (error) {
        resultDiv.innerHTML = `<div class="error"><i class="fas fa-exclamation-triangle"></i> خطا در دریافت اطلاعات: ${error.message}</div>`;
      } finally {
        loadingDiv.style.display = 'none';
      }
    }

    // شبیه‌سازی دریافت داده از چندین منبع
    async function fetchMultiSourceData(cryptoName) {
      // در یک نسخه واقعی، اینجا APIهای مختلف صرافی‌ها فراخوانی می‌شوند
      // این یک شبیه‌سازی با داده‌های نمونه است
      
      return new Promise((resolve) => {
        setTimeout(() => {
          const mockData = {
            symbol: cryptoName.toUpperCase(),
            name: cryptoName === 'btc' ? 'بیت‌کوین' : 
                  cryptoName === 'eth' ? 'اتریوم' : 
                  cryptoName === 'usdt' ? 'تتر' : cryptoName,
            prices: {
              binance: { price: getRandomPrice(30000, 50000), volume: getRandomNumber(1000, 5000) },
              coinbase: { price: getRandomPrice(30000, 50000), volume: getRandomNumber(800, 4000) },
              kucoin: { price: getRandomPrice(30000, 50000), volume: getRandomNumber(500, 3000) },
              okx: { price: getRandomPrice(30000, 50000), volume: getRandomNumber(700, 3500) },
              bybit: { price: getRandomPrice(30000, 50000), volume: getRandomNumber(600, 3200) },
              aave: { price: getRandomPrice(30000, 50000), apy: getRandomNumber(2, 8) },
              compound: { price: getRandomPrice(30000, 50000), apy: getRandomNumber(1.5, 7) },
              makerdao: { price: getRandomPrice(30000, 50000), apy: getRandomNumber(3, 9) }
            },
            chartData: generateChartData(),
            news: [
              { title: 'تحلیل هفتگی بازار رمزارزها', source: 'CoinDesk فارسی', date: '2 ساعت پیش' },
              { title: 'تأثیر تصمیمات فدرال رزرو بر قیمت بیت‌کوین', source: 'CryptoNews', date: '5 ساعت پیش' },
              { title: 'به‌روزرسانی شبکه اتریوم و تأثیر آن بر قیمت', source: 'Ethereum Blog', date: '1 روز پیش' }
            ]
          };
          resolve(mockData);
        }, 1500);
      });
    }

    // نمایش نتایج تحلیل
    function displayAnalysisResults(data) {
      const resultDiv = document.getElementById('analysisResult');
      const newsContent = document.getElementById('newsContent');
      const chartDiv = document.getElementById('priceChart');
      
      // نمایش اطلاعات قیمت از صرافی‌های مختلف
      let priceHtml = '<div class="price-section">';
      
      for (const [exchange, info] of Object.entries(data.prices)) {
        const exchangeInfo = [...exchanges, ...lendingPlatforms].find(e => e.id === exchange);
        const exchangeName = exchangeInfo ? exchangeInfo.name : exchange;
        const exchangeIcon = exchangeInfo ? exchangeInfo.icon : 'fas fa-exchange-alt';
        
        priceHtml += `
          <div class="price-card">
            <h3><i class="${exchangeIcon}"></i> ${exchangeName}</h3>
            <p>قیمت: <span class="price-value">${info.price.toLocaleString('en')} USD</span></p>
            ${info.volume ? `<p>حجم معاملات: ${info.volume.toLocaleString('en')} BTC</p>` : ''}
            ${info.apy ? `<p>سود سالانه: ${info.apy}%</p>` : ''}
            <p>تومان: ${(info.price * exchangeRates.irr).toLocaleString('fa-IR')}</p>
          </div>
        `;
      }
      
      priceHtml += '</div>';
      
      // تحلیل تکنیکال
      const analysisHtml = `
        <div class="analysis-section">
          <h3><i class="fas fa-chart-bar"></i> تحلیل فنی</h3>
          <p>💰 قیمت میانگین: ${calculateAveragePrice(data.prices).toLocaleString('en')} USD</p>
          <p>📉 حد ضرر پیشنهادی: ${(calculateAveragePrice(data.prices) * 0.92).toLocaleString('en')} USD (8% پایین‌تر)</p>
          <p>🛒 قیمت خرید مناسب: ${(calculateAveragePrice(data.prices) * 0.97).toLocaleString('en')} USD (3% پایین‌تر)</p>
          <p>📈 قیمت فروش مناسب: ${(calculateAveragePrice(data.prices) * 1.03).toLocaleString('en')} USD (3% بالاتر)</p>
          <p>🔍 اختلاف قیمت بین صرافی‌ها: ${calculatePriceDifference(data.prices)}%</p>
        </div>
      `;
      
      // نمایش اخبار
      let newsHtml = '';
      data.news.forEach(item => {
        newsHtml += `
          <div class="news-item">
            <h4>${item.title}</h4>
            <p><small>منبع: ${item.source} - ${item.date}</small></p>
          </div>
        `;
      });
      
      // نمایش نمودار (شبیه‌سازی)
      chartDiv.innerHTML = '<canvas id="priceChartCanvas" width="800" height="300"></canvas>';
      renderMockChart(data.chartData);
      
      // ترکیب تمام بخش‌ها
      resultDiv.innerHTML = `
        <h2 style="text-align: center;">تحلیل ${data.name} (${data.symbol})</h2>
        ${priceHtml}
        ${analysisHtml}
      `;
      
      newsContent.innerHTML = newsHtml;
    }

    // توابع کمکی
    function getRandomPrice(min, max) {
      return (Math.random() * (max - min) + min).toFixed(2);
    }

    function getRandomNumber(min, max) {
      return (Math.random() * (max - min) + min).toFixed(2);
    }

    function calculateAveragePrice(prices) {
      const values = Object.values(prices).map(item => item.price);
      return (values.reduce((a, b) => a + parseFloat(b), 0) / values.length).toFixed(2);
    }

    function calculatePriceDifference(prices) {
      const values = Object.values(prices).map(item => item.price);
      const max = Math.max(...values);
      const min = Math.min(...values);
      return (((max - min) / ((max + min) / 2)) * 100).toFixed(2);
    }

    function generateChartData() {
      const data = [];
      for (let i = 0; i < 30; i++) {
        data.push({
          date: new Date(Date.now() - (29 - i) * 24 * 60 * 60 * 1000),
          price: 30000 + Math.random() * 20000
        });
      }
      return data;
    }

    function renderMockChart(data) {
      const canvas = document.getElementById('priceChartCanvas');
      if (!canvas) return;
      
      const ctx = canvas.getContext('2d');
      
      // شبیه‌سازی یک نمودار ساده
      ctx.fillStyle = '#f9f9f9';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      ctx.strokeStyle = '#3498db';
      ctx.lineWidth = 2;
      ctx.beginPath();
      
      const step = canvas.width / (data.length - 1);
      const minPrice = Math.min(...data.map(d => d.price));
      const maxPrice = Math.max(...data.map(d => d.price));
      const priceRange = maxPrice - minPrice;
      
      data.forEach((point, i) => {
        const x = i * step;
        const y = canvas.height - ((point.price - minPrice) / priceRange) * canvas.height * 0.8 - canvas.height * 0.1;
        
        if (i === 0) {
          ctx.moveTo(x, y);
        } else {
          ctx.lineTo(x, y);
        }
        
        // نقاط داده
        ctx.fillStyle = '#3498db';
        ctx.beginPath();
        ctx.arc(x, y, 3, 0, Math.PI * 2);
        ctx.fill();
      });
      
      ctx.stroke();
      
      // محورها
      ctx.strokeStyle = '#ccc';
      ctx.lineWidth = 1;
      
      // محور X
      ctx.beginPath();
      ctx.moveTo(0, canvas.height * 0.9);
      ctx.lineTo(canvas.width, canvas.height * 0.9);
      ctx.stroke();
      
      // محور Y
      ctx.beginPath();
      ctx.moveTo(50, 0);
      ctx.lineTo(50, canvas.height);
      ctx.stroke();
      
      // برچسب‌ها
      ctx.fillStyle = '#7f8c8d';
      ctx.font = '10px Arial';
      ctx.textAlign = 'center';
      
      // برچسب محور X
      const firstDate = data[0].date.toLocaleDateString('fa-IR').split('/').slice(1).join('/');
      const lastDate = data[data.length - 1].date.toLocaleDateString('fa-IR').split('/').slice(1).join('/');
      
      ctx.fillText(firstDate, 0, canvas.height - 5);
      ctx.fillText(lastDate, canvas.width, canvas.height - 5);
      
      // برچسب محور Y
      ctx.textAlign = 'right';
      ctx.fillText(maxPrice.toFixed(0), 45, 15);
      ctx.fillText(minPrice.toFixed(0), 45, canvas.height - 5);
    }

    // مقدار دهی اولیه هنگام بارگذاری صفحه
    window.onload = function() {
      initExchangeTabs();
    };
  </script>
</body>
</html>
