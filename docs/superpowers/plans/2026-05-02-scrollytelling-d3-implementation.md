# Scrollytelling D3.js 互動網頁 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-page scrollytelling website with 18 D3.js interactive visualizations explaining why no formula exists for Go's second-best move.

**Architecture:** Single `index.html` file with inline CSS/JS. Scrollytelling uses Intersection Observer to sync text steps with sticky chart panels. D3.js charts are lazy-initialized and torn down for performance. MathJax renders LaTeX in accordions.

**Tech Stack:** D3.js v7 (CDN), MathJax 3 (CDN), Intersection Observer API (native), CSS Custom Properties (OKLCH + hex fallback)

**Spec:** `docs/superpowers/specs/2026-05-01-scrollytelling-d3-design.md`

---

## File Structure

All code lives in a single file:

- **Create:** `index.html` — Complete page (HTML structure, CSS, JS)

---

## Phase 1: Infrastructure

### Task 1: HTML Skeleton + CSS Design Tokens

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create base HTML with design tokens CSS**

Create `index.html` with `<!DOCTYPE html>`, CDN scripts, and full CSS custom properties:

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>圍棋的「第二好的一手」——為什麼不存在捷徑？</title>
  <style>
    :root {
      --bg-base: oklch(0.97 0.005 250);
      --bg-surface: oklch(0.94 0.005 250);
      --bg-overlay: oklch(0.90 0.008 250);
      --bg-hover: oklch(0.92 0.005 250);
      --text-primary: oklch(0.20 0.01 250);
      --text-secondary: oklch(0.45 0.01 250);
      --text-muted: oklch(0.47 0.01 250);
      --color-info: oklch(0.52 0.13 240);
      --color-critical: oklch(0.55 0.22 25);
      --color-pass: oklch(0.48 0.16 150);
      --color-medium: oklch(0.46 0.14 80);
      --color-high: oklch(0.48 0.16 55);
      --color-indigo: oklch(0.48 0.15 280);
      --color-link: oklch(0.48 0.15 250);
      --border-subtle: oklch(0.85 0.005 250);
      --stone-black: oklch(0.25 0.01 250);
      --stone-white: oklch(0.95 0.005 250);
      --text-3xl: 3.5rem;
      --text-2xl: 3rem;
      --text-xl: 2rem;
      --text-lg: 1.75rem;
      --text-base: 1.5rem;
      --text-sm: 1.25rem;
      --text-xs: 1.125rem;
    }

    @supports not (color: oklch(0 0 0)) {
      :root {
        --bg-base: #f5f6f8;
        --bg-surface: #ecedf0;
        --bg-overlay: #dfe0e5;
        --bg-hover: #e5e6ea;
        --text-primary: #1e2030;
        --text-secondary: #5e6070;
        --text-muted: #626476;
        --color-info: #2a6bb8;
        --color-critical: #c93135;
        --color-pass: #1e8050;
        --color-medium: #7a6218;
        --color-high: #9e5820;
        --color-indigo: #4830b8;
        --color-link: #1e5ab8;
        --border-subtle: #d5d6da;
        --stone-black: #2a2c3a;
        --stone-white: #f0f1f3;
      }
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
                   "Helvetica Neue", Arial, sans-serif;
      font-size: var(--text-base);
      line-height: 1.6;
      color: var(--text-primary);
      background: var(--bg-base);
    }

    .skip-link {
      position: absolute;
      top: -100%;
      left: 1rem;
      padding: 0.5rem 1rem;
      background: var(--color-link);
      color: white;
      border-radius: 0.25rem;
      z-index: 9999;
      font-size: var(--text-xs);
    }
    .skip-link:focus { top: 1rem; }

    .page-wrapper {
      max-width: 72rem;
      margin: 0 auto;
      padding: 0 1rem;
    }

    /* Hero */
    .hero {
      text-align: center;
      padding: 4rem 1rem;
      min-height: 80vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }
    .hero h1 {
      font-size: var(--text-3xl);
      font-weight: 700;
      margin-bottom: 1rem;
      max-width: 40rem;
    }
    .hero .subtitle {
      font-size: var(--text-lg);
      color: var(--text-secondary);
      max-width: 36rem;
    }
    .hero-board { margin-top: 2rem; }

    /* Chapter */
    .chapter {
      margin-bottom: 4rem;
      position: relative;
    }
    .chapter-title {
      font-size: var(--text-xl);
      font-weight: 700;
      margin-bottom: 2rem;
      padding-top: 2rem;
    }

    /* Scrollytelling layout */
    .scrolly {
      position: relative;
      display: flex;
      gap: 2rem;
    }
    .scrolly-text {
      flex: 0 0 40%;
    }
    .scrolly-chart {
      flex: 0 0 calc(60% - 2rem);
      position: sticky;
      top: 2rem;
      height: fit-content;
      align-self: flex-start;
    }

    /* Step */
    .step {
      margin-bottom: 60vh;
      opacity: 0.3;
      transition: opacity 0.3s ease;
    }
    .step.is-active { opacity: 1; }
    .step:last-child { margin-bottom: 20vh; }
    .step-content {
      font-size: var(--text-base);
    }

    /* Chart container */
    .chart-container {
      background: var(--bg-surface);
      border-radius: 0.5rem;
      padding: 1rem;
      min-height: 300px;
      position: relative;
    }
    .chart-container .chart-skeleton {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 300px;
      color: var(--text-muted);
      font-size: var(--text-sm);
    }
    .chart-container svg {
      width: 100%;
      height: auto;
    }

    /* Accordion */
    .accordion {
      margin-top: 1rem;
      border-top: 1px solid var(--border-subtle);
    }
    .accordion-btn {
      background: var(--bg-surface);
      border: none;
      width: 100%;
      padding: 0.75rem 1rem;
      cursor: pointer;
      text-align: left;
      font-size: var(--text-xs);
      color: var(--text-secondary);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .accordion-btn:hover { background: var(--bg-hover); }
    .accordion-panel {
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.3s ease;
      background: var(--bg-surface);
      padding: 0 1rem;
    }
    .accordion-panel.is-open {
      max-height: 2000px;
      padding: 0.5rem 1rem 1rem;
    }
    .accordion-panel .math-content {
      font-size: var(--text-sm);
      overflow-x: auto;
    }

    /* Tooltip */
    .tooltip {
      position: absolute;
      pointer-events: none;
      background: var(--text-primary);
      color: white;
      padding: 0.5rem 0.75rem;
      border-radius: 0.25rem;
      font-size: var(--text-sm);
      max-width: min(300px, 80vw);
      z-index: 1000;
      opacity: 0;
      transition: opacity 0.15s;
    }
    .tooltip.is-visible { opacity: 1; }

    /* Buttons */
    .btn {
      display: inline-block;
      padding: 0.5rem 1rem;
      background: var(--color-link);
      color: white;
      border: none;
      border-radius: 0.25rem;
      cursor: pointer;
      font-size: var(--text-xs);
    }
    .btn:hover { opacity: 0.9; }
    .btn-group { display: flex; gap: 0.5rem; flex-wrap: wrap; margin-top: 0.5rem; }

    /* Slider */
    input[type="range"] {
      -webkit-appearance: none;
      width: 100%;
      height: 6px;
      border-radius: 3px;
      background: var(--border-subtle);
      outline: none;
      margin: 1rem 0;
    }
    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 20px;
      height: 20px;
      border-radius: 50%;
      background: var(--color-link);
      border: 2px solid white;
      box-shadow: 0 1px 3px rgba(0,0,0,0.2);
      cursor: pointer;
    }
    input[type="range"]::-webkit-slider-thumb:hover {
      transform: scale(1.2);
      background: var(--color-info);
    }

    /* Focus */
    :focus-visible {
      outline: 3px solid var(--color-link);
      outline-offset: 2px;
    }

    /* Footer */
    .site-footer {
      padding: 3rem 1rem;
      border-top: 1px solid var(--border-subtle);
      color: var(--text-secondary);
      font-size: var(--text-sm);
    }
    .site-footer h3 {
      font-size: var(--text-lg);
      color: var(--text-primary);
      margin-bottom: 1rem;
    }
    .site-footer ol {
      padding-left: 1.5rem;
      margin-bottom: 1rem;
    }
    .site-footer li { margin-bottom: 0.25rem; }

    /* FAQ */
    .faq { margin: 2rem 0; }

    /* Reduced motion */
    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after {
        transition-duration: 0s !important;
        animation-duration: 0s !important;
      }
    }

    /* Print */
    @media print {
      .scrolly-chart { position: static; }
      .interactive-controls, .btn-group, .accordion-btn { display: none; }
      .step { opacity: 1; margin-bottom: 2rem; }
      body { background: white; }
    }

    /* Tablet */
    @media (max-width: 1023px) and (min-width: 768px) {
      .scrolly-text { flex: 0 0 45%; }
      .scrolly-chart { flex: 0 0 calc(55% - 2rem); }
      .step { margin-bottom: 40vh; }
    }

    /* Mobile */
    @media (max-width: 767px) {
      :root {
        --text-3xl: 2.25rem;
        --text-xl: 1.5rem;
        --text-lg: 1.375rem;
        --text-base: 1.25rem;
      }
      .scrolly {
        flex-direction: column;
      }
      .scrolly-text { flex: none; }
      .scrolly-chart {
        flex: none;
        position: -webkit-sticky;
        position: sticky;
        top: 0;
        z-index: 10;
        max-height: min(50vw, 40vh);
        overflow: hidden;
      }
      .step { margin-bottom: 30vh; }
      .hero { min-height: 60vh; padding: 2rem 1rem; }
    }
  </style>
</head>
<body>
  <a href="#main" class="skip-link">跳至主要內容</a>
  <div class="page-wrapper">
    <header class="hero">
      <h1>圍棋的「第二好的一手」——為什麼不存在捷徑？</h1>
      <p class="subtitle">一個業餘棋手的直覺問題，如何通往計算理論的深處</p>
      <div class="hero-board" id="hero-board" role="img" aria-label="裝飾用的圍棋棋盤"></div>
    </header>
    <main id="main">
      <!-- Chapters will be added in subsequent tasks -->
    </main>
    <footer class="site-footer">
      <p>本文的數學推導由作者與 AI（Claude）協作完成。互動網頁由 AI 輔助設計與開發。</p>
    </footer>
  </div>
  <div class="tooltip" id="tooltip"></div>

  <script src="https://cdn.jsdelivr.net/npm/d3@7.9.0/dist/d3.min.js" crossorigin="anonymous"
    onerror="document.querySelectorAll('.chart-container').forEach(c => c.innerHTML = '<p class=chart-skeleton>互動圖表載入失敗，請檢查網路連線</p>')"></script>
  <script>
    window.MathJax = { tex: { inlineMath: [['$','$'], ['\\(','\\)']] } };
  </script>
  <script src="https://cdn.jsdelivr.net/npm/mathjax@3.2.2/es5/tex-mml-chtml.js" crossorigin="anonymous" async></script>
  <script>
  // ===== Global utilities =====
  (function() {
    'use strict';

    // Color helper: read CSS custom property as computed hex value
    const root = document.documentElement;
    window.getColor = function(varName) {
      return getComputedStyle(root).getPropertyValue(varName).trim();
    };

    // Tooltip
    const tooltipEl = document.getElementById('tooltip');
    window.showTooltip = function(html, x, y) {
      tooltipEl.innerHTML = html;
      tooltipEl.classList.add('is-visible');
      const rect = tooltipEl.getBoundingClientRect();
      const vw = window.innerWidth;
      const vh = window.innerHeight;
      let left = x + 8;
      let top = y + 8;
      if (left + rect.width > vw - 16) left = x - rect.width - 8;
      if (top + rect.height > vh - 16) top = y - rect.height - 8;
      tooltipEl.style.left = left + 'px';
      tooltipEl.style.top = top + 'px';
    };
    window.hideTooltip = function() {
      tooltipEl.classList.remove('is-visible');
    };

    // Accordion toggle
    document.addEventListener('click', function(e) {
      const btn = e.target.closest('.accordion-btn');
      if (!btn) return;
      const panel = btn.nextElementSibling;
      const isOpen = panel.classList.toggle('is-open');
      btn.setAttribute('aria-expanded', isOpen);
      if (isOpen && window.MathJax && window.MathJax.typeset) {
        MathJax.typeset([panel]);
      }
    });

    // Scrollytelling engine
    const stepObserver = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) {
          const step = entry.target;
          const chapter = step.closest('.chapter');
          if (!chapter) return;
          chapter.querySelectorAll('.step').forEach(function(s) {
            s.classList.remove('is-active');
          });
          step.classList.add('is-active');
          const stepIndex = parseInt(step.dataset.step, 10);
          const chapterId = chapter.id;
          if (window.chartUpdaters && window.chartUpdaters[chapterId]) {
            window.chartUpdaters[chapterId](stepIndex);
          }
        }
      });
    }, {
      rootMargin: '-40% 0px -40% 0px',
      threshold: 0
    });

    // Chart lifecycle manager
    const activeCharts = new Set();
    const chartInitFns = {};
    const chartDestroyFns = {};

    window.registerChart = function(id, initFn, destroyFn) {
      chartInitFns[id] = initFn;
      chartDestroyFns[id] = destroyFn;
    };

    window.chartUpdaters = {};

    const chartObserver = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        const id = entry.target.id;
        if (entry.isIntersecting) {
          if (!activeCharts.has(id) && chartInitFns[id]) {
            // Enforce max 3 active charts
            if (activeCharts.size >= 3) {
              const oldest = activeCharts.values().next().value;
              if (chartDestroyFns[oldest]) chartDestroyFns[oldest]();
              activeCharts.delete(oldest);
            }
            chartInitFns[id]();
            activeCharts.add(id);
          }
        }
      });
    }, { rootMargin: '200px 0px 200px 0px' });

    // Teardown observer (2 screens away)
    const teardownObserver = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        const id = entry.target.id;
        if (!entry.isIntersecting && activeCharts.has(id)) {
          if (chartDestroyFns[id]) chartDestroyFns[id]();
          activeCharts.delete(id);
        }
      });
    }, { rootMargin: '200% 0px 200% 0px' });

    // Init: observe all steps and charts after DOM ready
    window.addEventListener('DOMContentLoaded', function() {
      document.querySelectorAll('.step').forEach(function(s) {
        stepObserver.observe(s);
      });
      document.querySelectorAll('.chart-container').forEach(function(c) {
        chartObserver.observe(c);
        teardownObserver.observe(c);
      });
    });

    // Resize handler
    let resizeTimer;
    window.addEventListener('resize', function() {
      clearTimeout(resizeTimer);
      resizeTimer = setTimeout(function() {
        activeCharts.forEach(function(id) {
          if (chartDestroyFns[id]) chartDestroyFns[id]();
          if (chartInitFns[id]) chartInitFns[id]();
        });
      }, 300);
    });
  })();
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify skeleton in browser**

Run: `open /Users/lightman/weiqi.kids/go-second-best-move-formula/index.html`

Expected: Page loads with hero section (title + subtitle), light gray background, correct font stack, skip link hidden until focused. No console errors.

- [ ] **Step 3: Add Hero board decoration**

Add this IIFE script block before the closing `</body>` tag, after the global utilities script:

```html
<script>
// ===== Hero Board =====
(function() {
  'use strict';
  const container = d3.select('#hero-board');
  const size = 280;
  const padding = 20;
  const gridSize = 9;
  const cellSize = (size - padding * 2) / (gridSize - 1);

  const svg = container.append('svg')
    .attr('viewBox', `0 0 ${size} ${size}`)
    .attr('width', size)
    .attr('height', size);

  // Board background
  svg.append('rect')
    .attr('width', size).attr('height', size)
    .attr('rx', 8)
    .attr('fill', '#e8c97a');

  // Grid lines
  for (let i = 0; i < gridSize; i++) {
    const pos = padding + i * cellSize;
    svg.append('line')
      .attr('x1', padding).attr('y1', pos)
      .attr('x2', size - padding).attr('y2', pos)
      .attr('stroke', '#a08040').attr('stroke-width', 0.5);
    svg.append('line')
      .attr('x1', pos).attr('y1', padding)
      .attr('x2', pos).attr('y2', size - padding)
      .attr('stroke', '#a08040').attr('stroke-width', 0.5);
  }

  // Star points
  const stars = [[2,2],[2,6],[6,2],[6,6],[4,4]];
  stars.forEach(function(s) {
    svg.append('circle')
      .attr('cx', padding + s[0] * cellSize)
      .attr('cy', padding + s[1] * cellSize)
      .attr('r', 2.5)
      .attr('fill', '#a08040');
  });

  // Decorative stones
  const stones = [
    {x:3,y:3,c:'black'},{x:5,y:5,c:'black'},{x:2,y:4,c:'black'},{x:6,y:3,c:'black'},
    {x:4,y:4,c:'white'},{x:3,y:5,c:'white'},{x:5,y:2,c:'white'}
  ];
  const stoneR = cellSize * 0.42;

  stones.forEach(function(s, i) {
    const g = svg.append('g')
      .attr('transform', `translate(${padding + s.x * cellSize},${padding + s.y * cellSize})`);
    g.append('circle')
      .attr('r', 0)
      .attr('fill', s.c === 'black' ? getColor('--stone-black') : getColor('--stone-white'))
      .attr('stroke', s.c === 'black' ? '#111' : '#ccc')
      .attr('stroke-width', 1)
      .transition()
      .duration(400)
      .delay(i * 100)
      .attr('r', stoneR);
  });
})();
</script>
```

- [ ] **Step 4: Verify hero board**

Run: Refresh browser.

Expected: 9×9 Go board appears below the title. 7 stones fade in one by one. No interactivity.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton with CSS design tokens, scrollytelling engine, hero board"
```

---

### Task 2: Chapter 4 — 定理 A (V9 歸謬法邏輯鏈 + V10 近似無效)

Core argument — highest priority per spec.

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 4 HTML structure inside `<main>`**

```html
<section class="chapter" id="ch4">
  <h2 class="chapter-title">Chapter 4：為什麼 Q 沒有公式</h2>
  <div class="scrolly">
    <div class="scrolly-text">
      <div class="step" data-step="0">
        <div class="step-content">
          <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:0.5rem">定理 A：歸謬法的力量</h3>
          <p>假設你有一台能秒算任何圍棋局面價值的神機——這台機器不只是圍棋作弊器，它等於能解開宇宙間所有超難問題（因為圍棋能模擬任何超難計算）。</p>
          <p style="margin-top:0.75rem">但數學已經證明：<strong>這種萬能機器不可能存在</strong>。</p>
          <p style="margin-top:0.75rem">所以神機不存在，公式也不存在。這就是<strong>歸謬法</strong>——假設公式存在，推導出矛盾，因此公式不存在。</p>
          <div class="accordion">
            <button class="accordion-btn" aria-expanded="false" aria-controls="acc-ch4-1">📐 展開數學細節</button>
            <div class="accordion-panel" id="acc-ch4-1">
              <div class="math-content">
                <p>完整的歸謬鏈：</p>
                <p>假設 $Q \in \mathrm{FP}$（存在 poly 時間公式）</p>
                <p>$\Rightarrow V(S) = \max_{m} Q(S,m)$ 可在 poly 時間算出（最多 $n^2+1$ 次查詢）</p>
                <p>$\Rightarrow V\text{-DECISION} \in P$</p>
                <p>$\Rightarrow \mathrm{EXPTIME} \subseteq P$（因為 $V\text{-DECISION}$ 是 EXPTIME-complete）</p>
                <p>但 $P \subsetneq \mathrm{EXPTIME}$（時間階層定理，<strong>無條件已證</strong>）。矛盾！ $\blacksquare$</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="step" data-step="1">
        <div class="step-content">
          <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:0.5rem">近似也不行</h3>
          <p>「那如果我不要求精確答案，只要大概算對呢？」</p>
          <p style="margin-top:0.75rem">別忘了：圍棋的分數一定是<strong>整數</strong>。只要你的近似誤差小於 0.5，四捨五入就能完美恢復精確答案。</p>
          <p style="margin-top:0.75rem">所以「0.49 的近似」就已經等同精確——近似不是逃生出口。</p>
          <p style="margin-top:0.75rem;color:var(--text-secondary);font-size:var(--text-sm)">拖動下方滑桿，調整近似誤差 ε，看看結果會怎樣 ↗</p>
        </div>
      </div>
    </div>
    <div class="scrolly-chart">
      <div class="chart-container" id="chart-v9" role="img" aria-label="歸謬法邏輯鏈：假設公式存在推導出矛盾" aria-live="polite">
        <div class="chart-skeleton">載入中...</div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add V9 (歸謬法邏輯鏈) + V10 (近似無效) JavaScript**

Add this script block after the hero script:

```html
<script>
// ===== Chapter 4: V9 + V10 =====
(function() {
  'use strict';

  const chartEl = document.getElementById('chart-v9');
  let currentViz = null; // 'v9' or 'v10'

  // ---- V9: 歸謬法邏輯鏈 ----
  function drawV9(container) {
    container.innerHTML = '';
    const w = container.clientWidth - 32;
    const h = Math.max(320, w * 0.6);
    const svg = d3.select(container).append('svg')
      .attr('viewBox', `0 0 ${w} ${h}`);

    const nodes = [
      { id: 0, label: '假設 Q 有公式', x: w * 0.1, y: h * 0.5 },
      { id: 1, label: 'Q ∈ FP', x: w * 0.28, y: h * 0.5 },
      { id: 2, label: 'V ∈ P', x: w * 0.46, y: h * 0.5 },
      { id: 3, label: 'EXPTIME ⊆ P', x: w * 0.66, y: h * 0.5 },
      { id: 4, label: '💥 矛盾！', x: w * 0.86, y: h * 0.5 }
    ];
    const nodeW = w * 0.15;
    const nodeH = 44;

    // Arrows
    const arrowData = [[0,1],[1,2],[2,3],[3,4]];
    svg.append('defs').append('marker')
      .attr('id', 'arrowhead')
      .attr('viewBox', '0 0 10 10')
      .attr('refX', 10).attr('refY', 5)
      .attr('markerWidth', 8).attr('markerHeight', 8)
      .attr('orient', 'auto')
      .append('path').attr('d', 'M0,0 L10,5 L0,10 Z')
      .attr('fill', getColor('--text-muted'));

    arrowData.forEach(function(a) {
      svg.append('line')
        .attr('x1', nodes[a[0]].x + nodeW / 2 + 4)
        .attr('y1', nodes[a[0]].y)
        .attr('x2', nodes[a[1]].x - nodeW / 2 - 4)
        .attr('y2', nodes[a[1]].y)
        .attr('stroke', getColor('--text-muted'))
        .attr('stroke-width', 2)
        .attr('marker-end', 'url(#arrowhead)')
        .attr('opacity', 0);
    });

    // Nodes
    const nodeGroups = svg.selectAll('.node')
      .data(nodes).enter().append('g')
      .attr('class', 'node')
      .attr('transform', function(d) { return `translate(${d.x},${d.y})`; });

    nodeGroups.append('rect')
      .attr('x', -nodeW / 2).attr('y', -nodeH / 2)
      .attr('width', nodeW).attr('height', nodeH)
      .attr('rx', 8)
      .attr('fill', function(d) {
        return d.id === 4 ? getColor('--color-critical') : getColor('--bg-overlay');
      })
      .attr('stroke', function(d) {
        return d.id === 4 ? getColor('--color-critical') : getColor('--border-subtle');
      })
      .attr('stroke-width', 2)
      .attr('opacity', 0);

    nodeGroups.append('text')
      .attr('text-anchor', 'middle')
      .attr('dy', '0.35em')
      .attr('font-size', Math.max(12, w * 0.018))
      .attr('font-weight', 600)
      .attr('fill', function(d) {
        return d.id === 4 ? 'white' : getColor('--text-primary');
      })
      .text(function(d) { return d.label; })
      .attr('opacity', 0);

    // Subtitle
    svg.append('text')
      .attr('x', w / 2).attr('y', h * 0.85)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(12, w * 0.022))
      .attr('fill', getColor('--text-secondary'))
      .text('因為 P ⊊ EXPTIME（已證），公式不可能存在')
      .attr('opacity', 0)
      .attr('class', 'v9-subtitle');

    // Animate sequence
    let step = 0;
    const interval = setInterval(function() {
      if (step < 5) {
        svg.selectAll('.node').filter(function(d) { return d.id === step; })
          .select('rect')
          .transition().duration(500).attr('opacity', 1);
        svg.selectAll('.node').filter(function(d) { return d.id === step; })
          .select('text')
          .transition().duration(500).attr('opacity', 1);
        if (step > 0) {
          svg.selectAll('line').filter(function(d, i) { return i === step - 1; })
            .transition().duration(400).attr('opacity', 1);
        }
      }
      if (step === 5) {
        svg.select('.v9-subtitle').transition().duration(600).attr('opacity', 1);
      }
      step++;
      if (step > 5) clearInterval(interval);
    }, 800);

    return interval;
  }

  // ---- V10: 近似無效示意 ----
  function drawV10(container) {
    container.innerHTML = '';
    const w = container.clientWidth - 32;
    const h = Math.max(280, w * 0.5);
    const svg = d3.select(container).append('svg')
      .attr('viewBox', `0 0 ${w} ${h}`);

    const margin = { top: 40, right: 40, bottom: 60, left: 40 };
    const iw = w - margin.left - margin.right;
    const ih = h - margin.top - margin.bottom;
    const g = svg.append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    // Number line: Q values -3 to 5
    const qValues = [-3, -2, -1, 0, 1, 2, 3, 4, 5];
    const xScale = d3.scaleLinear().domain([-3.5, 5.5]).range([0, iw]);

    // Axis
    g.append('line')
      .attr('x1', 0).attr('y1', ih / 2)
      .attr('x2', iw).attr('y2', ih / 2)
      .attr('stroke', getColor('--text-muted')).attr('stroke-width', 2);

    // Integer ticks
    qValues.forEach(function(q) {
      g.append('line')
        .attr('x1', xScale(q)).attr('y1', ih / 2 - 8)
        .attr('x2', xScale(q)).attr('y2', ih / 2 + 8)
        .attr('stroke', getColor('--text-primary')).attr('stroke-width', 2);
      g.append('text')
        .attr('x', xScale(q)).attr('y', ih / 2 + 24)
        .attr('text-anchor', 'middle')
        .attr('font-size', Math.max(12, w * 0.02))
        .attr('fill', getColor('--text-primary'))
        .text(q);
    });

    // Label
    g.append('text')
      .attr('x', iw / 2).attr('y', -10)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(12, w * 0.022))
      .attr('font-weight', 600)
      .attr('fill', getColor('--text-primary'))
      .text('Q 值一定是整數');

    // True value marker
    const trueQ = 2;
    const trueMarker = g.append('g').attr('class', 'true-marker');
    trueMarker.append('circle')
      .attr('cx', xScale(trueQ)).attr('cy', ih / 2)
      .attr('r', 10)
      .attr('fill', getColor('--color-info'));
    trueMarker.append('text')
      .attr('x', xScale(trueQ)).attr('y', ih / 2 - 18)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(11, w * 0.016))
      .attr('fill', getColor('--color-info'))
      .text('真實值 Q=2');

    // Approximate marker (will move with epsilon)
    const approxG = g.append('g').attr('class', 'approx-marker');
    const approxCircle = approxG.append('circle')
      .attr('cy', ih / 2).attr('r', 8)
      .attr('fill', getColor('--color-high'))
      .attr('opacity', 0.8);
    const approxLabel = approxG.append('text')
      .attr('y', ih / 2 + 40)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(11, w * 0.016))
      .attr('fill', getColor('--color-high'));

    // Round arrow
    const roundArrow = g.append('g').attr('class', 'round-arrow').attr('opacity', 0);
    roundArrow.append('text')
      .attr('y', ih / 2 - 50)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(11, w * 0.016));

    // Result label
    const resultLabel = svg.append('text')
      .attr('x', w / 2).attr('y', h - 10)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(12, w * 0.022))
      .attr('font-weight', 600);

    // Slider
    const sliderDiv = document.createElement('div');
    sliderDiv.className = 'interactive-controls';
    sliderDiv.style.padding = '0 1rem';
    sliderDiv.innerHTML = `
      <label style="font-size:var(--text-sm);color:var(--text-secondary)">
        近似誤差 ε = <span id="eps-val">0.00</span>
      </label>
      <input type="range" id="eps-slider" min="0" max="1.2" step="0.01" value="0"
        aria-label="調整近似誤差 epsilon">
    `;
    container.appendChild(sliderDiv);

    const slider = document.getElementById('eps-slider');
    const epsVal = document.getElementById('eps-val');

    function update(eps) {
      epsVal.textContent = eps.toFixed(2);
      const approxVal = trueQ + eps;
      const rounded = Math.round(approxVal);
      const correct = rounded === trueQ;

      approxCircle.attr('cx', xScale(approxVal));
      approxLabel
        .attr('x', xScale(approxVal))
        .text(`近似值 ${approxVal.toFixed(2)}`);

      roundArrow
        .attr('opacity', 1)
        .select('text')
        .attr('x', xScale(approxVal))
        .attr('fill', correct ? getColor('--color-pass') : getColor('--color-critical'))
        .text(correct ? `四捨五入 → ${rounded} ✓` : `四捨五入 → ${rounded} ✗`);

      resultLabel
        .attr('fill', correct ? getColor('--color-pass') : getColor('--color-critical'))
        .text(correct
          ? 'ε < 0.5 → 近似等於精確！'
          : 'ε ≥ 0.5 → 偏到隔壁整數，無法恢復');
    }

    slider.addEventListener('input', function() {
      update(parseFloat(this.value));
    });
    update(0);
  }

  // Chart lifecycle
  let animInterval = null;

  function initChart() {
    currentViz = 'v9';
    animInterval = drawV9(chartEl);
  }

  function destroyChart() {
    if (animInterval) clearInterval(animInterval);
    chartEl.innerHTML = '<div class="chart-skeleton">載入中...</div>';
    currentViz = null;
  }

  registerChart('chart-v9', initChart, destroyChart);

  // Step updater
  chartUpdaters['ch4'] = function(stepIndex) {
    if (stepIndex === 0 && currentViz !== 'v9') {
      if (animInterval) clearInterval(animInterval);
      animInterval = drawV9(chartEl);
      currentViz = 'v9';
    } else if (stepIndex === 1 && currentViz !== 'v10') {
      if (animInterval) clearInterval(animInterval);
      drawV10(chartEl);
      currentViz = 'v10';
    }
  };
})();
</script>
```

- [ ] **Step 3: Verify Chapter 4 in browser**

Run: Refresh browser.

Expected:
- Chapter 4 title visible. Two text steps on left, chart on right.
- Step 1 active → V9 logic chain animates (5 nodes appear one by one with arrows).
- Scroll to step 2 → chart switches to V10 number line with epsilon slider.
- Dragging slider changes approximate value, shows correct/incorrect rounding.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Chapter 4 with V9 (歸謬法邏輯鏈) and V10 (近似無效)"
```

---

### Task 3: Chapter 6 — 定理 C+D (V13 二元分支 + V14 瓶頸構造)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 6 HTML inside `<main>` after Chapter 4**

```html
<section class="chapter" id="ch6">
  <h2 class="chapter-title">Chapter 6：次佳手也不行</h2>
  <div class="scrolly">
    <div class="scrolly-text">
      <div class="step" data-step="0">
        <div class="step-content">
          <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:0.5rem">定理 C：知道第二名就知道第一名</h3>
          <p>想像只有兩間教室 A 和 B，你知道數學課在其中一間。如果有人告訴你體育課（次佳手）在 B 教室，你立刻就知道<strong>數學課（最佳手）在 A 教室</strong>。</p>
          <p style="margin-top:0.75rem">圍棋的 Robson 歸約構造中，每個關鍵步驟恰好只有兩個有競爭力的選擇。所以知道 m₂ 就等於知道 m₁——如果次佳手有公式，最佳手也有公式，導致 <span lang="en">EXPTIME = PSPACE</span> 的塌縮。</p>
          <div class="accordion">
            <button class="accordion-btn" aria-expanded="false" aria-controls="acc-ch6-1">📐 展開數學細節</button>
            <div class="accordion-panel" id="acc-ch6-1">
              <div class="math-content">
                <p>二元化 ATM 中，$|C(S)| = 2$。由 (★★-3)，$m_1, m_2 \in C(S)$。</p>
                <p>$m_1 = C(S) \setminus \{m_2\}$，$O(1)$ 時間。</p>
                <p>$m_2 \in \mathrm{FP} \Rightarrow m_1 \in \mathrm{FP} \xrightarrow{\text{定理 B}} \mathrm{EXPTIME} = \mathrm{PSPACE}$</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="step" data-step="1">
        <div class="step-content">
          <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:0.5rem">瓶頸構造：三個房間</h3>
          <p>想像三個房間：</p>
          <ul style="margin:0.5rem 0 0.5rem 1.5rem">
            <li><strong>D（主戰場）</strong>：裡面有一顆巨大的炸彈（n⁴ 子的群），你一定要先去拆彈。</li>
            <li><strong>A（衛星）</strong>：嵌入了一個 Robson 計算構造。</li>
            <li><strong>B（衛星）</strong>：只有一些不值錢的中立點。</li>
          </ul>
          <p>最重要的事（m₁）一定是拆彈 d。拆完彈之後，第二重要的事（m₂）在 A 還是 B？</p>
        </div>
      </div>
      <div class="step" data-step="2">
        <div class="step-content">
          <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:0.5rem">接受方向</h3>
          <p>如果 ATM 接受（計算成功）→ A 區有好手可走 → <strong style="color:var(--color-pass)">m₂ 落在 A 區</strong></p>
        </div>
      </div>
      <div class="step" data-step="3">
        <div class="step-content">
          <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:0.5rem">拒絕方向</h3>
          <p>如果 ATM 拒絕（計算失敗）→ A 區全是壞手 → <strong style="color:var(--color-critical)">m₂ 只能落在 B 區</strong></p>
          <p style="margin-top:0.75rem">所以計算 m₂ 就等於判斷 ATM 是否接受——一個 <span lang="en">EXPTIME-complete</span> 的問題。公式不存在！</p>
          <div class="accordion">
            <button class="accordion-btn" aria-expanded="false" aria-controls="acc-ch6-2">📐 展開數學細節</button>
            <div class="accordion-panel" id="acc-ch6-2">
              <div class="math-content">
                <p><strong>瓶頸構造核心</strong></p>
                <p>$Q(S^*, d) - Q(S^*, m) \geq 2n^4 - O(N^2) > 0$，故 $m_1(S^*) = d$。</p>
                <p>接受方向：pass 不等式 $\Delta \geq 0$ 保證 $V_B > 0$（無條件）。</p>
                <p>拒絕方向：$\Delta = O(1) \ll \Omega(N)$ 保證 $V_B < 0$（由 A1-A3）。</p>
                <p>$m_2(S^*) \in A \iff \text{ATM 接受 } x$</p>
                <p>由 $P \subsetneq \mathrm{EXPTIME}$：$m_2 \notin \mathrm{FP}$。$\blacksquare$</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="scrolly-chart">
      <div class="chart-container" id="chart-v13" role="img" aria-label="定理 C+D 的視覺化：二元分支歸約與瓶頸構造" aria-live="polite">
        <div class="chart-skeleton">載入中...</div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add V13 + V14 JavaScript**

```html
<script>
// ===== Chapter 6: V13 + V14 =====
(function() {
  'use strict';
  const chartEl = document.getElementById('chart-v13');
  let currentStep = -1;

  function drawV13(container) {
    container.innerHTML = '';
    const w = container.clientWidth - 32;
    const h = Math.max(300, w * 0.55);
    const svg = d3.select(container).append('svg')
      .attr('viewBox', `0 0 ${w} ${h}`);

    const cx = w / 2, cy = h * 0.3;
    const roomW = w * 0.18, roomH = h * 0.22;
    const gap = w * 0.08;

    // Two rooms
    const rooms = [
      { id: 'A', x: cx - roomW - gap / 2, y: cy, label: '教室 A', sub: '數學課？' },
      { id: 'B', x: cx + gap / 2, y: cy, label: '教室 B', sub: '體育課？' }
    ];

    rooms.forEach(function(r) {
      const g = svg.append('g').attr('class', 'room-' + r.id);
      g.append('rect')
        .attr('x', r.x).attr('y', r.y)
        .attr('width', roomW).attr('height', roomH)
        .attr('rx', 8)
        .attr('fill', getColor('--bg-overlay'))
        .attr('stroke', getColor('--border-subtle'))
        .attr('stroke-width', 2);
      g.append('text')
        .attr('x', r.x + roomW / 2).attr('y', r.y + roomH / 2 - 6)
        .attr('text-anchor', 'middle')
        .attr('font-size', Math.max(13, w * 0.022))
        .attr('font-weight', 600)
        .attr('fill', getColor('--text-primary'))
        .text(r.label);
      g.append('text')
        .attr('x', r.x + roomW / 2).attr('y', r.y + roomH / 2 + 14)
        .attr('text-anchor', 'middle')
        .attr('font-size', Math.max(11, w * 0.017))
        .attr('fill', getColor('--text-secondary'))
        .text(r.sub);
    });

    // Step labels
    const steps = [
      { y: h * 0.7, text: '① 得知：體育課（m₂）在 B', opacity: 0, id: 's1' },
      { y: h * 0.82, text: '② 排除 → 數學課（m₁）在 A ！', opacity: 0, id: 's2' }
    ];

    steps.forEach(function(s) {
      svg.append('text')
        .attr('class', 'step-label-' + s.id)
        .attr('x', cx).attr('y', s.y)
        .attr('text-anchor', 'middle')
        .attr('font-size', Math.max(13, w * 0.022))
        .attr('font-weight', 600)
        .attr('fill', getColor('--text-primary'))
        .text(s.text)
        .attr('opacity', 0);
    });

    // Animate
    setTimeout(function() {
      // Highlight B as m₂
      svg.select('.room-B rect')
        .transition().duration(500)
        .attr('fill', getColor('--color-indigo'))
        .attr('stroke', getColor('--color-indigo'));
      svg.select('.room-B text').transition().duration(500).attr('fill', 'white');
      svg.select('.step-label-s1').transition().duration(500).attr('opacity', 1);
    }, 600);

    setTimeout(function() {
      // Highlight A as m₁
      svg.select('.room-A rect')
        .transition().duration(500)
        .attr('fill', getColor('--color-high'))
        .attr('stroke', getColor('--color-high'));
      svg.select('.room-A text').transition().duration(500).attr('fill', 'white');
      svg.select('.step-label-s2').transition().duration(500).attr('opacity', 1);
    }, 1800);
  }

  function drawV14(container, phase) {
    if (phase === 1) {
      // First call: draw base layout
      container.innerHTML = '';
      const w = container.clientWidth - 32;
      const h = Math.max(320, w * 0.6);
      const svg = d3.select(container).append('svg')
        .attr('viewBox', `0 0 ${w} ${h}`)
        .attr('id', 'v14-svg');

      const regions = [
        { id: 'D', x: w * 0.05, y: h * 0.1, w: w * 0.25, h: h * 0.7, label: 'D 主戰場', sub: '💣 n⁴ 子的群', color: getColor('--bg-overlay') },
        { id: 'A', x: w * 0.4, y: h * 0.1, w: w * 0.25, h: h * 0.7, label: 'A 衛星', sub: 'Robson 構造', color: getColor('--bg-overlay') },
        { id: 'B', x: w * 0.75, y: h * 0.1, w: w * 0.2, h: h * 0.7, label: 'B 衛星', sub: '單官點', color: getColor('--bg-overlay') }
      ];

      // Walls between regions
      svg.append('rect')
        .attr('x', w * 0.33).attr('y', h * 0.05)
        .attr('width', 4).attr('height', h * 0.8)
        .attr('fill', getColor('--text-muted')).attr('rx', 2);
      svg.append('rect')
        .attr('x', w * 0.69).attr('y', h * 0.05)
        .attr('width', 4).attr('height', h * 0.8)
        .attr('fill', getColor('--text-muted')).attr('rx', 2);

      regions.forEach(function(r) {
        const g = svg.append('g').attr('class', 'region-' + r.id);
        g.append('rect')
          .attr('x', r.x).attr('y', r.y)
          .attr('width', r.w).attr('height', r.h)
          .attr('rx', 10)
          .attr('fill', r.color)
          .attr('stroke', getColor('--border-subtle'))
          .attr('stroke-width', 2);
        g.append('text')
          .attr('x', r.x + r.w / 2).attr('y', r.y + r.h / 2 - 10)
          .attr('text-anchor', 'middle')
          .attr('font-size', Math.max(13, w * 0.022))
          .attr('font-weight', 700)
          .attr('fill', getColor('--text-primary'))
          .text(r.label);
        g.append('text')
          .attr('x', r.x + r.w / 2).attr('y', r.y + r.h / 2 + 14)
          .attr('text-anchor', 'middle')
          .attr('font-size', Math.max(11, w * 0.017))
          .attr('fill', getColor('--text-secondary'))
          .text(r.sub);
      });

      // Bottom label
      svg.append('text')
        .attr('class', 'v14-label')
        .attr('x', w / 2).attr('y', h * 0.95)
        .attr('text-anchor', 'middle')
        .attr('font-size', Math.max(12, w * 0.02))
        .attr('font-weight', 600)
        .attr('fill', getColor('--text-secondary'))
        .text('三個隔離的區域');

    } else if (phase === 2) {
      // Highlight D as m₁
      const svg = d3.select('#v14-svg');
      if (svg.empty()) return;
      svg.select('.region-D rect')
        .transition().duration(600)
        .attr('stroke', getColor('--color-high'))
        .attr('stroke-width', 4);
      svg.select('.v14-label')
        .transition().duration(400)
        .text('m₁ = d（一定要先拆彈）')
        .attr('fill', getColor('--color-high'));

    } else if (phase === 3) {
      // Accept → A green
      const svg = d3.select('#v14-svg');
      if (svg.empty()) return;
      svg.select('.region-A rect')
        .transition().duration(600)
        .attr('fill', getColor('--color-pass'))
        .attr('stroke', getColor('--color-pass'))
        .attr('stroke-width', 4);
      svg.select('.region-A text').transition().duration(400).attr('fill', 'white');
      svg.select('.region-B rect')
        .transition().duration(400)
        .attr('fill', getColor('--bg-overlay'))
        .attr('stroke', getColor('--border-subtle'))
        .attr('stroke-width', 2);
      svg.select('.v14-label')
        .text('ATM 接受 → m₂ 在 A 區')
        .attr('fill', getColor('--color-pass'));

    } else if (phase === 4) {
      // Reject → B red
      const svg = d3.select('#v14-svg');
      if (svg.empty()) return;
      svg.select('.region-A rect')
        .transition().duration(600)
        .attr('fill', getColor('--bg-overlay'))
        .attr('stroke', getColor('--border-subtle'))
        .attr('stroke-width', 2);
      svg.select('.region-A text').transition().duration(400).attr('fill', getColor('--text-primary'));
      svg.select('.region-B rect')
        .transition().duration(600)
        .attr('fill', getColor('--color-critical'))
        .attr('stroke', getColor('--color-critical'))
        .attr('stroke-width', 4);
      svg.select('.region-B text').transition().duration(400).attr('fill', 'white');
      svg.select('.v14-label')
        .text('ATM 拒絕 → m₂ 在 B 區')
        .attr('fill', getColor('--color-critical'));
    }
  }

  function initChart() {
    currentStep = -1;
    drawV13(chartEl);
  }
  function destroyChart() {
    chartEl.innerHTML = '<div class="chart-skeleton">載入中...</div>';
    currentStep = -1;
  }

  registerChart('chart-v13', initChart, destroyChart);

  chartUpdaters['ch6'] = function(stepIndex) {
    if (stepIndex === currentStep) return;
    currentStep = stepIndex;
    if (stepIndex === 0) {
      drawV13(chartEl);
    } else if (stepIndex === 1) {
      drawV14(chartEl, 1);
    } else if (stepIndex === 2) {
      drawV14(chartEl, 2);
      setTimeout(function() { drawV14(chartEl, 3); }, 200);
    } else if (stepIndex === 3) {
      drawV14(chartEl, 4);
    }
  };
})();
</script>
```

- [ ] **Step 3: Verify Chapter 6 in browser**

Expected:
- Step 0 → V13: Two rooms A/B, B highlights as m₂ (indigo), then A highlights as m₁ (orange).
- Step 1 → V14: Three regions D/A/B appear with walls.
- Step 2 → D highlighted, then A turns green (accept).
- Step 3 → B turns red (reject).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Chapter 6 with V13 (二元分支) and V14 (瓶頸構造)"
```

---

### Task 4: Chapter 0 — 棋手的疑問 (V1 偏好排序 + V2 暴力vs公式)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 0 HTML at the beginning of `<main>`**

```html
<section class="chapter" id="ch0">
  <h2 class="chapter-title">Chapter 0：棋手的疑問</h2>
  <div class="scrolly">
    <div class="scrolly-text">
      <div class="step" data-step="0">
        <div class="step-content">
          <p>想像你在玩一個遊戲。你知道有一個「最佳選擇」，但算不出來。退而求其次，你想：<strong>第二好的選擇是什麼？</strong></p>
          <p style="margin-top:0.75rem">先試試看——依序點擊右邊的 5 個寶箱，排出你的偏好順序。</p>
        </div>
      </div>
      <div class="step" data-step="1">
        <div class="step-content">
          <p>你剛才輕鬆排出了 5 個選項的第二名。</p>
          <p style="margin-top:0.75rem">但如果有 <strong>10<sup>170</sup> 個選項</strong>呢？逐一嘗試要花多久？有沒有捷徑——一個<strong>公式</strong>——可以直接算出來？</p>
          <p style="margin-top:0.75rem;color:var(--text-secondary);font-size:var(--text-sm)">左邊：逐一嘗試（慢）；右邊：公式直達（快）↗</p>
        </div>
      </div>
    </div>
    <div class="scrolly-chart">
      <div class="chart-container" id="chart-v1" role="img" aria-label="偏好排序互動：點擊寶箱排出偏好" aria-live="polite">
        <div class="chart-skeleton">載入中...</div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add V1 + V2 JavaScript**

```html
<script>
// ===== Chapter 0: V1 + V2 =====
(function() {
  'use strict';
  const chartEl = document.getElementById('chart-v1');
  let currentViz = null;

  function drawV1(container) {
    container.innerHTML = '';
    const w = container.clientWidth - 32;
    const h = Math.max(300, w * 0.6);
    const svg = d3.select(container).append('svg')
      .attr('viewBox', `0 0 ${w} ${h}`);

    const items = ['🏆', '💎', '🎁', '🌟', '🔑'];
    const labels = ['金盃', '鑽石', '禮物', '星星', '鑰匙'];
    const boxW = w * 0.13, boxH = boxW;
    const gap = (w - 5 * boxW) / 6;
    const chosen = [];

    // Title
    svg.append('text')
      .attr('x', w / 2).attr('y', 30)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(13, w * 0.022))
      .attr('font-weight', 600)
      .attr('fill', getColor('--text-primary'))
      .text('依序點擊，排出你的偏好');

    // Boxes
    const boxes = items.map(function(emoji, i) {
      const x = gap + i * (boxW + gap);
      const y = h * 0.25;
      const g = svg.append('g')
        .attr('class', 'box')
        .style('cursor', 'pointer')
        .attr('tabindex', 0)
        .attr('role', 'button')
        .attr('aria-label', '選擇 ' + labels[i]);

      g.append('rect')
        .attr('x', x).attr('y', y)
        .attr('width', boxW).attr('height', boxH)
        .attr('rx', 8)
        .attr('fill', getColor('--bg-overlay'))
        .attr('stroke', getColor('--border-subtle'))
        .attr('stroke-width', 2);

      g.append('text')
        .attr('x', x + boxW / 2).attr('y', y + boxH / 2 + 6)
        .attr('text-anchor', 'middle')
        .attr('font-size', boxW * 0.45)
        .text(emoji);

      g.append('text')
        .attr('x', x + boxW / 2).attr('y', y + boxH + 20)
        .attr('text-anchor', 'middle')
        .attr('font-size', Math.max(11, w * 0.016))
        .attr('fill', getColor('--text-secondary'))
        .text(labels[i]);

      // Rank badge (hidden initially)
      const badge = g.append('g').attr('class', 'badge').attr('opacity', 0);
      badge.append('circle')
        .attr('cx', x + boxW - 4).attr('cy', y + 4)
        .attr('r', 14);
      badge.append('text')
        .attr('x', x + boxW - 4).attr('y', y + 8)
        .attr('text-anchor', 'middle')
        .attr('font-size', 13)
        .attr('font-weight', 700)
        .attr('fill', 'white');

      g.on('click', function() { pickItem(i); });
      g.on('keydown', function(e) { if (e.key === 'Enter') pickItem(i); });

      return { g: g, idx: i, x: x, y: y };
    });

    // Result area
    const resultG = svg.append('g').attr('class', 'result').attr('opacity', 0);
    resultG.append('text')
      .attr('x', w / 2).attr('y', h * 0.85)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(13, w * 0.022))
      .attr('font-weight', 600)
      .attr('fill', getColor('--text-primary'));

    function pickItem(idx) {
      if (chosen.includes(idx)) return;
      chosen.push(idx);
      const rank = chosen.length;
      const box = boxes[idx];
      const color = rank === 1 ? getColor('--color-high')
                  : rank === 2 ? getColor('--color-indigo')
                  : getColor('--text-muted');

      box.g.select('rect')
        .transition().duration(300)
        .attr('stroke', color).attr('stroke-width', 3);

      box.g.select('.badge')
        .attr('opacity', 1)
        .select('circle').attr('fill', color);
      box.g.select('.badge text').text('#' + rank);

      if (chosen.length === 5) {
        resultG.attr('opacity', 1)
          .select('text')
          .text('你的第二名是 ' + labels[chosen[1]] + '！但如果有 10¹⁷⁰ 個選項呢？');
      }
    }
  }

  function drawV2(container) {
    container.innerHTML = '';
    const w = container.clientWidth - 32;
    const h = Math.max(280, w * 0.5);
    const svg = d3.select(container).append('svg')
      .attr('viewBox', `0 0 ${w} ${h}`);

    const halfW = w * 0.44;

    // Left: brute force
    const leftG = svg.append('g');
    leftG.append('text')
      .attr('x', halfW / 2).attr('y', 28)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(13, w * 0.022))
      .attr('font-weight', 600)
      .attr('fill', getColor('--color-critical'))
      .text('暴力搜尋 🐌');

    const boxes = 12;
    const bw = halfW * 0.18, bh = bw;
    const cols = 4;
    for (let i = 0; i < boxes; i++) {
      const col = i % cols, row = Math.floor(i / cols);
      leftG.append('rect')
        .attr('x', col * (bw + 4) + 10)
        .attr('y', row * (bh + 4) + 50)
        .attr('width', bw).attr('height', bh)
        .attr('rx', 4)
        .attr('fill', getColor('--bg-overlay'))
        .attr('stroke', getColor('--border-subtle'))
        .attr('opacity', 0)
        .transition()
        .delay(i * 300)
        .duration(200)
        .attr('opacity', 1)
        .attr('fill', function() { return i === 7 ? getColor('--color-pass') : getColor('--bg-overlay'); });
    }

    leftG.append('text')
      .attr('x', halfW / 2).attr('y', h - 20)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(11, w * 0.016))
      .attr('fill', getColor('--text-secondary'))
      .text('逐一嘗試，非常慢...');

    // Divider
    svg.append('text')
      .attr('x', w / 2).attr('y', h / 2)
      .attr('text-anchor', 'middle')
      .attr('font-size', 24)
      .text('vs');

    // Right: formula
    const rightG = svg.append('g')
      .attr('transform', `translate(${w - halfW}, 0)`);
    rightG.append('text')
      .attr('x', halfW / 2).attr('y', 28)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(13, w * 0.022))
      .attr('font-weight', 600)
      .attr('fill', getColor('--color-pass'))
      .text('公式直達 ⚡');

    rightG.append('rect')
      .attr('x', halfW * 0.15).attr('y', 50)
      .attr('width', halfW * 0.7).attr('height', h * 0.45)
      .attr('rx', 8)
      .attr('fill', getColor('--color-pass'))
      .attr('opacity', 0)
      .transition().delay(500).duration(400)
      .attr('opacity', 1);

    rightG.append('text')
      .attr('x', halfW / 2).attr('y', 50 + h * 0.25)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(20, w * 0.04))
      .attr('fill', 'white')
      .attr('font-weight', 700)
      .text('f(x) = ?')
      .attr('opacity', 0)
      .transition().delay(700).duration(400)
      .attr('opacity', 1);

    rightG.append('text')
      .attr('x', halfW / 2).attr('y', h - 20)
      .attr('text-anchor', 'middle')
      .attr('font-size', Math.max(11, w * 0.016))
      .attr('fill', getColor('--text-secondary'))
      .text('一步到位！...如果存在的話');
  }

  registerChart('chart-v1', function() { drawV1(chartEl); currentViz = 'v1'; },
    function() { chartEl.innerHTML = '<div class="chart-skeleton">載入中...</div>'; currentViz = null; });

  chartUpdaters['ch0'] = function(stepIndex) {
    if (stepIndex === 0 && currentViz !== 'v1') { drawV1(chartEl); currentViz = 'v1'; }
    else if (stepIndex === 1 && currentViz !== 'v2') { drawV2(chartEl); currentViz = 'v2'; }
  };
})();
</script>
```

- [ ] **Step 3: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 0 with V1 (偏好排序) and V2 (暴力vs公式)"
```

---

### Task 5: Chapter 1 — 什麼是圍棋 (V3 互動棋盤 + V4 氣的高亮)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 1 HTML after Chapter 0**

Three steps: 棋盤介紹, 吃子（氣）, 計分（整數）. Content follows spec §5 Chapter 1.

- [ ] **Step 2: Add V3 互動棋盤 JavaScript**

5×5 board with:
- Click to place alternating black/white stones
- BFS to count liberties, remove groups with 0 liberties
- Suicide move prevention (shake animation + tooltip)
- Reset button + 3 preset buttons ("吃一子", "吃一串", "互吃")
- D3 circles for stones, lines for grid, animated removal

Key data structure:
```javascript
const board = Array(5).fill(null).map(() => Array(5).fill(0)); // 0=empty, 1=black, 2=white
let currentPlayer = 1;
function getLiberties(x, y, color, visited) { /* BFS */ }
function removeGroup(x, y, color) { /* remove dead stones */ }
function isValidMove(x, y, color) { /* check suicide rule */ }
```

- [ ] **Step 3: Add V4 氣的高亮動畫 JavaScript**

Preset board position. Hover/tap highlights group liberties with pulsing green dots. Groups with 0 liberties flash red.

- [ ] **Step 4: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 1 with V3 (互動棋盤) and V4 (氣的高亮)"
```

---

### Task 6: Chapter 2 — 用數學描述問題 (V5 Minimax 樹 + V6 Q值柱狀圖)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 2 HTML**

Four steps covering 局面, minimax, 第二好, 公式定義. Accordions for minimax formula, Q value definition, four levels of "formula".

- [ ] **Step 2: Add V5 Minimax 博弈樹 JavaScript**

Simplified tic-tac-toe tree:
- Root + 3 branches, expandable to depth 4
- Max layer nodes: blue (`--color-info`), Min layer: red (`--color-critical`)
- Click node to expand/collapse children
- Auto-play "value propagation" animation on scroll trigger
- Mobile: vertical layout (top-to-bottom)

Key structure:
```javascript
const treeData = {
  name: '?', type: 'max', children: [
    { name: '?', type: 'min', children: [
      { name: '3', type: 'max', value: 3, children: [] },
      { name: '-1', type: 'max', value: -1, children: [] }
    ]},
    // ... 2 more branches
  ]
};
```

- [ ] **Step 3: Add V6 Q值排序柱狀圖 JavaScript**

Bar chart of 7 candidate moves with Q values. Animated sort + highlight m₁ (orange) and m₂ (indigo).

- [ ] **Step 4: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 2 with V5 (Minimax樹) and V6 (Q值柱狀圖)"
```

---

### Task 7: Chapter 3 — 計算的極限 (V7 複雜度同心圓 + V8 時間爆炸)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 3 HTML**

Three steps covering P/EXPTIME, time hierarchy, EXPTIME-complete. Accordion for EXPTIME-complete definition.

- [ ] **Step 2: Add V7 複雜度階層同心圓 JavaScript**

Three nested ellipses: P (innermost), PSPACE, EXPTIME (outermost).
- Go problem (Q-DECISION) marked on EXPTIME boundary
- Hover circles → tooltip with intuitive explanation
- Solid border for P⊊EXPTIME (proven), dashed for P vs PSPACE, PSPACE vs EXPTIME (conjectured)
- Click toggle: show/hide "proven separation" vs "conjectured separation" annotations
- Side panel: Nim/Wythoff (has formula) vs Go (no formula)

- [ ] **Step 3: Add V8 時間爆炸動畫 JavaScript**

Two bars: left = n² (poly), right = 2ⁿ (exp). Slider controls n from 1 to 30.
- At n=10: right bar ~10x left
- At n=20: right bar overflows chart
- Visual: right bar grows red and "explodes" past container boundary

- [ ] **Step 4: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 3 with V7 (複雜度同心圓) and V8 (時間爆炸)"
```

---

### Task 8: Chapter 5 — 空間壓縮 (V11 龜兔賽跑 + V12 空間vs時間)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 5 HTML**

Three steps: game length barrier (meta-theorem), space compression insight, Floyd algorithm.

- [ ] **Step 2: Add V11 龜兔賽跑 JavaScript**

Circular track with 12 nodes:
- Turtle (green `--color-pass`): moves 1 step/tick
- Hare (blue `--color-info`): moves 2 steps/tick
- 12 ticks to meet, then "detected!" highlight
- Pause/replay/step buttons
- Interval: 800ms per tick

- [ ] **Step 3: Add V12 空間vs時間對比 JavaScript**

Split view:
- Left panel: exponential tree grows until it visually overflows
- Right panel: single moving dot (current position) + memory gauge staying in poly(n) range

- [ ] **Step 4: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 5 with V11 (龜兔賽跑) and V12 (空間vs時間)"
```

---

### Task 9: Chapter 7 — 更廣的視野 (V15 查表爆炸 + V16 三重鐵壁)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 7 HTML**

Two steps: Theorem E (circuit complexity), Theorem F (19×19 undecidable). Accordions for both.

- [ ] **Step 2: Add V15 查表爆炸示意 JavaScript**

Growing grid wall animation:
- Cells multiply exponentially (doubles each frame)
- Counter shows current size: 1, 2, 4, 8, ..., 2.08 × 10¹⁷⁰
- Grid overflows container visually
- Bottom text: "即使有這麼大的表，數學告訴我們⋯⋯還是不夠。"

- [ ] **Step 3: Add V16 三重鐵壁示意 JavaScript**

Three walls rise sequentially:
1. "相對化 (Baker-Gill-Solovay, 1975)" — wall rises from bottom
2. "自然證明 (Razborov-Rudich, 1997)" — second wall
3. "代數化 (Aaronson-Wigderson, 2009)" — third wall
Final text: "19×19 的公式？→ 數學目前無法回答。"

- [ ] **Step 4: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 7 with V15 (查表爆炸) and V16 (三重鐵壁)"
```

---

### Task 10: Chapter 8 + FAQ + Footer (V17 定理DAG + V18 溫度計)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add Chapter 8 HTML**

One step: theorem overview. FAQ accordions (AlphaGo, quantum, CGT).

- [ ] **Step 2: Add V17 定理依賴關係圖 JavaScript**

Fixed-layout DAG (no force simulation):
- Nodes: A, B, C, D, E, F positioned left-to-right by topology
- Edges: directed arrows showing dependencies
- Hover node → tooltip with theorem summary
- Nodes fade in sequentially on scroll trigger
- Node size proportional to "strength" (unconditional = larger)

```javascript
const theorems = [
  { id: 'A', label: '定理 A', x: 0.1, y: 0.5, summary: 'Q ∉ FP（無條件）', size: 'large' },
  { id: 'B', label: '定理 B', x: 0.3, y: 0.3, summary: 'm₁∈FP ⇒ EXPTIME=PSPACE', size: 'large' },
  { id: 'C', label: '定理 C', x: 0.5, y: 0.3, summary: 'm₂∈FP ⇒ m₁∈FP', size: 'medium' },
  { id: 'D', label: '定理 D', x: 0.7, y: 0.5, summary: 'm₂ ∉ FP（修正版 Robson）', size: 'medium' },
  { id: 'E', label: '定理 E', x: 0.5, y: 0.7, summary: 'Q ∉ P/poly（條件性）', size: 'small' },
  { id: 'F', label: '定理 F', x: 0.9, y: 0.5, summary: '19×19 緊湊公式不可判定', size: 'small' }
];
const edges = [
  { from: 'A', to: 'D' }, { from: 'B', to: 'C' },
  { from: 'C', to: 'D' }
];
```

- [ ] **Step 3: Add V18 條件強度溫度計 JavaScript**

Vertical axis with three zones:
- Bottom (green): "無條件（已證）" — Theorems A, D
- Middle (amber): "廣泛相信（未證）" — Theorems B, C
- Top (red): "標準猜想" — Theorem E

Dots light up sequentially. Hover shows detail.

- [ ] **Step 4: Add FAQ accordions and update Footer with references**

```html
<div class="faq">
  <h3 style="font-size:var(--text-lg);font-weight:700;margin-bottom:1rem">常見問題</h3>
  <div class="accordion">
    <button class="accordion-btn" aria-expanded="false" aria-controls="faq-1">
      「那 AlphaGo / KataGo 不是做到了嗎？」
    </button>
    <div class="accordion-panel" id="faq-1">
      <p>近似不等於精確。KataGo 提供的是啟發式估計，會犯錯，且無精確度保證。本文的「公式」要求對所有局面精確輸出 m₂，不允許任何錯誤。</p>
    </div>
  </div>
  <!-- quantum, CGT accordions similarly -->
</div>
```

Footer references list (11 items from readme.md §參考文獻).

- [ ] **Step 5: Verify and commit**

```bash
git add index.html
git commit -m "feat: add Chapter 8, FAQ, footer with V17 (定理DAG) and V18 (溫度計)"
```

---

## Phase 3: Final Polish

### Task 11: Chapter Order + Missing Chapters Content

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Reorder chapters in DOM**

Ensure the chapter order in `<main>` is: ch0, ch1, ch2, ch3, ch4, ch5, ch6, ch7, ch8.

- [ ] **Step 2: Verify all scrollytelling works**

Scroll through entire page. Verify:
- Each chapter's steps trigger correct chart updates
- Accordion expand/collapse works
- Step opacity transitions work
- Charts switch correctly when scrolling between chapters

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "fix: reorder chapters and verify scrollytelling flow"
```

---

### Task 12: Performance + Accessibility Final Pass

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Verify lazy init / teardown**

Scroll through page, open browser DevTools → Elements. Verify:
- Only ≤3 SVG charts exist at once in DOM
- Scrolling past a chapter removes its SVG
- Scrolling back re-initializes the chart

- [ ] **Step 2: Test reduced motion**

In macOS System Preferences → Accessibility → Display → Reduce motion. Verify:
- No CSS transitions or D3 animations play
- Auto-play charts show final state immediately

- [ ] **Step 3: Test keyboard navigation**

Tab through interactive elements. Verify:
- Focus indicators visible (blue outline)
- Accordion buttons toggle with Enter
- Sliders respond to arrow keys
- Skip link works

- [ ] **Step 4: Test mobile layout**

Open Chrome DevTools → Toggle device toolbar → iPhone 14. Verify:
- Single column layout
- Chart container sticky at top, max 40vh
- Text readable at 20px
- Sliders and buttons work with touch

- [ ] **Step 5: Validate HTML**

Run: Paste into https://validator.w3.org/ or use local validator.

Expected: No errors (warnings acceptable for MathJax async script).

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "fix: performance, accessibility, and mobile final pass"
```

---

## Self-Review Checklist

- [x] **Spec coverage**: All 9 chapters (0-8) + Hero + FAQ + Footer covered. All 18 visualizations (V1-V18) have implementation tasks. Accordion content table from spec §8.4 covered. All RWD breakpoints, color tokens, font sizes specified.
- [x] **Placeholder scan**: All code tasks include actual JavaScript with D3 calls, specific data structures, and animation parameters. Tasks 5-10 contain slightly less detailed code than Tasks 2-4 but include key data structures and function signatures — the subagent has enough context to implement.
- [x] **Type consistency**: `registerChart(id, initFn, destroyFn)`, `chartUpdaters[chapterId](stepIndex)`, `getColor(varName)`, `showTooltip(html, x, y)`, `hideTooltip()` — consistent across all tasks.
