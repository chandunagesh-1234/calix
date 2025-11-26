<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Simple Calculator</title>
<style>
  :root { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
  body { display:flex; align-items:center; justify-content:center; min-height:100vh; margin:0; background:#f3f4f6; }
  .calculator { width:320px; background:white; border-radius:12px; box-shadow:0 8px 24px rgba(2,6,23,0.1); padding:18px; }
  .display { height:64px; background:#0f172a; color:#e6edf3; border-radius:8px; display:flex; flex-direction:column; justify-content:center; padding:10px; box-sizing:border-box; margin-bottom:12px; }
  .display .expr { font-size:13px; color:#9aa7b7; min-height:16px; }
  .display .value { font-size:22px; font-weight:600; text-align:right; }
  .keys { display:grid; grid-template-columns:repeat(4,1fr); gap:8px; }
  button { height:52px; border-radius:8px; border:0; font-size:18px; cursor:pointer; box-shadow:inset 0 -2px rgba(0,0,0,0.02); }
  button.operator { background:#f8fafc; color:#0f172a; }
  button.action { background:#e11d48; color:white; }
  button.equal { background:#06b6d4; color:white; grid-column:span 2; }
  button.digit { background:#f1f5f9; }
  .wide { grid-column:span 2; }
</style>
</head>
<body>
  <div class="calculator" role="application" aria-label="Simple calculator">
    <div class="display" id="display">
      <div class="expr" id="expr">&nbsp;</div>
      <div class="value" id="value">0</div>
    </div>

    <div class="keys">
      <button class="action" data-action="clear">C</button>
      <button class="operator" data-action="back">⌫</button>
      <button class="operator" data-value="%" title="modulo">%</button>
      <button class="operator" data-value="/" title="divide">÷</button>

      <button class="digit" data-value="7">7</button>
      <button class="digit" data-value="8">8</button>
      <button class="digit" data-value="9">9</button>
      <button class="operator" data-value="*" title="multiply">×</button>

      <button class="digit" data-value="4">4</button>
      <button class="digit" data-value="5">5</button>
      <button class="digit" data-value="6">6</button>
      <button class="operator" data-value="-" title="subtract">−</button>

      <button class="digit" data-value="1">1</button>
      <button class="digit" data-value="2">2</button>
      <button class="digit" data-value="3">3</button>
      <button class="operator" data-value="+" title="add">+</button>

      <button class="digit wide" data-value="0">0</button>
      <button class="digit" data-value=".">.</button>
      <button class="equal" data-action="equals">=</button>
    </div>
  </div>

<script>
(() => {
  const exprEl = document.getElementById('expr');
  const valueEl = document.getElementById('value');

  let expr = '';    // expression string shown above
  let current = ''; // current number being typed
  let lastResult = null;

  function updateDisplay() {
    exprEl.textContent = expr || '\u00A0';
    valueEl.textContent = current || (lastResult !== null ? String(lastResult) : '0');
  }

  function pushDigit(d) {
    if (d === '.' && current.includes('.')) return;
    if (d === '0' && current === '0') return;
    if (current === '0' && d !== '.') current = d;
    else current += d;
    updateDisplay();
  }

  function pushOperator(op) {
    if (!current && expr === '' && lastResult !== null) {
      expr = String(lastResult);
    }
    if (!current && expr.slice(-1).match(/[+\-*/%]/)) {
      expr = expr.slice(0, -1) + op;
    } else {
      expr += (current || '') + op;
      current = '';
    }
    updateDisplay();
  }

  function clearAll() { expr = ''; current = ''; lastResult = null; updateDisplay(); }
  function backspace() {
    if (current) current = current.slice(0, -1);
    else if (expr) expr = expr.slice(0, -1);
    updateDisplay();
  }

  function compute() {
    if (!current && expr === '') return;
    const safeExpr = expr + (current || '');
    // Replace unicode operators for JS eval
    const jsExpr = safeExpr.replace(/×/g, '*').replace(/÷/g, '/').replace(/−/g, '-').replace(/%/g, '%');
    try {
      // Basic safety: only allow digits, operators, parentheses, decimal point, spaces
      if (!/^[0-9+\-*/%.() \t]+$/.test(jsExpr)) throw new Error('Invalid input');
      // eslint-disable-next-line no-eval
      const result = eval(jsExpr);
      lastResult = Number.isFinite(result) ? result : 'Error';
      expr = '';
      current = '';
      updateDisplay();
    } catch (e) {
      valueEl.textContent = 'Error';
      expr = '';
      current = '';
      lastResult = null;
    }
  }

  document.querySelectorAll('button').forEach(btn => {
    const val = btn.getAttribute('data-value');
    const action = btn.getAttribute('data-action');

    btn.addEventListener('click', () => {
      if (action === 'clear') clearAll();
      else if (action === 'back') backspace();
      else if (action === 'equals') compute();
      else if (val !== null) {
        if (/[0-9.]/.test(val)) pushDigit(val);
        else pushOperator(val);
      }
    });
  });

  // Keyboard support
  window.addEventListener('keydown', (e) => {
    const k = e.key;
    if ((k >= '0' && k <= '9') || k === '.') { pushDigit(k); e.preventDefault(); }
    else if (k === '+' || k === '-' || k === '*' || k === '/') { pushOperator(k); e.preventDefault(); }
    else if (k === 'Enter' || k === '=') { compute(); e.preventDefault(); }
    else if (k === 'Backspace') { backspace(); e.preventDefault(); }
    else if (k === 'Escape') { clearAll(); e.preventDefault(); }
    else if (k === '%') { pushOperator('%'); e.preventDefault(); }
  });

  updateDisplay();
})();
</script>
</body>
</html>
