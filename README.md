<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Age Calculator</title>
  <style>
    :root {
      --bg: #0b1020;
      --card: #121a35;
      --muted: #9fb0ff;
      --text: #e9edff;
      --accent: #7aa2ff;
      --bad: #ff6b6b;
      --good: #6bff95;
    }
    html, body {
      height: 100%;
      margin: 0;
      background: radial-gradient(1200px 600px at 70% -20%, #1a2658 0%, #0b1020 60%);
      color: var(--text);
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    }
    .wrapper {
      min-height: 100%;
      display: grid;
      place-items: center;
      padding: 24px;
    }
    .card {
      width: min(680px, 92vw);
      background: linear-gradient(180deg, rgba(255,255,255,0.04), rgba(255,255,255,0.02));
      border: 1px solid rgba(255,255,255,0.12);
      border-radius: 18px;
      padding: 24px;
      box-shadow: 0 12px 40px rgba(0,0,0,0.35);
      backdrop-filter: blur(6px);
    }
    h1 {
      margin: 0 0 12px;
      font-size: clamp(1.4rem, 1rem + 2vw, 2rem);
      letter-spacing: 0.2px;
    }
    p.sub {
      margin: 0 0 20px;
      color: var(--muted);
    }
    .row {
      display: grid;
      grid-template-columns: 1fr auto;
      gap: 12px;
      align-items: end;
      margin: 18px 0 8px;
    }
    label {
      display: block;
      font-size: 0.95rem;
      margin-bottom: 6px;
      color: var(--muted);
    }
    input[type="date"] {
      width: 100%;
      appearance: none;
      background: var(--card);
      color: var(--text);
      border: 1px solid rgba(255,255,255,0.16);
      border-radius: 12px;
      padding: 12px 14px;
      font-size: 1rem;
      outline: none;
      transition: border .15s ease, box-shadow .15s ease;
    }
    input[type="date"]:focus {
      border-color: var(--accent);
      box-shadow: 0 0 0 4px rgba(122,162,255,0.18);
    }
    .buttons {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
    }
    button {
      border: 1px solid rgba(255,255,255,0.16);
      background: var(--card);
      color: var(--text);
      padding: 10px 14px;
      border-radius: 12px;
      font-size: 0.95rem;
      cursor: pointer;
      transition: transform .06s ease, border .15s ease, background .15s ease;
    }
    button.primary {
      border-color: rgba(122,162,255,0.65);
      background: linear-gradient(180deg, #1d2b5c, #162249);
    }
    button:hover { transform: translateY(-1px); }
    .error {
      color: var(--bad);
      margin: 6px 0 0;
      min-height: 1.2em;
    }
    .grid {
      margin-top: 18px;
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
      gap: 12px;
    }
    .tile {
      background: rgba(255,255,255,0.06);
      border: 1px solid rgba(255,255,255,0.12);
      border-radius: 14px;
      padding: 14px;
    }
    .tile h3 {
      margin: 0;
      font-size: 0.95rem;
      color: var(--muted);
    }
    .tile p {
      margin: 6px 0 0;
      font-weight: 700;
      font-size: 1.4rem;
      letter-spacing: .3px;
    }
    .muted-line {
      margin-top: 12px;
      color: var(--muted);
      font-size: 0.95rem;
    }
    footer {
      margin-top: 18px;
      font-size: 0.85rem;
      color: var(--muted);
    }
    .sr-only {
      position: absolute !important;
      width: 1px; height: 1px;
      padding: 0; margin: -1px; overflow: hidden; clip: rect(0,0,0,0);
      border: 0;
    }
  </style>
</head>
<body>
  <main class="wrapper">
    <section class="card" role="region" aria-labelledby="title">
      <h1 id="title">Age Calculator</h1>
      <p class="sub">Enter your date of birth to compute your exact age in years, months, and days.</p>

      <div class="row">
        <div>
          <label for="dob">Date of birth</label>
          <input id="dob" name="dob" type="date" required />
          <div id="error" class="error" aria-live="polite"></div>
        </div>
        <div class="buttons">
          <button class="primary" id="calcBtn" type="button">Calculate</button>
          <button id="clearBtn" type="button">Clear</button>
          <button id="todayBtn" type="button" title="Quick fill today">Today</button>
        </div>
      </div>

      <div class="grid" aria-live="polite" aria-atomic="true">
        <div class="tile">
          <h3>Age (years)</h3>
          <p id="ageYears">—</p>
        </div>
        <div class="tile">
          <h3>Months</h3>
          <p id="ageMonths">—</p>
        </div>
        <div class="tile">
          <h3>Days</h3>
          <p id="ageDays">—</p>
        </div>
        <div class="tile">
          <h3>Days to next birthday</h3>
          <p id="nextBday">—</p>
        </div>
      </div>

      <p class="muted-line" id="summary" aria-live="polite"></p>

      <footer>
        Tip: The calculation uses UTC dates to avoid timezone off-by-one issues.
      </footer>
    </section>
  </main>

  <script>
    // Utility: construct a "UTC midnight" Date from yyyy-mm-dd (or Date parts)
    function dateUTC(y, m, d) {
      return new Date(Date.UTC(y, m - 1, d));
    }
    // Parse yyyy-mm-dd into a UTC Date (midnight)
    function parseLocalDateStringToUTC(str) {
      const [y, m, d] = str.split("-").map(Number);
      return dateUTC(y, m, d);
    }
    // Get today's date at UTC midnight
    function todayUTC() {
      const now = new Date();
      return dateUTC(now.getUTCFullYear(), now.getUTCMonth() + 1, now.getUTCDate());
    }
    // Days between two UTC-midnight dates
    function daysBetweenUTC(a, b) {
      const MS = 24 * 60 * 60 * 1000;
      return Math.round((b - a) / MS);
    }

    // Compute Y/M/D age precisely by "stepping" years then months, then days.
    function diffYMD(fromUTC, toUTC) {
      if (toUTC < fromUTC) return { y: 0, m: 0, d: 0 };

      // years
      let y = toUTC.getUTCFullYear() - fromUTC.getUTCFullYear();
      let annivY = dateUTC(fromUTC.getUTCFullYear() + y, fromUTC.getUTCMonth() + 1, fromUTC.getUTCDate());
      if (annivY > toUTC) {
        y -= 1;
        annivY = dateUTC(fromUTC.getUTCFullYear() + y, fromUTC.getUTCMonth() + 1, fromUTC.getUTCDate());
      }

      // months
      let m = (toUTC.getUTCFullYear() - annivY.getUTCFullYear()) * 12 +
              (toUTC.getUTCMonth() - annivY.getUTCMonth());
      let annivM = dateUTC(annivY.getUTCFullYear(), annivY.getUTCMonth() + 1 + m, fromUTC.getUTCDate());
      if (annivM > toUTC) {
        m -= 1;
        annivM = dateUTC(annivY.getUTCFullYear(), annivY.getUTCMonth() + 1 + m, fromUTC.getUTCDate());
      }

      // days
      let d = daysBetweenUTC(annivM, toUTC);
      return { y, m, d };
    }

    function nextBirthday(fromUTC, todayUTCDate) {
      let y = todayUTCDate.getUTCFullYear();
      // Handle Feb 29 birthdays: next valid anniversary
      const isFeb29 = (fromUTC.getUTCMonth() === 1 && fromUTC.getUTCDate() === 29);
      let next = isFeb29 ? dateUTC(y, 3, 1) : dateUTC(y, fromUTC.getUTCMonth() + 1, fromUTC.getUTCDate());
      if (next < todayUTCDate || (isFeb29 && !isLeap(y) && next.getUTCMonth() === 2 && next.getUTCDate() === 1)) {
        y += 1;
        next = isFeb29 ? (isLeap(y) ? dateUTC(y, 2, 29) : dateUTC(y, 3, 1))
                       : dateUTC(y, fromUTC.getUTCMonth() + 1, fromUTC.getUTCDate());
      }
      return next;
    }

    function isLeap(y) {
      return (y % 4 === 0 && y % 100 !== 0) || (y % 400 === 0);
    }

    // UI wiring
    const dobInput = document.getElementById("dob");
    const err = document.getElementById("error");
    const btn = document.getElementById("calcBtn");
    const clearBtn = document.getElementById("clearBtn");
    const todayBtn = document.getElementById("todayBtn");

    const ageY = document.getElementById("ageYears");
    const ageM = document.getElementById("ageMonths");
    const ageD = document.getElementById("ageDays");
    const nextB = document.getElementById("nextBday");
    const summary = document.getElementById("summary");

    // Set max to today (prevents future date selection in most browsers)
    (function init() {
      const t = todayUTC();
      const yyyy = t.getUTCFullYear();
      const mm = String(t.getUTCMonth() + 1).padStart(2, "0");
      const dd = String(t.getUTCDate()).padStart(2, "0");
      dobInput.max = `${yyyy}-${mm}-${dd}`;
    })();

    function showError(msg) {
      err.textContent = msg || "";
    }
    function resetUI() {
      ageY.textContent = "—";
      ageM.textContent = "—";
      ageD.textContent = "—";
      nextB.textContent = "—";
      summary.textContent = "";
      showError("");
    }

    function calculate() {
      resetUI();
      const v = dobInput.value;
      if (!v) {
        showError("Please select your date of birth.");
        return;
      }
      const dobUTC = parseLocalDateStringToUTC(v);
      const nowUTC = todayUTC();

      if (dobUTC > nowUTC) {
        showError("Birth date cannot be in the future.");
        return;
      }

      const { y, m, d } = diffYMD(dobUTC, nowUTC);
      const next = nextBirthday(dobUTC, nowUTC);
      const daysToNext = daysBetweenUTC(nowUTC, next);

      ageY.textContent = String(y);
      ageM.textContent = String(m);
      ageD.textContent = String(d);
      nextB.textContent = String(daysToNext);

      const dobNice = dobInput.value; // already yyyy-mm-dd
      const nextNice = `${next.getUTCFullYear()}-${String(next.getUTCMonth()+1).padStart(2,"0")}-${String(next.getUTCDate()).padStart(2,"0")}`;
      summary.textContent = `You are ${y} years, ${m} months, and ${d} days old as of today. Next birthday: ${nextNice} (${daysToNext} day${daysToNext===1?"":"s"} left).`;
    }

    btn.addEventListener("click", calculate);
    dobInput.addEventListener("keydown", (e) => {
      if (e.key === "Enter") calculate();
    });
    clearBtn.addEventListener("click", () => {
      dobInput.value = "";
      resetUI();
      dobInput.focus();
    });
    todayBtn.addEventListener("click", () => {
      const t = todayUTC();
      const yyyy = t.getUTCFullYear();
      const mm = String(t.getUTCMonth() + 1).padStart(2, "0");
      const dd = String(t.getUTCDate()).padStart(2, "0");
      dobInput.value = `${yyyy}-${mm}-${dd}`;
      calculate();
    });
  </script>
</body>
</html>
