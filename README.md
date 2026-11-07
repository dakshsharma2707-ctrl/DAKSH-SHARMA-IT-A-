<img width="959" height="599" alt="Screenshot 2026-11-07 170606" src="https://github.com/user-attachments/assets/6814064d-87b7-4dc1-a77b-2f8362ba5691" />
<img width="959" height="599" alt="Screenshot 2026-11-07 170541" src="https://github.com/user-attachments/assets/120f5f46-2166-4a2d-841a-6adbdfa728b5" />
<img width="959" height="599" alt="Screenshot 2026-11-07 170508" src="https://github.com/user-attachments/assets/840cb66a-b028-48b9-bd94-ce832a539af7" />
<img width="959" height="592" alt="Screenshot 2026-11-07 170457" src="https://github.com/user-attachments/assets/17f4e0ec-3105-4da6-ad02-31d209706953" />
<img width="959" height="599" alt="Screenshot 2026-11-07 170447" src="https://github.com/user-attachments/assets/5350acf0-dab3-4698-a031-84f7a2ee3c40" />
<img width="959" height="599" alt="Screenshot 2026-11-07 170435" src="https://github.com/user-attachments/assets/227b772a-bff2-4e54-a935-145751fc4aa8" />
# DAKSH-SHARMA-IT-A-
'use strict';
const $=s=>document.querySelector(s);
const load=(k,d)=>{try{return JSON.parse(localStorage.getItem(k))||d}catch(e){return d}};
const save=(k,v)=>localStorage.setItem(k,JSON.stringify(v));
let tx=load('et_tx',[]);
let cfg=Object.assign({cur:'₹',budget:0,warn:80,dark:false},load('et_cfg',{}));
const saveTx=()=>save('et_tx',tx),saveCfg=()=>save('et_cfg',cfg);
const CATS={expense:['Food','Transport','Shopping','Bills','Health','Entertainment','Education','Other'],income:['Salary','Freelance','Gift','Other']};
const ALL=[...new Set([...CATS.expense,...CATS.income])];
const esc=s=>String(s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const fmt=n=>(n<0?'-':'')+cfg.cur+Math.abs(n).toLocaleString('en-IN',{minimumFractionDigits:2,maximumFractionDigits:2});
const iso=d=>new Date(d-d.getTimezoneOffset()*6e4).toISOString().slice(0,10);
const today=()=>iso(new Date());
const curMonth=()=>today().slice(0,7);
const sum=(l,t)=>l.filter(x=>x.type===t).reduce((s,x)=>s+x.amount,0);
const inMonth=m=>tx.filter(x=>x.date.slice(0,7)===m);
const sorted=l=>[...l].sort((a,z)=>z.date.localeCompare(a.date)||z.id-a.id);

/* layout: nav + footer */
const PAGES=[['index','Dashboard'],['add','Add'],['history','History'],['budget','Budget'],['reports','Reports'],['tips','Saving Tips'],['settings','Settings']];
const page=document.body.dataset.page;
document.documentElement.dataset.theme=cfg.dark?'dark':'light';
$('#nav').innerHTML=`<a class="logo" href="index.html">Expense Tracker</a><button id="menu" aria-label="Toggle menu">☰</button><nav id="links">${PAGES.map(([p,t])=>`<a href="${p}.html"${p===page?' class="on"':''}>${t}</a>`).join('')}</nav>`;
$('#menu').onclick=()=>$('#links').classList.toggle('open');
document.body.insertAdjacentHTML('beforeend','<footer>Your data is stored only in this browser (localStorage).</footer>');

/* helpers */
function alertHTML(){const b=+cfg.budget;if(!b)return '';
 const s=sum(inMonth(curMonth()),'expense'),p=s/b*100;
 if(p>=100)return `<div class="alert bad">Over budget: you have spent ${fmt(s)} of ${fmt(b)} this month.</div>`;
 if(p>=cfg.warn)return `<div class="alert warn">Warning: you have used ${p.toFixed(0)}% of this month's budget.</div>`;
 return ''}
function withBal(){let b=0;const m={};[...tx].sort((a,z)=>a.date.localeCompare(z.date)||a.id-z.id).forEach(x=>{b+=x.type==='income'?x.amount:-x.amount;m[x.id]=b});return m}
function table(list,acts){if(!list.length)return '<p class="muted">No transactions found. Add one to get started.</p>';const bal=withBal();
 return `<div class="scroll"><table><thead><tr><th>Date</th><th>Category</th><th>Note</th><th>Amount</th><th>Balance</th>${acts?'<th>Actions</th>':''}</tr></thead><tbody>${list.map(x=>`<tr><td>${x.date}</td><td>${esc(x.category)}</td><td>${esc(x.note||'-')}</td><td class="${x.type}">${x.type==='income'?'+':'-'}${fmt(x.amount)}</td><td>${fmt(bal[x.id])}</td>${acts?`<td class="act"><a href="add.html?id=${x.id}">Edit</a><button data-del="${x.id}">Delete</button></td>`:''}</tr>`).join('')}</tbody></table></div>`}
function exportCSV(list){const q=v=>'"'+String(v).replace(/"/g,'""')+'"';
 const rows=[['Date','Type','Category','Amount','Note'],...list.map(x=>[x.date,x.type,x.category,x.amount,x.note||''])];
 const a=document.createElement('a');a.href=URL.createObjectURL(new Blob([rows.map(r=>r.map(q).join(',')).join('\n')],{type:'text/csv'}));a.download='transactions.csv';a.click()}
function months(n){const d=new Date(),o=[];for(let i=n-1;i>=0;i--){const t=new Date(d.getFullYear(),d.getMonth()-i,1);o.push([t.getFullYear()+'-'+String(t.getMonth()+1).padStart(2,'0'),t.toLocaleString('en',{month:'short'})])}return o}
function drawChart(id){const c=$(id),ms=months(6),w=c.width=c.parentNode.clientWidth,h=c.height=240,x=c.getContext('2d');
 const dk=cfg.dark,sets=[['income',dk?'#4ade80':'#15803d'],['expense',dk?'#fb923c':'#c2410c']].map(([t,col])=>({col,d:ms.map(m=>sum(inMonth(m[0]),t))}));
 const mx=Math.max(1,...sets.flatMap(s=>s.d)),bw=w/ms.length,gw=bw*.3;
 x.clearRect(0,0,w,h);x.font='12px system-ui,sans-serif';x.textAlign='center';
 const tc=getComputedStyle(document.body).color;
 ms.forEach((m,i)=>{sets.forEach((s,j)=>{const bh=(h-45)*s.d[i]/mx;x.fillStyle=s.col;x.fillRect(i*bw+bw*.2+j*gw,h-25-bh,gw-2,bh)});x.fillStyle=tc;x.fillText(m[1],i*bw+bw/2,h-8)})}
addEventListener('resize',()=>$('#chart')&&drawChart('#chart'));
function catBars(list){const t={};list.filter(x=>x.type==='expense').forEach(x=>t[x.category]=(t[x.category]||0)+x.amount);
 const e=Object.entries(t).sort((a,b)=>b[1]-a[1]);if(!e.length)return '<p class="muted">No expenses recorded for this month.</p>';
 return e.map(([k,v])=>`<div class="cb"><span>${esc(k)}</span><div class="bar"><i style="width:${v/e[0][1]*100}%"></i></div><b>${fmt(v)}</b></div>`).join('')}
const stat=([a,v,c])=>`<div class="card stat"><small>${a}</small><strong class="${c}">${v}</strong></div>`;

/* page logic */
const R={
index(){const m=inMonth(curMonth()),bal=sum(tx,'income')-sum(tx,'expense'),sp=sum(m,'expense'),b=+cfg.budget;
 $('#alert').innerHTML=alertHTML();
 $('#cards').innerHTML=[['Balance',fmt(bal),bal<0?'expense':''],['Income this month',fmt(sum(m,'income')),'income'],['Expenses this month',fmt(sp),'expense'],['Budget left',b?fmt(b-sp):'Not set',b&&b-sp<0?'expense':'']].map(stat).join('');
 $('#recent').innerHTML=table(sorted(tx).slice(0,5),false);drawChart('#chart')},
add(){const id=+new URLSearchParams(location.search).get('id'),ed=tx.find(x=>x.id===id),f=$('#f'),E=f.elements;
 const fill=()=>{E.category.innerHTML=CATS[E.type.value].map(c=>`<option>${c}</option>`).join('')};
 E.type.onchange=fill;fill();E.date.value=today();
 if(ed){$('h1').textContent='Edit transaction';E.type.value=ed.type;fill();E.category.value=ed.category;E.amount.value=ed.amount;E.date.value=ed.date;E.note.value=ed.note||''}
 f.onsubmit=e=>{e.preventDefault();const a=parseFloat(E.amount.value);
  if(!(a>0)){$('#msg').innerHTML='<div class="alert bad">Enter an amount greater than 0.</div>';return}
  const o={type:E.type.value,category:E.category.value,amount:a,date:E.date.value,note:E.note.value.trim()};
  if(ed){Object.assign(ed,o);saveTx();location.href='history.html';return}
  tx.push(Object.assign({id:Date.now()},o));saveTx();f.reset();fill();E.date.value=today();
  $('#msg').innerHTML='<div class="alert ok">Transaction added.</div>'+alertHTML()}},
history(){const f=$('#flt'),E=f.elements;let cur=[];
 E.cat.innerHTML='<option value="">All categories</option>'+ALL.map(c=>`<option>${c}</option>`).join('');
 const draw=()=>{cur=sorted(tx.filter(x=>(!E.cat.value||x.category===E.cat.value)&&(!E.type.value||x.type===E.type.value)&&(!E.from.value||x.date>=E.from.value)&&(!E.to.value||x.date<=E.to.value)));
  $('#list').innerHTML=table(cur,true);$('#count').textContent=cur.length+' transaction(s). Net: '+fmt(sum(cur,'income')-sum(cur,'expense'))};
 f.oninput=draw;f.onreset=()=>setTimeout(draw);draw();
 $('#list').onclick=e=>{const id=+e.target.dataset.del;if(id&&confirm('Delete this transaction?')){tx=tx.filter(x=>x.id!==id);saveTx();draw()}};
 $('#csv').onclick=()=>exportCSV(cur)},
budget(){const f=$('#bf');f.elements.amt.value=cfg.budget||'';
 const draw=()=>{const s=sum(inMonth(curMonth()),'expense'),b=+cfg.budget,p=b?s/b*100:0;
  $('#alert').innerHTML=alertHTML();
  $('#prog').innerHTML=b?`<div class="bar big"><i class="${p>=100?'bad':p>=cfg.warn?'warn':''}" style="width:${Math.min(100,p)}%"></i></div><p>${fmt(s)} spent of ${fmt(b)} (${p.toFixed(0)}%). ${s>b?'Over by '+fmt(s-b):fmt(b-s)+' left'}.</p>`:'<p class="muted">Set a monthly budget above to track your spending.</p>';
  $('#cats').innerHTML=catBars(inMonth(curMonth()))};
 f.onsubmit=e=>{e.preventDefault();cfg.budget=Math.max(0,+f.elements.amt.value)||0;saveCfg();draw()};draw()},
reports(){const mi=$('#month');mi.value=curMonth();
 const draw=()=>{const l=inMonth(mi.value),i=sum(l,'income'),e=sum(l,'expense');
  $('#sum').innerHTML=[['Income',fmt(i),'income'],['Expenses',fmt(e),'expense'],['Saved',fmt(i-e),i-e<0?'expense':'income']].map(stat).join('');
  $('#cats').innerHTML=catBars(l);drawChart('#chart')};
 mi.onchange=draw;draw();$('#csv').onclick=()=>exportCSV(inMonth(mi.value))},
tips(){const m=inMonth(curMonth()),i=sum(m,'income'),e=sum(m,'expense'),t=[],c={};
 m.filter(x=>x.type==='expense').forEach(x=>c[x.category]=(c[x.category]||0)+x.amount);
 const k=Object.entries(c).sort((a,b)=>b[1]-a[1])[0];
 if(k)t.push(`Your biggest expense this month is <b>${esc(k[0])}</b> (${fmt(k[1])}). Try cutting it by 10%.`);
 if(i>0)t.push(`Your savings rate this month is <b>${((i-e)/i*100).toFixed(0)}%</b>. Aim for 20% or more.`);
 $('#smart').innerHTML=t.length?t.map(x=>`<div class="card">${x}</div>`).join(''):'<p class="muted">Add income and expenses to see tips based on your own spending.</p>'},
settings(){const f=$('#sf'),E=f.elements,msg=h=>$('#msg').innerHTML=h;
 E.cur.value=cfg.cur;E.warn.value=cfg.warn;E.dark.checked=cfg.dark;
 f.onsubmit=e=>{e.preventDefault();cfg.cur=E.cur.value;cfg.warn=Math.min(100,Math.max(1,+E.warn.value||80));cfg.dark=E.dark.checked;saveCfg();document.documentElement.dataset.theme=cfg.dark?'dark':'light';msg('<div class="alert ok">Settings saved.</div>')};
 $('#csv').onclick=()=>exportCSV(tx);
 $('#demo').onclick=()=>{const s=[['income','Salary',30000,0,'Monthly salary'],['expense','Food',3500,1,'Groceries'],['expense','Bills',2200,2,'Electricity and internet'],['expense','Transport',1200,3,'Bus pass'],['expense','Shopping',2800,4,'Clothes'],['expense','Entertainment',900,4,'Movies']];let id=Date.now();
  for(let m=0;m<3;m++)s.forEach(([t,c,a,d,n])=>{const dt=new Date();dt.setMonth(dt.getMonth()-m,d+1);tx.push({id:id++,type:t,category:c,amount:a+m*150*(t==='expense'),date:iso(dt),note:n})});
  saveTx();msg('<div class="alert ok">Sample data added.</div>')};
 $('#clear').onclick=()=>{if(confirm('Delete ALL transactions? This cannot be undone.')){tx=[];saveTx();msg('<div class="alert ok">All transactions deleted.</div>')}}}
};
R[page]&&R[page]();
