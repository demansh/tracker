---
layout: default
title: Tracker
---

<style>
  :root {
    --bg-card: #ffffff;
    --text-main: #2d3748;
    --text-muted: #718096;
    --border-color: #e2e8f0;
    --shadow-sm: 0 1px 3px rgba(0,0,0,0.05), 0 1px 2px rgba(0,0,0,0.03);
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
    letter-spacing: -0.02em;
  }

  .ui-card p {
    color: var(--text-muted);
    font-size: 0.9rem;
    margin-bottom: 20px;
  }

  /* Адаптивный контейнер для нативной формы Google */
  .form-wrapper {
    position: relative;
    width: 100%;
    height: 750px; 
    border-radius: calc(var(--radius) - 4px);
    overflow: hidden;
  }

  .form-wrapper iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border: none;
  }

  /* Контейнер для адаптивного графика */
  .chart-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    padding: 10px 0;
  }

  .chart-wrapper img {
    width: 100%;
    height: auto;
    max-width: 100%;
    object-fit: contain;
    transition: opacity 0.3s ease;
  }

  /* Индикатор обновления в стиле iOS/Material */
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
    width: 12px;
    height: 12px;
    border: 2px solid #cbd5e0;
    border-top-color: #4a5568;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    display: none;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  /* Оптимизация под мобильные устройства */
  @media (max-width: 480px) {
    .dashboard-container {
      padding: 12px 8px;
    }
    .ui-card {
      padding: 16px;
      margin-bottom: 16px;
      border-radius: var(--radius);
    }
    .form-wrapper {
      height: 920px; 
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
        <p>Данные обновляются автоматически каждые 7 секунд после начала взаимодействия.</p>
      </div>
      <div class="refresh-badge" id="status-badge">
        <div class="spinner" id="refresh-spinner"></div>
        <span id="status-text">Автообновление</span>
      </div>
    </div>

    <div class="chart-wrapper">
      <img id="responsive-image-chart" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vT5y16P0vfS6N0x70eDhHBIM-pPb1e0k--lfKurgEcWIBHtlv-KD9LyXVkP3RnLuOFnSEOSWE5AIb7N/pubchart?oid=24048480&format=image" alt="График аналитики">
    </div>
  </div>

</div>

<script>
  let userInteracted = false;
  let updateInterval = null;
  const chartImg = document.getElementById('responsive-image-chart');
  const spinner = document.getElementById('refresh-spinner');
  const statusText = document.getElementById('status-text');
  const formWrapper = document.getElementById('form-card-wrapper');

  function reloadChartData() {
    if (!chartImg) return;
    
    spinner.style.display = 'block';
    statusText.innerText = 'Обновление...';
    chartImg.style.opacity = '0.7';

    const originalSrc = chartImg.src.split('&cacheBust=')[0];
    
    const newImg = new Image();
    newImg.src = originalSrc + '&cacheBust=' + new Date().getTime();
    
    newImg.onload = function() {
      chartImg.src = newImg.src;
      chartImg.style.opacity = '1';
      spinner.style.display = 'none';
      statusText.innerText = 'Обновлено';
      
      setTimeout(() => {
        statusText.innerText = 'Автообновление';
      }, 2000);
    };
  }

  // Активация таймера при кликах / тапах по форме
  formWrapper.addEventListener('click', startSmartTimer);
  formWrapper.addEventListener('touchstart', startSmartTimer);

  // Для ПК: уход фокуса в iframe формы
  window.addEventListener('blur', function() {
    if (document.activeElement.tagName === 'IFRAME') {
      startSmartTimer();
    }
  });

  function startSmartTimer() {
    if (userInteracted) return;
    userInteracted = true;
    
    // Запуск цикла обновления раз в 7 секунд
    updateInterval = setInterval(reloadChartData, 7000);
  }
</script>