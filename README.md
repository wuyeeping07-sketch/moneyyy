<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>奶油黃累積預算日曆記帳 App</title>
  <style>
    :root {
      --bg-color: #fefce8;
      --card-bg: #ffffff;
      --header-bg: #fef08a;
      --primary-color: #d97706;
      --text-main: #451a03;
      --text-muted: #78350f;
      --border-color: #fde68a;
      --green-bg: #dcfce7;
      --green-text: #15803d;
      --red-bg: #fee2e2;
      --red-text: #b91c1c;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; -webkit-tap-highlight-color: transparent; }
    body { background-color: var(--bg-color); color: var(--text-main); padding-bottom: calc(24px + env(safe-area-inset-bottom)); }

    header { background: var(--header-bg); padding: 14px 16px; border-bottom: 1px solid var(--border-color); position: sticky; top: 0; z-index: 10; box-shadow: 0 2px 6px rgba(180, 83, 9, 0.08); padding-top: calc(14px + env(safe-area-inset-top)); }
    .month-picker { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
    .month-picker h2 { font-size: 1.15rem; font-weight: 700; color: var(--text-main); }
    .btn-icon { background: #ffffff; border: 1px solid var(--border-color); border-radius: 8px; padding: 8px 16px; cursor: pointer; font-size: 1rem; font-weight: bold; color: var(--text-main); }

    .summary-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 8px; background: #fffbebf5; padding: 10px; border-radius: 10px; text-align: center; border: 1px solid var(--border-color); }
    .summary-item .label { font-size: 0.7rem; color: var(--text-muted); }
    .summary-item .value { font-size: 0.95rem; font-weight: 700; margin-top: 2px; }

    .header-actions { display: flex; justify-content: space-between; align-items: center; margin-top: 10px; }
    .btn-secondary { background: #ffffff; color: var(--primary-color); border: 1px solid var(--primary-color); border-radius: 6px; padding: 6px 12px; font-size: 0.8rem; font-weight: 600; cursor: pointer; }

    .calendar-container { padding: 8px; }
    .weekdays { display: grid; grid-template-columns: repeat(7, 1fr); text-align: center; font-size: 0.75rem; font-weight: 700; color: var(--text-muted); margin-bottom: 6px; }
    .days-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 3px; }
    .day-cell { background: var(--card-bg); border: 1px solid var(--border-color); border-radius: 8px; min-height: 85px; padding: 3px; display: flex; flex-direction: column; justify-content: space-between; cursor: pointer; transition: background 0.1s; overflow: hidden; }
    .day-cell:active { background: #fef08a; }
    .day-cell.empty { background: transparent; border: none; cursor: default; }
    .day-number { font-size: 0.75rem; font-weight: 700; color: var(--text-main); }
    
    .day-items-list { display: flex; flex-direction: column; gap: 2px; margin-top: 2px; }
    .day-item-tag { font-size: 0.58rem; padding: 1px 3px; border-radius: 3px; color: #ffffff; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 100%; display: block; }
    
    .amount-info { font-size: 0.6rem; text-align: right; margin-top: 2px; }
    .spent-text { color: var(--red-text); font-weight: 600; font-size: 0.6rem; }
    .avail-text { font-size: 0.55rem; color: var(--text-muted); }
    .remain-tag { padding: 1px 3px; border-radius: 4px; font-weight: 700; margin-top: 1px; display: inline-block; font-size: 0.58rem; }
    .remain-tag.safe { background: var(--green-bg); color: var(--green-text); }
    .remain-tag.over { background: var(--red-bg); color: var(--red-text); }

    .cat-summary-section { margin: 16px 8px 24px; background: var(--card-bg); border: 1px solid var(--border-color); border-radius: 12px; padding: 12px; }
    .cat-summary-section h3 { font-size: 0.9rem; font-weight: 700; color: var(--text-main); margin-bottom: 12px; display: flex; align-items: center; justify-content: space-between; }
    .cat-progress-card { margin-bottom: 10px; }
    .cat-progress-header { display: flex; justify-content: space-between; align-items: center; font-size: 0.78rem; margin-bottom: 4px; }
    .cat-progress-bar-bg { background: var(--bg-color); height: 8px; border-radius: 4px; overflow: hidden; border: 1px solid var(--border-color); }
    .cat-progress-bar-fill { height: 100%; border-radius: 4px; transition: width 0.3s ease; }

    .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(69, 26, 3, 0.4); justify-content: center; align-items: flex-end; z-index: 100; }
    .modal.active { display: flex; }
    .modal-content { background: var(--card-bg); width: 100%; max-width: 500px; border-radius: 16px 16px 0 0; padding: 18px; max-height: 85vh; overflow-y: auto; border-top: 3px solid var(--header-bg); padding-bottom: calc(18px + env(safe-area-inset-bottom)); }
    .modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 14px; }
    .close-btn { background: none; border: none; font-size: 1.5rem; cursor: pointer; color: var(--text-muted); padding: 4px 8px; }

    .form-group { margin-bottom: 12px; }
    .form-group label { display: block; font-size: 0.8rem; color: var(--text-muted); margin-bottom: 4px; }
    /* 防止 iOS 自動放大頁面 (font-size >= 16px) */
    .form-group input, .form-group select { width: 100%; padding: 10px 12px; border: 1px solid var(--border-color); border-radius: 8px; font-size: 16px; background: var(--bg-color); color: var(--text-main); }
    .btn-submit { width: 100%; padding: 12px; background: var(--primary-color); color: white; border: none; border-radius: 8px; font-size: 1rem; font-weight: 600; cursor: pointer; }

    .cat-pill { display: inline-flex; align-items: center; gap: 4px; padding: 2px 6px; border-radius: 12px; font-size: 0.75rem; color: #fff; font-weight: 600; }

    .expense-list { margin-top: 14px; border-top: 1px solid var(--border-color); padding-top: 10px; }
    .expense-item { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px dashed var(--border-color); font-size: 0.85rem; }
    .btn-delete { background: none; border: none; color: var(--red-text); font-size: 0.8rem; cursor: pointer; margin-left: 8px; padding: 4px; }

    .cat-budget-row { display: flex; justify-content: space-between; align-items: center; padding: 8px 0; border-bottom: 1px solid var(--border-color); }
    .cat-budget-row input { width: 100px; padding: 6px; border: 1px solid var(--border-color); border-radius: 6px; text-align: right; font-size: 16px; }
  </style>
</head>
<body>

  <header>
    <div class="month-picker">
      <button class="btn-icon" onclick="changeMonth(-1)">&lt;</button>
      <h2 id="currentMonthTitle">2026 年 8 月</h2>
      <button class="btn-icon" onclick="changeMonth(1)">&gt;</button>
    </div>

    <div class="summary-grid">
      <div class="summary-item">
        <div class="label">本月總預算</div>
        <div class="value" id="dispTotalBudget">$0</div>
      </div>
      <div class="summary-item">
        <div class="label">本月已消費</div>
        <div class="value" style="color: var(--red-text);" id="dispTotalSpent">$0</div>
      </div>
      <div class="summary-item">
        <div class="label">本月總結餘</div>
        <div class="value" style="color: var(--green-text);" id="dispTotalRemain">$0</div>
      </div>
    </div>

    <div class="header-actions">
      <span style="font-size:0.7rem; color:var(--text-muted);">✨ 點選日期可記帳</span>
      <button class="btn-secondary" onclick="openBudgetModal()">⚙️ 設定預算</button>
    </div>
  </header>

  <main class="calendar-container">
    <div class="weekdays">
      <div>日</div><div>一</div><div>二</div><div>三</div><div>四</div><div>五</div><div>六</div>
    </div>
    <div class="days-grid" id="daysGrid"></div>
  </main>

  <section class="cat-summary-section">
    <h3>
      <span>📊 各類別預算及使用量</span>
      <span style="font-size: 0.7rem; font-weight: normal; color: var(--text-muted);">（本月累計）</span>
    </h3>
    <div id="catSummaryList"></div>
  </section>

  <div class="modal" id="budgetModal">
    <div class="modal-content">
      <div class="modal-header">
        <h3>設定本月類別預算</h3>
        <button class="close-btn" onclick="closeBudgetModal()">&times;</button>
      </div>
      <div id="catBudgetInputs"></div>
      <button class="btn-submit" style="margin-top:14px;" onclick="saveCatBudgets()">儲存預算設定</button>
    </div>
  </div>

  <div class="modal" id="dayModal">
    <div class="modal-content">
      <div class="modal-header">
        <h3 id="modalDateTitle">2026-08-05</h3>
        <button class="close-btn" onclick="closeModal()">&times;</button>
      </div>

      <form onsubmit="addExpense(event)">
        <div class="form-group">
          <label>金額 ($)</label>
          <input type="number" id="expAmount" step="any" inputmode="decimal" placeholder="例如: 150" required>
        </div>
        <div class="form-group">
          <label>類別</label>
          <select id="expCategory"></select>
        </div>
        <div class="form-group">
          <label>備註/名稱（選填）</label>
          <input type="text" id="expNote" placeholder="例如: 鼓棒 / 午餐">
        </div>
        <button type="submit" class="btn-submit">新增消費記錄</button>
      </form>

      <div class="expense-list">
        <h4>當日消費明細</h4>
        <div id="expenseItemsContainer" style="margin-top: 8px;"></div>
      </div>
    </div>
  </div>

  <script>
    const CATEGORIES = {
      "飲食": "#f97316",
      "交通": "#06b6d4",
      "額外消費": "#64748b",
      "玩樂": "#a855f7",
      "禮物": "#ec4899",
      "電子產品": "#0284c7",
      "旅行": "#f59e0b",
      "日用": "#10b981",
      "打鼓": "#ef4444",
      "興趣班": "#6366f1"
    };

    let currentDate = new Date();
    let selectedDateStr = "";

    let db = JSON.parse(localStorage.getItem('accumulative_budget_data')) || {
      catBudgets: {
        "飲食": 4500, "交通": 1200, "額外消費": 900, "玩樂": 1500,
        "禮物": 600, "電子產品": 1200, "旅行": 1200, "日用": 1200,
        "打鼓": 900, "興趣班": 1200
      },
      expenses: []
    };

    function saveData() {
      try {
        localStorage.setItem('accumulative_budget_data', JSON.stringify(db));
      } catch (e) {
        console.warn("無法儲存資料至 localStorage");
      }
    }

    function getMonthKey(date) {
      const y = date.getFullYear();
      const m = String(date.getMonth() + 1).padStart(2, '0');
      return `${y}-${m}`;
    }

    function getTotalMonthlyBudget() {
      return Object.values(db.catBudgets).reduce((a, b) => a + (parseFloat(b) || 0), 0);
    }

    function changeMonth(delta) {
      currentDate.setMonth(currentDate.getMonth() + delta);
      renderApp();
    }

    function renderApp() {
      const year = currentDate.getFullYear();
      const month = currentDate.getMonth();
      const monthKey = getMonthKey(currentDate);

      document.getElementById('currentMonthTitle').innerText = `${year} 年 ${month + 1} 月`;
      
      const totalBudget = getTotalMonthlyBudget();
      const monthExpenses = db.expenses.filter(e => e.date && e.date.startsWith(monthKey));
      const totalSpent = monthExpenses.reduce((sum, e) => sum + e.amount, 0);
      const totalRemain = totalBudget - totalSpent;

      document.getElementById('dispTotalBudget').innerText = `$${totalBudget.toLocaleString()}`;
      document.getElementById('dispTotalSpent').innerText = `$${Math.round(totalSpent).toLocaleString()}`;
      document.getElementById('dispTotalRemain').innerText = `$${Math.round(totalRemain).toLocaleString()}`;

      const daysGrid = document.getElementById('daysGrid');
      daysGrid.innerHTML = '';

      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();

      for (let i = 0; i < firstDay; i++) {
        const emptyCell = document.createElement('div');
        emptyCell.className = 'day-cell empty';
        daysGrid.appendChild(emptyCell);
      }

      const baseDailyAllowance = totalBudget / daysInMonth;
      let previousBalance = 0;

      for (let day = 1; day <= daysInMonth; day++) {
        const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
        const dayExpenses = monthExpenses.filter(e => e.date === dateStr);
        const daySpent = dayExpenses.reduce((sum, e) => sum + e.amount, 0);

        const availableBudget = baseDailyAllowance + previousBalance;
        const netRemain = availableBudget - daySpent;
        previousBalance = netRemain;

        const cell = document.createElement('div');
        cell.className = 'day-cell';
        cell.onclick = () => openModal(dateStr);

        let itemsHtml = '';
        if (dayExpenses.length > 0) {
          itemsHtml += `<div class="day-items-list">`;
          dayExpenses.slice(0, 2).forEach(exp => {
            const color = CATEGORIES[exp.category] || '#64748b';
            const displayName = exp.note && exp.note.trim() ? exp.note : exp.category;
            itemsHtml += `<span class="day-item-tag" style="background-color: ${color};">${displayName}</span>`;
          });
          if (dayExpenses.length > 2) {
            itemsHtml += `<span style="font-size:0.5rem; color:var(--text-muted);">+${dayExpenses.length - 2}筆</span>`;
          }
          itemsHtml += `</div>`;
        }

        const isOver = netRemain < 0;
        const remainClass = isOver ? 'over' : 'safe';

        cell.innerHTML = `
          <div>
            <div class="day-number">${day}</div>
            ${itemsHtml}
          </div>
          <div class="amount-info">
            <div class="avail-text">可用:$${Math.round(availableBudget)}</div>
            ${daySpent > 0 ? `<div class="spent-text">-${Math.round(daySpent)}</div>` : ''}
            <div class="remain-tag ${remainClass}">
              結餘:$${Math.round(netRemain)}
            </div>
          </div>
        `;
        daysGrid.appendChild(cell);
      }

      renderCategorySummary(monthExpenses);
    }

    function renderCategorySummary(monthExpenses) {
      const container = document.getElementById('catSummaryList');
      container.innerHTML = '';

      Object.keys(CATEGORIES).forEach(cat => {
        const budget = db.catBudgets[cat] || 0;
        const catSpent = monthExpenses.filter(e => e.category === cat).reduce((sum, e) => sum + e.amount, 0);
        const color = CATEGORIES[cat];
        const pct = budget > 0 ? Math.min(100, Math.round((catSpent / budget) * 100)) : (catSpent > 0 ? 100 : 0);
        const isOver = catSpent > budget;

        const card = document.createElement('div');
        card.className = 'cat-progress-card';
        card.innerHTML = `
          <div class="cat-progress-header">
            <div>
              <span class="cat-pill" style="background:${color};">${cat}</span>
            </div>
            <div>
              <span style="font-weight:700; ${isOver ? 'color:var(--red-text);' : ''}">$${Math.round(catSpent).toLocaleString()}</span>
              <span style="color:var(--text-muted);"> / $${budget.toLocaleString()} (${pct}%)</span>
            </div>
          </div>
          <div class="cat-progress-bar-bg">
            <div class="cat-progress-bar-fill" style="width: ${pct}%; background-color: ${isOver ? 'var(--red-text)' : color};"></div>
          </div>
        `;
        container.appendChild(card);
      });
    }

    function initCategorySelect() {
      const select = document.getElementById('expCategory');
      select.innerHTML = '';
      Object.keys(CATEGORIES).forEach(cat => {
        const opt = document.createElement('option');
        opt.value = cat;
        opt.innerText = cat;
        select.appendChild(opt);
      });
    }

    function openBudgetModal() {
      const container = document.getElementById('catBudgetInputs');
      container.innerHTML = '';
      Object.keys(CATEGORIES).forEach(cat => {
        const val = db.catBudgets[cat] || 0;
        const color = CATEGORIES[cat];
        const row = document.createElement('div');
        row.className = 'cat-budget-row';
        row.innerHTML = `
          <span class="cat-pill" style="background:${color};">${cat}</span>
          <div>$<input type="number" inputmode="decimal" data-cat="${cat}" value="${val}"></div>
        `;
        container.appendChild(row);
      });
      document.getElementById('budgetModal').classList.add('active');
    }

    function closeBudgetModal() {
      document.getElementById('budgetModal').classList.remove('active');
    }

    function saveCatBudgets() {
      const inputs = document.querySelectorAll('#catBudgetInputs input');
      inputs.forEach(input => {
        const cat = input.getAttribute('data-cat');
        db.catBudgets[cat] = parseFloat(input.value) || 0;
      });
      saveData();
      closeBudgetModal();
      renderApp();
    }

    function openModal(dateStr) {
      selectedDateStr = dateStr;
      document.getElementById('modalDateTitle').innerText = dateStr;
      document.getElementById('dayModal').classList.add('active');
      renderDayExpenses();
    }

    function closeModal() {
      document.getElementById('dayModal').classList.remove('active');
    }

    function addExpense(e) {
      e.preventDefault();
      const amount = parseFloat(document.getElementById('expAmount').value);
      const category = document.getElementById('expCategory').value;
      const note = document.getElementById('expNote').value;

      if (!amount) return;

      db.expenses.push({
        id: 'exp_' + Date.now(),
        date: selectedDateStr,
        amount: amount,
        category: category,
        note: note
      });

      saveData();
      document.getElementById('expAmount').value = '';
      document.getElementById('expNote').value = '';

      renderDayExpenses();
      renderApp();
    }

    function deleteExpense(id) {
      db.expenses = db.expenses.filter(e => e.id !== id);
      saveData();
      renderDayExpenses();
      renderApp();
    }

    function renderDayExpenses() {
      const container = document.getElementById('expenseItemsContainer');
      container.innerHTML = '';
      const dayExpenses = db.expenses.filter(e => e.date === selectedDateStr);

      if (dayExpenses.length === 0) {
        container.innerHTML = '<div style="color:var(--text-muted); font-size:0.85rem;">當天無消費記錄</div>';
        return;
      }

      dayExpenses.forEach(item => {
        const color = CATEGORIES[item.category] || '#64748b';
        const div = document.createElement('div');
        div.className = 'expense-item';
        div.innerHTML = `
          <div>
            <span class="cat-pill" style="background:${color};">${item.category}</span>
            <span style="margin-left:6px;">${item.note || ''}</span>
          </div>
          <div>
            <span style="color:var(--red-text); font-weight:600;">-$${item.amount}</span>
            <button class="btn-delete" onclick="deleteExpense('${item.id}')">刪除</button>
          </div>
        `;
        container.appendChild(div);
      });
    }

    initCategorySelect();
    renderApp();
  </script>
</body>
</html>
