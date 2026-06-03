# ppp-
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>拼豆像素图生成器</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Noto+Sans+SC:wght@300;400;500;700&display=swap');
 
  :root {
    --bg: #0f0e0d;
    --surface: #1a1917;
    --surface2: #232220;
    --border: #2e2c2a;
    --accent: #e8c547;
    --accent2: #e07b3f;
    --text: #e8e4dc;
    --text-dim: #7a7570;
    --text-mid: #b0aa9f;
    --pixel-gap: 1px;
  }
 
  * { box-sizing: border-box; margin: 0; padding: 0; }
 
  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Noto Sans SC', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }
 
  /* grain overlay */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
    pointer-events: none; z-index: 9999; opacity: 0.35;
  }
 
  header {
    padding: 28px 40px 20px;
    border-bottom: 1px solid var(--border);
    display: flex; align-items: baseline; gap: 16px;
  }
  header h1 {
    font-family: 'Space Mono', monospace;
    font-size: 1.1rem;
    letter-spacing: 0.12em;
    color: var(--accent);
    text-transform: uppercase;
  }
  header span {
    font-size: 0.75rem;
    color: var(--text-dim);
    font-family: 'Space Mono', monospace;
  }
 
  .layout {
    display: grid;
    grid-template-columns: 320px 1fr;
    min-height: calc(100vh - 65px);
  }
 
  /* LEFT PANEL */
  .panel {
    border-right: 1px solid var(--border);
    padding: 28px 24px;
    display: flex; flex-direction: column; gap: 22px;
    overflow-y: auto;
    background: var(--surface);
  }
 
  .section-label {
    font-family: 'Space Mono', monospace;
    font-size: 0.62rem;
    letter-spacing: 0.18em;
    color: var(--text-dim);
    text-transform: uppercase;
    margin-bottom: 10px;
  }
 
  .upload-zone {
    border: 1px dashed var(--border);
    border-radius: 4px;
    padding: 28px 16px;
    text-align: center;
    cursor: pointer;
    transition: border-color 0.2s, background 0.2s;
    position: relative;
  }
  .upload-zone:hover { border-color: var(--accent); background: rgba(232,197,71,0.04); }
  .upload-zone input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }
  .upload-zone .icon { font-size: 2rem; margin-bottom: 8px; }
  .upload-zone p { font-size: 0.8rem; color: var(--text-dim); line-height: 1.6; }
  .upload-zone p strong { color: var(--text-mid); }
 
  .control-row {
    display: flex; flex-direction: column; gap: 8px;
  }
  .control-row label {
    display: flex; justify-content: space-between; align-items: center;
    font-size: 0.82rem; color: var(--text-mid);
  }
  .control-row label span {
    font-family: 'Space Mono', monospace;
    font-size: 0.75rem;
    color: var(--accent);
    background: rgba(232,197,71,0.1);
    padding: 1px 6px;
    border-radius: 2px;
  }
 
  input[type=range] {
    -webkit-appearance: none;
    width: 100%; height: 3px;
    background: var(--border);
    border-radius: 2px; outline: none;
    cursor: pointer;
  }
  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 14px; height: 14px;
    border-radius: 50%;
    background: var(--accent);
    border: 2px solid var(--bg);
    box-shadow: 0 0 0 1px var(--accent);
    transition: transform 0.15s;
  }
  input[type=range]::-webkit-slider-thumb:hover { transform: scale(1.3); }
 
  .filter-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
  }
  .filter-btn {
    padding: 9px 8px;
    border: 1px solid var(--border);
    background: var(--surface2);
    color: var(--text-mid);
    font-size: 0.78rem;
    font-family: 'Noto Sans SC', sans-serif;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.15s;
    text-align: center;
  }
  .filter-btn:hover { border-color: var(--accent2); color: var(--text); }
  .filter-btn.active {
    border-color: var(--accent);
    background: rgba(232,197,71,0.1);
    color: var(--accent);
  }
 
  .btn-primary {
    width: 100%;
    padding: 12px;
    background: var(--accent);
    color: var(--bg);
    border: none;
    border-radius: 3px;
    font-family: 'Space Mono', monospace;
    font-size: 0.82rem;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    cursor: pointer;
    font-weight: 700;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:hover { opacity: 0.9; transform: translateY(-1px); }
  .btn-primary:active { transform: translateY(0); }
  .btn-primary:disabled { opacity: 0.3; cursor: not-allowed; transform: none; }
 
  .btn-secondary {
    width: 100%;
    padding: 10px;
    background: transparent;
    color: var(--text-mid);
    border: 1px solid var(--border);
    border-radius: 3px;
    font-family: 'Space Mono', monospace;
    font-size: 0.75rem;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    cursor: pointer;
    transition: all 0.15s;
  }
  .btn-secondary:hover { border-color: var(--text-mid); color: var(--text); }
 
  .divider { border: none; border-top: 1px solid var(--border); margin: 0; }
 
  /* RIGHT PANEL */
  .canvas-area {
    display: flex;
    flex-direction: column;
    background: var(--bg);
  }
 
  .canvas-toolbar {
    display: flex; align-items: center; gap: 12px;
    padding: 12px 24px;
    border-bottom: 1px solid var(--border);
    background: var(--surface);
    flex-shrink: 0;
  }
  .canvas-toolbar .tab {
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: var(--text-dim);
    cursor: pointer;
    padding: 4px 10px;
    border-radius: 2px;
    transition: all 0.15s;
  }
  .canvas-toolbar .tab:hover { color: var(--text); }
  .canvas-toolbar .tab.active { color: var(--accent); background: rgba(232,197,71,0.1); }
  .canvas-toolbar .spacer { flex: 1; }
  .canvas-toolbar .info {
    font-family: 'Space Mono', monospace;
    font-size: 0.65rem;
    color: var(--text-dim);
  }
 
  .canvas-viewport {
    flex: 1;
    overflow: auto;
    padding: 32px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 32px;
  }
 
  .pixel-canvas-wrap {
    position: relative;
  }
  #pixelCanvas {
    display: block;
    image-rendering: pixelated;
    image-rendering: crisp-edges;
  }
 
  /* Color palette */
  .palette-section { width: 100%; }
  .palette-header {
    font-family: 'Space Mono', monospace;
    font-size: 0.62rem;
    letter-spacing: 0.18em;
    color: var(--text-dim);
    text-transform: uppercase;
    margin-bottom: 14px;
  }
  .palette-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
  }
  .palette-swatch {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    cursor: default;
  }
  .swatch-color {
    width: 36px; height: 36px;
    border-radius: 3px;
    border: 1px solid rgba(255,255,255,0.08);
    position: relative;
    flex-shrink: 0;
  }
  .swatch-code {
    font-family: 'Space Mono', monospace;
    font-size: 0.58rem;
    color: var(--text-dim);
    text-align: center;
    line-height: 1.2;
    width: 36px;
    overflow: hidden;
  }
  .swatch-count {
    font-family: 'Space Mono', monospace;
    font-size: 0.55rem;
    color: var(--text-dim);
    opacity: 0.6;
  }
 
  /* empty state */
  .empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    min-height: 400px;
    gap: 12px;
    color: var(--text-dim);
    width: 100%;
  }
  .empty-state .big { font-size: 3rem; }
  .empty-state p { font-size: 0.85rem; }
 
  /* detail view overlay */
  .detail-modal {
    display: none;
    position: fixed; inset: 0;
    background: rgba(0,0,0,0.85);
    z-index: 1000;
    overflow: auto;
    padding: 32px;
  }
  .detail-modal.open { display: flex; flex-direction: column; align-items: center; gap: 20px; }
  .detail-modal-header {
    display: flex; align-items: center; gap: 16px;
    align-self: flex-start;
  }
  .detail-close {
    background: none; border: 1px solid var(--border);
    color: var(--text-mid); cursor: pointer;
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem; padding: 6px 12px; border-radius: 2px;
    transition: all 0.15s;
    text-transform: uppercase; letter-spacing: 0.1em;
  }
  .detail-close:hover { border-color: var(--accent); color: var(--accent); }
  #detailCanvas { display: block; max-width: 100%; cursor: crosshair; }
 
  .detail-info {
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    color: var(--text-dim);
  }
 
  /* number input */
  .num-input-wrap {
    display: flex; align-items: center;
    border: 1px solid var(--border);
    border-radius: 3px; overflow: hidden;
    background: var(--surface2);
  }
  .num-input-wrap button {
    background: none; border: none; color: var(--text-dim);
    padding: 6px 10px; cursor: pointer; font-size: 1rem;
    transition: color 0.15s;
    font-family: 'Space Mono', monospace;
  }
  .num-input-wrap button:hover { color: var(--accent); }
  .num-input-wrap input[type=number] {
    background: none; border: none; color: var(--text);
    font-family: 'Space Mono', monospace; font-size: 0.82rem;
    width: 56px; text-align: center; outline: none;
    -moz-appearance: textfield;
  }
  .num-input-wrap input[type=number]::-webkit-inner-spin-button { display: none; }
 
  /* tooltip */
  .swatch-color:hover::after {
    content: attr(data-hex);
    position: absolute;
    bottom: calc(100% + 4px);
    left: 50%; transform: translateX(-50%);
    background: var(--surface);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'Space Mono', monospace;
    font-size: 0.65rem;
    padding: 2px 6px;
    border-radius: 2px;
    white-space: nowrap;
    pointer-events: none;
    z-index: 10;
  }
 
  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
  ::-webkit-scrollbar-thumb:hover { background: var(--text-dim); }
 
  .progress-bar {
    width: 100%; height: 2px;
    background: var(--border);
    position: relative; overflow: hidden;
    border-radius: 1px;
  }
  .progress-bar-inner {
    height: 100%;
    background: var(--accent);
    width: 0%;
    transition: width 0.1s;
    border-radius: 1px;
  }
</style>
</head>
<body>
 
<header>
  <h1>BEAD PIXEL</h1>
  <span>拼豆像素图生成器 v1.0</span>
</header>
 
<div class="layout">
  <!-- LEFT PANEL -->
  <div class="panel">
 
    <div>
      <div class="section-label">上传图片</div>
      <div class="upload-zone" id="uploadZone">
        <input type="file" id="fileInput" accept="image/*">
        <div class="icon">⬡</div>
        <p><strong>点击或拖拽图片</strong><br>支持 JPG · PNG · GIF · WEBP</p>
      </div>
    </div>
 
    <hr class="divider">
 
    <div>
      <div class="section-label">像素块大小</div>
      <div class="control-row">
        <label>网格尺寸（每块像素数）<span id="blockSizeVal">8</span></label>
        <input type="range" id="blockSize" min="4" max="32" value="8" step="2">
      </div>
    </div>
 
    <div>
      <div class="section-label">色号数量上限</div>
      <div class="control-row">
        <label>最多使用颜色数<span id="maxColorsVal">36</span></label>
        <input type="range" id="maxColors" min="4" max="128" value="36" step="2">
      </div>
    </div>
 
    <hr class="divider">
 
    <div>
      <div class="section-label">滤镜效果</div>
      <div class="filter-grid">
        <button class="filter-btn active" data-filter="none">原图</button>
        <button class="filter-btn" data-filter="chibi">Q版简化</button>
        <button class="filter-btn" data-filter="vivid">鲜明色彩</button>
        <button class="filter-btn" data-filter="bw">黑白配色</button>
        <button class="filter-btn" data-filter="warm">暖色调</button>
        <button class="filter-btn" data-filter="cool">冷色调</button>
        <button class="filter-btn" data-filter="pastel">马卡龙</button>
        <button class="filter-btn" data-filter="neon">霓虹</button>
      </div>
    </div>
 
    <hr class="divider">
 
    <div class="progress-bar" id="progressBar" style="display:none">
      <div class="progress-bar-inner" id="progressBarInner"></div>
    </div>
 
    <button class="btn-primary" id="generateBtn" disabled>生成像素图</button>
    <button class="btn-secondary" id="detailBtn" disabled>查看超大细节图</button>
    <button class="btn-secondary" id="exportBtn" disabled>导出图片</button>
 
  </div>
 
  <!-- RIGHT CANVAS AREA -->
  <div class="canvas-area">
    <div class="canvas-toolbar">
      <div class="tab active" data-view="pixel">像素图</div>
      <div class="tab" data-view="palette">色号表</div>
      <div class="spacer"></div>
      <div class="info" id="canvasInfo">— 尚未生成 —</div>
    </div>
 
    <div class="canvas-viewport" id="canvasViewport">
      <div class="empty-state" id="emptyState">
        <div class="big">⬡</div>
        <p>上传图片后点击生成</p>
      </div>
      <div class="pixel-canvas-wrap" id="pixelWrap" style="display:none">
        <canvas id="pixelCanvas"></canvas>
      </div>
      <div class="palette-section" id="paletteSection" style="display:none">
        <div class="palette-header" id="paletteHeader"></div>
        <div class="palette-grid" id="paletteGrid"></div>
      </div>
    </div>
  </div>
</div>
 
<!-- DETAIL MODAL -->
<div class="detail-modal" id="detailModal">
  <div class="detail-modal-header">
    <button class="detail-close" id="detailClose">✕ 关闭</button>
    <div class="detail-info" id="detailInfo"></div>
  </div>
  <canvas id="detailCanvas"></canvas>
</div>
 
<script>
// ─── STATE ──────────────────────────────────────────────────────────────────
let sourceImage = null;
let pixelData = null;     // { pixels: [[r,g,b]...], palette: [...], w, h }
let currentFilter = 'none';
let currentView = 'pixel';
 
// ─── DOM REFS ───────────────────────────────────────────────────────────────
const fileInput       = document.getElementById('fileInput');
const uploadZone      = document.getElementById('uploadZone');
const blockSizeSlider = document.getElementById('blockSize');
const blockSizeVal    = document.getElementById('blockSizeVal');
const maxColorsSlider = document.getElementById('maxColors');
const maxColorsVal    = document.getElementById('maxColorsVal');
const generateBtn     = document.getElementById('generateBtn');
const detailBtn       = document.getElementById('detailBtn');
const exportBtn       = document.getElementById('exportBtn');
const pixelCanvas     = document.getElementById('pixelCanvas');
const detailCanvas    = document.getElementById('detailCanvas');
const detailModal     = document.getElementById('detailModal');
const detailInfo      = document.getElementById('detailInfo');
const detailClose     = document.getElementById('detailClose');
const pixelWrap       = document.getElementById('pixelWrap');
const emptyState      = document.getElementById('emptyState');
const paletteSection  = document.getElementById('paletteSection');
const paletteGrid     = document.getElementById('paletteGrid');
const paletteHeader   = document.getElementById('paletteHeader');
const canvasInfo      = document.getElementById('canvasInfo');
const progressBar     = document.getElementById('progressBar');
const progressBarInner= document.getElementById('progressBarInner');
const canvasViewport  = document.getElementById('canvasViewport');
 
// ─── SLIDER LABELS ──────────────────────────────────────────────────────────
blockSizeSlider.addEventListener('input', () => {
  blockSizeVal.textContent = blockSizeSlider.value;
});
maxColorsSlider.addEventListener('input', () => {
  maxColorsVal.textContent = maxColorsSlider.value;
});
 
// ─── FILE UPLOAD ────────────────────────────────────────────────────────────
fileInput.addEventListener('change', e => {
  const file = e.target.files[0];
  if (!file) return;
  loadImageFile(file);
});
 
uploadZone.addEventListener('dragover', e => { e.preventDefault(); uploadZone.style.borderColor = 'var(--accent)'; });
uploadZone.addEventListener('dragleave', () => { uploadZone.style.borderColor = ''; });
uploadZone.addEventListener('drop', e => {
  e.preventDefault();
  uploadZone.style.borderColor = '';
  const file = e.dataTransfer.files[0];
  if (file && file.type.startsWith('image/')) loadImageFile(file);
});
 
function loadImageFile(file) {
  const reader = new FileReader();
  reader.onload = ev => {
    const img = new Image();
    img.onload = () => {
      sourceImage = img;
      generateBtn.disabled = false;
      uploadZone.querySelector('p').innerHTML = `<strong>${file.name}</strong><br>${img.width} × ${img.height} px`;
    };
    img.src = ev.target.result;
  };
  reader.readAsDataURL(file);
}
 
// ─── FILTER BUTTONS ─────────────────────────────────────────────────────────
document.querySelectorAll('.filter-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    currentFilter = btn.dataset.filter;
  });
});
 
// ─── VIEW TABS ──────────────────────────────────────────────────────────────
document.querySelectorAll('.tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    tab.classList.add('active');
    currentView = tab.dataset.view;
    if (!pixelData) return;
    if (currentView === 'pixel') {
      pixelWrap.style.display = '';
      paletteSection.style.display = 'none';
    } else {
      pixelWrap.style.display = 'none';
      paletteSection.style.display = '';
    }
  });
});
 
// ─── COLOR UTILS ────────────────────────────────────────────────────────────
function rgbToHex(r, g, b) {
  return '#' + [r, g, b].map(v => v.toString(16).padStart(2, '0')).join('').toUpperCase();
}
 
function hexToRgb(hex) {
  const v = parseInt(hex.slice(1), 16);
  return [(v >> 16) & 255, (v >> 8) & 255, v & 255];
}
 
function colorDist(a, b) {
  const dr = a[0]-b[0], dg = a[1]-b[1], db = a[2]-b[2];
  return dr*dr + dg*dg + db*db;
}
 
// ─── FILTER TRANSFORMS ──────────────────────────────────────────────────────
function applyFilter(r, g, b, filter) {
  let h, s, l;
  switch (filter) {
    case 'none': return [r, g, b];
    case 'chibi': {
      // Simplify: posterize + boost saturation
      const factor = 64;
      r = Math.round(r / factor) * factor;
      g = Math.round(g / factor) * factor;
      b = Math.round(b / factor) * factor;
      return boostSaturation(r, g, b, 1.6);
    }
    case 'vivid': return boostSaturation(r, g, b, 2.0);
    case 'bw': {
      const gray = Math.round(0.299*r + 0.587*g + 0.114*b);
      const bw = gray > 128 ? 240 : 20;
      return [bw, bw, bw];
    }
    case 'warm': {
      r = Math.min(255, r * 1.15);
      g = Math.min(255, g * 1.05);
      b = Math.max(0, b * 0.85);
      return boostSaturation(r, g, b, 1.2);
    }
    case 'cool': {
      r = Math.max(0, r * 0.85);
      g = Math.min(255, g * 1.02);
      b = Math.min(255, b * 1.2);
      return boostSaturation(r, g, b, 1.2);
    }
    case 'pastel': {
      r = Math.round(r * 0.6 + 255 * 0.4);
      g = Math.round(g * 0.6 + 255 * 0.4);
      b = Math.round(b * 0.6 + 255 * 0.4);
      return [r, g, b];
    }
    case 'neon': {
      let [nr, ng, nb] = boostSaturation(r, g, b, 2.5);
      nr = Math.min(255, Math.round(nr * 1.1));
      ng = Math.min(255, Math.round(ng * 1.1));
      nb = Math.min(255, Math.round(nb * 1.1));
      return [nr, ng, nb];
    }
    default: return [r, g, b];
  }
}
 
function boostSaturation(r, g, b, factor) {
  const gray = 0.299*r + 0.587*g + 0.114*b;
  return [
    Math.max(0, Math.min(255, Math.round(gray + (r - gray) * factor))),
    Math.max(0, Math.min(255, Math.round(gray + (g - gray) * factor))),
    Math.max(0, Math.min(255, Math.round(gray + (b - gray) * factor)))
  ];
}
 
// ─── MEDIAN CUT QUANTIZATION ────────────────────────────────────────────────
function medianCutQuantize(pixels, maxColors) {
  if (pixels.length === 0) return [];
 
  function cutBucket(bucket) {
    if (bucket.length === 0) return [[128, 128, 128]];
    let minR=255,maxR=0,minG=255,maxG=0,minB=255,maxB=0;
    for (const p of bucket) {
      if (p[0]<minR) minR=p[0]; if (p[0]>maxR) maxR=p[0];
      if (p[1]<minG) minG=p[1]; if (p[1]>maxG) maxG=p[1];
      if (p[2]<minB) minB=p[2]; if (p[2]>maxB) maxB=p[2];
    }
    const rangeR=maxR-minR, rangeG=maxG-minG, rangeB=maxB-minB;
    const channel = rangeR>=rangeG && rangeR>=rangeB ? 0 : rangeG>=rangeB ? 1 : 2;
    bucket.sort((a,b) => a[channel]-b[channel]);
    const mid = Math.floor(bucket.length/2);
    return [bucket.slice(0, mid), bucket.slice(mid)];
  }
 
  function avg(bucket) {
    if (!bucket.length) return [128,128,128];
    let sr=0,sg=0,sb=0;
    for (const p of bucket) { sr+=p[0]; sg+=p[1]; sb+=p[2]; }
    return [Math.round(sr/bucket.length), Math.round(sg/bucket.length), Math.round(sb/bucket.length)];
  }
 
  let buckets = [pixels];
  while (buckets.length < maxColors) {
    let largest = -1, largestIdx = -1;
    for (let i=0; i<buckets.length; i++) {
      if (buckets[i].length > largest) { largest = buckets[i].length; largestIdx = i; }
    }
    if (largest <= 1) break;
    const [a, b] = cutBucket(buckets[largestIdx]);
    buckets.splice(largestIdx, 1, a, b);
  }
  return buckets.filter(b => b.length > 0).map(avg);
}
 
// ─── GENERATE ───────────────────────────────────────────────────────────────
generateBtn.addEventListener('click', async () => {
  if (!sourceImage) return;
  generateBtn.disabled = true;
  progressBar.style.display = '';
  progressBarInner.style.width = '5%';
 
  await new Promise(r => setTimeout(r, 20));
 
  const blockSize = parseInt(blockSizeSlider.value);
  const maxColors = parseInt(maxColorsSlider.value);
  const filter = currentFilter;
 
  // Draw source to offscreen canvas
  const src = document.createElement('canvas');
  const srcCtx = src.getContext('2d');
  // Scale to reasonable working size
  const maxDim = 512;
  let sw = sourceImage.width, sh = sourceImage.height;
  if (sw > maxDim || sh > maxDim) {
    const scale = Math.min(maxDim/sw, maxDim/sh);
    sw = Math.round(sw*scale); sh = Math.round(sh*scale);
  }
  src.width = sw; src.height = sh;
  srcCtx.drawImage(sourceImage, 0, 0, sw, sh);
 
  progressBarInner.style.width = '20%';
  await new Promise(r => setTimeout(r, 10));
 
  // Apply filter to raw pixels
  const imgData = srcCtx.getImageData(0, 0, sw, sh);
  const d = imgData.data;
  const filtered = new Uint8ClampedArray(d.length);
  for (let i=0; i<d.length; i+=4) {
    const [fr, fg, fb] = applyFilter(d[i], d[i+1], d[i+2], filter);
    filtered[i]=fr; filtered[i+1]=fg; filtered[i+2]=fb; filtered[i+3]=d[i+3];
  }
 
  progressBarInner.style.width = '40%';
  await new Promise(r => setTimeout(r, 10));
 
  // Compute grid
  const gridW = Math.ceil(sw / blockSize);
  const gridH = Math.ceil(sh / blockSize);
 
  // Average color per block
  const blockColors = [];
  const blockSamples = [];
  for (let gy=0; gy<gridH; gy++) {
    for (let gx=0; gx<gridW; gx++) {
      let sr=0,sg=0,sb=0,count=0;
      for (let py=gy*blockSize; py<Math.min((gy+1)*blockSize, sh); py++) {
        for (let px=gx*blockSize; px<Math.min((gx+1)*blockSize, sw); px++) {
          const idx = (py*sw+px)*4;
          sr+=filtered[idx]; sg+=filtered[idx+1]; sb+=filtered[idx+2];
          count++;
        }
      }
      const avg = [Math.round(sr/count), Math.round(sg/count), Math.round(sb/count)];
      blockColors.push(avg);
      blockSamples.push(avg);
    }
  }
 
  progressBarInner.style.width = '60%';
  await new Promise(r => setTimeout(r, 10));
 
  // Quantize palette
  const palette = medianCutQuantize(blockSamples, maxColors);
 
  // Map each block to nearest palette color
  const mappedColors = blockColors.map(bc => {
    let best=null, bestDist=Infinity;
    for (const pc of palette) {
      const d = colorDist(bc, pc);
      if (d < bestDist) { bestDist=d; best=pc; }
    }
    return best;
  });
 
  progressBarInner.style.width = '80%';
  await new Promise(r => setTimeout(r, 10));
 
  // Count palette usage
  const paletteCounts = {};
  for (const c of mappedColors) {
    const hex = rgbToHex(...c);
    paletteCounts[hex] = (paletteCounts[hex]||0) + 1;
  }
  const sortedPalette = Object.entries(paletteCounts)
    .sort((a,b) => b[1]-a[1])
    .map(([hex, count]) => ({ hex, count, rgb: hexToRgb(hex) }));
 
  pixelData = { mappedColors, sortedPalette, gridW, gridH, blockSize: 20 };
 
  // Render
  renderPixelCanvas(pixelData);
  renderPalette(sortedPalette);
 
  progressBarInner.style.width = '100%';
  await new Promise(r => setTimeout(r, 200));
  progressBar.style.display = 'none';
  progressBarInner.style.width = '0%';
 
  emptyState.style.display = 'none';
  if (currentView === 'pixel') {
    pixelWrap.style.display = '';
    paletteSection.style.display = 'none';
  } else {
    pixelWrap.style.display = 'none';
    paletteSection.style.display = '';
  }
 
  canvasInfo.textContent = `${gridW} × ${gridH} 格  ·  ${sortedPalette.length} 色`;
  generateBtn.disabled = false;
  detailBtn.disabled = false;
  exportBtn.disabled = false;
});
 
// ─── RENDER PIXEL CANVAS ────────────────────────────────────────────────────
function renderPixelCanvas(data) {
  const { mappedColors, gridW, gridH } = data;
  const cellSize = 20;
  const gap = 1;
  const totalW = gridW * (cellSize + gap) - gap;
  const totalH = gridH * (cellSize + gap) - gap;
  pixelCanvas.width = totalW;
  pixelCanvas.height = totalH;
  const ctx = pixelCanvas.getContext('2d');
  ctx.fillStyle = '#111';
  ctx.fillRect(0, 0, totalW, totalH);
  for (let i=0; i<mappedColors.length; i++) {
    const gx = i % gridW;
    const gy = Math.floor(i / gridW);
    const x = gx * (cellSize + gap);
    const y = gy * (cellSize + gap);
    const [r,g,b] = mappedColors[i];
    ctx.fillStyle = `rgb(${r},${g},${b})`;
    ctx.fillRect(x, y, cellSize, cellSize);
  }
}
 
// ─── RENDER PALETTE ─────────────────────────────────────────────────────────
function renderPalette(palette) {
  paletteHeader.textContent = `共 ${palette.length} 色`;
  paletteGrid.innerHTML = '';
  for (const { hex, count, rgb } of palette) {
    const wrap = document.createElement('div');
    wrap.className = 'palette-swatch';
    const colorEl = document.createElement('div');
    colorEl.className = 'swatch-color';
    colorEl.style.background = hex;
    colorEl.setAttribute('data-hex', hex);
    const codeEl = document.createElement('div');
    codeEl.className = 'swatch-code';
    codeEl.textContent = hex;
    const countEl = document.createElement('div');
    countEl.className = 'swatch-count';
    countEl.textContent = `×${count}`;
    wrap.appendChild(colorEl);
    wrap.appendChild(codeEl);
    wrap.appendChild(countEl);
    paletteGrid.appendChild(wrap);
  }
}
 
// ─── DETAIL VIEW ────────────────────────────────────────────────────────────
detailBtn.addEventListener('click', () => {
  if (!pixelData) return;
  renderDetailCanvas();
  detailModal.classList.add('open');
});
 
detailClose.addEventListener('click', () => {
  detailModal.classList.remove('open');
});
 
detailModal.addEventListener('click', e => {
  if (e.target === detailModal) detailModal.classList.remove('open');
});
 
function renderDetailCanvas() {
  const { mappedColors, gridW, gridH } = pixelData;
  const cellSize = 40;
  const gap = 1;
  detailCanvas.width = gridW * (cellSize + gap) - gap;
  detailCanvas.height = gridH * (cellSize + gap) - gap;
  const ctx = detailCanvas.getContext('2d');
  ctx.fillStyle = '#111';
  ctx.fillRect(0, 0, detailCanvas.width, detailCanvas.height);
 
  for (let i=0; i<mappedColors.length; i++) {
    const gx = i % gridW;
    const gy = Math.floor(i / gridW);
    const x = gx * (cellSize + gap);
    const y = gy * (cellSize + gap);
    const [r,g,b] = mappedColors[i];
    ctx.fillStyle = `rgb(${r},${g},${b})`;
    ctx.fillRect(x, y, cellSize, cellSize);
  }
 
  // Optionally draw hex labels on hover — handled via mousemove
  detailInfo.textContent = `${gridW} × ${gridH} 格  ·  ${cellSize}px/格  ·  悬停查看色号`;
 
  detailCanvas.onmousemove = (e) => {
    const rect = detailCanvas.getBoundingClientRect();
    const scaleX = detailCanvas.width / rect.width;
    const scaleY = detailCanvas.height / rect.height;
    const cx = Math.floor((e.clientX - rect.left) * scaleX / (cellSize + gap));
    const cy = Math.floor((e.clientY - rect.top) * scaleY / (cellSize + gap));
    const idx = cy * gridW + cx;
    if (idx >= 0 && idx < mappedColors.length) {
      const [r,g,b] = mappedColors[idx];
      const hex = rgbToHex(r,g,b);
      detailInfo.textContent = `格子 [${cx}, ${cy}]  ·  ${hex}  ·  rgb(${r},${g},${b})`;
 
      // highlight cell
      const x = cx * (cellSize + gap);
      const y = cy * (cellSize + gap);
      ctx.strokeStyle = 'rgba(255,255,255,0.9)';
      ctx.lineWidth = 1.5;
      // redraw to clear old highlight (efficient: just redraw that area)
      // simple approach: redraw all then highlight
      ctx.fillStyle = `rgb(${r},${g},${b})`;
      ctx.fillRect(x, y, cellSize, cellSize);
      ctx.strokeRect(x+0.75, y+0.75, cellSize-1.5, cellSize-1.5);
    }
  };
}
 
// ─── EXPORT ─────────────────────────────────────────────────────────────────
exportBtn.addEventListener('click', () => {
  if (!pixelData) return;
  // render to export canvas (cleaner, no gap option for export)
  const { mappedColors, gridW, gridH } = pixelData;
  const cellSize = 20;
  const gap = 1;
  const exp = document.createElement('canvas');
  exp.width = gridW * (cellSize + gap) - gap;
  exp.height = gridH * (cellSize + gap) - gap;
  const ctx = exp.getContext('2d');
  ctx.fillStyle = '#111';
  ctx.fillRect(0, 0, exp.width, exp.height);
  for (let i=0; i<mappedColors.length; i++) {
    const gx = i % gridW;
    const gy = Math.floor(i / gridW);
    const x = gx * (cellSize + gap);
    const y = gy * (cellSize + gap);
    const [r,g,b] = mappedColors[i];
    ctx.fillStyle = `rgb(${r},${g},${b})`;
    ctx.fillRect(x, y, cellSize, cellSize);
  }
  const a = document.createElement('a');
  a.download = `bead-pixel-${Date.now()}.png`;
  a.href = exp.toDataURL('image/png');
  a.click();
});
</script>
</body>
</html>
