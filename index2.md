<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
  <title>Plant Wheel</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    :root{
      /* 🎨 Твои тона (меняй эти значения под себя) */
      --bg1: #0f1a14;
      --bg2: #12251c;
      --card: rgba(255,255,255,.08);
      --card2: rgba(255,255,255,.06);
      --text: rgba(255,255,255,.92);
      --muted: rgba(255,255,255,.70);

      --accent: #7ee2b8;
      --accent2:#b7f7dc;

      --danger: #ff6b6b;
      --ok: #7ee2b8;

      --radius: 22px;
      --shadow: 0 18px 55px rgba(0,0,0,.30);

      --max-width: 460px;
      --wheel-size: min(90vw, 62svh); /* колесо крупнее и по центру */
      --gap: 12px;

      --border: rgba(255,255,255,.10);
      --border2: rgba(255,255,255,.14);
    }

    *{ box-sizing: border-box; }
    html, body{ height:100%; }
    body{
      margin:0;
      color: var(--text);
      font-family: system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif;
      background:
        radial-gradient(1000px 700px at 20% 0%, var(--bg2), var(--bg1));
    }

    /* Safe area для Telegram / iOS */
    .page{
      min-height: 100svh;
      display:flex;
      align-items:stretch;
      justify-content:center;
      padding:
        calc(12px + env(safe-area-inset-top))
        12px
        calc(12px + env(safe-area-inset-bottom))
        12px;
    }

    .shell{
      width: min(var(--max-width), 100%);
      display:flex;
      flex-direction:column;
      gap: var(--gap);
      padding-bottom: 74px; /* место под нижнюю навигацию */
    }

    .card{
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      padding: 14px;
      backdrop-filter: blur(10px);
    }

    .header{
      display:flex;
      align-items:flex-start;
      justify-content:space-between;
      gap: 10px;
      margin-bottom: 8px;
    }
    .h1{
      font-size: 18px;
      font-weight: 900;
      letter-spacing: .2px;
      margin:0;
    }
    .sub{
      margin: 4px 0 0 0;
      font-size: 13px;
      color: var(--muted);
    }

    .pill{
      display:inline-flex;
      gap: 8px;
      align-items:center;
      padding: 8px 10px;
      border-radius: 999px;
      background: rgba(0,0,0,.18);
      border: 1px solid var(--border);
      color: var(--muted);
      font-size: 12px;
      user-select:none;
      white-space:nowrap;
    }

    /* Tabs */
    .tab{ display:none; }
    .tab.active{ display:block; }

    /* Wheel */
    .stageWrap{
      display:flex;
      align-items:center;
      justify-content:center;
      padding: 8px 0 4px;
    }
    .stage{
      position: relative;
      width: var(--wheel-size);
      height: var(--wheel-size);
      margin: 6px auto 10px;
    }
    #wheelCanvas{
      width:100%;
      height:100%;
      display:block;
      border-radius: 50%;
      box-shadow: 0 10px 30px rgba(0,0,0,.25);
      border: 1px solid var(--border2);
      transition: transform 3.2s cubic-bezier(.1,.8,.2,1);
      background: rgba(0,0,0,.08);
    }
    .pointer{
      position:absolute;
      left:50%;
      top:-10px;
      transform: translateX(-50%);
      width:0; height:0;
      border-left: 14px solid transparent;
      border-right: 14px solid transparent;
      border-bottom: 26px solid var(--accent);
      filter: drop-shadow(0 6px 10px rgba(0,0,0,.35));
    }
    .centerDot{
      position:absolute;
      left:50%;
      top:50%;
      transform: translate(-50%,-50%);
      width: 22%;
      height: 22%;
      border-radius: 50%;
      background: radial-gradient(circle at 30% 30%, rgba(255,255,255,.20), rgba(0,0,0,.25));
      border: 1px solid var(--border);
      box-shadow: 0 10px 22px rgba(0,0,0,.25);
      pointer-events:none;
    }

    .btn{
      width:100%;
      padding: 14px 16px;
      border:0;
      border-radius: 16px;
      font-weight: 900;
      font-size: 16px;
      cursor:pointer;
      color:#0c1410;
      background: linear-gradient(135deg, var(--accent), var(--accent2));
      box-shadow: 0 12px 26px rgba(0,0,0,.22);
    }
    .btn:disabled{ opacity:.6; cursor:not-allowed; }
    .btn.secondary{
      background: rgba(255,255,255,.14);
      color: var(--text);
      border: 1px solid var(--border);
      box-shadow: none;
    }
    .row{
      display:flex;
      gap: 10px;
      align-items:center;
    }
    .row > * { flex: 1; }

    .result{
      margin-top: 12px;
      padding: 14px;
      border-radius: 16px;
      background: rgba(0,0,0,.20);
      border: 1px solid var(--border);
    }
    .resultTitle{
      font-weight: 900;
      font-size: 14px;
      color: var(--muted);
    }
    .resultAction{
      margin-top: 6px;
      font-weight: 950;
      font-size: 18px;
      letter-spacing: .2px;
    }
    .timer{
      font-variant-numeric: tabular-nums;
      font-weight: 950;
      font-size: 28px;
      margin-top: 8px;
    }

    details{
      margin-top: 12px;
      border-radius: 16px;
      background: rgba(255,255,255,.06);
      border: 1px solid var(--border);
      padding: 10px 12px;
    }
    summary{
      cursor:pointer;
      font-weight: 850;
      color: var(--text);
      list-style:none;
    }
    summary::-webkit-details-marker{ display:none; }
    .settingsHint{
      margin-top: 6px;
      font-size: 12px;
      color: var(--muted);
    }
    .checks{
      display:grid;
      gap: 10px;
      margin-top: 10px;
    }
    .checks label{
      display:flex;
      gap: 10px;
      align-items:flex-start;
      line-height: 1.2;
      color: var(--text);
    }
    .checks input{ margin-top: 2px; }

    /* Plants */
    .form{
      display:flex;
      flex-direction:column;
      gap: 10px;
    }
    .input{
      width:100%;
      padding: 12px 12px;
      border-radius: 14px;
      border: 1px solid var(--border);
      background: rgba(0,0,0,.18);
      color: var(--text);
      outline: none;
    }
    .input::placeholder{ color: rgba(255,255,255,.45); }
    .list{
      display:flex;
      flex-direction:column;
      gap: 10px;
      margin-top: 10px;
    }
    .item{
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap: 10px;
      padding: 12px 12px;
      border-radius: 16px;
      background: rgba(255,255,255,.06);
      border: 1px solid var(--border);
    }
    .itemName{
      font-weight: 900;
    }
    .itemMeta{
      margin-top: 2px;
      font-size: 12px;
      color: var(--muted);
    }
    .iconBtn{
      border: 1px solid var(--border);
      background: rgba(0,0,0,.16);
      color: var(--text);
      border-radius: 14px;
      padding: 10px 12px;
      cursor:pointer;
      font-weight: 850;
      user-select:none;
    }
    .iconBtn.danger{ border-color: rgba(255,107,107,.35); color: #ffd1d1; }

    /* Calendar */
    .calTop{
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap: 10px;
      margin-bottom: 10px;
    }
    .calTitle{
      font-weight: 950;
      font-size: 16px;
    }
    .calNav{
      display:flex;
      gap: 8px;
    }
    .calGrid{
      display:grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 8px;
      margin-top: 10px;
    }
    .dow{
      font-size: 11px;
      color: var(--muted);
      text-align:center;
      padding: 4px 0;
      user-select:none;
    }
    .day{
      border: 1px solid var(--border);
      background: rgba(0,0,0,.14);
      border-radius: 14px;
      padding: 8px 8px 6px;
      min-height: 58px;
      cursor:pointer;
      position:relative;
      overflow:hidden;
      user-select:none;
    }
    .day.muted{ opacity: .45; }
    .day.active{
      outline: 2px solid rgba(126,226,184,.45);
      border-color: rgba(126,226,184,.35);
    }
    .dayNum{
      font-weight: 950;
      font-size: 13px;
    }
    .markers{
      position:absolute;
      left: 8px;
      right: 8px;
      bottom: 6px;
      display:flex;
      gap: 6px;
      align-items:center;
      flex-wrap:wrap;
      font-size: 12px;
      color: var(--muted);
    }
    .badge{
      padding: 2px 8px;
      border-radius: 999px;
      border: 1px solid var(--border);
      background: rgba(255,255,255,.06);
      font-size: 11px;
      color: var(--muted);
    }

    .dayPanel{
      margin-top: 12px;
      padding: 12px;
      border-radius: 16px;
      border: 1px solid var(--border);
      background: rgba(255,255,255,.06);
    }
    .dayPanelTop{
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap: 10px;
      margin-bottom: 10px;
    }
    .dayPanelTitle{
      font-weight: 950;
    }

    .plantLog{
      padding: 12px;
      border-radius: 16px;
      border: 1px solid var(--border);
      background: rgba(0,0,0,.16);
      margin-top: 10px;
    }
    .plantLogName{
      font-weight: 950;
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap: 10px;
    }
    .moodRow{
      display:flex;
      gap: 8px;
      margin-top: 10px;
      flex-wrap:wrap;
    }
    .moodBtn{
      border: 1px solid var(--border);
      background: rgba(255,255,255,.08);
      border-radius: 14px;
      padding: 8px 10px;
      cursor:pointer;
      font-weight: 900;
      user-select:none;
    }
    .moodBtn.active{
      border-color: rgba(126,226,184,.45);
      box-shadow: 0 10px 18px rgba(0,0,0,.18);
    }
    .toggles{
      display:flex;
      gap: 10px;
      margin-top: 10px;
      flex-wrap:wrap;
    }
    .toggle{
      display:inline-flex;
      align-items:center;
      gap: 8px;
      padding: 10px 12px;
      border-radius: 14px;
      border: 1px solid var(--border);
      background: rgba(255,255,255,.06);
      cursor:pointer;
      user-select:none;
      font-weight: 850;
      color: var(--text);
    }
    .toggle input{ accent-color: var(--accent); }

    .note{
      margin-top: 10px;
      width:100%;
      min-height: 42px;
      resize: vertical;
    }

    /* Bottom nav */
    .bottomNav{
      position: fixed;
      left: 12px;
      right: 12px;
      bottom: calc(10px + env(safe-area-inset-bottom));
      margin: 0 auto;
      width: min(var(--max-width), calc(100% - 24px));
      display:flex;
      gap: 10px;
      padding: 10px;
      border-radius: 22px;
      background: rgba(0,0,0,.28);
      border: 1px solid var(--border);
      backdrop-filter: blur(10px);
      box-shadow: 0 18px 55px rgba(0,0,0,.35);
      z-index: 10;
    }
    .navBtn{
      flex:1;
      border: 1px solid var(--border);
      background: rgba(255,255,255,.06);
      color: var(--text);
      border-radius: 18px;
      padding: 10px 10px;
      cursor:pointer;
      font-weight: 950;
      font-size: 12px;
      display:flex;
      flex-direction:column;
      align-items:center;
      gap: 4px;
      user-select:none;
    }
    .navBtn.active{
      border-color: rgba(126,226,184,.45);
      background: rgba(126,226,184,.12);
    }
    .navIcon{ font-size: 16px; line-height: 1; }

    .small{
      font-size: 12px;
      color: var(--muted);
      margin-top: 8px;
    }

    @media (max-height: 640px){
      :root{ --wheel-size: min(92vw, 56svh); }
    }
  </style>
</head>

<body>
  <div class="page">
    <div class="shell">

      <!-- TAB: WHEEL -->
      <div class="tab card active" id="tab-wheel">
        <div class="header">
          <div>
            <h1 class="h1">Колесо ухода 🌿</h1>
            <p class="sub">Крути — и делай 2 минуты. Иногда выпадет “ничего не делай — ты молодец”.</p>
          </div>
          <div class="pill" id="savePill">💾 сохранено</div>
        </div>

        <div class="stageWrap">
          <div class="stage">
            <canvas id="wheelCanvas"></canvas>
            <div class="pointer" aria-hidden="true"></div>
            <div class="centerDot" aria-hidden="true"></div>
          </div>
        </div>

        <div class="row">
          <button class="btn" id="spinBtn">Крутить</button>
        </div>

        <div class="result" id="resultBox" style="display:none;">
          <div class="resultTitle">Выпало</div>
          <div class="resultAction" id="resultAction">—</div>
          <div class="timer" id="timerText">2:00</div>
          <div class="row" style="margin-top:10px;">
            <button class="btn secondary" id="doneBtn">Готово</button>
          </div>
        </div>

        <details>
          <summary>Настройки колеса: исключить действия</summary>
          <div class="settingsHint">Отметь, что не хочешь видеть. Сохраняется автоматически.</div>
          <div class="checks" id="excludeChecks"></div>
        </details>

        <div class="small">
          Подсказка: список растений и календарь — во вкладках снизу.
        </div>
      </div>

      <!-- TAB: PLANTS -->
      <div class="tab card" id="tab-plants">
        <div class="header">
          <div>
            <h1 class="h1">Мои растения 🪴</h1>
            <p class="sub">Добавь растения, чтобы отмечать уход в календаре.</p>
          </div>
          <div class="pill" id="plantsCountPill">0 шт.</div>
        </div>

        <div class="form">
          <input class="input" id="plantName" placeholder="Название (например, Монстера)" maxlength="40" />
          <input class="input" id="plantKind" placeholder="Вид/сорт (необязательно)" maxlength="60" />
          <button class="btn" id="addPlantBtn">Добавить растение</button>
        </div>

        <div class="list" id="plantsList"></div>
      </div>

      <!-- TAB: CALENDAR -->
      <div class="tab card" id="tab-calendar">
        <div class="header">
          <div>
            <h1 class="h1">Календарь ухода 📅</h1>
            <p class="sub">Выбирай день и отмечай: самочувствие смайликом + полив/обработка.</p>
          </div>
          <div class="pill" id="selectedDatePill">—</div>
        </div>

        <div class="calTop">
          <div class="calTitle" id="calTitle">—</div>
          <div class="calNav">
            <button class="iconBtn" id="prevMonthBtn">←</button>
            <button class="iconBtn" id="todayBtn">Сегодня</button>
            <button class="iconBtn" id="nextMonthBtn">→</button>
          </div>
        </div>

        <div class="calGrid" id="calDow"></div>
        <div class="calGrid" id="calGrid"></div>

        <div class="dayPanel" id="dayPanel" style="display:none;">
          <div class="dayPanelTop">
            <div class="dayPanelTitle" id="dayPanelTitle">—</div>
            <span class="badge" id="daySummaryBadge">—</span>
          </div>

          <div id="dayPlants"></div>

          <div class="small">
            Всё сохраняется автоматически. В Telegram синхронизируется через CloudStorage (если доступно).
          </div>
        </div>
      </div>

    </div>
  </div>

  <div class="bottomNav">
    <button class="navBtn active" data-tab="wheel">
      <div class="navIcon">🌀</div>
      <div>Колесо</div>
    </button>
    <button class="navBtn" data-tab="plants">
      <div class="navIcon">🪴</div>
      <div>Растения</div>
    </button>
    <button class="navBtn" data-tab="calendar">
      <div class="navIcon">📅</div>
      <div>Календарь</div>
    </button>
  </div>

  <script>
    (() => {
      // ===== Telegram init (не ломается вне Telegram) =====
      const tg = window.Telegram?.WebApp;
      try { tg?.ready?.(); } catch (_) {}

      // Подхватываем тему Telegram, если есть (мягко)
      function applyTelegramTheme(){
        if (!tg?.themeParams) return;
        const p = tg.themeParams;
        // Подстраиваемся аккуратно: только если значения есть
        const css = document.documentElement.style;
        if (p.bg_color) css.setProperty('--bg1', p.bg_color);
        if (p.secondary_bg_color) css.setProperty('--bg2', p.secondary_bg_color);
        if (p.text_color) css.setProperty('--text', p.text_color);
        if (p.hint_color) css.setProperty('--muted', p.hint_color);
        if (p.button_color) css.setProperty('--accent', p.button_color);
        if (p.button_text_color) css.setProperty('--accent2', p.button_text_color);
      }
      try { applyTelegramTheme(); tg?.onEvent?.('themeChanged', applyTelegramTheme); } catch(_) {}

      // ===== Storage: Telegram CloudStorage fallback to localStorage =====
      const STORAGE_KEY = "plantwheel_state_v1";

      const storage = {
        async get(key) {
          if (tg?.CloudStorage?.getItem) {
            return await new Promise((resolve) => {
              tg.CloudStorage.getItem(key, (err, val) => resolve(err ? null : val));
            });
          }
          return localStorage.getItem(key);
        },
        async set(key, val) {
          if (tg?.CloudStorage?.setItem) {
            return await new Promise((resolve) => {
              tg.CloudStorage.setItem(key, val, () => resolve());
            });
          }
          localStorage.setItem(key, val);
        }
      };

      // ===== App state =====
      const DEFAULT_ACTIONS = [
        { id:"water",   label:"Полей 2 минуты 💧", weight:4 },
        { id:"spray",   label:"Опрыскай 2 минуты 🌫️", weight:3 },
        { id:"wipe",    label:"Протри листья 2 минуты ✨", weight:2 },
        { id:"rotate",  label:"Поверни горшок к свету ☀️", weight:2 },
        { id:"check",   label:"Проверь грунт/влажность 🪴", weight:3 },
        { id:"nothing", label:"Ничего не делай — ты молодец 💚", weight:2 },
      ];

      const MOODS = ["😄","🙂","😐","😟","😵","🌿"]; // выбери, какие нравятся

      let state = {
        version: 1,
        actions: DEFAULT_ACTIONS,
        excludedActionIds: [],   // персонализация колеса
        plants: [],              // {id,name,kind,createdAt}
        logs: {},                // "YYYY-MM-DD": { plantId: { mood, water, treat, note } }
        ui: {                    // UI prefs
          calYear: null,
          calMonth: null,        // 0-11
          selectedDate: null     // "YYYY-MM-DD"
        }
      };

      const savePill = document.getElementById("savePill");
      let saveTimer = null;

      function showSavedPulse(){
        savePill.textContent = "💾 сохранено";
        savePill.style.opacity = "1";
        clearTimeout(saveTimer);
        saveTimer = setTimeout(() => { savePill.style.opacity = "0.6"; }, 900);
      }

      async function loadState(){
        const raw = await storage.get(STORAGE_KEY);
        if (raw) {
          try {
            const parsed = JSON.parse(raw);
            // мягкий мердж
            state = {
              ...state,
              ...parsed,
              actions: Array.isArray(parsed.actions) && parsed.actions.length ? parsed.actions : state.actions,
              excludedActionIds: Array.isArray(parsed.excludedActionIds) ? parsed.excludedActionIds : [],
              plants: Array.isArray(parsed.plants) ? parsed.plants : [],
              logs: parsed.logs && typeof parsed.logs === "object" ? parsed.logs : {},
              ui: { ...state.ui, ...(parsed.ui || {}) }
            };
          } catch (_) {}
        }
        // дефолт для календаря
        const now = new Date();
        if (state.ui.calYear == null) state.ui.calYear = now.getFullYear();
        if (state.ui.calMonth == null) state.ui.calMonth = now.getMonth();
        if (!state.ui.selectedDate) state.ui.selectedDate = toISODate(now);
      }

      async function saveState(){
        await storage.set(STORAGE_KEY, JSON.stringify(state));
        showSavedPulse();
      }

      function saveStateDebounced(){
        // чтобы не писать в storage на каждое нажатие без нужды
        clearTimeout(saveStateDebounced._t);
        saveStateDebounced._t = setTimeout(saveState, 250);
      }

      // ===== Utilities =====
      function uid(){
        return Math.random().toString(16).slice(2) + Date.now().toString(16);
      }

      function pad2(n){ return String(n).padStart(2,"0"); }
      function toISODate(d){
        return `${d.getFullYear()}-${pad2(d.getMonth()+1)}-${pad2(d.getDate())}`;
      }
      function fromISODate(s){
        const [y,m,dd] = s.split("-").map(Number);
        return new Date(y, m-1, dd);
      }
      function formatDateRu(iso){
        const d = fromISODate(iso);
        return d.toLocaleDateString("ru-RU", { year:"numeric", month:"long", day:"numeric" });
      }
      function monthTitleRu(year, monthIndex){
        const d = new Date(year, monthIndex, 1);
        return d.toLocaleDateString("ru-RU", { year:"numeric", month:"long" });
      }

      // ===== Tabs =====
      const tabs = {
        wheel: document.getElementById("tab-wheel"),
        plants: document.getElementById("tab-plants"),
        calendar: document.getElementById("tab-calendar"),
      };
      const navBtns = Array.from(document.querySelectorAll(".navBtn"));

      function setActiveTab(name){
        for (const k of Object.keys(tabs)) tabs[k].classList.toggle("active", k === name);
        navBtns.forEach(b => b.classList.toggle("active", b.dataset.tab === name));

        // перерисуем нужное
        if (name === "wheel") redrawWheel();
        if (name === "plants") renderPlants();
        if (name === "calendar") renderCalendar();
      }

      navBtns.forEach(btn => btn.addEventListener("click", () => setActiveTab(btn.dataset.tab)));

      // ===== Wheel (canvas + spin) =====
      const canvas = document.getElementById("wheelCanvas");
      const ctx = canvas.getContext("2d");
      const spinBtn = document.getElementById("spinBtn");
      const doneBtn = document.getElementById("doneBtn");
      const resultBox = document.getElementById("resultBox");
      const resultAction = document.getElementById("resultAction");
      const timerText = document.getElementById("timerText");
      const excludeChecks = document.getElementById("excludeChecks");

      let currentRotation = 0;
      let spinning = false;
      let timerId = null;

      function resizeCanvasToDisplaySize(){
        const rect = canvas.getBoundingClientRect();
        const dpr = window.devicePixelRatio || 1;
        const w = Math.max(1, Math.round(rect.width * dpr));
        const h = Math.max(1, Math.round(rect.height * dpr));
        if (canvas.width !== w || canvas.height !== h){
          canvas.width = w;
          canvas.height = h;
        }
      }

      function getAvailableActions(){
        const excluded = new Set(state.excludedActionIds || []);
        const available = (state.actions || DEFAULT_ACTIONS).filter(a => !excluded.has(a.id));
        return available.length ? available : (state.actions || DEFAULT_ACTIONS);
      }

      function drawWheel(items){
        resizeCanvasToDisplaySize();
        const W = canvas.width, H = canvas.height;
        const cx = W/2, cy = H/2;
        const r = Math.min(W,H)/2 - Math.max(10, Math.round(Math.min(W,H) * 0.03));

        ctx.clearRect(0,0,W,H);

        const n = items.length;
        const seg = (Math.PI * 2) / n;

        // фон-кольцо
        ctx.beginPath();
        ctx.arc(cx,cy,r+2,0,Math.PI*2);
        ctx.fillStyle = "rgba(0,0,0,.10)";
        ctx.fill();

        for (let i=0; i<n; i++){
          const start = -Math.PI/2 + i*seg;
          const end = start + seg;

          // сектора: чередуем + лёгкий градиентный эффект
          ctx.beginPath();
          ctx.moveTo(cx,cy);
          ctx.arc(cx,cy,r,start,end);
          ctx.closePath();

          const isEven = i%2===0;
          ctx.fillStyle = isEven ? "rgba(255,255,255,.12)" : "rgba(126,226,184,.12)";
          ctx.fill();

          // линия раздела
          ctx.strokeStyle = "rgba(255,255,255,.10)";
          ctx.lineWidth = Math.max(1, Math.round(Math.min(W,H) * 0.004));
          ctx.stroke();

          // текст
          ctx.save();
          ctx.translate(cx,cy);
          ctx.rotate(start + seg/2);

          const fontSize = Math.max(12, Math.round(Math.min(W,H) * 0.04));
          ctx.font = `900 ${fontSize}px system-ui, -apple-system, Segoe UI, Roboto, Arial`;
          ctx.textAlign = "right";
          ctx.textBaseline = "middle";
          ctx.fillStyle = "rgba(255,255,255,.92)";

          const label = items[i].label;
          const maxChars = 18;
          const text = label.length > maxChars ? label.slice(0,maxChars) + "…" : label;
          ctx.fillText(text, r - Math.round(r*0.12), 0);
          ctx.restore();
        }

        // обводка
        ctx.beginPath();
        ctx.arc(cx,cy,r,0,Math.PI*2);
        ctx.strokeStyle = "rgba(255,255,255,.14)";
        ctx.lineWidth = Math.max(2, Math.round(Math.min(W,H) * 0.02));
        ctx.stroke();
      }

      function redrawWheel(){
        const items = getAvailableActions();
        drawWheel(items);
        renderExcludeChecks();
      }

      function pickWeighted(items){
        const total = items.reduce((s,x)=>s+(x.weight||1),0);
        let rnd = Math.random() * total;
        for(const x of items){
          rnd -= (x.weight||1);
          if (rnd <= 0) return x;
        }
        return items[items.length-1];
      }

      function startTimer(seconds=120){
        clearInterval(timerId);
        let left = seconds;

        const tick = () => {
          const m = Math.floor(left/60);
          const s = left%60;
          timerText.textContent = `${m}:${String(s).padStart(2,"0")}`;
          if (left <= 0) {
            clearInterval(timerId);
            // маленький “готово”
            try { tg?.HapticFeedback?.notificationOccurred?.("success"); } catch(_) {}
          }
          left--;
        };
        tick();
        timerId = setInterval(tick, 1000);
      }

      function stopTimer(){
        clearInterval(timerId);
        timerId = null;
        timerText.textContent = "2:00";
      }

      function setResultVisible(visible){
        resultBox.style.display = visible ? "block" : "none";
      }

      function spin(){
        if (spinning) return;
        spinning = true;
        spinBtn.disabled = true;
        setResultVisible(false);
        stopTimer();

        const items = getAvailableActions();
        drawWheel(items);

        const picked = pickWeighted(items);
        const pickedIndex = items.findIndex(x => x.id === picked.id);

        const n = items.length;
        const segDeg = 360 / n;

        // цель: чтобы центр выбранного сегмента оказался под стрелкой (сверху)
        const spins = 5; // полных оборотов
        const target = (spins * 360) + (360 - (pickedIndex + 0.5) * segDeg);

        currentRotation += target;
        canvas.style.transform = `rotate(${currentRotation}deg)`;

        try { tg?.HapticFeedback?.impactOccurred?.("light"); } catch(_) {}

        setTimeout(() => {
          resultAction.textContent = picked.label;
          setResultVisible(true);
          startTimer(120);

          spinBtn.disabled = false;
          spinning = false;
        }, 3300);
      }

      spinBtn.addEventListener("click", spin);
      doneBtn.addEventListener("click", () => {
        setResultVisible(false);
        stopTimer();
      });

      // Exclusions UI
      function renderExcludeChecks(){
        excludeChecks.innerHTML = "";
        const excluded = new Set(state.excludedActionIds || []);
        const actions = state.actions || DEFAULT_ACTIONS;

        for (const a of actions){
          const row = document.createElement("label");
          row.innerHTML = `<input type="checkbox" /> <span>${escapeHtml(a.label)}</span>`;
          const input = row.querySelector("input");
          input.checked = excluded.has(a.id);

          input.addEventListener("change", () => {
            const set = new Set(state.excludedActionIds || []);
            if (input.checked) set.add(a.id);
            else set.delete(a.id);
            state.excludedActionIds = [...set];
            saveStateDebounced();
            redrawWheel();
          });

          excludeChecks.appendChild(row);
        }
      }

      // ===== Plants =====
      const plantName = document.getElementById("plantName");
      const plantKind = document.getElementById("plantKind");
      const addPlantBtn = document.getElementById("addPlantBtn");
      const plantsList = document.getElementById("plantsList");
      const plantsCountPill = document.getElementById("plantsCountPill");

      addPlantBtn.addEventListener("click", () => {
        const name = (plantName.value || "").trim();
        const kind = (plantKind.value || "").trim();
        if (!name) {
          try { tg?.HapticFeedback?.notificationOccurred?.("error"); } catch(_) {}
          plantName.focus();
          return;
        }
        state.plants.unshift({
          id: uid(),
          name,
          kind,
          createdAt: toISODate(new Date())
        });
        plantName.value = "";
        plantKind.value = "";
        saveStateDebounced();
        renderPlants();
        try { tg?.HapticFeedback?.notificationOccurred?.("success"); } catch(_) {}
      });

      function renderPlants(){
        plantsCountPill.textContent = `${state.plants.length} шт.`;
        plantsList.innerHTML = "";

        if (!state.plants.length){
          const empty = document.createElement("div");
          empty.className = "item";
          empty.innerHTML = `<div>
              <div class="itemName">Пока пусто</div>
              <div class="itemMeta">Добавь растения выше — и отмечай уход в календаре.</div>
            </div>`;
          plantsList.appendChild(empty);
          return;
        }

        for (const p of state.plants){
          const el = document.createElement("div");
          el.className = "item";
          el.innerHTML = `
            <div>
              <div class="itemName">${escapeHtml(p.name)}</div>
              <div class="itemMeta">${p.kind ? escapeHtml(p.kind) : "—"} · добавлено ${escapeHtml(p.createdAt || "")}</div>
            </div>
            <button class="iconBtn danger" title="Удалить">Удалить</button>
          `;
          el.querySelector("button").addEventListener("click", () => {
            // удаляем растение
            state.plants = state.plants.filter(x => x.id !== p.id);
            // чистим логи
            for (const dayKey of Object.keys(state.logs || {})){
              if (state.logs[dayKey] && state.logs[dayKey][p.id]) {
                delete state.logs[dayKey][p.id];
              }
            }
            saveStateDebounced();
            renderPlants();
            renderCalendar(); // обновить маркеры
          });
          plantsList.appendChild(el);
        }
      }

      // ===== Calendar =====
      const calTitle = document.getElementById("calTitle");
      const calDow = document.getElementById("calDow");
      const calGrid = document.getElementById("calGrid");
      const prevMonthBtn = document.getElementById("prevMonthBtn");
      const nextMonthBtn = document.getElementById("nextMonthBtn");
      const todayBtn = document.getElementById("todayBtn");
      const selectedDatePill = document.getElementById("selectedDatePill");

      const dayPanel = document.getElementById("dayPanel");
      const dayPanelTitle = document.getElementById("dayPanelTitle");
      const dayPlants = document.getElementById("dayPlants");
      const daySummaryBadge = document.getElementById("daySummaryBadge");

      const DOW = ["Пн","Вт","Ср","Чт","Пт","Сб","Вс"];

      prevMonthBtn.addEventListener("click", () => {
        let y = state.ui.calYear, m = state.ui.calMonth;
        m -= 1;
        if (m < 0){ m = 11; y -= 1; }
        state.ui.calYear = y; state.ui.calMonth = m;
        saveStateDebounced();
        renderCalendar();
      });

      nextMonthBtn.addEventListener("click", () => {
        let y = state.ui.calYear, m = state.ui.calMonth;
        m += 1;
        if (m > 11){ m = 0; y += 1; }
        state.ui.calYear = y; state.ui.calMonth = m;
        saveStateDebounced();
        renderCalendar();
      });

      todayBtn.addEventListener("click", () => {
        const now = new Date();
        state.ui.calYear = now.getFullYear();
        state.ui.calMonth = now.getMonth();
        state.ui.selectedDate = toISODate(now);
        saveStateDebounced();
        renderCalendar();
      });

      function getDayLog(iso, plantId){
        state.logs ||= {};
        state.logs[iso] ||= {};
        state.logs[iso][plantId] ||= { mood: "", water: false, treat: false, note: "" };
        return state.logs[iso][plantId];
      }

      function getDaySummary(iso){
        const day = state.logs?.[iso] || {};
        let waterCount = 0, treatCount = 0, moodCount = 0;
        for (const pid of Object.keys(day)){
          const x = day[pid];
          if (!x) continue;
          if (x.water) waterCount++;
          if (x.treat) treatCount++;
          if (x.mood) moodCount++;
        }
        return { waterCount, treatCount, moodCount };
      }

      function renderCalendar(){
        // DOW header
        calDow.innerHTML = "";
        for (const d of DOW){
          const el = document.createElement("div");
          el.className = "dow";
          el.textContent = d;
          calDow.appendChild(el);
        }

        const y = state.ui.calYear;
        const m = state.ui.calMonth;

        calTitle.textContent = monthTitleRu(y, m);

        // month grid (Mon-start)
        const first = new Date(y, m, 1);
        const firstDow = (first.getDay() + 6) % 7; // convert Sun(0) -> 6, Mon(1)->0
        const daysInMonth = new Date(y, m+1, 0).getDate();

        // previous month days shown
        const prevMonthDays = new Date(y, m, 0).getDate();

        const cells = [];
        for (let i=0; i<42; i++){
          const dayNum = i - firstDow + 1;
          let cellDate, inMonth = true;

          if (dayNum < 1){
            // prev month
            const d = prevMonthDays + dayNum;
            const pm = m - 1;
            if (pm < 0) cellDate = new Date(y-1, 11, d);
            else cellDate = new Date(y, pm, d);
            inMonth = false;
          } else if (dayNum > daysInMonth){
            // next month
            const d = dayNum - daysInMonth;
            const nm = m + 1;
            if (nm > 11) cellDate = new Date(y+1, 0, d);
            else cellDate = new Date(y, nm, d);
            inMonth = false;
          } else {
            cellDate = new Date(y, m, dayNum);
          }

          cells.push({ iso: toISODate(cellDate), day: cellDate.getDate(), inMonth });
        }

        const selectedIso = state.ui.selectedDate;
        selectedDatePill.textContent = selectedIso ? formatDateRu(selectedIso) : "—";

        calGrid.innerHTML = "";
        for (const c of cells){
          const el = document.createElement("div");
          el.className = "day" + (c.inMonth ? "" : " muted") + (c.iso === selectedIso ? " active" : "");
          el.innerHTML = `<div class="dayNum">${c.day}</div><div class="markers"></div>`;

          // markers (агрегированно по дню)
          const markers = el.querySelector(".markers");
          const sum = getDaySummary(c.iso);

          // маленький “муд” — если есть хоть один
          if (sum.moodCount > 0) markers.appendChild(markerChip("🙂"));
          if (sum.waterCount > 0) markers.appendChild(markerChip(`💧${sum.waterCount}`));
          if (sum.treatCount > 0) markers.appendChild(markerChip(`🧴${sum.treatCount}`));

          // если вообще ничего — оставим пусто
          el.addEventListener("click", () => {
            state.ui.selectedDate = c.iso;
            saveStateDebounced();
            renderCalendar();
          });

          calGrid.appendChild(el);
        }

        renderDayPanel();
      }

      function markerChip(text){
        const s = document.createElement("span");
        s.className = "badge";
        s.textContent = text;
        return s;
      }

      function renderDayPanel(){
        const iso = state.ui.selectedDate;
        if (!iso){
          dayPanel.style.display = "none";
          return;
        }

        dayPanel.style.display = "block";
        dayPanelTitle.textContent = formatDateRu(iso);

        if (!state.plants.length){
          dayPlants.innerHTML = `
            <div class="plantLog">
              <div class="plantLogName">Нет растений</div>
              <div class="small">Сначала добавь их во вкладке “Растения”.</div>
            </div>
          `;
          daySummaryBadge.textContent = "—";
          return;
        }

        const sum = getDaySummary(iso);
        daySummaryBadge.textContent = `💧 ${sum.waterCount} · 🧴 ${sum.treatCount} · 🙂 ${sum.moodCount}`;

        dayPlants.innerHTML = "";
        for (const p of state.plants){
          const log = getDayLog(iso, p.id);

          const box = document.createElement("div");
          box.className = "plantLog";

          box.innerHTML = `
            <div class="plantLogName">
              <span>${escapeHtml(p.name)}</span>
              <span class="badge">${escapeHtml(p.kind || "—")}</span>
            </div>

            <div class="moodRow"></div>

            <div class="toggles">
              <label class="toggle"><input type="checkbox" class="waterChk"> 💧 Полила</label>
              <label class="toggle"><input type="checkbox" class="treatChk"> 🧴 Обработала</label>
            </div>

            <textarea class="input note" placeholder="Заметка (необязательно)"></textarea>
          `;

          // Mood buttons
          const moodRow = box.querySelector(".moodRow");
          for (const m of MOODS){
            const b = document.createElement("button");
            b.type = "button";
            b.className = "moodBtn" + (log.mood === m ? " active" : "");
            b.textContent = m;
            b.addEventListener("click", () => {
              // toggle: same -> clear
              const next = (log.mood === m) ? "" : m;
              log.mood = next;
              // repaint active
              moodRow.querySelectorAll(".moodBtn").forEach(x => x.classList.toggle("active", x.textContent === next));
              saveStateDebounced();
              renderCalendar(); // обновит маркеры и summary
            });
            moodRow.appendChild(b);
          }

          // Toggles
          const waterChk = box.querySelector(".waterChk");
          const treatChk = box.querySelector(".treatChk");
          waterChk.checked = !!log.water;
          treatChk.checked = !!log.treat;

          waterChk.addEventListener("change", () => {
            log.water = waterChk.checked;
            saveStateDebounced();
            renderCalendar();
          });
          treatChk.addEventListener("change", () => {
            log.treat = treatChk.checked;
            saveStateDebounced();
            renderCalendar();
          });

          // Note
          const note = box.querySelector("textarea");
          note.value = log.note || "";
          note.addEventListener("input", () => {
            log.note = note.value.slice(0, 600);
            saveStateDebounced();
          });

          dayPlants.appendChild(box);
        }
      }

      // ===== Helpers =====
      function escapeHtml(s){
        return String(s)
          .replaceAll("&","&amp;")
          .replaceAll("<","&lt;")
          .replaceAll(">","&gt;")
          .replaceAll('"',"&quot;")
          .replaceAll("'","&#039;");
      }

      // ===== Boot =====
      async function boot(){
        await loadState();

        // initial UI
        renderPlants();
        renderCalendar();
        redrawWheel();

        // Resize wheel on viewport changes
        window.addEventListener("resize", () => {
          if (tabs.wheel.classList.contains("active")) redrawWheel();
        });

        // initial save pill fade
        savePill.style.opacity = "0.85";
      }

      boot();
    })();
  </script>
</body>
</html>
