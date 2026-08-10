<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>«Звёздный портал» — T-003: стелс-бонус + разрушаемость, тренировочная база</title>
<style>
  :root{
    --ink:#0d0c0a; --panel:#16130f; --panel2:#1e1a14;
    --brass:#c8a45c; --brass-d:#8a7038; --steel:#3a3a3c;
    --paper:#d8cfba; --muted:#8a8171; --red:#a8443a; --blue:#7fa8bd; --green:#7d9464;
  }
  *{box-sizing:border-box}
  html,body{margin:0;padding:0;background:var(--ink);color:var(--paper);
    font-family:"Iowan Old Style","Palatino Linotype",Georgia,serif;
    -webkit-font-smoothing:antialiased;overflow:hidden}
  #app{display:flex;height:100vh;width:100vw}
  #stage{position:relative;flex:1;min-width:0;background:
    radial-gradient(ellipse at 50% 35%, #1a1713 0%, #0d0c0a 70%)}
  canvas{display:block;width:100%;height:100%}
  #side{width:300px;flex:0 0 300px;background:var(--panel);
    border-left:1px solid #2b251c;display:flex;flex-direction:column;overflow:hidden}
  .sec{padding:10px 14px;border-bottom:1px solid #2b251c}
  .sec h3{margin:0 0 8px;font-size:11px;letter-spacing:.18em;text-transform:uppercase;
    color:var(--brass);font-weight:600;font-family:"Helvetica Neue",Arial,sans-serif}
  .unit{padding:8px 10px;margin-bottom:6px;background:var(--panel2);
    border-left:3px solid #444;cursor:pointer;transition:.12s}
  .unit.sel{border-left-color:var(--brass);background:#2a2318}
  .unit.dead{opacity:.35;cursor:default}
  .unit .nm{font-size:14px;display:flex;justify-content:space-between;align-items:baseline}
  .unit .nm small{font-family:"Helvetica Neue",Arial,sans-serif;font-size:10px;
    letter-spacing:.1em;color:var(--muted)}
  .bar{height:5px;background:#000;margin-top:5px}
  .bar i{display:block;height:100%}
  .bar.hp i{background:var(--green)}
  .acts{display:grid;grid-template-columns:1fr 1fr;gap:6px}
  button{font-family:"Helvetica Neue",Arial,sans-serif;font-size:11px;letter-spacing:.06em;
    text-transform:uppercase;padding:9px 6px;background:var(--panel2);color:var(--paper);
    border:1px solid #3a3227;cursor:pointer;transition:.12s}
  button:hover:not(:disabled){background:#302819;border-color:var(--brass-d)}
  button.on{background:var(--brass);color:#17130c;border-color:var(--brass)}
  button:disabled{opacity:.3;cursor:not-allowed}
  button.wide{grid-column:1/3}
  #pred{font-size:12px;line-height:1.55;color:var(--muted);min-height:60px}
  #log{position:absolute;left:14px;bottom:14px;width:400px;max-height:210px;overflow:hidden;
    font-size:12.5px;line-height:1.5;pointer-events:none;
    display:flex;flex-direction:column-reverse}
  #log div{margin-top:3px;text-shadow:0 1px 3px #000;opacity:.92}
  #log .big{color:var(--brass);font-size:13.5px}
  #log .bad{color:#c9705f}
  #log .good{color:#9fbd7f}
  #stealthBadge{position:absolute;top:14px;left:50%;transform:translateX(-50%);
    font-family:"Helvetica Neue",Arial,sans-serif;font-size:11px;letter-spacing:.2em;
    text-transform:uppercase;padding:6px 16px;border-radius:2px}
  #stealthBadge.hidden{background:rgba(127,148,100,.22);color:#9fbd7f;border:1px solid #5c6f45}
  #stealthBadge.spotted{background:rgba(168,68,58,.28);color:#e0958a;border:1px solid #8a4038}
  .leg{font-size:11.5px;line-height:1.8;color:var(--muted)}
  .leg i{display:inline-block;width:9px;height:9px;margin-right:6px;vertical-align:baseline}
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
    <div id="stealthBadge" class="hidden">СКРЫТНОСТЬ · отряд не замечен</div>
    <div id="log"></div>
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
        <button id="btnEnd" class="wide">Завершить ход отряда</button>
      </div>
    </div>
    <div class="sec" style="flex:1">
      <h3>Прибор говорит</h3>
      <div id="pred">Отряд ещё не замечен — двигайтесь к позиции для скрытой атаки.</div>
    </div>
    <div class="sec">
      <h3>Обозначения</h3>
      <div class="leg">
        <i style="background:#5d5346"></i>стена — не разрушается<br>
        <i style="background:#7a6544"></i>ящик — 1 попадание, низкое укрытие<br>
        <i style="background:#7a5a3a"></i>бочка — взрыв 2 урона по соседям<br>
        <i style="background:#5c6a78"></i>слабая стена — только сильный/крит
      </div>
    </div>
  </aside>
</div>

<script>
"use strict";
/* ==========================================================================
   T-003 · спайк: разрушаемость + бонус скрытой атаки поверх канонической
   пошаговой боёвки (AP, три барабана, укрытия — как в prototype_scene.html,
   этот файл не трогает канон, только расширяет его логику в отдельном файле).
   Сцена — тренировочная база Аненербе (Миссия 2 «Гул в темноте»).
   ========================================================================== */

const MAPSTR = [
  "############",
  "#....x.....#",
  "#..o.......#",
  "#....w.w...#",
  "#..........#",
  "#..x....o..#",
  "#..........#",
  "#....##....#",
  "#.x..##..x.#",
  "#..........#",
  "#....o.....#",
  "############"
];
const W = 12, H = 12, TW = 68, TH = 34;

const grid = MAPSTR.map(r => r.split(""));
// destructibles: ключ "x,y" -> {type:'crate'|'barrel'|'weak', hp:1, alive:true}
const DEST_HP = { x: 1, o: 1, w: 1 };
const destructibles = new Map();
for (let y = 0; y < H; y++) for (let x = 0; x < W; x++) {
  const c = grid[y][x];
  if (c === "x" || c === "o" || c === "w") {
    destructibles.set(x + "," + y, { type: c, alive: true });
  }
}
function tile(x, y) { return (x < 0 || y < 0 || x >= W || y >= H) ? "#" : grid[y][x]; }
function destAt(x, y) { const d = destructibles.get(x + "," + y); return (d && d.alive) ? d : null; }
function isWalk(x, y) {
  const t = tile(x, y);
  if (t === "#") return false;
  const d = destAt(x, y);
  if (d && d.type === "w") return false;   // слабая стена блокирует проход, пока цела
  if (d && (d.type === "x" || d.type === "o")) return false; // ящик/бочка тоже физически стоят на клетке
  return true;
}
const dist = (a, b) => Math.hypot(a.x - b.x, a.y - b.y);
const sleep = ms => new Promise(r => setTimeout(r, ms));

/* ------------------------------ юниты -------------------------------------- */
let units = [
  { id:"k", name:"Кессель", role:"солдат", side:"p", x:3, y:9, hp:8, maxHp:8, ap:2, move:5, dead:false, col:"#c8a45c" },
  { id:"v", name:"Виктор", role:"учёный", side:"p", x:5, y:9, hp:6, maxHp:6, ap:2, move:4, dead:false, col:"#7fa8bd" },
  { id:"e1", name:"Инструктор · мишень", role:"противник", side:"e", x:5, y:2, hp:6, maxHp:6, ap:2, move:4, dead:false, col:"#a35b4a" },
  { id:"e2", name:"Инструктор · мишень", role:"противник", side:"e", x:10, y:8, hp:6, maxHp:6, ap:2, move:4, dead:false, col:"#8f4f42" }
];
const alive = () => units.filter(u => !u.dead);
const unitAt = (x, y) => alive().find(u => u.x === x && u.y === y);
const players = () => alive().filter(u => u.side === "p");
const enemies = () => alive().filter(u => u.side === "e");

/* ------------------------------ состояние ----------------------------------- */
let sel = units[0];
let mode = null;
let hoverTile = { x:-1, y:-1 };
let reach = new Map();
let busy = false, over = false;
let stealth = true;          // отряд не обнаружен
let stealthShotUsed = false; // бонус скрытой атаки уже потрачен
let turn = 1;

/* ------------------------------ утилиты -------------------------------------- */
const $ = id => document.getElementById(id);
const cv = $("cv"), ctx = cv.getContext("2d");
let ORX = 0, ORY = 0, DPR = 1;
function resize() {
  DPR = Math.min(window.devicePixelRatio || 1, 2);
  const r = cv.getBoundingClientRect();
  cv.width = r.width * DPR; cv.height = r.height * DPR;
  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
  ORX = r.width / 2; ORY = r.height / 2 - (W + H) * TH / 4 + 30;
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

/* ------------------------------ LOS / укрытие --------------------------------- */
function blocksLOS(x, y) {
  if (tile(x, y) === "#") return true;
  const d = destAt(x, y);
  return !!(d && d.type === "w");
}
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
    if (blocksLOS(x0, y0) && Math.hypot(x0 - x1, y0 - y1) > 1.5) return false;
  }
  return false;
}
function coverOf(shooter, target) {
  const dx = Math.sign(shooter.x - target.x), dy = Math.sign(shooter.y - target.y);
  let best = 0;
  const probes = [[dx, dy], [dx, 0], [0, dy]].filter(p => p[0] || p[1]);
  for (const [px, py] of probes) {
    const t = tile(target.x + px, target.y + py);
    const d = destAt(target.x + px, target.y + py);
    if (t === "#" || (d && d.type === "w")) best = Math.max(best, 3);
    else if (d && (d.type === "x" || d.type === "o")) best = Math.max(best, 2);
  }
  return best;
}
function modsFor(shooter, target) {
  const m = [];
  const c = coverOf(shooter, target);
  if (c === 3) m.push({ t: "полное укрытие цели", v: -3 });
  else if (c === 2) m.push({ t: "низкое укрытие цели", v: -2 });
  const d = dist(shooter, target);
  if (d > 7) m.push({ t: "дальняя дистанция", v: -1 });
  else if (d <= 2.2) m.push({ t: "вплотную", v: +1 });
  if (stealth && !stealthShotUsed && shooter.side === "p") m.push({ t: "скрытая атака", v: +3 });
  return m;
}
const sumMods = m => m.reduce((s, x) => s + x.v, 0);

/* ------------------------------ бросок (без визуальных барабанов — фокус на числах) */
function roll3() { return [1,2,3].map(() => 1 + Math.floor(Math.random() * 3)); }
function outcomeShoot(eff) {
  if (eff <= 3) return { dmg: 0, t: "ПРОВАЛ — промах", cls: "bad" };
  if (eff <= 5) return { dmg: 1, t: "скользящее попадание", cls: "" };
  if (eff <= 7) return { dmg: 2, t: "попадание", cls: "" };
  if (eff <= 8) return { dmg: 3, t: "плотное попадание", cls: "good" };
  return { dmg: 4, t: "ИДЕАЛЬНЫЙ РЕЗОНАНС — крит", cls: "good" };
}
function resolveRoll(mods, forceMin) {
  const barrels = roll3();
  const raw = barrels[0] + barrels[1] + barrels[2];
  let eff = raw + sumMods(mods);
  if (forceMin !== undefined) eff = Math.max(eff, forceMin);
  return { raw, eff, barrels };
}

/* ------------------------------ разрушаемые объекты ---------------------------- */
function destructibleOutcome(type, eff) {
  if (type === "w") return eff >= 8;      // слабая стена — только сильный/крит
  return eff >= 4;                        // ящик/бочка — любой не-провал
}
function destroyAt(x, y, fromChain) {
  const d = destAt(x, y);
  if (!d) return;
  d.alive = false;
  if (d.type === "x") log("Ящик разбит — укрытие снято.", "");
  if (d.type === "w") log("Слабая стена обрушена — открылся проём.", "good");
  if (d.type === "o") {
    log((fromChain ? "Цепная детонация: " : "") + "бочка взрывается!", "big");
    for (let dx = -1; dx <= 1; dx++) for (let dy = -1; dy <= 1; dy++) {
      if (!dx && !dy) continue;
      const nx = x + dx, ny = y + dy;
      const u = unitAt(nx, ny);
      if (u) {
        u.hp -= 2;
        log("  " + u.name + " задет взрывом (−2)", u.side === "p" ? "bad" : "good");
        if (u.hp <= 0) { u.hp = 0; u.dead = true; log("  " + u.name + " выбывает.", "big"); }
      }
      const nd = destAt(nx, ny);
      if (nd && nd.type === "o") destroyAt(nx, ny, true);   // цепная реакция бочка->бочка
      else if (nd && nd.type === "x") destroyAt(nx, ny, true);
    }
  }
}
async function shootDestructible(shooter, x, y) {
  const d = destAt(x, y);
  const mods = [];
  const dd = dist(shooter, { x, y });
  if (dd > 7) mods.push({ t: "дальняя дистанция", v: -1 });
  else if (dd <= 2.2) mods.push({ t: "вплотную", v: +1 });
  if (stealth && !stealthShotUsed) mods.push({ t: "скрытая атака", v: +3 });
  const r = resolveRoll(mods, (stealth && !stealthShotUsed) ? 4 : undefined);
  log(shooter.name + " стреляет по объекту: барабаны " + r.raw + " → итог " + r.eff, "");
  if (destructibleOutcome(d.type, r.eff)) destroyAt(x, y);
  else log("Объект выдержал попадание.", "bad");
  onFirstShotFired(shooter);
  shooter.ap = 0;
  afterAction();
}

/* ------------------------------ действия по юнитам ------------------------------ */
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
function onFirstShotFired(shooter) {
  if (stealth) {
    if (shooter.side === "p" && !stealthShotUsed) {
      stealthShotUsed = true;
      log("Скрытая атака нанесена — бонус потрачен, отряд обнаружен.", "big");
    }
    stealth = false;
    paintStealthBadge();
  }
}
async function doShoot(shooter, target) {
  const mods = modsFor(shooter, target);
  const forceMin = (stealth && !stealthShotUsed && shooter.side === "p") ? 4 : undefined;
  const r = resolveRoll(mods, forceMin);
  const modTxt = mods.length ? " " + mods.map(m => (m.v>0?"+":"")+m.v+" "+m.t).join(",") : "";
  log(shooter.name + " → " + target.name + ": барабаны " + r.raw + modTxt + " → итог " + r.eff, "");
  const o = outcomeShoot(r.eff);
  if (o.dmg > 0) {
    target.hp -= o.dmg;
    log("  " + o.t + " (−" + o.dmg + ")", o.cls);
  } else log("  " + o.t, "bad");
  if (target.hp <= 0) { target.hp = 0; target.dead = true; log(target.name + " выбывает.", "big"); }
  onFirstShotFired(shooter);
  shooter.ap = 0;
  afterAction();
}

function afterAction() { mode = null; reach = new Map(); checkEnd(); render(); }
function checkEnd() {
  if (over) return;
  if (!enemies().length) finish("Цель поражена", "Тренировочная зачистка пройдена. Скрытая атака или нет — принцип один: думать раньше, чем стрелять.");
  else if (!players().length) finish("Отряд выведен из строя", "Даже на тренировке гул и невнимательность наказывают одинаково.");
}
function finish(t, p) { over = true; $("endt").textContent = t; $("endp").textContent = p; $("endcard").style.display = "flex"; }

/* ------------------------------ ход противника (детекция + бой) ------------------ */
async function enemyTurn() {
  busy = true; sel = null; mode = null; reach = new Map(); render();

  // Пока отряд в стелсе — противники не действуют по кулдауну оружия,
  // но с шансом 12% за проверку замечают отряд (раздел 2 задачи T-003).
  if (stealth) {
    for (const e of enemies()) {
      const seesSomeone = players().some(p => hasLOS(e, p) && dist(e, p) <= 4);
      if (seesSomeone && Math.random() < 0.12) {
        stealth = false;
        paintStealthBadge();
        log(e.name + " замечает отряд! Скрытность потеряна, бонус не получен.", "big");
        break;
      }
    }
  }

  if (!stealth) {
    log("— ход противника —", "big");
    for (const e of enemies()) {
      e.ap = 2;
      while (e.ap > 0 && players().length && !over) {
        await sleep(260);
        const tgt = players().filter(p => hasLOS(e, p) && dist(e, p) <= 9)
          .sort((a, b) => dist(e, a) - dist(e, b))[0];
        if (tgt) { await shootAI(e, tgt); }
        else {
          const goal = players().sort((a, b) => dist(e, a) - dist(e, b))[0];
          if (!goal) break;
          stepToward(e, goal); e.ap--;
        }
        render();
      }
      if (over) break;
    }
  } else {
    log("Отряд остаётся незамеченным — ход противника пропущен.", "good");
  }

  if (!over) { turn++; players().forEach(p => p.ap = 2); log("— ход " + turn + " · отряд —", "big"); }
  checkEnd(); busy = false; render();
}
async function shootAI(e, tgt) {
  const mods = modsFor(e, tgt);
  const r = resolveRoll(mods);
  const o = outcomeShoot(r.eff);
  if (o.dmg > 0) { tgt.hp -= o.dmg; log(e.name + " → " + tgt.name + ": " + o.t + " (−" + o.dmg + ")", "bad"); }
  else log(e.name + " → " + tgt.name + ": промах", "");
  if (tgt.hp <= 0) { tgt.hp = 0; tgt.dead = true; log(tgt.name + " выбывает.", "big"); }
  e.ap = 0;
}
function stepToward(u, goal) {
  const key = (x, y) => x + "," + y;
  const prev = new Map(); const q = [[u.x, u.y]]; const seen = new Set([key(u.x, u.y)]);
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
  const path = []; let cur = found;
  while (cur && !(cur[0] === u.x && cur[1] === u.y)) { path.unshift(cur); cur = prev.get(key(cur[0], cur[1])); }
  path.pop();
  const stepsAllowed = Math.min(u.move, path.length);
  if (stepsAllowed > 0) { const d = path[stepsAllowed - 1]; u.x = d[0]; u.y = d[1]; }
}

/* ------------------------------ отрисовка ---------------------------------------- */
function tileTop(x, y) {
  const t = tile(x, y);
  if (t === "#") return "#5d5346";
  const d = destAt(x, y);
  if (d) return d.type === "x" ? "#7a6544" : d.type === "o" ? "#7a5a3a" : "#5c6a78";
  return "#3b372f";
}
function drawTile(x, y) {
  const px = sx(x, y), py = sy(x, y);
  ctx.beginPath();
  ctx.moveTo(px, py); ctx.lineTo(px + TW/2, py + TH/2);
  ctx.lineTo(px, py + TH); ctx.lineTo(px - TW/2, py + TH/2);
  ctx.closePath();
  ctx.fillStyle = tileTop(x, y); ctx.fill();
  ctx.strokeStyle = "rgba(0,0,0,.4)"; ctx.lineWidth = 1; ctx.stroke();
  const k = x + "," + y;
  if (mode === "move" && reach.has(k)) { ctx.fillStyle = "rgba(200,164,92,.20)"; ctx.fill(); ctx.strokeStyle = "rgba(200,164,92,.5)"; ctx.stroke(); }
  if (hoverTile.x === x && hoverTile.y === y) { ctx.strokeStyle = "rgba(236,224,196,.85)"; ctx.lineWidth = 2; ctx.stroke(); }
  const t = tile(x, y); const d = destAt(x, y);
  if (t === "#" || d) {
    const hgt = t === "#" ? 30 : d.type === "w" ? 26 : 16;
    const top = py - hgt;
    ctx.beginPath();
    ctx.moveTo(px, top); ctx.lineTo(px + TW/2, top + TH/2);
    ctx.lineTo(px, top + TH); ctx.lineTo(px - TW/2, top + TH/2);
    ctx.closePath();
    ctx.fillStyle = t === "#" ? "#6d6252" : d.type === "w" ? "#6f7f8d" : d.type === "o" ? "#8a6a42" : "#93794f";
    ctx.fill(); ctx.strokeStyle = "rgba(0,0,0,.5)"; ctx.lineWidth = 1; ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(px - TW/2, top + TH/2); ctx.lineTo(px, top + TH); ctx.lineTo(px, py + TH); ctx.lineTo(px - TW/2, py + TH/2);
    ctx.closePath(); ctx.fillStyle = "rgba(0,0,0,.35)"; ctx.fill();
    ctx.beginPath();
    ctx.moveTo(px + TW/2, top + TH/2); ctx.lineTo(px, top + TH); ctx.lineTo(px, py + TH); ctx.lineTo(px + TW/2, py + TH/2);
    ctx.closePath(); ctx.fillStyle = "rgba(0,0,0,.18)"; ctx.fill();
  }
}
function drawUnit(u) {
  const px = sx(u.x, u.y), py = sy(u.x, u.y) + TH / 2;
  ctx.beginPath(); ctx.ellipse(px, py, 17, 8, 0, 0, Math.PI * 2); ctx.fillStyle = "rgba(0,0,0,.45)"; ctx.fill();
  if (sel === u) { ctx.beginPath(); ctx.ellipse(px, py, 22, 11, 0, 0, Math.PI * 2); ctx.strokeStyle = "#c8a45c"; ctx.lineWidth = 2; ctx.stroke(); }
  const dim = (stealth && u.side === "e") ? 0.55 : 1;
  ctx.globalAlpha = dim;
  ctx.beginPath();
  ctx.roundRect ? ctx.roundRect(px - 8, py - 34, 16, 28, 5) : ctx.rect(px - 8, py - 34, 16, 28);
  ctx.fillStyle = u.col; ctx.fill(); ctx.strokeStyle = "rgba(0,0,0,.6)"; ctx.lineWidth = 1.2; ctx.stroke();
  ctx.beginPath(); ctx.arc(px, py - 41, 7, 0, Math.PI * 2); ctx.fillStyle = u.col; ctx.fill(); ctx.stroke();
  ctx.globalAlpha = 1;
  const bw = 26;
  ctx.fillStyle = "#000"; ctx.fillRect(px - bw/2, py - 56, bw, 4);
  ctx.fillStyle = u.side === "p" ? "#7d9464" : "#a8443a";
  ctx.fillRect(px - bw/2, py - 56, bw * (u.hp / u.maxHp), 4);
}
function draw() {
  const r = cv.getBoundingClientRect();
  ctx.clearRect(0, 0, r.width, r.height);
  for (let y = 0; y < H; y++) for (let x = 0; x < W; x++) drawTile(x, y);
  alive().slice().sort((a, b) => (a.x + a.y) - (b.x + b.y)).forEach(drawUnit);
  if (mode === "shoot" && sel) {
    const t = unitAt(hoverTile.x, hoverTile.y) || destAt(hoverTile.x, hoverTile.y);
    if (t) {
      const tx = t.x !== undefined ? t.x : hoverTile.x, ty = t.y !== undefined ? t.y : hoverTile.y;
      ctx.beginPath();
      ctx.moveTo(sx(sel.x, sel.y), sy(sel.x, sel.y) + TH/2 - 20);
      ctx.lineTo(sx(hoverTile.x, hoverTile.y), sy(hoverTile.x, hoverTile.y) + TH/2 - 20);
      ctx.strokeStyle = "rgba(236,224,196,.75)"; ctx.lineWidth = 1.5; ctx.setLineDash([5,4]); ctx.stroke(); ctx.setLineDash([]);
    }
  }
}
setInterval(() => { if (!over) draw(); }, 60);

/* ------------------------------ интерфейс ------------------------------------- */
function paintStealthBadge() {
  const b = $("stealthBadge");
  if (stealth) { b.textContent = "СКРЫТНОСТЬ · отряд не замечен"; b.className = "hidden"; }
  else { b.textContent = "ОБНАРУЖЕНЫ · бой"; b.className = "spotted"; }
}
function render() {
  const box = $("roster"); box.innerHTML = "";
  units.forEach(u => {
    if (u.side !== "p") return;
    const d = document.createElement("div");
    d.className = "unit" + (sel === u ? " sel" : "") + (u.dead ? " dead" : "");
    d.innerHTML = '<div class="nm">' + u.name + ' <small>' + u.role + '</small></div>' +
      '<div class="bar hp"><i style="width:' + (100*Math.max(0,u.hp)/u.maxHp) + '%"></i></div>';
    if (!u.dead) d.onclick = () => { if (!busy) { sel = u; mode = null; render(); } };
    box.appendChild(d);
  });
  const can = sel && !sel.dead && sel.ap > 0 && !busy && !over;
  $("btnMove").disabled = !can; $("btnShoot").disabled = !can; $("btnEnd").disabled = busy || over;
  $("btnMove").classList.toggle("on", mode === "move");
  $("btnShoot").classList.toggle("on", mode === "shoot");
  updatePred(); draw();
}
function updatePred() {
  const p = $("pred");
  if (!sel) { p.textContent = "Выберите бойца."; return; }
  if (mode === "shoot") {
    const tgt = unitAt(hoverTile.x, hoverTile.y);
    const d = destAt(hoverTile.x, hoverTile.y);
    if (tgt && tgt.side === "e") {
      if (!hasLOS(sel, tgt)) { p.innerHTML = "<b>Нет линии огня.</b>"; return; }
      const mods = modsFor(sel, tgt); const m = sumMods(mods);
      p.innerHTML = "<b>" + tgt.name + "</b><br>" + mods.map(x => (x.v>0?"+":"")+x.v+" "+x.t).join("<br>") +
        "<br>диапазон: " + (3+m) + "–" + (9+m) +
        (stealth && !stealthShotUsed ? "<br><b>скрытая атака: мин. итог 4</b>" : "");
      return;
    }
    if (d) { p.innerHTML = "<b>" + (d.type==="x"?"Ящик":d.type==="o"?"Бочка":"Слабая стена") + "</b><br>" +
      (d.type==="w" ? "нужен «сильный»(8) или крит(9)" : "разрушается любым попаданием"); return; }
    p.innerHTML = "Наведите на цель.";
    return;
  }
  if (mode === "move") { p.innerHTML = "Выберите клетку."; return; }
  p.innerHTML = "<b>" + sel.name + "</b> · AP " + sel.ap +
    (stealth ? "<br>Отряд в стелсе — первый выстрел получит +3 и не провалится ниже 4." : "<br>Бой идёт открыто.");
}
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
  const d = destAt(t.x, t.y);
  if (mode === "shoot" && sel && sel.ap > 0) {
    if (u && u.side === "e") { if (!hasLOS(sel, u)) { log("Нет линии огня.", "bad"); return; } await doShoot(sel, u); return; }
    if (d && hasLOS(sel, { x: t.x, y: t.y })) { await shootDestructible(sel, t.x, t.y); return; }
  }
  if (mode === "move" && sel && reach.has(t.x + "," + t.y) && sel.ap > 0) { sel.x = t.x; sel.y = t.y; sel.ap--; afterAction(); return; }
  if (u && u.side === "p") { sel = u; mode = null; render(); return; }
  mode = null; reach = new Map(); render();
});
$("btnMove").onclick = () => { mode = mode === "move" ? null : "move"; if (mode) computeReach(sel); render(); };
$("btnShoot").onclick = () => { mode = mode === "shoot" ? null : "shoot"; render(); };
$("btnEnd").onclick = async () => { if (!busy && !over) await enemyTurn(); };

window.addEventListener("resize", resize);
resize(); render();
</script>
</body>
</html>
