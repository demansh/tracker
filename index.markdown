<style>
  :root {
    --bg-card: #ffffff;
    --text-main: #2d3748;
    --text-muted: #718096;
    --border-color: #e2e8f0;
    --shadow-md: 0 4px 6px -1px rgba(0,0,0,0.05), 0 2px 4px -1px rgba(0,0,0,0.03);
    --radius: 12px;
  }

  .dashboard-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px 16px;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    color: var(--text-main);
    background-color: #f7fafc;
  }

  .ui-card {
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: var(--radius);
    box-shadow: var(--shadow-md);
    padding: 24px;
    margin-bottom: 24px;
    overflow: hidden;
  }

  .ui-card h2 {
    margin-top: 0;
    margin-bottom: 8px;
    font-size: 1.25rem;
    font-weight: 600;
  }

  .ui-card p {
    color: var(--text-muted);
    font-size: 0.9rem;
    margin-bottom: 20px;
  }

  .form-wrapper {
    position: relative;
    width: 100%;
    height: 750px; 
    border-radius: calc(var(--radius) - 4px);
    overflow: hidden;
  }

  .form-wrapper iframe {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    border: none;
  }

  /* СВЕРХАДАПТИВНЫЙ КОНТЕЙНЕР ДЛЯ ГРАФИКА */
  .chart-responsive-container {
    position: relative;
    width: 100%;
    height: 400px; /* Фиксированная высота для десктопа */
    overflow: hidden;
  }

  /* На компах график просто заполняет область */
  .chart-responsive-container iframe {
    width: 100%;
    height: 100%;
    border: none;
    transition: opacity 0.3s ease;
  }

  .refresh-badge {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: #edf2f7;
    color: var(--text-muted);
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 0.75rem;
    font-weight: 500;
    margin-bottom: 16px;
  }

  .spinner {
    width: 12px; height: 12px;
    border: 2px solid #cbd5e0;
    border-top-color: #4a5568;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    display: none;
  }

  @keyframes spin { to { transform: rotate(360deg); } }

  /* ТРЮК ДЛЯ СМАРТФОНОВ: Масштабируем интерактивный фрейм */
  @media (max-width: 600px) {
    .dashboard-container {
      padding: 12px 8px;
    }
    .ui-card {
      padding: 16px;
      margin-bottom: 16px;
    }
    .form-wrapper {
      height: 920px; 
    }
    
    /* Меняем логику контейнера на мобильных */
    .chart-responsive-container {
      height: 260px; /* Уменьшаем высоту контейнера, так как сам график сожмется */
    }
    
    .chart-responsive-container iframe {
      width: 160%; /* Делаем виртуальный экран шире */
      height: 160%;
      /* Сжимаем картинку интерактивного графика обратно в границы экрана телефона */
      transform: scale(0.625); 
      transform-origin: top left;
    }
  }

  @media (max-width: 400px) {
    .chart-responsive-container {
      height: 210px;
    }
    .chart-responsive-container iframe {
      width: 200%;
      height: 200%;
      transform: scale(0.5);
    }
  }
</style>

<div class="dashboard-container">

  <div class="ui-card" id="form-card-wrapper">
    <h2>Заполнить форму</h2>
    <p>Пожалуйста, внесите ваши актуальные данные ниже. Результаты сразу отобразятся на графике.</p>
    
    <div class="form-wrapper">
      <iframe src="https://docs.google.com/forms/d/e/1FAIpQLSei-sxllN2AEFnCsEOc4huBwBXRNbnFq50oV5usLWZbdJT_Yg/viewform?embedded=true">Загрузка формы…</iframe>
    </div>
  </div>

  <div class="ui-card">
    <div style="display: flex; justify-content: space-between; align-items: flex-start;">
      <div>
        <h2>Текущая статистика</h2>
        <p>Интерактивные живые данные без задержек кэширования.</p>
      </div>
      <div class="refresh-badge" id="status-badge">
        <div class="spinner" id="refresh-spinner"></div>
        <span id="status-text">Автообновление</span>
      </div>
    </div>

    <div class="chart-responsive-container">
      <iframe id="responsive-interactive-chart" 
              src="https://docs.google.com/spreadsheets/d/e/2PACX-1vT5y16P0vfS6N0x70eDhHBIM-pPb1e0k--lfKurgEcWIBHtlv-KD9LyXVkP3RnLuOFnSEOSWE5AIb7N/pubchart?oid=24048480&format=interactive">
      </iframe>
    </div>
  </div>

</div>

<script>
  let userInteracted = false;
  let updateInterval = null;
  const chartIframe = document.getElementById('responsive-interactive-chart');
  const spinner = document.getElementById('refresh-spinner');
  const statusText = document.getElementById('status-text');
  const formWrapper = document.getElementById('form-card-wrapper');

  function reloadChartData() {
    if (!chartIframe) return;
    
    spinner.style.display = 'block';
    statusText.innerText = 'Обновление...';
    chartIframe.style.opacity = '0.5';

    const originalSrc = chartIframe.src.split('&cacheBust=')[0];
    chartIframe.src = originalSrc + '&cacheBust=' + new Date().getTime();
    
    chartIframe.onload = function() {
      chartIframe.style.opacity = '1';
      spinner.style.display = 'none';
      statusText.innerText = 'Обновлено';
      
      setTimeout(() => {
        statusText.innerText = 'Автообновление';
      }, 2000);
    };
  }

  formWrapper.addEventListener('click', startSmartTimer);
  formWrapper.addEventListener('touchstart', startSmartTimer);

  window.addEventListener('blur', function() {
    if (document.activeElement.tagName === 'IFRAME' && document.activeElement.id !== 'responsive-interactive-chart') {
      startSmartTimer();
    }
  });

  function startSmartTimer() {
    if (userInteracted) return;
    userInteracted = true;
    updateInterval = setInterval(reloadChartData, 10000);
  }
</script>