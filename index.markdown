---
layout: default
title: Автоматический ввод веса
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
    background-color: #f7fafc;
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

  /* Состояние блокировки до загрузки ID */
  .custom-input:disabled {
    background-color: #edf2f7;
    cursor: not-allowed;
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
  .custom-btn:disabled { background-color: #cbd5e0; cursor: not-allowed; }

  .chart-responsive-container { position: relative; width: 100%; height: 380px; overflow: hidden; border-radius: 8px; }
  .chart-responsive-container iframe { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none; transition: opacity 0.3s ease; }

  .refresh-badge { display: inline-flex; align-items: center; gap: 6px; background: #edf2f7; color: var(--text-muted); padding: 6px 12px; border-radius: 20px; font-size: 0.75rem; font-weight: 500; margin-bottom: 16px; }
  .spinner { width: 12px; height: 12px; border: 2px solid #cbd5e0; border-top-color: #4a5568; border-radius: 50%; animation: spin 1s linear infinite; display: none; }
  @keyframes spin { to { transform: rotate(360deg); } }

  @media (max-width: 480px) {
    .dashboard-container { padding: 12px 8px; }
    .ui-card { padding: 16px; margin-bottom: 16px; }
    .weight-form-group { flex-direction: column; gap: 10px; }
    .custom-btn { padding: 14px; width: 100%; }
    .chart-responsive-container { height: 250px; }
    .chart-responsive-container iframe { width: 150%; height: 150%; transform: scale(0.666); transform-origin: top left; }
  }
</style>

<div class="dashboard-container">

  <div class="ui-card">
    <h2>Ввести вес</h2>
    <p id="form-status">Синхронизация с Google Forms...</p>
    
    <form id="custom-weight-form" action="" method="POST" target="hidden_iframe">
      <div class="weight-form-group">
        <div class="input-wrapper">
          <input type="number" step="0.1" id="weight-input" name="" placeholder="Загрузка..." class="custom-input" disabled required>
          <span class="input-suffix">кг</span>
        </div>
        <button type="submit" id="submit-btn" class="custom-btn" disabled>Сохранить</button>
      </div>
    </form>
  </div>

  <div class="ui-card">
    <div style="display: flex; justify-content: space-between; align-items: flex-start;">
      <div>
        <h2>История изменений</h2>
        <p>График обновится автоматически через 4 секунды после сохранения.</p>
      </div>
      <div class="refresh-badge">
        <div class="spinner" id="refresh-spinner"></div>
        <span id="status-text">Автообновление</span>
      </div>
    </div>

    <div class="chart-responsive-container">
      <iframe id="responsive-interactive-chart" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vT5y16P0vfS6N0x70eDhHBIM-pPb1e0k--lfKurgEcWIBHtlv-KD9LyXVkP3RnLuOFnSEOSWE5AIb7N/pubchart?oid=24048480&format=interactive"></iframe>
    </div>
  </div>

</div>

<iframe name="hidden_iframe" id="hidden_iframe" style="display:none;"></iframe>

<script>
  // ⚠️ ЕСЛИ МЕНЯЕТЕ ФОРМУ — МЕНЯЙТЕ ТОЛЬКО ЭТУ ССЫЛКУ:
  const GOOGLE_FORM_URL = 'https://docs.google.com/forms/d/e/1FAIpQLSei-sxllN2AEFnCsEOc4huBwBXRNbnFq50oV5usLWZbdJT_Yg/viewform';

  const formElement = document.getElementById('custom-weight-form');
  const inputElement = document.getElementById('weight-input');
  const btnElement = document.getElementById('submit-btn');
  const formStatus = document.getElementById('form-status');
  const chartIframe = document.getElementById('responsive-interactive-chart');
  const spinner = document.getElementById('refresh-spinner');
  const statusText = document.getElementById('status-text');

  // Устанавливаем action для отправки данных формы
  formElement.action = GOOGLE_FORM_URL.replace('/viewform', '/formResponse');

  // ФУНКЦИЯ ПАРСИНГА ФОРМЫ НА СТОРОНЕ КЛИЕНТА
  async function autoDiscoverFieldId() {
    try {
      // Используем бесплатный прокси allorigins для обхода CORS ограничений Google
      const proxyUrl = `https://api.allorigins.win/get?url=${encodeURIComponent(GOOGLE_FORM_URL)}`;
      const response = await fetch(proxyUrl);
      const data = await response.json();
      const htmlText = data.contents;

      // Регулярное выражение ищет технический массив Google, где лежат ID полей
      const regex = /FB_PUBLIC_LOAD_DATA_\s*=\s*(.*?);/s;
      const match = htmlText.match(regex);

      if (match && match[1]) {
        // Парсим массив данных Google Формы
        const rawData = JSON.parse(match[1]);
        const formFields = rawData[1][1];
        
        // Берем ID самого первого поля в форме
        const firstFieldId = formFields[0][4][0][0];
        const entryId = `entry.${firstFieldId}`;

        // Настраиваем HTML-элементы найденным ID
        inputElement.name = entryId;
        inputElement.disabled = false;
        inputElement.placeholder = "0.0";
        btnElement.disabled = false;
        formStatus.innerText = "Система готова к вводу веса.";
        console.log(`Успешно обнаружен ID поля: ${entryId}`);
      } else {
        throw new Error("Не удалось прочитать структуру формы");
      }
    } catch (error) {
      console.error(error);
      formStatus.innerHTML = "<span style='color: #e53e3e;'>Ошибка синхронизации. Введите ID вручную.</span>";
    }
  }

  // Запускаем автопоиск ID сразу при загрузке страницы
  autoDiscoverFieldId();

  // Логика отправки формы и обновления графика (осталась прежней)
  formElement.addEventListener('submit', function() {
    spinner.style.display = 'block';
    statusText.innerText = 'Сохранение...';
    chartIframe.style.opacity = '0.4';

    setTimeout(function() {
      if (!chartIframe) return;
      statusText.innerText = 'Обновление графика...';
      
      const originalSrc = chartIframe.src.split('&cacheBust=')[0];
      chartIframe.src = originalSrc + '&cacheBust=' + new Date().getTime();
      
      chartIframe.onload = function() {
        chartIframe.style.opacity = '1';
        spinner.style.display = 'none';
        statusText.innerText = 'Успешно обновлено';
        inputElement.value = '';
        setTimeout(() => { statusText.innerText = 'Автообновление'; }, 3000);
      };
    }, 4000);
  });
</script>