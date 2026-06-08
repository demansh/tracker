---
layout: default
title: Универсальный Контроль Веса
---

<style>
  :root {
    --bg-card: #ffffff;
    --text-main: #2d3748;
    --text-muted: #718096;
    --primary: #3182ce;
    --primary-hover: #2b6cb0;
    --border-color: #e2e8f0;
    --shadow-md: 0 4px 6px -1px rgba(0,0,0,0.05), 0 2px 4px -1px rgba(0,0,0,0.03);
    --radius: 12px;
  }

  .dashboard-container {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px 16px;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    color: var(--text-main);
  }

  .ui-card {
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: var(--radius);
    box-shadow: var(--shadow-md);
    padding: 24px;
    margin-bottom: 24px;
  }

  .ui-card h2 { margin-top: 0; margin-bottom: 6px; font-size: 1.25rem; font-weight: 600; }
  .ui-card p { color: var(--text-muted); font-size: 0.9rem; margin-bottom: 20px; }

  /* Кастомная форма ввода */
  #custom-weight-form { display: none; } /* По умолчанию скрыта, включается при успехе */
  .weight-form-group { display: flex; gap: 12px; width: 100%; }
  .input-wrapper { position: relative; flex-grow: 1; }

  .custom-input {
    width: 100%;
    padding: 14px 45px 14px 16px;
    font-size: 1.1rem;
    font-weight: 500;
    border: 2px solid var(--border-color);
    border-radius: 8px;
    outline: none;
    transition: all 0.2s ease;
    box-sizing: border-box;
  }

  .custom-input:focus {
    border-color: var(--primary);
    box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.15);
  }

  .input-suffix {
    position: absolute;
    right: 16px; top: 50%; transform: translateY(-50%);
    color: var(--text-muted); font-weight: 500; pointer-events: none;
  }

  .custom-btn {
    background-color: var(--primary); color: white; border: none; padding: 0 24px;
    font-size: 1rem; font-weight: 600; border-radius: 8px; cursor: pointer;
    transition: background-color 0.2s ease; white-space: nowrap;
  }

  .custom-btn:hover { background-color: var(--primary-hover); }

  /* Фрейм для оригинальной формы (резервный вариант) */
  .original-form-wrapper {
    display: none; /* Включается только при ошибке парсинга */
    position: relative;
    width: 100%;
    height: 280px; /* Так как поле всего одно, сделаем фрейм компактным */
    border-radius: 8px;
    overflow: hidden;
    border: 1px solid var(--border-color);
  }

  .original-form-wrapper iframe {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%; border: none;
  }

  /* Адаптивный контейнер для интерактивного графика */
  .chart-responsive-container { position: relative; width: 100%; height: 380px; overflow: hidden; border-radius: 8px; }
  .chart-responsive-container iframe { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none; transition: opacity 0.3s ease; }

  .refresh-badge { display: inline-flex; align-items: center; gap: 6px; background: #edf2f7; color: var(--text-muted); padding: 6px 12px; border-radius: 20px; font-size: 0.75rem; font-weight: 500; margin-bottom: 16px; }
  .spinner { width: 12px; height: 12px; border: 2px solid #cbd5e0; border-top-color: #4a5568; border-radius: 50%; animation: spin 1s linear infinite; display: none; }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* Окно настройки, если в URL нет параметров */
  .welcome-box { text-align: center; padding: 40px 20px; }
  .welcome-box input { width: 100%; padding: 12px; margin-bottom: 12px; border: 1px solid var(--border-color); border-radius: 6px; }
  
  @media (max-width: 480px) {
    .dashboard-container { padding: 12px 8px; }
    .ui-card { padding: 16px; margin-bottom: 16px; }
    .weight-form-group { flex-direction: column; gap: 10px; }
    .custom-btn { padding: 14px; width: 100%; }
    .chart-responsive-container { height: 250px; }
    .chart-responsive-container iframe { width: 150%; height: 150%; transform: scale(0.666); transform-origin: top left; }
    .original-form-wrapper { height: 340px; }
  }
</style>

<div class="dashboard-container">

  <div class="ui-card welcome-box" id="welcome-card" style="display: none;">
    <h2>Создать свой Dashboard</h2>
    <p>Вставьте ссылки из адресной строки вашего браузера, чтобы сгенерировать персональный дашборд.</p>
    <input type="text" id="setup-form-url" placeholder="Ссылка на Google Форму (viewform)">
    <input type="text" id="setup-chart-url" placeholder="Ссылка на Google График (pubchart)">
    <button class="custom-btn" onclick="generateDashboardLink()" style="padding: 12px 24px;">Создать дашборд</button>
  </div>

  <div id="main-dashboard" style="display: none;">
    
    <div class="ui-card" id="form-card-wrapper">
      <h2>Ввести вес</h2>
      <p id="form-status" style="color: #4a5568;">Синхронизация с вашей Google Формой...</p>
      
      <form id="custom-weight-form" action="" method="POST" target="hidden_iframe">
        <div class="weight-form-group">
          <div class="input-wrapper">
            <input type="number" step="0.1" id="weight-input" name="" placeholder="0.0" class="custom-input" required>
            <span class="input-suffix">кг</span>
          </div>
          <button type="submit" id="submit-btn" class="custom-btn">Сохранить</button>
        </div>
      </form>

      <div class="original-form-wrapper" id="fallback-iframe-wrapper">
        <iframe id="original-google-iframe" src="">Загрузка формы…</iframe>
      </div>
    </div>

    <div class="ui-card">
      <div style="display: flex; justify-content: space-between; align-items: flex-start;">
        <div>
          <h2>История изменений</h2>
          <p>График обновится автоматически через 4 секунды после сохранения или взаимодействия.</p>
        </div>
        <div class="refresh-badge">
          <div class="spinner" id="refresh-spinner"></div>
          <span id="status-text">Автообновление</span>
        </div>
      </div>

      <div class="chart-responsive-container">
        <iframe id="responsive-interactive-chart" src=""></iframe>
      </div>
    </div>

  </div>
</div>

<iframe name="hidden_iframe" id="hidden_iframe" style="display:none;"></iframe>

<script>
  const urlParams = new URLSearchParams(window.location.search);
  const rawFormUrl = urlParams.get('form');
  const rawChartUrl = urlParams.get('chart');

  const welcomeCard = document.getElementById('welcome-card');
  const mainDashboard = document.getElementById('main-dashboard');
  const formElement = document.getElementById('custom-weight-form');
  const inputElement = document.getElementById('weight-input');
  const btnElement = document.getElementById('submit-btn');
  const formStatus = document.getElementById('form-status');
  const chartIframe = document.getElementById('responsive-interactive-chart');
  const spinner = document.getElementById('refresh-spinner');
  const statusText = document.getElementById('status-text');
  
  const fallbackWrapper = document.getElementById('fallback-iframe-wrapper');
  const originalIframe = document.getElementById('original-google-iframe');
  const formCardWrapper = document.getElementById('form-card-wrapper');

  let userInteracted = false;
  let updateInterval = null;

  if (!rawFormUrl || !rawChartUrl) {
    welcomeCard.style.display = 'block';
  } else {
    mainDashboard.style.display = 'block';
    
    const cleanedFormUrl = rawFormUrl.split('?')[0].replace('/formResponse', '/viewform');
    let cleanedChartUrl = rawChartUrl;
    if (!cleanedChartUrl.includes('format=interactive')) {
      cleanedChartUrl = cleanedChartUrl.split('&format=')[0].split('?format=')[0] + '&format=interactive';
    }

    formElement.action = cleanedFormUrl.replace('/viewform', '/formResponse');
    chartIframe.src = cleanedChartUrl;

    // Сразу настраиваем резервный iframe на случай неудачи парсинга
    originalIframe.src = cleanedFormUrl + '?embedded=true';

    // Пробуем кастомизировать
    autoDiscoverFieldId(cleanedFormUrl);
  }

  async function autoDiscoverFieldId(targetFormUrl) {
    try {
      const proxyUrl = `https://corsproxy.io/?${encodeURIComponent(targetFormUrl)}`;
      const response = await fetch(proxyUrl);
      if(!response.ok) throw new Error("CORS Proxy failed");
      
      const htmlText = await response.text();
      const regex = /FB_PUBLIC_LOAD_DATA_\s*=\s*(.*?);/s;
      const match = htmlText.match(regex);

      if (match && match[1]) {
        const rawData = JSON.parse(match[1]);
        const firstFieldId = rawData[1][1][0][4][0][0];
        
        // Успех! Активируем кастомную форму
        inputElement.name = `entry.${firstFieldId}`;
        formElement.style.display = 'block';
        formStatus.innerText = "Система готова к быстрому вводу веса.";
        
        // Навешиваем обработчик на кастомную форму
        initCustomFormLogic();
      } else {
        throw new Error("Парсинг структуры не удался");
      }
    } catch (error) {
      console.warn("Кастомизация не удалась. Включаем бесшовный откат к оригиналу.");
      
      // Откат: прячем кастомную форму, показываем оригинальный фрейм Google
      formElement.style.display = 'none';
      fallbackWrapper.style.display = 'block';
      formStatus.innerHTML = "Используется стандартный интерфейс ввода.";
      
      // Запускаем отслеживание активности для оригинального фрейма (чтобы график всё равно обновлялся по таймеру)
      initFallbackTimerLogic();
    }
  }

  // СЦЕНАРИЙ А: Логика работы идеальной кастомной формы
  function initCustomFormLogic() {
    formElement.addEventListener('submit', function() {
      triggerChartRefresh(4000); // Обновляем через 4 секунды после клика "Сохранить"
      setTimeout(() => { inputElement.value = ''; }, 500);
    });
  }

  // СЦЕНАРИЙ Б: Логика для оригинального Iframe (запуск фонового обновления при тапе/клике)
  function initFallbackTimerLogic() {
    formCardWrapper.addEventListener('click', startSmartTimer);
    formCardWrapper.addEventListener('touchstart', startSmartTimer);

    window.addEventListener('blur', function() {
      if (document.activeElement.tagName === 'IFRAME' && document.activeElement.id === 'original-google-iframe') {
        startSmartTimer();
      }
    });
  }

  function startSmartTimer() {
    if (userInteracted) return;
    userInteracted = true;
    // В режиме iframe обновляем каждые 10 секунд после того, как пользователь нажал на форму
    updateInterval = setInterval(() => { triggerChartRefresh(0); }, 10000);
  }

  // Универсальный движок перезагрузки графика в обход кэша
  function triggerChartRefresh(delayMs) {
    setTimeout(function() {
      if (!chartIframe) return;
      spinner.style.display = 'block';
      statusText.innerText = 'Обновление...';
      chartIframe.style.opacity = '0.4';

      const originalSrc = chartIframe.src.split('&cacheBust=')[0];
      chartIframe.src = originalSrc + '&cacheBust=' + new Date().getTime();
      
      chartIframe.onload = function() {
        chartIframe.style.opacity = '1';
        spinner.style.display = 'none';
        statusText.innerText = 'Успешно обновлено';
        setTimeout(() => { statusText.innerText = 'Автообновление'; }, 3000);
      };
    }, delayMs);
  }

  // Генератор ссылок для главного экрана
  function generateDashboardLink() {
    const fUrl = document.getElementById('setup-form-url').value.trim();
    const cUrl = document.getElementById('setup-chart-url').value.trim();
    if(fUrl && cUrl) {
      window.location.href = `${window.location.origin}${window.location.pathname}?form=${encodeURIComponent(fUrl)}&chart=${encodeURIComponent(cUrl)}`;
    } else {
      alert("Пожалуйста, заполните обе ссылки!");
    }
  }
</script>