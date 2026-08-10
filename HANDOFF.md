<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>«Звёздный портал» — тестовая сцена: Святилище, Тибет, 1938</title>
<style>
  :root{
    --ink:#0d0c0a;
    --panel:#16130f;
    --panel2:#1e1a14;
    --brass:#c8a45c;
    --brass-d:#8a7038;
    --steel:#3a3a3c;
    --paper:#d8cfba;
    --muted:#8a8171;
    --red:#a8443a;
    --blue:#7fa8bd;
    --green:#7d9464;
  }
  *{box-sizing:border-box}
  html,body{margin:0;padding:0;background:var(--ink);color:var(--paper);
    font-family:"Iowan Old Style","Palatino Linotype",Georgia,serif;
    -webkit-font-smoothing:antialiased;overflow:hidden}
  #app{display:flex;height:100vh;width:100vw}
  #stage{position:relative;flex:1;min-width:0;background:
    radial-gradient(ellipse at 50% 35%, #1a1713 0%, #0d0c0a 70%)}
  canvas{display:block;width:100%;height:100%}
  /* ---------- боковая панель ---------- */
  #side{width:300px;flex:0 0 300px;background:var(--panel);
    border-left:1px solid #2b251c;display:flex;flex-direction:column;overflow:hidden}
  .sec{padding:10px 14px;border-bottom:1px solid #2b251c}
  .sec h3{margin:0 0 8px;font-size:11px;letter-spacing:.18em;text-transform:uppercase;
    color:var(--brass);font-weight:600;font-family:"Helvetica Neue",Arial,sans-serif}
  .unit{padding:8px 10px;margin-bottom:6px;background:var(--panel2);
    border-left:3px solid #444;cursor:pointer;transition:.12s}
  .unit:hover{background:#252017}
  .unit.sel{border-left-color:var(--brass);background:#2a2318}
  .unit.dead{opacity:.35;cursor:default}
  .unit .nm{font-size:14px;display:flex;justify-content:space-between;align-items:baseline}
  .unit .nm small{font-family:"Helvetica Neue",Arial,sans-serif;font-size:10px;
    letter-spacing:.1em;color:var(--muted)}
  .bar{height:5px;background:#000;margin-top:5px;position:relative;overflow:hidden}
  .bar i{display:block;height:100%;transition:width .25s}
  .bar.hp i{background:var(--green)}
  .bar.hum i{background:var(--red)}
  .aps{margin-top:5px;display:flex;gap:4px}
  .ap{width:16px;height:6px;background:#2e2a22}
  .ap.on{background:var(--brass)}
  /* ---------- кнопки действий ---------- */
  .acts{display:grid;grid-template-columns:1fr 1fr;gap:6px}
  button{font-family:"Helvetica Neue",Arial,sans-serif;font-size:11px;letter-spacing:.08em;
    text-transform:uppercase;padding:9px 6px;background:var(--panel2);color:var(--paper);
    border:1px solid #3a3227;cursor:pointer;transition:.12s}
  button:hover:not(:disabled){background:#302819;border-color:var(--brass-d)}
  button.on{background:var(--brass);color:#17130c;border-color:var(--brass)}
  button:disabled{opacity:.3;cursor:not-allowed}
  button.wide{grid-column:1/3}
  /* ---------- предсказание броска ---------- */
  #pred{font-size:12px;line-height:1.55;color:var(--muted);min-height:74px}
  #pred b{color:var(--paper);font-weight:normal}
  #pred .plus{color:var(--green)}
  #pred .minus{color:var(--red)}
  /* ---------- лог ---------- */
  #log{position:absolute;left:14px;bottom:14px;width:390px;max-height:190px;overflow:hidden;
    font-size:12.5px;line-height:1.5;pointer-events:none;
    display:flex;flex-direction:column-reverse}
  #log div{margin-top:3px;text-shadow:0 1px 3px #000;opacity:.92}
  #log .big{color:var(--brass);font-size:13.5px}
  #log .bad{color:#c9705f}
  #log .good{color:#9fbd7f}
  /* ---------- резонатор ---------- */
  #reso{position:absolute;right:16px;bottom:16px;width:246px;
    background:linear-gradient(#211d17,#141110);
    border:2px solid #3b3125;border-radius:5px;padding:11px;
    box-shadow:0 12px 40px rgba(0,0,0,.8), inset 0 1px 0 rgba(255,255,255,.05)}
  #reso .plate{background:linear-gradient(#c9a961,#9d7f42);color:#221a0c;
    font-family:"Helvetica Neue",Arial,sans-serif;text-align:center;
    padding:5px 4px;border:1px solid #6d5527;border-radius:2px;
    font-size:10.5px;letter-spacing:.11em;line-height:1.35;font-weight:700}
  #reso .plate small{display:block;font-weight:400;font-size:8.5px;letter-spacing:.09em;
    opacity:.82;margin-top:1px}
  #reso .win{margin-top:10px;background:#0a0908;border:2px solid #4a3d2c;border-radius:3px;
    padding:9px;display:flex;align-items:center;gap:10px;
    box-shadow:inset 0 3px 12px rgba(0,0,0,.9)}
  .barrels{display:flex;gap:5px}
  .barrel{width:34px;height:44px;border-radius:3px;
    background:linear-gradient(#3a352c 0%,#191712 18%,#26221b 50%,#191712 82%,#3a352c 100%);
    border:1px solid #57493a;display:flex;align-items:center;justify-content:center;
    font-family:"Helvetica Neue",Arial,sans-serif;font-size:22px;font-weight:700;
    color:#e2d6b8;text-shadow:0 1px 2px #000}
  .barrel.spin{color:#8d8474}
  #gauge{width:56px;height:56px}
  #reso .sum{margin-top:9px;text-align:center;font-family:"Helvetica Neue",Arial,sans-serif;
    font-size:10.5px;letter-spacing:.13em;color:var(--muted);text-transform:uppercase;
    min-height:15px}
  #reso .sum b{color:var(--brass);font-size:13px}
  .toggles{display:flex;gap:26px;justify-content:center;margin-top:9px}
  .tog{width:15px;height:26px;background:linear-gradient(#4d4237,#2a241d);
    border:1px solid #5d5040;border-radius:3px;position:relative}
  .tog::after{content:'';position:absolute;left:3px;right:3px;top:3px;height:9px;
    background:#b8a882;border-radius:2px;transition:.2s}
  .tog.up::after{top:13px}
  /* ---------- хроника (интро) ---------- */
  #chron{position:absolute;inset:0;background:#000;z-index:50;
    display:flex;align-items:center;justify-content:center;cursor:pointer}
  #chron .inner{max-width:620px;padding:34px;text-align:center;filter:contrast(1.1)}
  #chron .stamp{font-family:"Helvetica Neue",Arial,sans-serif;font-size:11px;
    letter-spacing:.42em;color:#7d7460;margin-bottom:20px}
  #chron h1{font-size:27px;margin:0 0 14px;letter-spacing:.05em;color:#cbbfa4;font-weight:400}
  #chron p{font-size:14.5px;line-height:1.75;color:#8f8571;margin:0 0 9px}
  #chron .go{margin-top:26px;font-family:"Helvetica Neue",Arial,sans-serif;font-size:10.5px;
    letter-spacing:.3em;color:#5f5849;animation:pulse 2s infinite}
  @keyframes pulse{0%,100%{opacity:.4}50%{opacity:1}}
  #chron .grain{position:absolute;inset:0;pointer-events:none;opacity:.16;
    background-image:repeating-linear-gradient(0deg,rgba(255,255,255,.09) 0 1px,transparent 1px 3px)}
  /* ---------- легенда ---------- */
  .leg{font-size:11.5px;line-height:1.7;color:var(--muted)}
  .leg i{display:inline-block;width:9px;height:9px;margin-right:6px;vertical-align:baseline}
  #banner{position:absolute;top:14px;left:50%;transform:translateX(-50%);
    font-family:"Helvetica Neue",Arial,sans-serif;font-size:11px;letter-spacing:.26em;
    color:#6f6757;text-transform:uppercase;pointer-events:none}
  #endcard{position:absolute;inset:0;background:rgba(6,5,4,.93);z-index:60;
    display:none;align-items:center;justify-content:center;text-align:center}
  #endcard h2{font-size:26px;font-weight:400;letter-spacing:.06em;color:var(--brass);margin:0 0 12px}
  #endcard p{color:#8f8571;font-size:14px;max-width:430px;line-height:1.7;margin:0 auto}
</style>
</head>
<body>
<div id="app">
  <div id="stage">
    <canvas id="cv"></canvas>
    <div id="banner">Тестовая сцена · Святилище · Лхаса · 1938</div>
    <div id="log"></div>

    <div id="reso">
      <div class="plate">RESONANZ&nbsp;ANZEIGER
        <small>V. N. — VERSUCH 3 · 1938</small>
      </div>
      <div class="win">
        <div class="barrels">
          <div class="barrel" id="b0">1</div>
          <div class="barrel" id="b1">2</div>
          <div class="barrel" id="b2">3</div>
        </div>
        <canvas id="gauge" width="112" height="112"></canvas>
      </div>
      <div class="sum" id="sumline">прибор в покое</div>
      <div class="toggles">
        <div class="tog up" id="tg0"></div>
        <div class="tog" id="tg1"></div>
      </div>
    </div>

    <div id="chron">
      <div class="grain"></div>
      <div class="inner">
        <div class="stamp">ГЕХАЙМ · АНЕНЕРБЕ · ФИЛЬМОТЕКА · РОЛИК 14</div>
        <h1>Святилище</h1>
        <p>Тибет, осень 1938 года. Монахи говорили о гуле и о «свете бога».
           Отряд им не поверил.</p>
        <p>Виктор Новак раздал приборы за час до спуска.
           «Устройство не делает вас сильнее. Оно только показывает правду».</p>
        <div class="go">— нажмите, чтобы принять управление —</div>
      </div>
    </div>

    <div id="endcard"><div><h2 id="endt"></h2><p id="endp"></p></div></div>
  </div>

  <aside id="side">
    <div class="sec">
      <h3>Отряд</h3>
      <div id="roster"></div>
    </div>
    <div class="sec">
      <h3>Действия</h3>
      <div class="acts">
        <button id="btnMove">Ход</button>
        <button id="btnShoot">Выстрел</button>
        <button id="btnJump">Прыжок</button>
        <button id="btnComp">Компенсатор</button>
        <button id="btnImp" class="wide">Резонансный импульс</button>
        <button id="btnEnd" class="wide">Завершить ход отряда</button>
      </div>
    </div>
    <div class="sec" style="flex:1">
      <h3>Прибор говорит</h3>
      <div id="pred">Выберите бойца и действие.</div>
    </div>
    <div class="sec">
      <h3>Обозначения</h3>
      <div class="leg">
        <i style="background:#5d5346"></i>стена — полное укрытие<br>
        <i style="background:#7a6544"></i>ящик — низкое укрытие<br>
        <i style="background:#241f2e"></i>разлом — только прыжком<br>
        <i style="background:#7e3b34"></i>зона гула излучателя
      </div>
    </div>
  </aside>
</div>

<script>
"use strict";
/* ==========================================================================
   «Звёздный портал» — тестовая сцена боевой механики
   Проверяем: броски резонатора, стрельбу, укрытия, прыжки, влияние излучателя
   ========================================================================== */

/* ------------------------------ карта ---------------------------------- */
const MAPSTR = [
  "############",
  "#....E.....#",
  "#.c......c.#",
  "#..........#",
  "#..........#",
  "#..~~~~~~..#",
  "#..........#",
  "#.c..##..c.#",
  "#....##....#",
  "#..........#",
  "#..........#",
  "############"
];
const W = 12, H = 12;
const TW = 68, TH = 34;
const EMIT = { x: 5, y: 1, r: 2.6 };

const grid = MAPSTR.map(r => r.split(""));
const tile = (x, y) => (x < 0 || y < 0 || x >= W || y >= H) ? "#" : grid[y][x];
const isWalk = (x, y) => tile(x, y) === ".";
const isChasm = (x, y) => tile(x, y) === "~";
const dist = (a, b) => Math.hypot(a.x - b.x, a.y - b.y);
const inHum = (x, y) => Math.hypot(x - EMIT.x, y - EMIT.y) <= EMIT.r;

/* ------------------------------ юниты ---------------------------------- */
let units = [];
function resetUnits() {
  units = [
    { id:"k", name:"Кессель", role:"солдат", side:"p", x:4, y:9,
      hp:8, maxHp:8, ap:2, hum:0, move:5, comp:1, imp:0, dead:false, col:"#c8a45c" },
    { id:"v", name:"Виктор", role:"учёный", side:"p", x:6, y:9,
      hp:6, maxHp:6, ap:2, hum:0, move:4, comp:1, imp:1, dead:false, col:"#7fa8bd" },
    { id:"e1", name:"Трискель · стрелок", role:"противник", side:"e", x:3, y:3,
      hp:6, maxHp:6, ap:2, hum:0, move:4, comp:0, imp:0, dead:false, col:"#a35b4a" },
    { id:"e2", name:"Трискель · стрелок", role:"противник", side:"e", x:8, y:4,
      hp:6, maxHp:6, ap:2, hum:0, move:4, comp:0, imp:0, dead:false, col:"#a35b4a" },
    { id:"e3", name:"Трискель · егерь", role:"противник", side:"e", x:7, y:7,
      hp:7, maxHp:7, ap:2, hum:0, move:5, comp:0, imp:0, dead:false, col:"#8f4f42" }
  ];
}
const alive = () => units.filter(u => !u.dead);
const unitAt = (x, y) => alive().find(u => u.x === x && u.y === y);
const players = () => alive().filter(u => u.side === "p");
const enemies = () => alive().filter(u => u.side === "e");

/* ------------------------------ состояние ------------------------------ */
let sel = null;          // выбранный боец
let mode = null;         // "move" | "shoot" | "jump"
let hoverTile = { x:-1, y:-1 };
let reach = new Map();   // достижимые клетки при mode==="move"
let jumpTargets = [];
let busy = false;        // идёт анимация/ход ИИ
let turn = 1;
let over = false;
let compArmed = false;   // компенсатор взведён на следующий бросок

/* ------------------------------ утилиты -------------------------------- */
const $ = id => document.getElementById(id);
const cv = $("cv"), ctx = cv.getContext("2d");
let ORX = 0, ORY = 0, DPR = 1;

function resize() {
  DPR = Math.min(window.devicePixelRatio || 1, 2);
  const r = cv.getBoundingClientRect();
  cv.width = r.width * DPR; cv.height = r.height * DPR;
  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
  ORX = r.width / 2;
  ORY = r.height / 2 - (W + H) * TH / 4 + 30;
  draw();
}
const sx = (x, y) => ORX + (x - y) * (TW / 2);
const sy = (x, y) => ORY + (x + y) * (TH / 2);
function pick(mx, my) {
  const dx = mx - ORX, dy = my - ORY;
  const fx = (dx / (TW / 2) + dy / (TH / 2)) / 2;
  const fy = (dy / (TH / 2) - dx / (TW / 2)) / 2;
  return { x: Math.floor(fx), y: Math.floor(fy) };
}

function log(msg, cls) {
  const d = document.createElement("div");
  if (cls) d.className = cls;
  d.textContent = msg;
  $("log").prepend(d);
  while ($("log").children.length > 9) $("log").lastChild.remove();
}

/* ------------------------------ линия огня ----------------------------- */
function hasLOS(a, b) {
  let x0 = a.x, y0 = a.y;
  const x1 = b.x, y1 = b.y;
  const dx = Math.abs(x1 - x0), dy = Math.abs(y1 - y0);
  const sxg = x0 < x1 ? 1 : -1, syg = y0 < y1 ? 1 : -1;
  let err = dx - dy;
  for (let guard = 0; guard < 64; guard++) {
    const e2 = 2 * err;
    if (e2 > -dy) { err -= dy; x0 += sxg; }
    if (e2 < dx)  { err += dx; y0 += syg; }
    if (x0 === x1 && y0 === y1) return true;
    // стена перекрывает линию; клетка вплотную к цели считается её укрытием
    if (tile(x0, y0) === "#" && Math.hypot(x0 - x1, y0 - y1) > 1.5) return false;
  }
  return false;
}

/* Укрытие цели относительно стрелка: клетка, примыкающая к цели в сторону стрелка */
function coverOf(shooter, target) {
  const dx = Math.sign(shooter.x - target.x), dy = Math.sign(shooter.y - target.y);
  let best = 0;
  const probes = [[dx, dy], [dx, 0], [0, dy]].filter(p => p[0] || p[1]);
  for (const [px, py] of probes) {
    const t = tile(target.x + px, target.y + py);
    if (t === "#") best = Math.max(best, 3);
    else if (t === "c" || t === "E") best = Math.max(best, 2);
  }
  return best;
}

/* ------------------------------ модификаторы --------------------------- */
function modsFor(shooter, target, kind) {
  const m = [];
  if (kind === "shoot") {
    const c = coverOf(shooter, target);
    if (c === 3) m.push({ t: "полное укрытие цели", v: -3 });
    else if (c === 2) m.push({ t: "низкое укрытие цели", v: -2 });
    const d = dist(shooter, target);
    if (d > 7) m.push({ t: "дальняя дистанция", v: -1 });
    else if (d <= 2.2) m.push({ t: "вплотную", v: +1 });
  }
  if (shooter.hum >= 3) m.push({ t: "гул в голове (≥3)", v: -1 });
  if (compArmed && shooter.comp > 0) m.push({ t: "компенсатор", v: +1 });
  return m;
}
const sumMods = m => m.reduce((s, x) => s + x.v, 0);

/* ------------------------------ бросок --------------------------------- */
function rollBarrels(unit) {
  const nervous = inHum(unit.x, unit.y);
  const out = [];
  for (let i = 0; i < 3; i++) {
    const a = 1 + Math.floor(Math.random() * 3);
    if (!nervous) { out.push(a); continue; }
    const b = 1 + Math.floor(Math.random() * 3);
    out.push(Math.min(a, b));           // рядом с излучателем барабаны «нервничают»
  }
  return out;
}

function paintGauge(val, max) {
  const g = $("gauge"), c = g.getContext("2d");
  const S = 112;
  c.clearRect(0, 0, S, S);
  c.save(); c.translate(S / 2, S / 2);
  c.beginPath(); c.arc(0, 0, 50, 0, Math.PI * 2);
  c.fillStyle = "#171410"; c.fill();
  c.strokeStyle = "#6b5836"; c.lineWidth = 4; c.stroke();
  const a0 = Math.PI * 0.78, a1 = Math.PI * 2.22;
  for (let i = 0; i <= 6; i++) {
    const a = a0 + (a1 - a0) * (i / 6);
    c.save(); c.rotate(a);
    c.beginPath(); c.moveTo(38, 0); c.lineTo(46, 0);
    c.strokeStyle = i < 2 ? "#a8443a" : (i > 4 ? "#9fbd7f" : "#8a8171");
    c.lineWidth = 2; c.stroke(); c.restore();
  }
  const t = Math.max(0, Math.min(1, (val - 3) / (max - 3)));
  const a = a0 + (a1 - a0) * t;
  c.save(); c.rotate(a);
  c.beginPath(); c.moveTo(-8, 0); c.lineTo(42, 0);
  c.strokeStyle = "#dcd0b0"; c.lineWidth = 2.5; c.lineCap = "round"; c.stroke();
  c.restore();
  c.beginPath(); c.arc(0, 0, 5, 0, Math.PI * 2);
  c.fillStyle = "#c8a45c"; c.fill();
  c.restore();
}
paintGauge(3, 9);

function spinTo(final) {
  return new Promise(res => {
    const els = [$("b0"), $("b1"), $("b2")];
    els.forEach(e => e.classList.add("spin"));
    let ticks = 0;
    const iv = setInterval(() => {
      ticks++;
      els.forEach((e, i) => {
        if (ticks < 9 + i * 5) e.textContent = 1 + Math.floor(Math.random() * 3);
        else { e.textContent = final[i]; e.classList.remove("spin"); }
      });
      paintGauge(3 + Math.random() * 6, 9);
      if (ticks >= 20) {
        clearInterval(iv);
        els.forEach((e, i) => { e.textContent = final[i]; e.classList.remove("spin"); });
        paintGauge(final[0] + final[1] + final[2], 9);
        res();
      }
    }, 42);
  });
}

/* Полный бросок: барабаны → сумма → модификаторы → итог */
async function resonate(unit, mods, forced) {
  busy = true;
  $("tg0").classList.toggle("up", true);
  const barrels = forced || rollBarrels(unit);
  await spinTo(barrels);
  const raw = barrels[0] + barrels[1] + barrels[2];
  const mod = sumMods(mods);
  const eff = raw + mod;
  const modTxt = mods.length ? "  " + mods.map(m => (m.v > 0 ? "+" : "") + m.v).join(" ") : "";
  $("sumline").innerHTML = "барабаны <b>" + raw + "</b>" + modTxt +
                           " → итог <b>" + eff + "</b>";
  if (compArmed && unit.comp > 0) { unit.comp--; compArmed = false; }
  busy = false;
  return { raw, eff, barrels };
}

/* ------------------------------ действия ------------------------------- */
function computeReach(u) {
  reach = new Map();
  const key = (x, y) => x + "," + y;
  const q = [[u.x, u.y, 0]];
  const seen = new Set([key(u.x, u.y)]);
  while (q.length) {
    const [x, y, d] = q.shift();
    if (d > 0) reach.set(key(x, y), d);
    if (d >= u.move) continue;
    for (const [dx, dy] of [[1,0],[-1,0],[0,1],[0,-1]]) {
      const nx = x + dx, ny = y + dy, k = key(nx, ny);
      if (seen.has(k) || !isWalk(nx, ny) || unitAt(nx, ny)) continue;
      seen.add(k); q.push([nx, ny, d + 1]);
    }
  }
}

function computeJumps(u) {
  jumpTargets = [];
  for (const [dx, dy] of [[1,0],[-1,0],[0,1],[0,-1]]) {
    if (!isChasm(u.x + dx, u.y + dy)) continue;
    for (let span = 2; span <= 3; span++) {
      const lx = u.x + dx * span, ly = u.y + dy * span;
      if (isChasm(lx, ly)) continue;
      if (isWalk(lx, ly) && !unitAt(lx, ly)) jumpTargets.push({ x: lx, y: ly, dx, dy });
      break;
    }
  }
}

function outcomeShoot(eff) {
  if (eff <= 3) return { dmg: 0, t: "ПРОВАЛ — промах, ствол увело", cls: "bad" };
  if (eff <= 5) return { dmg: 1, t: "скользящее попадание", cls: "" };
  if (eff <= 7) return { dmg: 2, t: "попадание", cls: "" };
  if (eff <= 8) return { dmg: 3, t: "плотное попадание", cls: "good" };
  return { dmg: 4, t: "ИДЕАЛЬНЫЙ РЕЗОНАНС — критическое попадание", cls: "good" };
}

async function doShoot(shooter, target) {
  const mods = modsFor(shooter, target, "shoot");
  const r = await resonate(shooter, mods);
  const o = outcomeShoot(r.eff);
  if (o.dmg > 0) {
    target.hp -= o.dmg;
    log(shooter.name + " → " + target.name + ": " + o.t + " (−" + o.dmg + ")", o.cls);
  } else {
    log(shooter.name + " → " + target.name + ": " + o.t, "bad");
    if (inHum(shooter.x, shooter.y)) {
      shooter.hum = Math.min(5, shooter.hum + 1);
      log("  гул усиливается: " + shooter.name + " держится хуже", "bad");
    }
  }
  if (target.hp <= 0) { target.hp = 0; target.dead = true; log(target.name + " выбывает.", "big"); }
  shooter.ap = 0;               // выстрел завершает ход бойца (правило XCOM)
  afterAction();
}

async function doJump(u, t) {
  const mods = [];
  if (u.hum >= 3) mods.push({ t: "гул в голове (≥3)", v: -1 });
  if (compArmed && u.comp > 0) mods.push({ t: "компенсатор", v: +1 });
  const r = await resonate(u, mods);
  if (r.eff <= 3) {
    u.hp -= 2;
    log(u.name + ": ПРОВАЛ прыжка — сорвался, чудом зацепился за край (−2)", "bad");
    u.ap = 0;
    if (u.hp <= 0) { u.hp = 0; u.dead = true; log(u.name + " срывается в разлом.", "big"); }
  } else if (r.eff <= 7) {
    u.x = t.x; u.y = t.y;
    log(u.name + ": перепрыгнул тяжело — приземление сбило дыхание", "");
    u.ap = 0;
  } else {
    u.x = t.x; u.y = t.y;
    log(u.name + ": чистый прыжок через разлом", "good");
    u.ap--;
  }
  afterAction();
}

async function doImpulse(u) {
  log(u.name + " продавливает резонансный импульс.", "big");
  const forced = Math.random() < 0.5 ? [3,3,2] : [3,3,3];
  const r = await resonate(u, [], forced);
  u.imp--;
  u.hum = Math.min(5, u.hum + 2);
  log("  прибор выдал " + r.raw + " — но из носа идёт кровь (гул +2)", "bad");
  const tgt = enemies().filter(e => hasLOS(u, e)).sort((a, b) => dist(u, a) - dist(u, b))[0];
  if (tgt) {
    const o = outcomeShoot(r.raw + sumMods(modsFor(u, tgt, "shoot")));
    if (o.dmg > 0) { tgt.hp -= o.dmg; log("  " + tgt.name + ": " + o.t + " (−" + o.dmg + ")", "good"); }
    else log("  и всё равно мимо — " + o.t, "bad");
    if (tgt.hp <= 0) { tgt.hp = 0; tgt.dead = true; log(tgt.name + " выбывает.", "big"); }
  }
  u.ap = 0;
  afterAction();
}

/* ------------------------------ конец действия ------------------------- */
function afterAction() {
  mode = null; reach = new Map(); jumpTargets = [];
  checkEnd();
  render();
}

function checkEnd() {
  if (over) return;
  if (!enemies().length) { finish("Зачищено", "Отряд Трискель отброшен. Излучатель ещё работает — но это уже другая миссия."); }
  else if (!players().length) { finish("Отряд потерян", "Гул победил раньше пуль. Так и погиб Тензин — только у него не было даже прибора."); }
}
function finish(t, p) {
  over = true;
  $("endt").textContent = t; $("endp").textContent = p;
  $("endcard").style.display = "flex";
}

/* ------------------------------ ход противника ------------------------- */
async function enemyTurn() {
  busy = true;
  sel = null; mode = null; reach = new Map(); jumpTargets = [];
  render();
  log("— ход Ордена Трискель —", "big");
  for (const e of enemies()) {
    e.ap = 2;
    while (e.ap > 0 && players().length && !over) {
      await sleep(280);
      const tgt = players()
        .filter(p => hasLOS(e, p) && dist(e, p) <= 9)
        .sort((a, b) => dist(e, a) - dist(e, b))[0];
      if (tgt) {
        await shootAI(e, tgt);
      } else {
        const goal = players().sort((a, b) => dist(e, a) - dist(e, b))[0];
        if (!goal) break;
        stepToward(e, goal);
        e.ap--;
      }
      render();
    }
    if (over) break;
  }
  if (!over) {
    turn++;
    players().forEach(p => {
      p.ap = 2;
      if (inHum(p.x, p.y)) {
        p.hum = Math.min(5, p.hum + 1);
        log(p.name + " стоит в зоне гула — рассудок сдаёт (гул " + p.hum + "/5)", "bad");
        if (p.hum >= 5) {
          p.hp -= 2;
          log(p.name + ": гул сломал его — как Ансельма на Бермудах (−2)", "big");
          if (p.hp <= 0) { p.hp = 0; p.dead = true; log(p.name + " выбывает.", "big"); }
        }
      } else if (p.hum > 0) p.hum--;
    });
    log("— ход " + (turn) + " · отряд —", "big");
  }
  checkEnd();
  busy = false;
  render();
}

async function shootAI(e, tgt) {
  const mods = modsFor(e, tgt, "shoot");
  const r = await resonate(e, mods);
  const o = outcomeShoot(r.eff);
  if (o.dmg > 0) {
    tgt.hp -= o.dmg;
    log(e.name + " → " + tgt.name + ": " + o.t + " (−" + o.dmg + ")", "bad");
  } else log(e.name + " → " + tgt.name + ": промах", "");
  if (tgt.hp <= 0) { tgt.hp = 0; tgt.dead = true; log(tgt.name + " выбывает.", "big"); }
  e.ap = 0;                     // выстрел завершает ход бойца
}

function stepToward(u, goal) {
  const key = (x, y) => x + "," + y;
  const prev = new Map();
  const q = [[u.x, u.y]];
  const seen = new Set([key(u.x, u.y)]);
  let found = null;
  while (q.length) {
    const [x, y] = q.shift();
    for (const [dx, dy] of [[1,0],[-1,0],[0,1],[0,-1]]) {
      const nx = x + dx, ny = y + dy, k = key(nx, ny);
      if (seen.has(k)) continue;
      if (nx === goal.x && ny === goal.y) { prev.set(k, [x, y]); found = [nx, ny]; q.length = 0; break; }
      if (!isWalk(nx, ny) || unitAt(nx, ny)) continue;
      seen.add(k); prev.set(k, [x, y]); q.push([nx, ny]);
    }
  }
  if (!found) return;
  const path = [];
  let cur = found;
  while (cur && !(cur[0] === u.x && cur[1] === u.y)) {
    path.unshift(cur);
    cur = prev.get(key(cur[0], cur[1]));
  }
  path.pop(); // не вставать на клетку цели
  const stepsAllowed = Math.min(u.move, path.length);
  if (stepsAllowed > 0) {
    const dest = path[stepsAllowed - 1];
    u.x = dest[0]; u.y = dest[1];
  }
}
const sleep = ms => new Promise(r => setTimeout(r, ms));

/* ------------------------------ отрисовка ------------------------------ */
function tileTop(x, y) {
  const t = tile(x, y);
  if (t === "#") return "#5d5346";
  if (t === "c") return "#7a6544";
  if (t === "E") return "#6d4a4a";
  if (t === "~") return "#241f2e";
  return inHum(x, y) ? "#4a3630" : "#3b372f";
}

function drawTile(x, y) {
  const px = sx(x, y), py = sy(x, y);
  const t = tile(x, y);
  ctx.beginPath();
  ctx.moveTo(px, py);
  ctx.lineTo(px + TW / 2, py + TH / 2);
  ctx.lineTo(px, py + TH);
  ctx.lineTo(px - TW / 2, py + TH / 2);
  ctx.closePath();
  ctx.fillStyle = tileTop(x, y);
  ctx.fill();
  ctx.strokeStyle = "rgba(0,0,0,.4)"; ctx.lineWidth = 1; ctx.stroke();

  // подсветки
  const k = x + "," + y;
  if (mode === "move" && reach.has(k)) {
    ctx.fillStyle = "rgba(200,164,92,.20)"; ctx.fill();
    ctx.strokeStyle = "rgba(200,164,92,.5)"; ctx.stroke();
  }
  if (mode === "jump" && jumpTargets.some(j => j.x === x && j.y === y)) {
    ctx.fillStyle = "rgba(127,168,189,.28)"; ctx.fill();
    ctx.strokeStyle = "rgba(127,168,189,.8)"; ctx.stroke();
  }
  if (hoverTile.x === x && hoverTile.y === y) {
    ctx.strokeStyle = "rgba(236,224,196,.85)"; ctx.lineWidth = 2; ctx.stroke();
  }

  // объёмные блоки
  if (t === "#" || t === "c" || t === "E") {
    const hgt = t === "#" ? 30 : (t === "E" ? 24 : 14);
    const top = py - hgt;
    ctx.beginPath();
    ctx.moveTo(px, top); ctx.lineTo(px + TW / 2, top + TH / 2);
    ctx.lineTo(px, top + TH); ctx.lineTo(px - TW / 2, top + TH / 2);
    ctx.closePath();
    ctx.fillStyle = t === "#" ? "#6d6252" : (t === "E" ? "#8a5b57" : "#93794f");
    ctx.fill(); ctx.strokeStyle = "rgba(0,0,0,.5)"; ctx.lineWidth = 1; ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(px - TW / 2, top + TH / 2); ctx.lineTo(px, top + TH);
    ctx.lineTo(px, py + TH); ctx.lineTo(px - TW / 2, py + TH / 2);
    ctx.closePath(); ctx.fillStyle = "rgba(0,0,0,.35)"; ctx.fill();
    ctx.beginPath();
    ctx.moveTo(px + TW / 2, top + TH / 2); ctx.lineTo(px, top + TH);
    ctx.lineTo(px, py + TH); ctx.lineTo(px + TW / 2, py + TH / 2);
    ctx.closePath(); ctx.fillStyle = "rgba(0,0,0,.18)"; ctx.fill();
  }
}

function drawUnit(u) {
  const px = sx(u.x, u.y), py = sy(u.x, u.y) + TH / 2;
  // тень
  ctx.beginPath(); ctx.ellipse(px, py, 17, 8, 0, 0, Math.PI * 2);
  ctx.fillStyle = "rgba(0,0,0,.45)"; ctx.fill();
  // выделение
  if (sel === u) {
    ctx.beginPath(); ctx.ellipse(px, py, 22, 11, 0, 0, Math.PI * 2);
    ctx.strokeStyle = "#c8a45c"; ctx.lineWidth = 2; ctx.stroke();
  }
  // тело
  ctx.beginPath();
  ctx.roundRect ? ctx.roundRect(px - 8, py - 34, 16, 28, 5)
                : ctx.rect(px - 8, py - 34, 16, 28);
  ctx.fillStyle = u.col; ctx.fill();
  ctx.strokeStyle = "rgba(0,0,0,.6)"; ctx.lineWidth = 1.2; ctx.stroke();
  // голова
  ctx.beginPath(); ctx.arc(px, py - 41, 7, 0, Math.PI * 2);
  ctx.fillStyle = u.col; ctx.fill(); ctx.stroke();
  // полоса здоровья
  const bw = 26;
  ctx.fillStyle = "#000"; ctx.fillRect(px - bw / 2, py - 56, bw, 4);
  ctx.fillStyle = u.side === "p" ? "#7d9464" : "#a8443a";
  ctx.fillRect(px - bw / 2, py - 56, bw * (u.hp / u.maxHp), 4);
  // гул
  if (u.hum > 0) {
    ctx.fillStyle = "#000"; ctx.fillRect(px - bw / 2, py - 61, bw, 3);
    ctx.fillStyle = "#a8443a"; ctx.fillRect(px - bw / 2, py - 61, bw * (u.hum / 5), 3);
  }
}

function draw() {
  const r = cv.getBoundingClientRect();
  ctx.clearRect(0, 0, r.width, r.height);
  for (let y = 0; y < H; y++) for (let x = 0; x < W; x++) drawTile(x, y);
  // пульсация излучателя
  const t = performance.now() / 700;
  const ex = sx(EMIT.x, EMIT.y), ey = sy(EMIT.x, EMIT.y) + TH / 2 - 20;
  ctx.beginPath();
  ctx.arc(ex, ey, 16 + Math.sin(t) * 5, 0, Math.PI * 2);
  ctx.strokeStyle = "rgba(200,90,70," + (0.5 + Math.sin(t) * 0.25) + ")";
  ctx.lineWidth = 2; ctx.stroke();
  // юниты по глубине
  alive().slice().sort((a, b) => (a.x + a.y) - (b.x + b.y)).forEach(drawUnit);
  // линия огня
  if (mode === "shoot" && sel) {
    const tgt = unitAt(hoverTile.x, hoverTile.y);
    if (tgt && tgt.side === "e") {
      const ok = hasLOS(sel, tgt);
      ctx.beginPath();
      ctx.moveTo(sx(sel.x, sel.y), sy(sel.x, sel.y) + TH / 2 - 20);
      ctx.lineTo(sx(tgt.x, tgt.y), sy(tgt.x, tgt.y) + TH / 2 - 20);
      ctx.strokeStyle = ok ? "rgba(236,224,196,.75)" : "rgba(168,68,58,.75)";
      ctx.lineWidth = 1.5; ctx.setLineDash([5, 4]); ctx.stroke(); ctx.setLineDash([]);
    }
  }
}
setInterval(() => { if (!over) draw(); }, 60);

/* ------------------------------ интерфейс ------------------------------ */
function render() {
  // ростер
  const box = $("roster"); box.innerHTML = "";
  units.forEach(u => {
    if (u.side !== "p") return;
    const d = document.createElement("div");
    d.className = "unit" + (sel === u ? " sel" : "") + (u.dead ? " dead" : "");
    d.innerHTML =
      '<div class="nm">' + u.name + ' <small>' + u.role + '</small></div>' +
      '<div class="bar hp"><i style="width:' + (100 * u.hp / u.maxHp) + '%"></i></div>' +
      '<div class="bar hum"><i style="width:' + (100 * u.hum / 5) + '%"></i></div>' +
      '<div class="aps">' + [0,1].map(i =>
        '<div class="ap' + (u.ap > i ? " on" : "") + '"></div>').join("") + '</div>';
    if (!u.dead) d.onclick = () => { if (!busy) { sel = u; mode = null; render(); } };
    box.appendChild(d);
  });
  // кнопки
  const can = sel && !sel.dead && sel.ap > 0 && !busy && !over;
  $("btnMove").disabled = !can;
  $("btnShoot").disabled = !can;
  $("btnJump").disabled = !can || (computeJumps(sel || {x:-9,y:-9}), jumpTargets.length === 0);
  $("btnComp").disabled = !sel || !sel.comp || busy || over;
  $("btnImp").disabled = !can || !sel.imp;
  $("btnEnd").disabled = busy || over;
  $("btnMove").classList.toggle("on", mode === "move");
  $("btnShoot").classList.toggle("on", mode === "shoot");
  $("btnJump").classList.toggle("on", mode === "jump");
  $("btnComp").classList.toggle("on", compArmed);
  $("tg1").classList.toggle("up", compArmed);
  updatePred();
  draw();
}

function updatePred() {
  const p = $("pred");
  if (!sel) { p.textContent = "Выберите бойца и действие."; return; }
  if (mode === "shoot") {
    const tgt = unitAt(hoverTile.x, hoverTile.y);
    if (tgt && tgt.side === "e") {
      if (!hasLOS(sel, tgt)) { p.innerHTML = "<b>Нет линии огня.</b> Стена перекрывает."; return; }
      const mods = modsFor(sel, tgt, "shoot");
      const m = sumMods(mods);
      const lines = mods.map(x =>
        '<span class="' + (x.v > 0 ? "plus" : "minus") + '">' +
        (x.v > 0 ? "+" : "") + x.v + "</span> " + x.t).join("<br>");
      const nervous = inHum(sel.x, sel.y);
      p.innerHTML = "<b>" + tgt.name + "</b><br>" + (lines || "без модификаторов") +
        "<br>диапазон итога: <b>" + (3 + m) + "–" + (9 + m) + "</b>" +
        (nervous ? '<br><span class="minus">барабаны нервничают: вы в зоне гула</span>' : "");
      return;
    }
    p.innerHTML = "Наведите на противника, чтобы увидеть расклад.";
    return;
  }
  if (mode === "jump") { p.innerHTML = "Прыжок через разлом.<br>≤3 — срыв (−2), 4–7 — тяжёлое приземление, ≥8 — чисто."; return; }
  if (mode === "move") { p.innerHTML = "Выберите клетку. Перемещение бросок не требует."; return; }
  p.innerHTML = "<b>" + sel.name + "</b> · гул " + sel.hum + "/5 · компенсатор " + sel.comp +
    (sel.imp ? " · импульс " + sel.imp : "") + "<br>3 барабана по 1–3. Сумма 3 — провал, 9 — идеальный резонанс.<br>Выстрел и импульс завершают ход бойца.";
}

/* ------------------------------ ввод ----------------------------------- */
cv.addEventListener("mousemove", e => {
  const r = cv.getBoundingClientRect();
  hoverTile = pick(e.clientX - r.left, e.clientY - r.top);
  if (mode === "shoot") updatePred();
});
cv.addEventListener("click", async e => {
  if (busy || over) return;
  const r = cv.getBoundingClientRect();
  const t = pick(e.clientX - r.left, e.clientY - r.top);
  const u = unitAt(t.x, t.y);
  if (mode === "shoot" && u && u.side === "e" && sel && sel.ap > 0) {
    if (!hasLOS(sel, u)) { log("Нет линии огня.", "bad"); return; }
    await doShoot(sel, u); return;
  }
  if (mode === "move" && sel && reach.has(t.x + "," + t.y) && sel.ap > 0) {
    sel.x = t.x; sel.y = t.y; sel.ap--; afterAction(); return;
  }
  if (mode === "jump" && sel && sel.ap > 0) {
    const j = jumpTargets.find(k => k.x === t.x && k.y === t.y);
    if (j) { await doJump(sel, j); return; }
  }
  if (u && u.side === "p") { sel = u; mode = null; render(); return; }
  mode = null; reach = new Map(); jumpTargets = []; render();
});

$("btnMove").onclick  = () => { mode = mode === "move" ? null : "move"; if (mode) computeReach(sel); render(); };
$("btnShoot").onclick = () => { mode = mode === "shoot" ? null : "shoot"; render(); };
$("btnJump").onclick  = () => { mode = mode === "jump" ? null : "jump"; if (mode) computeJumps(sel); render(); };
$("btnComp").onclick  = () => { compArmed = !compArmed; render(); };
$("btnImp").onclick   = async () => { if (sel && sel.imp && sel.ap > 0) await doImpulse(sel); };
$("btnEnd").onclick   = async () => { if (!busy && !over) await enemyTurn(); };

$("chron").onclick = () => {
  $("chron").style.display = "none";
  log("— ход 1 · отряд —", "big");
  log("Виктор: «Верь только приборам. И даже им — не до конца».");
};

window.addEventListener("resize", resize);
resetUnits();
resize();
render();
</script>
</body>
</html>
