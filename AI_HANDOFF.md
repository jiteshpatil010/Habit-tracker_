# AI Handoff - Habit Tracker

Read this first. It lets any AI assistant or developer continue the project without the original chat.

## 1. What this is
A single-file, mobile-first Habit Tracker PWA (vanilla HTML/CSS/JS, no framework, no build). Built to replace a handwritten notebook tracker (columns = habits, rows = days of the month, X marks, totals row, line graph). Hosted on GitHub Pages. Data lives in `localStorage` (key `ht1`). Test-data seeding and the Settings "Test data" section were removed from the final build. Logo: user-supplied glossy blue calendar/check/bar-chart icon (`icons/`).

Status: working trial, tested on Chromium at 390x844. NOT yet verified on real iPhone Safari, and the real PDF/Excel output files have not been inspected (see section 8).

## 2. Condensed chat history (not verbatim)
1. User asked if a habit tracker (like their earlier expense app) is feasible and about token cost; they were hitting "limit exhausted". Advice: build in stages, small edits, one chat per app.
2. User shared a photo of their notebook tracker: habits as columns, days as rows, X per done day, totals row, line graph of per-habit totals with up/down arrows. Requirements: those habits as defaults; add/remove extra habits; month-end report like the photo plus in-month progress; iOS-style UI; edit only previous 2 days; live graphs; make a trial app.
3. Round 2: good colour grading, user-selectable theme (light/dark), menu to reach previous/current months, settings, backup/restore, link to try on iPhone, random test data for the last 5 months.
4. Round 3: menu should show a "Months" button that expands history; streaks limited to the month (no carry-over) and change the fire icon; frozen header row (habit names) and Day column in the monthly report; report header shows max 20 characters of a habit name so row height stays fixed; backup/restore as a file instead of copy-paste; remove Habits tab from bottom bar (moved to menu).
5. Round 4: user asked PDF vs Excel opinion; recommendation was PDF as primary (fixed record, graphs reliable) with Excel as optional data export; user chose to offer both after tapping "Download report".
6. Round 5: user asked for testing (UI, bugs, speed with a year of data). Found/fixed: unescaped HTML in habit names, negative hours, stale date after midnight, sticky column overlapping tab bar. 1-5 years of data renders in <13 ms.
7. Round 6: package for GitHub deployment with the uploaded logo, plus this handoff file. Changes for standalone hosting: standard browser download/share instead of the Claude-sandbox download API, PWA manifest + service worker + icons, no auto test data on first run, default accent "Ocean" to match logo.
8. Final cleanup: removed the Test data section (Load test data / Clear all logs), its seed code, and bumped the service worker to ht-v2.

## 3. Requirements and rules (keep these)
- Default habits (in order): 6 AM wake, Exercise, No films, Read book (1 hr), Geetha, No fast food, 4 L water, Minox (morning), Minox (evening), Journal, Learn something new, Sleep by 11:30 pm, YouTube (hrs, Hours type), OTT (hrs, Hours type). "No films" is an interpretation of unclear handwriting ("No F(l/m)") - confirm with the user.
- Users can add/remove habits (Yes/No or Hours). Removing hides the habit; logs remain in storage.
- Editing is allowed only for today and the 2 previous days (`ed(k)`: days-ago 0..2). Future and older days are locked everywhere (UI and `set()`).
- Streak = consecutive done days ending today (or yesterday if today not done), limited to the current calendar month.
- Report header shows habit name truncated to 20 chars with an ellipsis; full name in the title tooltip and elsewhere.
- Report grid: frozen header row and Day column; totals row; shaded rows = editable days.
- Hours-type habits are not counted in completion % or streaks; their totals are summed.
- All user text must go through `esc()` before being inserted via innerHTML.
- Backup/restore is file based (`.json`), restore validates `habits` array, `logs` object and habit id format.

## 4. File map
- `index.html` - the whole app (HTML + CSS + JS)
- `manifest.webmanifest` - PWA manifest
- `sw.js` - service worker (offline cache; bump `VERSION` each release)
- `icons/` - `icon-192.png`, `icon-512.png`, `maskable-512.png`, `apple-touch-icon.png`, `favicon-32.png` (cropped full-bleed from the uploaded logo so iOS rounding has no black corners)
- `.nojekyll`, `README.md`, `AI_HANDOFF.md`
- External scripts (cdnjs, deferred): jsPDF 2.5.1, SheetJS xlsx 0.18.5

## 5. Data model (`localStorage['ht1']`)
```json
{"habits":[{"id":"h0","n":"6 AM wake","e":"⏰","t":"c"}],
 "logs":{"2026-11-01":{"h0":true,"h12":2.5}},
 "set":{"theme":"auto|light|dark","ac":0}}
```
`t` is `c` (Yes/No, value `true`) or `h` (Hours, number 0-24). Dates are local `YYYY-MM-DD`. Custom habit ids are `u<timestamp>`.

## 6. Logic overview (functions in index.html)
- State: `S` (data), `tab` (today|month|manage|set), `sel` (0-2 days ago on Today), `mon` ({y,m}), `dr` (drawer), `mo` (months expanded), `rp` (report sheet).
- `key(d)`, `dayK(n)`, `ago(k)`, `ed(k)`, `get`, `set` - date helpers and guarded writes (`set` re-renders).
- `streak(id)`, `mp(y,m)` (month completion %), `md(y,m)` (month data for exports).
- Views: `today()`, `month()`, `manage()`, `settings()`, `drawer()`, `rpt()`; `render()` rebuilds `#app`; one delegated click handler `act(el)` driven by `data-a` attributes.
- Charts: `line(values,labels,max)` returns inline SVG (per-habit totals, daily completed count).
- Theme: `theme()` applies auto/light/dark and accent via CSS variables (`--ac`, `--a1`, `--g2`); palette array `AC`.
- Exports: `exportPdf` (jsPDF: grid page + graph page), `exportXlsx` (SheetJS: "Monthly report" sheet with 1/0 cells and SUM formulas, "Summary" sheet; no charts), `dl()` (share sheet on touch devices if files can be shared, else anchor download).
- Backup: `bkSave()` -> `.json`; `restoreFile(file)`.
- `visibilitychange` refreshes "today" after midnight.

## 7. Deployment
GitHub Pages from branch (`main`, `/ root`). See README. After edits, bump `VERSION` in `sw.js`.

## 8. Testing status and known limits
Passed (Chromium, 390x844): layout/no horizontal overflow, sticky header/column, edit lock, month-limited streaks, menu expansion, themes, backup/restore, offline load, HTML-escaping, performance (5 years: ~12 ms slowest render, 216 KB storage).
Not verified: real iPhone Safari behaviour; real generated PDF/XLSX appearance (tested with stand-ins); month rollover on a live device; multi-device sync (not supported - each device has its own data).
Known limits: Excel export has no charts (SheetJS community edition); PDF cannot render emoji or non-Latin names; iOS may purge site data for long-unused non-installed sites (advise Home Screen install + backups); numeric input uses dot decimals.

## 9. Ideas / backlog
Per-month habit lists, reminders, habit notes, goals/targets per habit, year overview, CSV import, optional cloud sync, up/down trend arrows on the totals graph (as in the notebook), weekday-specific habits.

## 10. Conventions for further changes
Keep it a single dependency-light file; use `esc()` for any user text in HTML; keep the 3-day edit rule inside `set()`; keep `data-a` delegated handlers; test at 390x844 with 5 years of seeded data; bump the service worker `VERSION`.

---
# Appendix A - index.html (complete source)
````html
<!DOCTYPE html><html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Habit Tracker</title>
<meta name="description" content="Daily habit tracker with monthly reports, streaks, graphs, PDF and Excel export. Works offline."><meta name="theme-color" content="#0b6fd6"><meta name="apple-mobile-web-app-capable" content="yes"><meta name="mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" content="default"><link rel="manifest" href="manifest.webmanifest"><link rel="icon" type="image/png" sizes="32x32" href="icons/favicon-32.png"><link rel="apple-touch-icon" href="icons/apple-touch-icon.png"><meta name="apple-mobile-web-app-title" content="Habits">
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script><script defer src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<style>
:root{--bg:#eef1f8;--card:#fff;--tx:#172033;--mu:#6a7590;--ac:#0b6fd6;--a1:#0b6fd6;--g2:#22d3ee;--ok:#10b981;--ln:#e2e7f1;--wn:#f59e0b;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#10131f;--card:#1a1f33;--tx:#eef0f8;--mu:#98a0bd;--ac:#7c93ff;--ok:#4cc38a;--ln:#2a3050;--wn:#ffa94d}}
:root[data-theme="dark"]{--bg:#10131f;--card:#1a1f33;--tx:#eef0f8;--mu:#98a0bd;--ac:#7c93ff;--ok:#4cc38a;--ln:#2a3050;--wn:#ffa94d}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--tx);font:15px/1.4 -apple-system,"Segoe UI",Roboto,sans-serif}
#app{max-width:760px;margin:0 auto;padding:16px 14px 90px}
h1{font-size:22px;margin:4px 0 12px}h2{font-size:16px;margin:0 0 8px}
.card{background:var(--card);border-radius:16px;padding:14px;margin-bottom:12px;box-shadow:0 1px 3px rgba(0,0,0,.08)}
.mu{color:var(--mu);font-size:13px}.row{display:flex;align-items:center;gap:10px}.sp{justify-content:space-between}
.chips{display:flex;gap:8px;margin-bottom:12px}.chip{flex:1;padding:8px;border-radius:12px;border:1px solid var(--ln);background:var(--card);color:var(--tx);text-align:center;font-size:13px}
.chip.on{background:var(--ac);color:#fff;border-color:var(--ac)}
.h{display:flex;align-items:center;gap:12px;padding:10px 0;border-top:1px solid var(--ln)}.h:first-of-type{border-top:0}
.h .n{flex:1}.h .s{font-size:12px;color:var(--wn)}
.ck{width:34px;height:34px;border-radius:50%;border:2px solid var(--ln);background:none;color:#fff;font-size:18px}
.ck.on{background:var(--ok);border-color:var(--ok)}
input,select,button{font:inherit}input,select{padding:8px;border-radius:10px;border:1px solid var(--ln);background:var(--bg);color:var(--tx)}
input[type=number]{width:78px;text-align:center}
.btn{padding:9px 14px;border-radius:10px;border:0;background:var(--ac);color:#fff}.sh{position:fixed;left:0;right:0;bottom:0;z-index:21;background:var(--card);padding:18px 16px calc(18px + env(safe-area-inset-bottom,0px));border-radius:18px 18px 0 0;max-width:760px;margin:0 auto}.btn.g{background:var(--ln);color:var(--tx)}.btn.r{background:#e03131}
.tbl{overflow:auto;max-height:72vh;border-radius:10px;-webkit-overflow-scrolling:touch}table{border-collapse:collapse;width:100%;min-width:560px;text-align:center;font-size:13px}
td,th{border:1px solid var(--ln);padding:0;height:30px;min-width:34px}th.v{height:165px;vertical-align:bottom;font-weight:500;padding:4px 0}th{position:sticky;top:0;z-index:2;background:var(--card)}th:first-child{left:0;z-index:3}
th.v span{writing-mode:vertical-rl;transform:rotate(180deg);white-space:nowrap;display:inline-block}
td.d{background:var(--bg);font-weight:600;position:sticky;left:0;z-index:1}td.e{cursor:pointer;background:rgba(59,91,219,.07)}td.f{opacity:.35}
td.x{color:var(--ac);font-weight:700;background:color-mix(in srgb,var(--ac) 20%,transparent)}tr.t td{font-weight:700;background:var(--bg)}
.nav{display:flex;align-items:center;justify-content:space-between;margin-bottom:12px}.nav button{background:var(--card);border:1px solid var(--ln);color:var(--tx);border-radius:10px;padding:6px 14px;font-size:18px}
.stats{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:12px}.stats div{background:var(--card);border-radius:14px;padding:10px;text-align:center}.stats b{display:block;font-size:20px}
.bar{z-index:10;position:fixed;left:0;right:0;bottom:0;display:flex;background:var(--card);border-top:1px solid var(--ln);padding:6px 0 calc(6px + env(safe-area-inset-bottom,0px))}
.bar button{flex:1;background:none;border:0;color:var(--mu);padding:6px;font-size:12px}.bar button.on{color:var(--ac);font-weight:600}.bar span{display:block;font-size:20px}
svg{width:100%;height:auto;display:block}
.top{display:flex;align-items:center;gap:10px;margin-bottom:12px}.top h1{margin:0;flex:1}.mb{background:var(--card);border:1px solid var(--ln);color:var(--tx);border-radius:12px;width:40px;height:40px;font-size:20px;padding:0!important}
.hero{background:linear-gradient(135deg,var(--a1),var(--g2));color:#fff}.hero .mu{color:rgba(255,255,255,.88)}
.ov{position:fixed;inset:0;background:rgba(0,0,0,.45);z-index:20}.dr{position:fixed;top:0;bottom:0;left:0;width:82%;max-width:320px;background:var(--card);z-index:21;padding:calc(16px + env(safe-area-inset-top,0px)) 14px 20px;overflow-y:auto}
.mi{display:flex;justify-content:space-between;width:100%;padding:11px 12px;margin-bottom:4px;border:0;border-radius:10px;background:none;color:var(--tx);text-align:left}.mi.on{background:color-mix(in srgb,var(--ac) 16%,transparent);color:var(--ac);font-weight:600}
.sw{display:flex;gap:12px}.sw button{width:36px;height:36px;border-radius:50%;border:3px solid transparent}.sw button.on{border-color:var(--tx)}
textarea{width:100%;height:90px;border-radius:10px;border:1px solid var(--ln);background:var(--bg);color:var(--tx);font:12px monospace;padding:8px}
</style></head><body><div id="app"></div>
<div class="bar" id="bar"></div>
<script>
const D=[["6 AM wake","⏰"],["Exercise","💪"],["No films","🎬"],["Read book (1 hr)","📚"],["Geetha","🕉️"],["No fast food","🍔"],["4 L water","💧"],["Minox (morning)","🧴"],["Minox (evening)","🧴"],["Journal","📓"],["Learn something new","💡"],["Sleep by 11:30 pm","😴"],["YouTube (hrs)","▶️","h"],["OTT (hrs)","📺","h"]];
let S;try{S=JSON.parse(localStorage.getItem('ht1'))}catch(e){}
if(!S||!S.habits)S={habits:D.map((d,i)=>({id:'h'+i,n:d[0],e:d[1],t:d[2]||'c'})),logs:{}};
S.set=S.set||{theme:'auto',ac:0};
const save=()=>{try{localStorage.setItem('ht1',JSON.stringify(S))}catch(e){}};
const AC=[['Ocean','#0b6fd6','#4da3ff','#22d3ee'],['Indigo','#4f46e5','#818cf8','#a78bfa'],['Teal','#0d9488','#2dd4bf','#38bdf8'],['Rose','#e11d48','#fb7185','#f59e0b'],['Amber','#d97706','#fbbf24','#f43f5e']];
function theme(){const t=S.set.theme,r=document.documentElement,dk=t==='dark'||(t==='auto'&&matchMedia('(prefers-color-scheme:dark)').matches),a=AC[S.set.ac]||AC[0];if(t==='auto')r.removeAttribute('data-theme');else r.dataset.theme=t;r.style.setProperty('--ac',dk?a[2]:a[1]);r.style.setProperty('--a1',a[1]);r.style.setProperty('--g2',a[3])}
function mp(y,m){const c=chk(),N=new Date(y,m+1,0).getDate();let t=0,n=0;for(let i=1;i<=N;i++){const k=y+'-'+pad(m+1)+'-'+pad(i);if(ago(k)<0)break;n++;c.forEach(h=>{if(get(k,h.id))t++})}return n&&c.length?Math.round(t/(n*c.length)*100):0}
const rgb=h=>[1,3,5].map(i=>parseInt(h.slice(i,i+2),16));
async function dl(name,data){const ty=name.endsWith('.pdf')?'application/pdf':name.endsWith('.xlsx')?'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet':'application/json',blob=new Blob([data],{type:ty});try{const f=new File([blob],name,{type:ty});if(matchMedia('(pointer:coarse)').matches&&navigator.canShare&&navigator.canShare({files:[f]})){await navigator.share({files:[f]});msg('Saved \u2713');return}}catch(e){if(e&&e.name==='AbortError'){msg('Cancelled');return}}try{const u=URL.createObjectURL(blob),a=document.createElement('a');a.href=u;a.download=name;document.body.appendChild(a);a.click();a.remove();setTimeout(()=>URL.revokeObjectURL(u),5000);msg('Saved \u2713')}catch(e){msg('Could not save the file')}}
function md(y,m){const N=new Date(y,m+1,0).getDate(),days=[...Array(N)].map((_,i)=>key(new Date(y,m,i+1))),H=S.habits,c=H.filter(h=>h.t==='c'),past=days.filter(k=>ago(k)>=0);return{N,days,H,c,past,tot:H.map(h=>days.reduce((s,k)=>s+(+get(k,h.id)||0),0)),daily:past.map(k=>c.filter(h=>get(k,h.id)).length),pc:mp(y,m),title:new Date(y,m,1).toLocaleDateString('en-IN',{month:'long',year:'numeric'})}}
function pdfLine(doc,a,x,y,w,h,v,l,mx,title){doc.setTextColor(30);doc.setFontSize(11);doc.text(title,x,y-5);doc.setDrawColor(210);doc.setLineWidth(.2);doc.setFontSize(7);const st=Math.ceil(mx/5)||1,px=i=>x+(v.length<2?w/2:i*w/(v.length-1)),py=q=>y+h-(q/mx)*h;for(let t=0;t<=mx;t+=st){doc.line(x,py(t),x+w,py(t));doc.text(String(t),x-2,py(t)+1,{align:'right'})}doc.setDrawColor(...a);doc.setFillColor(...a);doc.setLineWidth(.6);for(let i=1;i<v.length;i++)doc.line(px(i-1),py(v[i-1]),px(i),py(v[i]));v.forEach((q,i)=>{doc.circle(px(i),py(q),.9,'F');doc.setFontSize(6);doc.text(String(q),px(i),py(q)-2,{align:'center'});doc.text(String(l[i]),px(i),y+h+4,{align:'center'})});doc.setLineWidth(.2)}
async function exportPdf(y,m){const J=window.jspdf;if(!J){msg('PDF tool did not load, check your internet');return}msg('Preparing PDF...');const d=md(y,m),doc=new J.jsPDF({unit:'mm',format:'a4'}),a=rgb(AC[S.set.ac][1]),n=d.H.length,x0=10,dw=10,cw=(190-dw)/n,hh=32,rh=5,y0=34,tint=a.map(c=>Math.round(c+(255-c)*.8));
doc.setFillColor(...a);doc.rect(0,0,210,24,'F');doc.setTextColor(255);doc.setFontSize(18);doc.text('Habit Tracker - '+d.title,10,14);doc.setFontSize(9);doc.text('Completion '+d.pc+'%   |   '+d.daily.reduce((s,v)=>s+v,0)+' ticks   |   '+d.c.length+' daily habits',10,20);
doc.setTextColor(30);doc.setDrawColor(200);doc.setLineWidth(.2);doc.setFontSize(7);doc.rect(x0,y0,dw,hh);doc.text('Day',x0+dw/2,y0+hh-2,{align:'center'});
d.H.forEach((h,i)=>{const x=x0+dw+i*cw;doc.rect(x,y0,cw,hh);doc.text(h.n.length>20?h.n.slice(0,19)+'...':h.n,x+cw/2+1.2,y0+hh-1.5,{angle:90})});
d.days.forEach((k,r)=>{const yy=y0+hh+r*rh;doc.rect(x0,yy,dw,rh);doc.text(String(r+1),x0+dw/2,yy+3.6,{align:'center'});d.H.forEach((h,i)=>{const x=x0+dw+i*cw,v=get(k,h.id);if(v&&h.t==='c'){doc.setFillColor(...tint);doc.rect(x,yy,cw,rh,'FD');doc.text('X',x+cw/2,yy+3.6,{align:'center'})}else{doc.rect(x,yy,cw,rh);if(v)doc.text(String(v),x+cw/2,yy+3.6,{align:'center'})}})});
const ty=y0+hh+d.N*rh;doc.setFont('helvetica','bold');doc.rect(x0,ty,dw,rh);doc.text('Total',x0+dw/2,ty+3.6,{align:'center'});d.H.forEach((h,i)=>{const x=x0+dw+i*cw;doc.rect(x,ty,cw,rh);doc.text(String(Math.round(d.tot[i]*100)/100),x+cw/2,ty+3.6,{align:'center'})});doc.setFont('helvetica','normal');
doc.addPage();const ci=d.H.map((h,i)=>[h,i]).filter(p=>p[0].t==='c'),cv=ci.map(p=>d.tot[p[1]]);pdfLine(doc,a,22,32,165,60,cv,ci.map(p=>p[1]+1),Math.max(5,...cv),'Days each habit was done (numbers match the legend)');doc.setFontSize(7);doc.setTextColor(60);d.H.forEach((h,i)=>doc.text((i+1)+' = '+h.n,22+(i%2)*85,106+Math.floor(i/2)*4));
if(d.past.length)pdfLine(doc,a,22,Math.max(170,112+Math.ceil(n/2)*4+20),165,60,d.daily,d.past.map(k=>+k.slice(8)),Math.max(5,d.c.length),'Habits completed per day');
await dl('habit-report-'+y+'-'+pad(m+1)+'.pdf',doc.output('arraybuffer'))}
async function exportXlsx(y,m){const X=window.XLSX;if(!X){msg('Excel tool did not load, check your internet');return}const d=md(y,m),rows=[['Day',...d.H.map(h=>h.n)]];d.days.forEach((k,i)=>rows.push([i+1,...d.H.map(h=>{const v=get(k,h.id);return h.t==='c'?(v?1:0):(v||0)})]));rows.push(['Total',...d.tot.map(t=>Math.round(t*100)/100)]);
const ws=X.utils.aoa_to_sheet(rows);ws['!cols']=[{wch:7},...d.H.map(h=>({wch:Math.max(9,Math.min(22,h.n.length+2))}))];d.H.forEach((h,i)=>{const c=X.utils.encode_col(i+1);ws[c+(d.N+2)]={t:'n',f:'SUM('+c+'2:'+c+(d.N+1)+')',v:Math.round(d.tot[i]*100)/100}});
const sm=[['Habit Tracker',d.title],['Completion %',d.pc],['Total ticks',d.daily.reduce((s,v)=>s+v,0)],[],['Habit','Type','Total','% of days done']];d.H.forEach((h,i)=>sm.push([h.n,h.t==='c'?'Yes/No':'Hours',Math.round(d.tot[i]*100)/100,h.t==='c'&&d.past.length?Math.round(d.tot[i]/d.past.length*100):'']));
const s2=X.utils.aoa_to_sheet(sm);s2['!cols']=[{wch:26},{wch:10},{wch:10},{wch:16}];const wb=X.utils.book_new();X.utils.book_append_sheet(wb,ws,'Monthly report');X.utils.book_append_sheet(wb,s2,'Summary');
await dl('habit-report-'+y+'-'+pad(m+1)+'.xlsx',X.write(wb,{type:'array',bookType:'xlsx'}))}
function rpt(){return `<div class="ov" data-a="rp" data-v="0"></div><div class="sh"><h2>${md(mon.y,mon.m).title} report</h2><button class="btn" style="width:100%;margin-bottom:8px" data-a="pdf">📄 PDF – grid + graphs, read-only</button><button class="btn g" style="width:100%" data-a="xls">📊 Excel – editable data, no graphs</button><div class="mu" id="msg" style="margin-top:10px"></div></div>`}
const esc=s=>String(s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const hdr=t=>`<div class="top"><button class="mb" data-a="dr" data-v="1">☰</button><h1>${t}</h1></div>`;
const msg=t=>{const m=document.getElementById('msg');if(m)m.textContent=t};
async function bkSave(){await dl('habit-backup-'+key(now)+'.json',JSON.stringify(S))}
async function restoreFile(f){if(!f)return;try{const o=JSON.parse(await f.text());if(!Array.isArray(o.habits)||typeof o.logs!=='object'||o.habits.some(h=>!/^[\w-]+$/.test(h.id)))throw 0;S=o;S.set=S.set||{theme:'auto',ac:0};save();theme();render();msg('Restored ✓ ('+Object.keys(S.logs).length+' days)')}catch(e){msg('That file is not a valid backup')}}
function drawer(){const c=new Date(now.getFullYear(),now.getMonth(),1),f=Object.keys(S.logs).sort()[0];let s=f?new Date(+f.slice(0,4),+f.slice(5,7)-1,1):new Date(c.getFullYear(),c.getMonth()-1,1);if(s>c)s=c;const L=[];for(let d=new Date(c);d>=s;d.setMonth(d.getMonth()-1))L.push(new Date(d));
return `<div class="ov" data-a="dr" data-v="0"></div><div class="dr"><h2>Habit Tracker</h2><button class="mi ${tab==='today'?'on':''}" data-a="go" data-t="today">✅ Today</button><button class="mi ${tab==='month'?'on':''}" data-a="mo"><span>📅 Months</span><span>${mo?'▴':'▾'}</span></button>${!mo?'':L.map(d=>`<button class="mi ${tab==='month'&&mon.y===d.getFullYear()&&mon.m===d.getMonth()?'on':''}" data-a="go" data-t="month" data-y="${d.getFullYear()}" data-m="${d.getMonth()}"><span>${d.toLocaleDateString('en-IN',{month:'long',year:'numeric'})}</span><span>${mp(d.getFullYear(),d.getMonth())}%</span></button>`).join('')}<div style="height:8px"></div><button class="mi" data-a="go" data-t="manage">📝 Manage habits</button><button class="mi" data-a="go" data-t="set">⚙️ Settings</button></div>`}
function settings(){const t=S.set.theme;return `${hdr('Settings')}<div class="card"><h2>Appearance</h2><div class="chips">${['auto','light','dark'].map(x=>`<button class="chip ${t===x?'on':''}" data-a="th" data-v="${x}">${{auto:'Auto',light:'☀️ Light',dark:'🌙 Dark'}[x]}</button>`).join('')}</div><div class="sw">${AC.map((a,i)=>`<button class="${S.set.ac===i?'on':''}" style="background:linear-gradient(135deg,${a[1]},${a[3]})" data-a="ac" data-v="${i}" title="${a[0]}"></button>`).join('')}</div></div>
<div class="card"><h2>Backup &amp; restore</h2><p class="mu">Save all your habits and history as a file, and import it later to restore.</p><div class="row" style="flex-wrap:wrap"><button class="btn" data-a="bk">⬇️ Create backup file</button><label class="btn g" style="display:inline-block">⬆️ Restore from file<input type="file" id="rf" accept=".json,application/json" hidden></label></div><div class="mu" id="msg" style="margin-top:8px"></div></div>`}
const pad=n=>String(n).padStart(2,'0'),key=d=>d.getFullYear()+'-'+pad(d.getMonth()+1)+'-'+pad(d.getDate());
const now=new Date();now.setHours(0,0,0,0);
const ago=k=>Math.round((now-new Date(k+'T00:00:00'))/864e5),ed=k=>{const a=ago(k);return a>=0&&a<=2};
const get=(k,id)=>(S.logs[k]||{})[id];
function set(k,id,v){if(!ed(k))return;S.logs[k]=S.logs[k]||{};if(v)S.logs[k][id]=v;else delete S.logs[k][id];save();render()}
const dayK=n=>{const d=new Date(now);d.setDate(d.getDate()-n);return key(d)};
const chk=()=>S.habits.filter(h=>h.t==='c');
let tab='today',sel=0,mon={y:now.getFullYear(),m:now.getMonth()};
function streak(id){const m=dayK(0).slice(0,7);let n=0,a=get(dayK(0),id)?0:1;for(;;a++){const k=dayK(a);if(k.slice(0,7)!==m||!get(k,id))break;n++}return n}
function line(v,l,mx){const W=600,H=200,p=30,n=v.length,x=i=>p+(n<2?W/2-p:i*(W-2*p)/(n-1)),y=a=>H-p-(a/mx)*(H-2*p-10);
let g='',st=Math.ceil(mx/5)||1;for(let t=0;t<=mx;t+=st)g+=`<line x1="${p}" x2="${W-p}" y1="${y(t)}" y2="${y(t)}" stroke="var(--ln)"/><text x="${p-6}" y="${y(t)+4}" text-anchor="end" font-size="11" fill="var(--mu)">${t}</text>`;
const pts=v.map((a,i)=>x(i)+','+y(a)).join(' ');
return `<svg viewBox="0 0 ${W} ${H}">${g}<polyline points="${pts}" fill="none" stroke="var(--ac)" stroke-width="2.5" stroke-linejoin="round"/>`+v.map((a,i)=>`<circle cx="${x(i)}" cy="${y(a)}" r="4" fill="var(--ac)"/><text x="${x(i)}" y="${y(a)-9}" text-anchor="middle" font-size="11" fill="var(--tx)">${a}</text><text x="${x(i)}" y="${H-10}" text-anchor="middle" font-size="${n>14?9:13}" fill="var(--mu)">${l[i]}</text>`).join('')+'</svg>'}
function today(){const k=dayK(sel),c=chk(),dn=c.filter(h=>get(k,h.id)).length,pc=c.length?dn/c.length:0,R=34,C=2*Math.PI*R;
const dt=new Date(k+'T00:00:00').toLocaleDateString('en-IN',{weekday:'long',day:'numeric',month:'long'});
return `${hdr('Habit Tracker')}<div class="chips">${['Today','Yesterday','2 days ago'].map((t,i)=>`<button class="chip ${i===sel?'on':''}" data-a="sel" data-i="${i}">${t}</button>`).join('')}</div>
<div class="card hero row"><svg viewBox="0 0 90 90" style="width:80px"><circle cx="45" cy="45" r="${R}" fill="none" stroke="rgba(255,255,255,.3)" stroke-width="9"/><circle cx="45" cy="45" r="${R}" fill="none" stroke="#fff" stroke-width="9" stroke-linecap="round" stroke-dasharray="${C*pc} ${C}" transform="rotate(-90 45 45)"/><text x="45" y="50" text-anchor="middle" font-size="16" font-weight="700" fill="#fff">${Math.round(pc*100)}%</text></svg>
<div><b>${dt}</b><div class="mu">${dn} of ${c.length} habits done</div><div class="mu">You can edit today and the previous 2 days.</div></div></div>
<div class="card">${S.habits.map(h=>{const s=h.t==='c'?streak(h.id):0;return `<div class="h"><span style="font-size:22px">${esc(h.e)}</span><div class="n">${esc(h.n)}${s>1?`<div class="s">⚡ ${s}-day streak</div>`:''}</div>`+(h.t==='c'?`<button class="ck ${get(k,h.id)?'on':''}" data-a="tg" data-id="${h.id}">${get(k,h.id)?'✓':''}</button>`:`<input type="number" min="0" step="0.25" data-a="hr" data-id="${h.id}" value="${get(k,h.id)||''}" placeholder="hrs">`)+'</div>'}).join('')}</div>`}
function month(){const {y,m}=mon,N=new Date(y,m+1,0).getDate(),c=chk(),days=[...Array(N)].map((_,i)=>key(new Date(y,m,i+1)));
const past=days.filter(k=>ago(k)>=0),tot=S.habits.map(h=>days.reduce((s,k)=>s+(+get(k,h.id)||0),0)),
daily=past.map(k=>c.filter(h=>get(k,h.id)).length),ticks=daily.reduce((a,b)=>a+b,0),pc=past.length&&c.length?Math.round(ticks/(past.length*c.length)*100):0;
const ci=S.habits.map((h,i)=>[h,tot[i]]).filter(a=>a[0].t==='c'),best=ci.sort((a,b)=>b[1]-a[1])[0];
const title=new Date(y,m,1).toLocaleDateString('en-IN',{month:'long',year:'numeric'});
const th=S.habits.map(h=>`<th class="v" title="${esc(h.n)}"><span>${esc(h.e)} ${esc(h.n.length>20?h.n.slice(0,19)+'…':h.n)}</span></th>`).join('');
const rows=days.map((k,i)=>`<tr><td class="d">${i+1}</td>`+S.habits.map(h=>{const v=get(k,h.id),f=ago(k)<0,e=ed(k);return `<td class="${f?'f':''} ${e?'e':''} ${v?'x':''}" ${e?`data-a="cell" data-k="${k}" data-id="${h.id}"`:''}>${h.t==='c'?(v?'✕':''):(v||'')}</td>`}).join('')+'</tr>').join('');
const cl=ci.length?S.habits.filter(h=>h.t==='c'):[],cv=cl.map(h=>tot[S.habits.indexOf(h)]);
return `<div class="nav"><button class="mb" data-a="dr" data-v="1">☰</button><button data-a="mm" data-i="-1">‹</button><h1 style="margin:0;font-size:19px">${title}</h1><button data-a="mm" data-i="1">›</button></div>
<div class="stats"><div><b>${pc}%</b><span class="mu">completion</span></div><div><b>${ticks}</b><span class="mu">ticks</span></div><div><b>${best&&best[1]?best[0].e:'–'}</b><span class="mu">top habit</span></div></div>
<button class="btn" style="width:100%;margin-bottom:12px" data-a="rp" data-v="1">⬇️ Download report</button><div class="card"><h2>Monthly report</h2><div class="tbl"><table><tr><th>Day</th>${th}</tr>${rows}<tr class="t"><td class="d">Σ</td>${tot.map(t=>`<td>${t||0}</td>`).join('')}</tr></table></div><div class="mu" style="margin-top:6px">Shaded rows are editable (today + 2 days back). Tap to mark.</div></div>
<div class="card"><h2>Days each habit was done</h2>${cl.length?line(cv,cl.map(h=>h.e),Math.max(5,...cv)):'<div class="mu">No habits yet.</div>'}</div>
<div class="card"><h2>Habits completed per day</h2>${past.length?line(daily,past.map(k=>+k.slice(8)),Math.max(5,c.length)):'<div class="mu">No days yet this month.</div>'}</div>`}
let del=null,dr=false,mo=false,rp=false;
function manage(){return `${hdr('Manage habits')}<div class="card"><h2>Add a habit</h2><div class="row" style="flex-wrap:wrap"><input id="ne" value="⭐" style="width:56px;text-align:center"><input id="nn" placeholder="Habit name" style="flex:1;min-width:130px"><select id="nt"><option value="c">Yes / No</option><option value="h">Hours</option></select><button class="btn" data-a="add">Add</button></div></div>
<div class="card">${S.habits.map(h=>`<div class="h"><span style="font-size:22px">${esc(h.e)}</span><div class="n">${esc(h.n)}<div class="mu">${h.t==='c'?'Yes / No':'Hours'}</div></div><button class="btn ${del===h.id?'r':'g'}" data-a="del" data-id="${h.id}">${del===h.id?'Sure?':'Remove'}</button></div>`).join('')||'<div class="mu">No habits.</div>'}</div>
<button class="btn g" data-a="reset">Restore default habits</button><p class="mu">Removing a habit hides it from the tracker. Restoring defaults keeps your logged days.</p>`}
function render(){const a=document.getElementById('app');a.innerHTML=(tab==='today'?today():tab==='month'?month():tab==='set'?settings():manage())+(dr?drawer():'')+(rp?rpt():'');
document.getElementById('bar').innerHTML=[['today','✅','Today'],['month','📊','Month'],['set','⚙️','Settings']].map(t=>`<button class="${tab===t[0]?'on':''}" data-a="tab" data-t="${t[0]}"><span>${t[1]}</span>${t[2]}</button>`).join('')}
function act(el){const a=el.dataset.a,d=el.dataset;
if(a==='tab'){tab=d.t;render()}else if(a==='sel'){sel=+d.i;render()}
else if(a==='tg'){const k=dayK(sel);set(k,d.id,!get(k,d.id))}
else if(a==='cell'){const h=S.habits.find(x=>x.id===d.id);if(h.t==='c')set(d.k,d.id,!get(d.k,d.id));else{const v=prompt('Hours for '+h.n,get(d.k,d.id)||'');if(v!==null)set(d.k,d.id,Math.max(0,Math.min(24,parseFloat(v)||0)))}}
else if(a==='mm'){const t=new Date(mon.y,mon.m+ +d.i,1);mon={y:t.getFullYear(),m:t.getMonth()};render()}
else if(a==='add'){const n=document.getElementById('nn').value.trim();if(!n)return;S.habits.push({id:'u'+Date.now(),n,e:document.getElementById('ne').value||'⭐',t:document.getElementById('nt').value});save();render()}
else if(a==='del'){if(del===d.id){S.habits=S.habits.filter(h=>h.id!==d.id);del=null;save()}else del=d.id;render()}
else if(a==='dr'){dr=d.v==='1';render()}else if(a==='go'){tab=d.t;if(d.y)mon={y:+d.y,m:+d.m};dr=false;render();scrollTo(0,0)}else if(a==='th'){S.set.theme=d.v;save();theme();render()}else if(a==='ac'){S.set.ac=+d.v;save();theme();render()}
else if(a==='rp'){rp=d.v==='1';render()}else if(a==='pdf'){exportPdf(mon.y,mon.m)}else if(a==='xls'){exportXlsx(mon.y,mon.m)}else if(a==='bk'){bkSave();return}else if(a==='mo'){mo=!mo;render()}else if(a==='bk0'){const j=JSON.stringify(S),t=document.getElementById('bk');t.value=j;t.select();try{navigator.clipboard.writeText(j).then(()=>msg('Backup copied ✓'),()=>msg('Select the text above and copy it'))}catch(e){msg('Select the text above and copy it')}}
else if(a==='rs'){try{const o=JSON.parse(document.getElementById('bk').value);if(!Array.isArray(o.habits)||typeof o.logs!=='object')throw 0;S=o;S.set=S.set||{theme:'auto',ac:0};save();theme();render();msg('Restored ✓')}catch(e){msg('That does not look like a valid backup')}}

else if(a==='reset'){const ids=S.habits.map(h=>h.id);D.forEach((x,i)=>{if(!ids.includes('h'+i))S.habits.push({id:'h'+i,n:x[0],e:x[1],t:x[2]||'c'})});save();render()}}
document.addEventListener('click',e=>{const el=e.target.closest('[data-a]');if(el&&el.tagName!=='INPUT')act(el)});
document.addEventListener('change',e=>{if(e.target.id==='rf')restoreFile(e.target.files[0]);if(e.target.dataset.a==='hr')set(dayK(sel),e.target.dataset.id,Math.max(0,Math.min(24,parseFloat(e.target.value)||0)))});
document.addEventListener('visibilitychange',()=>{const t=new Date();t.setHours(0,0,0,0);if(t.getTime()!==now.getTime()){now.setTime(t.getTime());sel=0;render()}});
theme();matchMedia('(prefers-color-scheme:dark)').addEventListener('change',theme);try{navigator.storage&&navigator.storage.persist&&navigator.storage.persist()}catch(e){}
render();
if('serviceWorker' in navigator&&location.protocol.indexOf('http')===0)addEventListener('load',()=>navigator.serviceWorker.register('sw.js').catch(()=>{}));
</script></body></html>

````

# Appendix B - sw.js
````js
// Offline cache. Bump VERSION on every release so phones pick up the new files.
const VERSION = 'ht-v2';
const CORE = ['./', 'index.html', 'manifest.webmanifest', 'icons/icon-192.png', 'icons/icon-512.png', 'icons/apple-touch-icon.png', 'icons/favicon-32.png'];
self.addEventListener('install', e => e.waitUntil(caches.open(VERSION).then(c => c.addAll(CORE)).then(() => self.skipWaiting())));
self.addEventListener('activate', e => e.waitUntil(caches.keys().then(k => Promise.all(k.filter(x => x !== VERSION).map(x => caches.delete(x)))).then(() => self.clients.claim())));
self.addEventListener('fetch', e => {
  const r = e.request, u = new URL(r.url);
  if (r.method !== 'GET' || (u.origin !== location.origin && u.hostname !== 'cdnjs.cloudflare.com')) return;
  e.respondWith(caches.match(r).then(hit => {
    const net = fetch(r).then(res => { if (res && (res.ok || res.type === 'opaque')) { const cp = res.clone(); caches.open(VERSION).then(c => c.put(r, cp)); } return res; })
      .catch(() => hit || (r.mode === 'navigate' ? caches.match('index.html') : undefined));
    return hit || net;
  }));
});
````

# Appendix C - manifest.webmanifest
````json
{
  "name": "Habit Tracker",
  "short_name": "Habits",
  "description": "Daily habit tracker with monthly reports, streaks and graphs.",
  "start_url": "./",
  "scope": "./",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#eef1f8",
  "theme_color": "#0b6fd6",
  "icons": [
    {"src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png", "purpose": "any"},
    {"src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any"},
    {"src": "icons/maskable-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable"}
  ]
}
````
