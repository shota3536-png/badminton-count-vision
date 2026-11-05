<!DOCTYPE html>  
<html lang="ja">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">  
<title>COURTVISION - バドミントン ミス記録 & 集計</title>  
<script src="https://cdn.tailwindcss.com"></script>  
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>  
  
<!-- PWA用マニフェストファイルの読み込み -->  
<link rel="manifest" href="manifest.json">  
  
<style>  
:root {  
  --app-bg: #0b1121;  
  --court-bg: #183d29;  
  --court-border: rgba(255, 255, 255, 0.8);  
  --court-line: rgba(255, 255, 255, 0.35);  
  --grid-line: rgba(255, 255, 255, 0.15);  
}  
body {  
  background-color: var(--app-bg);  
  overscroll-behavior-y: none;  
  -webkit-tap-highlight-color: transparent;  
  touch-action: manipulation;  
  user-select: none;  
  font-family: 'Inter', sans-serif;  
}  
.court-container {  
  background-color: var(--court-bg);  
  border: 3px solid var(--court-border);  
  position: relative;  
  width: min(72vw, calc(76vh / 2.1));  
  aspect-ratio: 1 / 2.1;  
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);  
}  
.real-court-lines {  
  position: absolute;  
  inset: 0;  
  pointer-events: none;  
  z-index: 1;  
}  
.real-court-lines::before, .real-court-lines::after {  
  content: '';  
  position: absolute;  
  top: 0;  
  bottom: 0;  
  width: 7.5%;  
  border-left: 1px solid var(--court-line);  
  border-right: 1px solid var(--court-line);  
}  
.real-court-lines::before { left: 0; }  
.real-court-lines::after { right: 0; }  
.inner-lines {  
  position: absolute;  
  inset: 0;  
}  
.center-line-top {  
  position: absolute;  
  top: 0;  
  bottom: 65%;  
  left: 50%;  
  border-left: 1px solid var(--court-line);  
  transform: translateX(-50%);  
}  
.center-line-bottom {  
  position: absolute;  
  top: 65%;  
  bottom: 0;  
  left: 50%;  
  border-left: 1px solid var(--court-line);  
  transform: translateX(-50%);  
}  
.short-service-top {  
  position: absolute;  
  top: 35%;  
  left: 0;  
  right: 0;  
  border-top: 1px solid var(--court-line);  
}  
.short-service-bottom {  
  position: absolute;  
  bottom: 35%;  
  left: 0;  
  right: 0;  
  border-top: 1px solid var(--court-line);  
}  
.long-service-top {  
  position: absolute;  
  top: 7.5%;  
  left: 7.5%;  
  right: 7.5%;  
  border-top: 1px solid var(--court-line);  
}  
.long-service-bottom {  
  position: absolute;  
  bottom: 7.5%;  
  left: 7.5%;  
  right: 7.5%;  
  border-top: 1px solid var(--court-line);  
}  
.net-visual {  
  position: absolute;  
  top: 50%;  
  left: -12px;  
  right: -12px;  
  height: 4px;  
  background-color: rgba(255,255,255,0.9);  
  transform: translateY(-50%);  
  z-index: 10;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  box-shadow: 0 4px 6px rgba(0,0,0,0.3);  
}  
.net-pole {  
  width: 8px;  
  height: 24px;  
  background: #e2e8f0;  
  border: 1px solid #94a3b8;  
  border-radius: 2px;  
}  
.interactive-grid {  
  position: absolute;  
  inset: 0;  
  display: flex;  
  flex-direction: column;  
  z-index: 20;  
  pointer-events: none; /* クリックイベントはキャンバスで拾うため無効化 */  
}  
.zone-cell {  
  position: relative;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  border: 1px solid var(--grid-line);  
}  
@keyframes popIn {  
  0% { transform: scale(0.5); opacity: 0; }  
  60% { transform: scale(1.2); opacity: 1; }  
  100% { transform: scale(1); opacity: 1; }  
}  
.anim-pop {  
  animation: popIn 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;  
}  
#trajectory-canvas {  
  position: absolute;  
  inset: 0;  
  width: 100%;  
  height: 100%;  
  z-index: 25;  
  pointer-events: auto; /* 線を描くために有効化 */  
}  
::-webkit-scrollbar { display: none; }  
.toolbar-btn {  
  font-size: 0.75rem;  
  font-weight: 700;  
  padding: 0.6rem 0.2rem;  
  border-radius: 0.5rem;  
  border: 2px solid transparent;  
  transition: all 0.2s ease;  
  text-align: center;  
  line-height: 1.2;  
  word-break: keep-all;  
}  
</style>  
</head>  
<body class="flex flex-col h-screen text-white">  
  
<!-- ヘッダー部 -->  
<header class="p-3 flex items-center justify-between border-b border-gray-800 bg-[#0b1121]/90 backdrop-blur-md z-30 shrink-0">  
  <div class="flex items-center gap-3">  
    <div class="w-8 h-8 rounded bg-teal-900 border border-teal-500 flex items-center justify-center shadow-[0_0_10px_rgba(20,184,166,0.3)]">  
      <svg class="w-4 h-4 text-teal-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path></svg>  
    </div>  
    <div>  
      <h1 class="font-black text-lg tracking-wider">COURTVISION <span class="text-teal-500 text-sm font-bold">AI</span></h1>  
    </div>  
  </div>  
  <button onclick="openGallery()" class="px-3 py-1.5 bg-gray-800 border border-gray-700 hover:bg-gray-700 rounded-lg text-xs font-bold flex items-center gap-1.5 text-teal-400 transition-colors">  
    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"/></svg>  
    保存BOX (<span id="saved-count">0</span>)  
  </button>  
</header>  
  
<!-- メインエリア -->  
<main class="flex-grow flex flex-row items-center justify-center p-2 sm:p-4 gap-2 sm:gap-4 overflow-hidden relative">  
  <!-- 左側：コートエリア -->  
  <div id="capture-area" class="flex flex-col items-center justify-center gap-1.5 shrink-0 bg-[#0b1121] p-2 rounded-xl">  
    <input id="player-top" type="text" placeholder="奥の選手名" class="text-center bg-gray-800/80 text-white border border-gray-600 rounded py-1 px-2 text-xs sm:text-sm focus:outline-none focus:border-teal-500 transition-colors shadow-sm" style="width: min(72vw, calc(76vh / 2.1));">  
      
    <!-- コート本体 -->  
    <div class="court-container shrink-0">  
      <div class="real-court-lines">  
        <div class="inner-lines">  
          <div class="center-line-top"></div>  
          <div class="center-line-bottom"></div>  
          <div class="short-service-top"></div>  
          <div class="short-service-bottom"></div>  
          <div class="long-service-top"></div>  
          <div class="long-service-bottom"></div>  
        </div>  
      </div>  
      <div class="net-visual">  
        <div class="net-pole -ml-[2px]"></div>  
        <div class="net-pole -mr-[2px]"></div>  
      </div>  
      <div class="interactive-grid h-full w-full">  
        <div class="flex-1 grid grid-cols-3 grid-rows-3" id="grid-opponent"></div>  
        <div class="flex-1 grid grid-cols-3 grid-rows-3" id="grid-player"></div>  
      </div>  
      <canvas id="trajectory-canvas" width="400" height="840"></canvas>  
    </div>  
      
    <input id="player-bottom" type="text" placeholder="手前の選手名" class="text-center bg-gray-800/80 text-white border border-gray-600 rounded py-1 px-2 text-xs sm:text-sm focus:outline-none focus:border-teal-500 transition-colors shadow-sm" style="width: min(72vw, calc(76vh / 2.1));">  
  </div>  
  
  <!-- 右側：ツールバー -->  
  <div class="flex flex-col justify-center gap-2 z-30 w-16 sm:w-20 shrink-0">  
    <button class="mode-btn toolbar-btn border-white opacity-100 shadow-lg bg-[#ef4444]/20 text-[#ef4444]" data-mode="smash">スマ</button>  
    <button class="mode-btn toolbar-btn opacity-50 bg-[#22c55e]/20 text-[#22c55e]" data-mode="drop">ドロ</button>  
    <button class="mode-btn toolbar-btn opacity-50 bg-[#eab308]/20 text-[#eab308]" data-mode="clear">クリ</button>  
    <button class="mode-btn toolbar-btn opacity-50 bg-[#3b82f6]/20 text-[#3b82f6]" data-mode="hairpin">ヘア<br>ロブ</button>  
    <div class="h-px w-full bg-gray-700 my-0.5"></div>  
    <button onclick="saveToGallery()" class="toolbar-btn border-blue-800 bg-blue-900/30 text-blue-400 hover:bg-blue-800/50">保存</button>  
    <button onclick="openModal()" class="toolbar-btn border-teal-800 bg-teal-900/30 text-teal-400 hover:bg-teal-800/50">リセット</button>  
  </div>  
</main>  
  
<!-- 保存BOX・集計モーダル -->  
<div id="gallery-modal" class="fixed inset-0 bg-black/85 z-50 flex items-center justify-center opacity-0 pointer-events-none transition-opacity duration-300 backdrop-blur-md p-4">  
  <div class="bg-[#111827] border border-gray-700 rounded-2xl w-full max-w-2xl max-h-[90vh] flex flex-col shadow-2xl transform scale-95 transition-transform duration-300 overflow-hidden" id="gallery-modal-content">  
    <div class="p-4 border-b border-gray-800 flex items-center justify-between bg-gray-900/50">  
      <div class="flex items-center gap-2">  
        <h3 class="text-lg font-bold text-white">保存シート＆集計BOX</h3>  
        <span class="text-xs text-teal-400 bg-teal-950 px-2 py-0.5 rounded-full border border-teal-800" id="modal-item-count">0件</span>  
      </div>  
      <div class="flex items-center gap-2">  
        <button onclick="exportCSV()" class="px-3 py-1.5 bg-emerald-700 hover:bg-emerald-600 text-white rounded-lg text-xs font-bold transition-colors shadow flex items-center gap-1">  
          <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/></svg>  
          CSV出力  
        </button>  
        <button onclick="closeGallery()" class="text-gray-400 hover:text-white p-1 rounded-lg hover:bg-gray-800 transition-colors">  
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>  
        </button>  
      </div>  
    </div>  
    <div class="p-4 overflow-y-auto flex-grow space-y-3" id="gallery-list"></div>  
  </div>  
</div>  
  
<!-- リセット確認モーダル -->  
<div id="reset-modal" class="fixed inset-0 bg-black/80 z-50 flex items-center justify-center opacity-0 pointer-events-none transition-opacity duration-300 backdrop-blur-sm p-4">  
  <div class="bg-[#111827] border border-gray-700 rounded-2xl p-6 w-full max-w-sm shadow-2xl transform scale-95 transition-transform duration-300" id="reset-modal-content">  
    <h3 class="text-xl font-bold text-white mb-2">記録のリセット</h3>  
    <p class="text-gray-400 text-sm mb-6">すべてのエリアのミス記録と軌跡をクリアします。<br>よろしいですか？</p>  
    <div class="flex gap-3">  
      <button onclick="closeModal()" class="flex-1 py-3 rounded-xl bg-gray-800 text-white font-semibold hover:bg-gray-700 transition-colors">キャンセル</button>  
      <button onclick="confirmReset()" class="flex-1 py-3 rounded-xl bg-teal-600 text-white font-semibold hover:bg-teal-500 shadow-[0_0_15px_rgba(13,148,136,0.4)] transition-colors">リセットする</button>  
    </div>  
  </div>  
</div>  
  
<script>  
// PWA サービスワーカーの登録  
if ('serviceWorker' in navigator) {  
  window.addEventListener('load', () => {  
    navigator.serviceWorker.register('sw.js').catch(err => {  
      console.log('ServiceWorker registration failed: ', err);  
    });  
  });  
}  
  
const labelsTop = [  
  "後衛 左", "後衛 中", "後衛 右",  
  "中衛 左", "中衛 中", "中衛 右",  
  "前衛 左", "前衛 中", "前衛 右"  
];  
const labelsBottom = [  
  "前衛 左", "前衛 中", "前衛 右",  
  "中衛 左", "中衛 中", "中衛 右",  
  "後衛 左", "後衛 中", "後衛 右"  
];  
let counts = new Array(18).fill(0);  
let savedSheets = [];   
  
document.addEventListener('DOMContentLoaded', () => {  
  const oppGrid = document.getElementById('grid-opponent');  
  const playGrid = document.getElementById('grid-player');  
  
  labelsTop.forEach((label, i) => {  
    oppGrid.appendChild(createZoneDisplay(i, label));  
  });  
  labelsBottom.forEach((label, i) => {  
    playGrid.appendChild(createZoneDisplay(i + 9, label));  
  });  
  loadFromStorage();  
});  
  
// 表示用セルの作成（タップ判定は無し）  
function createZoneDisplay(index, labelText) {  
  const zone = document.createElement('div');  
  zone.className = 'zone-cell group';  
    
  const label = document.createElement('span');  
  label.className = 'absolute text-[9px] sm:text-[10px] text-white/30 font-medium tracking-widest pointer-events-none transition-opacity duration-300 text-center';  
  label.innerText = labelText.replace(" ", "\n");  
  label.id = `label-${index}`;  
    
  const countDisplay = document.createElement('span');  
  countDisplay.className = 'text-2xl sm:text-4xl font-black text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)] pointer-events-none opacity-0 hidden';  
  countDisplay.id = `count-${index}`;  
  countDisplay.innerText = '0';  
    
  zone.appendChild(label);  
  zone.appendChild(countDisplay);  
  return zone;  
}  
  
function updateDisplay(index) {  
  const countEl = document.getElementById(`count-${index}`);  
  const labelEl = document.getElementById(`label-${index}`);  
  countEl.innerText = counts[index];  
    
  if (counts[index] > 0) {  
    countEl.classList.remove('hidden');  
    countEl.classList.remove('anim-pop');  
    void countEl.offsetWidth;   
    countEl.classList.add('anim-pop');  
    labelEl.classList.replace('text-white/30', 'text-white/10');  
  } else {  
    countEl.classList.add('hidden');  
    countEl.classList.remove('anim-pop');  
    labelEl.classList.replace('text-white/10', 'text-white/30');  
  }  
}  
  
// 座標から属するエリア(0~17)を計算する  
function getZoneIndex(x, y) {  
  const cw = canvas.width;  
  const ch = canvas.height;  
  const col = Math.min(2, Math.max(0, Math.floor((x / cw) * 3)));  
  let row;  
    
  if (y < ch / 2) {  
    // 上半分 (0~8)  
    row = Math.min(2, Math.max(0, Math.floor((y / (ch / 2)) * 3)));  
    return row * 3 + col;  
  } else {  
    // 下半分 (9~17)  
    row = Math.min(2, Math.max(0, Math.floor(((y - ch / 2) / (ch / 2)) * 3)));  
    return 9 + row * 3 + col;  
  }  
}  
  
// --- 描画・消しゴムロジック ---  
const canvas = document.getElementById('trajectory-canvas');  
const ctx = canvas.getContext('2d', { willReadFrequently: true });  
ctx.lineCap = 'round';  
ctx.lineJoin = 'round';  
  
let isDrawing = false;  
let startPos = { x: 0, y: 0 };  
let currentMode = 'smash'; // デフォルト  
let drawnLines = [];  
let longPressTimer = null;  
let isLongPress = false;  
let touchStartPos = null;  
  
const strokeColors = {  
  smash: '#ef4444',  
  drop: '#22c55e',  
  clear: '#eab308',  
  hairpin: '#3b82f6'  
};  
  
document.querySelectorAll('.mode-btn').forEach(btn => {  
  btn.addEventListener('click', (e) => {  
    document.querySelectorAll('.mode-btn').forEach(b => {  
      b.classList.remove('border-white', 'opacity-100', 'shadow-lg');  
      b.classList.add('border-transparent', 'opacity-50');  
    });  
    const target = e.currentTarget;  
    target.classList.remove('border-transparent', 'opacity-50');  
    target.classList.add('border-white', 'opacity-100', 'shadow-lg');  
    currentMode = target.dataset.mode;  
  });  
});  
  
function getCanvasPos(e, isTouchEnd = false) {  
  const rect = canvas.getBoundingClientRect();  
  let clientX, clientY;  
  if (e.type.includes('touch')) {  
    const touch = isTouchEnd ? e.changedTouches[0] : e.touches[0];  
    clientX = touch.clientX;  
    clientY = touch.clientY;  
  } else {  
    clientX = e.clientX;  
    clientY = e.clientY;  
  }  
  return {  
    x: (clientX - rect.left) * (canvas.width / rect.width),  
    y: (clientY - rect.top) * (canvas.height / rect.height)  
  };  
}  
  
function drawStar(ctx, cx, cy, spikes, outerRadius, innerRadius, color) {  
  let rot = Math.PI / 2 * 3;  
  let x = cx, y = cy;  
  let step = Math.PI / spikes;  
  ctx.save();  
  ctx.beginPath();  
  ctx.moveTo(cx, cy - outerRadius);  
  for (let i = 0; i < spikes; i++) {  
    x = cx + Math.cos(rot) * outerRadius;  
    y = cy + Math.sin(rot) * outerRadius;  
    ctx.lineTo(x, y);  
    rot += step;  
    x = cx + Math.cos(rot) * innerRadius;  
    y = cy + Math.sin(rot) * innerRadius;  
    ctx.lineTo(x, y);  
    rot += step;  
  }  
  ctx.lineTo(cx, cy - outerRadius);  
  ctx.closePath();  
  ctx.fillStyle = color;  
  ctx.shadowBlur = 4;  
  ctx.shadowColor = 'rgba(0,0,0,0.5)';  
  ctx.fill();  
  ctx.restore();  
}  
  
function drawArrowHead(ctx, fromX, fromY, toX, toY, color) {  
  const headlen = 16;  
  const angle = Math.atan2(toY - fromY, toX - fromX);  
  ctx.save();  
  ctx.beginPath();  
  ctx.moveTo(toX, toY);  
  ctx.lineTo(toX - headlen * Math.cos(angle - Math.PI / 6), toY - headlen * Math.sin(angle - Math.PI / 6));  
  ctx.moveTo(toX, toY);  
  ctx.lineTo(toX - headlen * Math.cos(angle + Math.PI / 6), toY - headlen * Math.sin(angle + Math.PI / 6));  
  ctx.strokeStyle = color;  
  ctx.lineWidth = 5;  
  ctx.stroke();  
  ctx.restore();  
}  
  
function drawStraightLineWithDecorations(p1, p2, color, isDashed) {  
  ctx.strokeStyle = color;  
  ctx.fillStyle = color;  
  ctx.lineWidth = 5;  
  ctx.shadowBlur = 4;  
  ctx.shadowColor = 'rgba(0, 0, 0, 0.4)';  
    
  ctx.beginPath();  
  if (isDashed) {  
    ctx.setLineDash([15, 15]);  
  } else {  
    ctx.setLineDash([]);  
  }  
  ctx.moveTo(p1.x, p1.y);  
  ctx.lineTo(p2.x, p2.y);  
  ctx.stroke();  
    
  ctx.setLineDash([]); // 星や矢印は必ず直線で描く  
    
  drawStar(ctx, p1.x, p1.y, 5, 12, 5, color);  
    
  if (Math.hypot(p2.x - p1.x, p2.y - p1.y) > 10) {  
    drawArrowHead(ctx, p1.x, p1.y, p2.x, p2.y, color);  
  }  
}  
  
function redrawAll() {  
  ctx.clearRect(0, 0, canvas.width, canvas.height);  
  drawnLines.forEach(line => {  
    drawStraightLineWithDecorations(line.start, line.end, line.color, line.isDashed);  
  });  
}  
  
function getDistanceToLine(p, p1, p2) {  
  const l2 = Math.pow(p1.x - p2.x, 2) + Math.pow(p1.y - p2.y, 2);  
  if (l2 === 0) return Math.hypot(p.x - p1.x, p.y - p1.y);  
  let t = ((p.x - p1.x) * (p2.x - p1.x) + (p.y - p1.y) * (p2.y - p1.y)) / l2;  
  t = Math.max(0, Math.min(1, t));  
  return Math.hypot(p.x - (p1.x + t * (p2.x - p1.x)), p.y - (p1.y + t * (p2.y - p1.y)));  
}  
  
function eraseLineAt(pos) {  
  let closestIndex = -1;  
  let minDistance = 30;  
  for (let i = drawnLines.length - 1; i >= 0; i--) {  
    if (getDistanceToLine(pos, drawnLines[i].start, drawnLines[i].end) < minDistance) {  
      closestIndex = i;  
      break;  
    }  
  }  
    
  if (closestIndex !== -1) {  
    // 線を削除し、対応するエリアのカウントを減らす  
    const erasedLine = drawnLines.splice(closestIndex, 1)[0];  
    if (counts[erasedLine.startIndex] > 0) {  
      counts[erasedLine.startIndex]--;  
      updateDisplay(erasedLine.startIndex);  
    }  
    redrawAll();  
    if (navigator.vibrate) navigator.vibrate(50);  
  }  
}  
  
function startDraw(e) {  
  e.preventDefault();  
  isDrawing = true;  
  isLongPress = false;  
  startPos = getCanvasPos(e);  
  touchStartPos = startPos;  
    
  // 長押し判定（消しゴム）  
  longPressTimer = setTimeout(() => {  
    isLongPress = true;  
    isDrawing = false;  
    eraseLineAt(touchStartPos);  
  }, 1000);  
}  
  
function draw(e) {  
  e.preventDefault();  
  const currentPos = getCanvasPos(e);  
    
  // 少しでも動いたら長押し（消しゴム）をキャンセル  
  if (touchStartPos && longPressTimer && Math.hypot(currentPos.x - touchStartPos.x, currentPos.y - touchStartPos.y) > 10) {  
    clearTimeout(longPressTimer);  
    longPressTimer = null;  
  }  
    
  if (!isDrawing || isLongPress) return;  
    
  redrawAll();  
    
  // プレビュー描画（奥のコートからスタートしたら点線）  
  const isTopCourt = startPos.y < canvas.height / 2;  
  drawStraightLineWithDecorations(startPos, currentPos, strokeColors[currentMode], isTopCourt);  
}  
  
function endDraw(e) {  
  if (e) e.preventDefault();  
  if (longPressTimer) {  
    clearTimeout(longPressTimer);  
    longPressTimer = null;  
  }  
    
  if (!isDrawing || isLongPress) {  
    isDrawing = false;  
    return;  
  }  
  isDrawing = false;  
    
  const finalPos = getCanvasPos(e, e.type.includes('touch'));  
    
  // ある程度の長さが描かれたら保存＆カウント追加  
  if (Math.hypot(finalPos.x - startPos.x, finalPos.y - startPos.y) > 5) {  
    const isTopCourt = startPos.y < canvas.height / 2;  
    const startIndex = getZoneIndex(startPos.x, startPos.y);  
      
    drawnLines.push({  
      start: startPos,  
      end: finalPos,  
      color: strokeColors[currentMode],  
      type: currentMode,  
      startIndex: startIndex,  
      isDashed: isTopCourt  
    });  
      
    // エリアのカウントを増やす  
    counts[startIndex]++;  
    updateDisplay(startIndex);  
  }  
  redrawAll();  
}  
  
canvas.addEventListener('touchstart', startDraw, { passive: false });  
canvas.addEventListener('touchmove', draw, { passive: false });  
canvas.addEventListener('touchend', endDraw, { passive: false });  
canvas.addEventListener('touchcancel', endDraw, { passive: false });  
  
canvas.addEventListener('mousedown', startDraw);  
canvas.addEventListener('mousemove', draw);  
canvas.addEventListener('mouseup', endDraw);  
canvas.addEventListener('mouseleave', (e) => { if (isDrawing) endDraw(e); });  
  
// --- 保存・集計管理 ---  
async function saveToGallery() {  
  for (let i = 0; i < 18; i++) {  
    const countEl = document.getElementById(`count-${i}`);  
    const labelEl = document.getElementById(`label-${i}`);  
    if (counts[i] === 0) {  
      countEl.classList.remove('hidden');  
      countEl.classList.add('text-white/50');  
      labelEl.classList.replace('text-white/30', 'text-white/10');  
    }  
  }  
    
  const captureArea = document.getElementById('capture-area');  
  try {  
    const canvasImg = await html2canvas(captureArea, {  
      backgroundColor: '#0b1121',  
      scale: 2,  
      logging: false  
    });  
      
    const pTop = document.getElementById('player-top').value || '奥';  
    const pBottom = document.getElementById('player-bottom').value || '手前';  
    const now = new Date();  
    const dateStr = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;  
      
    const shotCounts = {  
      smash: drawnLines.filter(l => l.type === 'smash').length,  
      drop: drawnLines.filter(l => l.type === 'drop').length,  
      clear: drawnLines.filter(l => l.type === 'clear').length,  
      hairpin: drawnLines.filter(l => l.type === 'hairpin').length,  
    };  
      
    const item = {  
      id: Date.now(),  
      date: dateStr,  
      playerTop: pTop,  
      playerBottom: pBottom,  
      totalMisses: counts.reduce((a, b) => a + b, 0),  
      counts: [...counts],  
      shotCounts: shotCounts,  
      dataUrl: canvasImg.toDataURL('image/png')  
    };  
      
    savedSheets.unshift(item);  
    saveToStorage();  
    updateSavedCount();  
      
    const link = document.createElement('a');  
    link.download = `CourtVision_${pTop}_vs_${pBottom}_${Date.now()}.png`;  
    link.href = item.dataUrl;  
    link.click();  
    openGallery();  
  } catch (error) {  
    console.error("保存失敗:", error);  
  } finally {  
    for (let i = 0; i < 18; i++) {  
      if (counts[i] === 0) {  
        const countEl = document.getElementById(`count-${i}`);  
        const labelEl = document.getElementById(`label-${i}`);  
        countEl.classList.add('hidden');  
        countEl.classList.remove('text-white/50');  
        labelEl.classList.replace('text-white/10', 'text-white/30');  
      }  
    }  
  }  
}  
  
function saveToStorage() {  
  try {  
    localStorage.setItem('badminton_sheets', JSON.stringify(savedSheets));  
  } catch(e) { console.error(e); }  
}  
  
function loadFromStorage() {  
  try {  
    const data = localStorage.getItem('badminton_sheets');  
    if (data) {  
      savedSheets = JSON.parse(data);  
      updateSavedCount();  
    }  
  } catch(e) { console.error(e); }  
}  
  
function updateSavedCount() {  
  document.getElementById('saved-count').innerText = savedSheets.length;  
  document.getElementById('modal-item-count').innerText = `${savedSheets.length}件`;  
}  
  
function openGallery() {  
  renderGalleryList();  
  const modal = document.getElementById('gallery-modal');  
  const content = document.getElementById('gallery-modal-content');  
  modal.classList.remove('pointer-events-none', 'opacity-0');  
  content.classList.remove('scale-95');  
  content.classList.add('scale-100');  
}  
  
function closeGallery() {  
  const modal = document.getElementById('gallery-modal');  
  const content = document.getElementById('gallery-modal-content');  
  modal.classList.add('opacity-0', 'pointer-events-none');  
  content.classList.remove('scale-100');  
  content.classList.add('scale-95');  
}  
  
function renderGalleryList() {  
  const listEl = document.getElementById('gallery-list');  
  if (savedSheets.length === 0) {  
    listEl.innerHTML = `<div class="text-center py-12 text-gray-500 text-sm">保存されたシートはありません。「保存」ボタンを押すとここに蓄積されます。</div>`;  
    return;  
  }  
    
  listEl.innerHTML = savedSheets.map(item => `  
    <div class="bg-gray-800/80 border border-gray-700 rounded-xl p-3 flex flex-col sm:flex-row gap-4 items-center">  
      <img src="${item.dataUrl}" class="w-28 aspect-[1/2.1] object-contain bg-black/40 rounded border border-gray-700 shrink-0" />  
      <div class="flex-grow text-xs space-y-1 w-full">  
        <div class="flex justify-between items-center text-gray-400">  
          <span>${item.date}</span>  
          <span class="text-teal-400 font-bold">総ミス: ${item.totalMisses}回</span>  
        </div>  
        <div class="font-bold text-sm text-white">${item.playerTop || '奥'} vs ${item.playerBottom || '手前'}</div>  
        <div class="flex gap-2 text-[11px] text-gray-300 pt-1 flex-wrap">  
          <span class="bg-red-950/60 text-red-400 px-1.5 py-0.5 rounded border border-red-900">スマ: ${item.shotCounts.smash}</span>  
          <span class="bg-green-950/60 text-green-400 px-1.5 py-0.5 rounded border border-green-900">ドロ: ${item.shotCounts.drop}</span>  
          <span class="bg-yellow-950/60 text-yellow-400 px-1.5 py-0.5 rounded border border-yellow-900">クリ: ${item.shotCounts.clear}</span>  
          <span class="bg-blue-950/60 text-blue-400 px-1.5 py-0.5 rounded border border-blue-900">ヘアロブ: ${item.shotCounts.hairpin}</span>  
        </div>  
      </div>  
      <div class="flex sm:flex-col gap-2 w-full sm:w-auto shrink-0">  
        <a href="${item.dataUrl}" download="CourtVision_${item.playerTop}_vs_${item.playerBottom}.png" class="flex-1 sm:flex-none text-center px-3 py-1.5 bg-teal-700 hover:bg-teal-600 text-white rounded font-bold text-xs transition-colors">ダウンロード</a>  
        <button onclick="deleteItem(${item.id})" class="flex-1 sm:flex-none px-3 py-1.5 bg-red-950/80 hover:bg-red-900 text-red-300 border border-red-900 rounded font-bold text-xs transition-colors">削除</button>  
      </div>  
    </div>`).join('');  
}  
  
function deleteItem(id) {  
  savedSheets = savedSheets.filter(i => i.id !== id);  
  saveToStorage();  
  updateSavedCount();  
  renderGalleryList();  
}  
  
function exportCSV() {  
  if (savedSheets.length === 0) {  
    alert("集計するデータがありません。");  
    return;  
  }  
  let csv = '\uFEFF';   
  csv += '日時,奥選手名,手前選手名,総ミス数,スマッシュ数,ドロップ数,クリア数,ヘアピンロブ数';  
  for (let i = 0; i < 9; i++) csv += `,奥陣地_${labelsTop[i].replace(' ', '')}`;  
  for (let i = 0; i < 9; i++) csv += `,手前陣地_${labelsBottom[i].replace(' ', '')}`;  
  csv += '\n';  
    
  savedSheets.forEach(item => {  
    csv += `"${item.date}","${item.playerTop}","${item.playerBottom}",${item.totalMisses},${item.shotCounts.smash},${item.shotCounts.drop},${item.shotCounts.clear},${item.shotCounts.hairpin}`;  
    item.counts.forEach(c => { csv += `,${c}`; });  
    csv += '\n';  
  });  
    
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });  
  const url = URL.createObjectURL(blob);  
  const link = document.createElement('a');  
  link.href = url;  
  link.download = `CourtVision_Summary_${new Date().toISOString().slice(0,10).replace(/-/g,'')}.csv`;  
  link.click();  
  URL.revokeObjectURL(url);  
}  
  
const modal = document.getElementById('reset-modal');  
const modalContent = document.getElementById('reset-modal-content');  
  
function openModal() {  
  modal.classList.remove('pointer-events-none', 'opacity-0');  
  modalContent.classList.remove('scale-95');  
  modalContent.classList.add('scale-100');  
}  
  
function closeModal() {  
  modal.classList.add('opacity-0', 'pointer-events-none');  
  modalContent.classList.remove('scale-100');  
  modalContent.classList.add('scale-95');  
}  
  
function confirmReset() {  
  counts.fill(0);  
  for(let i = 0; i < 18; i++) updateDisplay(i);  
  drawnLines = [];  
  ctx.clearRect(0, 0, canvas.width, canvas.height);  
  closeModal();  
}  
</script>  
</body>  
</html>  
