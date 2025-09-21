<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Miniflix — Netflix-style UI (Demo)</title>
<meta name="description" content="A static Netflix-style clone UI demo with carousels, modal player, search, and watchlist." />
<style>
  /* -------------------------
     DESIGN TOKENS & LAYOUT
     ------------------------- */
  :root{
    --bg:#050608;
    --card:#0b0d10;
    --muted:rgba(255,255,255,0.65);
    --glass: rgba(255,255,255,0.03);
    --accent:#e50914;
    --radius:12px;
    --gap:18px;
    --max-w:1200px;
    --ease:cubic-bezier(.17,.95,.3,1);
    color-scheme: dark;
    font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0; background:linear-gradient(180deg,#031018 0%, #050608 100%); color:#fff;
    -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale;
    line-height:1.35;
  }
  .app{
    max-width:var(--max-w); margin:28px auto; padding:20px; min-height:100vh;
  }

  /* NAV */
  header.nav{display:flex; align-items:center; gap:18px; justify-content:space-between; margin-bottom:22px}
  .logo{display:flex; gap:12px; align-items:center}
  .logo .mark{width:44px;height:28px;border-radius:4px;background:linear-gradient(90deg,var(--accent),#ff6b6b);display:grid;place-items:center;font-weight:800;color:#fff}
  .logo b{font-weight:800; letter-spacing: -0.02em; font-size:18px}
  .search{flex:1; max-width:560px; display:flex; align-items:center; gap:8px}
  .search input{flex:1;padding:10px 12px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
  .controls{display:flex; gap:8px; align-items:center}
  .btn{background:transparent;color:var(--muted);padding:8px 12px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);cursor:pointer}
  .btn.primary{background:var(--accent); color:white; border:0; box-shadow:0 8px 30px rgba(233,9,20,0.12)}

  /* HERO */
  .hero{position:relative;margin-bottom:28px;border-radius:14px;overflow:hidden;background:linear-gradient(90deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); padding:28px; display:flex; gap:18px; align-items:center}
  .hero .left{flex:1}
  .hero h1{margin:0;font-size:28px; font-weight:800}
  .hero p{margin:8px 0 0;color:var(--muted)}
  .hero .cta{margin-top:14px;display:flex;gap:8px}
  .hero .poster{width:360px; min-width:220px; height:160px; border-radius:10px; overflow:hidden; background:linear-gradient(180deg,#111 0%,#070809 100%); display:grid;place-items:center}

  /* ROWS / CAROUSELS */
  .row{margin-bottom:22px}
  .row h3{margin:6px 6px 10px 6px;font-size:16px}
  .strip{display:flex; gap:12px; overflow:auto; padding:8px 6px 14px; scroll-behavior:smooth}
  .card{
    width:160px; min-width:160px; height:240px; border-radius:10px; background:var(--card);
    display:flex; flex-direction:column; overflow:hidden; cursor:pointer; transition:transform 260ms var(--ease);
    border:1px solid rgba(255,255,255,0.03);
  }
  .card:focus{outline:3px solid rgba(112,76,255,0.16)}
  .card:hover{transform:translateY(-6px)}
  .thumb{flex:1;background:linear-gradient(180deg,#222,#111); display:grid;place-items:center}
  .thumb img{width:100%; height:100%; object-fit:cover; display:block}
  .meta{padding:8px 10px}
  .meta .title{font-weight:700;font-size:14px}
  .meta .meta-muted{font-size:12px;color:var(--muted); margin-top:6px}

  /* modal player */
  .modal{position:fixed; inset:0; display:none; align-items:center; justify-content:center; z-index:999; background:linear-gradient(180deg, rgba(0,0,0,0.6), rgba(0,0,0,0.85)); padding:24px}
  .modal .paper{width:min(1100px,96vw); background:#06080a; border-radius:12px; padding:18px; border:1px solid rgba(255,255,255,0.03)}
  .modal .close{float:right; background:transparent;border:0;color:var(--muted); font-weight:700; cursor:pointer}
  .player-wrap{position:relative; padding-top:56.25%; background:#000; border-radius:8px; overflow:hidden}
  .player-wrap iframe, .player-wrap video{position:absolute; inset:0; width:100%; height:100%; border:0}

  /* watchlist badge */
  .watch-badge{position:fixed; right:22px; bottom:22px; background:linear-gradient(90deg,var(--accent),#ff6b6b); padding:10px 14px; border-radius:999px; box-shadow:0 12px 36px rgba(233,9,20,0.14); font-weight:700; cursor:pointer; z-index:1200}

  /* responsive */
  @media (max-width:900px){
    .hero{flex-direction:column; align-items:flex-start}
    .hero .poster{width:100%; height:180px}
    .card{width:140px; min-width:140px; height:210px}
  }

  /* tiny scrollbar style */
  .strip::-webkit-scrollbar{height:10px}
  .strip::-webkit-scrollbar-thumb{background:rgba(255,255,255,0.06);border-radius:10px}
</style>
</head>
<body>
  <div class="app" id="app" aria-live="polite">
    <header class="nav" role="banner">
      <div class="logo" aria-hidden="true">
        <div class="mark">M</div>
        <b>Miniflix</b>
      </div>

      <div class="search" role="search">
        <input id="search" placeholder="Search movies, series, genres..." aria-label="Search movies" />
      </div>

      <div class="controls" role="navigation">
        <button class="btn" id="homeBtn">Home</button>
        <button class="btn" id="randBtn" title="Surprise me">🎲</button>
        <button class="btn primary" id="signinBtn">Sign In</button>
      </div>
    </header>

    <!-- HERO: rotating featured -->
    <section class="hero" aria-label="Featured">
      <div class="left">
        <h1 id="featuredTitle">Featured — The Dev Dojo</h1>
        <p id="featuredOverview">High-stakes refactor drama. Live coding duels. A story about discipline and deploys.</p>
        <div class="cta">
          <button class="btn primary" id="playFeatured">Play</button>
          <button class="btn" id="moreInfoFeatured">More Info</button>
        </div>
      </div>
      <div class="poster" id="featuredPoster" aria-hidden="true">
        <!-- poster placeholder is injected by JS -->
      </div>
    </section>

    <!-- rows -->
    <main>
      <!-- dynamic rows will be inserted here -->
      <div id="rows"></div>
    </main>

    <button class="watch-badge" id="watchlistBtn" aria-haspopup="dialog">Watchlist (0)</button>
  </div>

  <!-- Modal player -->
  <div class="modal" id="modal" role="dialog" aria-modal="true" aria-hidden="true">
    <div class="paper" role="document">
      <button class="close" id="closeModal" aria-label="Close player">Close ✕</button>
      <div id="modalTitle" style="font-weight:800;margin-bottom:8px"></div>
      <div class="player-wrap" id="playerWrap" tabindex="-1">
        <!-- player iframe/video goes here -->
      </div>
      <div style="margin-top:12px; color:var(--muted)" id="modalOverview"></div>
    </div>
  </div>

<script>
/* ============================
   Demo Data — replace with real API
   ============================ */
const DB = {
  featured: { id: 'f1', title: 'The Dev Dojo', overview: 'A cinematic tale of refactors, deploys and discipline.', posterColor:'#704CFF', trailer: 'https://www.youtube.com/embed/ScMzIvxBSi4' },
  rows: [
    { id:'r1', title:'Trending Now', items: [
      {id:'m1', title:'Refactor: Act I', year:2024, length: '1h 34m', color:'#6b6', desc:'A small team faces a giant legacy codebase.', trailer:'https://www.youtube.com/embed/ScMzIvxBSi4'},
      {id:'m2', title:'Silent Debugging', year:2023, length:'0h 45m', color:'#f66', desc:'Debugging in absolute silence — intense.'},
      {id:'m3', title:'Merge Ritual', year:2022, length:'1h 12m', color:'#59f', desc:'When two branches become one.'},
      {id:'m4', title:'One-line TDD', year:2025, length:'0h 30m', color:'#f9a', desc:'Speed and precision.'},
      {id:'m5', title:'Pair Kata', year:2024, length:'0h 50m', color:'#4cf', desc:'Head-to-head code duels.'}
    ]},
    { id:'r2', title:'Editor Essentials', items: [
      {id:'m6', title:'Keys of Power', year:2021, length:'1h 40m', color:'#d48', desc:'Mastering your editor.'},
      {id:'m7', title:'Terminal Zen', year:2020, length:'0h 37m', color:'#a6f', desc:'The discipline of CLI.'},
      {id:'m8', title:'Latency Whisperer', year:2022, length:'1h 05m', color:'#2a9', desc:'Performance tuning.'}
    ]},
    { id:'r3', title:'Documentaries', items: [
      {id:'m9', title:'Architecture of Calm', year:2019, length:'1h 55m', color:'#ffc', desc:'System architecture as meditation.'},
      {id:'m10', title:'Design Systems', year:2021, length:'0h 48m', color:'#66c', desc:'Scale and beauty.'}
    ]}
  ]
};

/* ============================
   State & Utilities
   ============================ */
const state = {
  watchlist: new Set(JSON.parse(localStorage.getItem('miniflix_watch')||'[]')),
};

function saveWatch(){
  localStorage.setItem('miniflix_watch', JSON.stringify(Array.from(state.watchlist)));
  updateWatchBadge();
}

function updateWatchBadge(){
  const el = document.getElementById('watchlistBtn');
  el.textContent = `Watchlist (${state.watchlist.size})`;
}

function el(tag,props={},children=[]){
  const node = document.createElement(tag);
  Object.entries(props || {}).forEach(([k,v])=>{
    if(k==='class') node.className=v;
    else if(k.startsWith('data-')) node.setAttribute(k,v);
    else if(k==='html') node.innerHTML=v;
    else node[k]=v;
  });
  (Array.isArray(children)?children:[children]).forEach(c=>{ if(c==null) return; node.append(typeof c === 'string'? document.createTextNode(c) : c); });
  return node;
}

/* ============================
   Rendering
   ============================ */
function renderHero(){
  const f = DB.featured;
  document.getElementById('featuredTitle').textContent = f.title;
  document.getElementById('featuredOverview').textContent = f.overview;
  const poster = document.getElementById('featuredPoster');
  poster.innerHTML = '';
  const svg = placeholderPosterSVG(f.title, f.posterColor, 360,160);
  poster.appendChild(svg);
  // play button
  document.getElementById('playFeatured').onclick = ()=>openModal(f);
  document.getElementById('moreInfoFeatured').onclick = ()=>openModal(f);
}

function renderRows(filterText=''){
  const rowsEl = document.getElementById('rows');
  rowsEl.innerHTML = '';
  DB.rows.forEach(row=>{
    const rDiv = el('section',{class:'row'});
    rDiv.appendChild(el('h3',{}, row.title));
    const strip = el('div',{class:'strip', tabindex:0, 'aria-label': row.title});
    // accessible left/right keyboard nav
    strip.addEventListener('keydown', (e)=>{
      if(e.key === 'ArrowRight') { strip.scrollBy({left:220, behavior:'smooth'}); e.preventDefault(); }
      if(e.key === 'ArrowLeft') { strip.scrollBy({left:-220, behavior:'smooth'}); e.preventDefault(); }
    });
    // cards
    row.items.forEach(item=>{
      if(filterText && !item.title.toLowerCase().includes(filterText.toLowerCase())) return;
      const card = el('button',{class:'card', onclick:()=>openModal(item), tabindex:0, 'aria-label': item.title});
      const thumb = el('div',{class:'thumb'});
      // lazy poster (svg placeholder)
      thumb.appendChild(placeholderPosterSVG(item.title, item.color || '#333', 160, 210));
      const meta = el('div',{class:'meta'});
      meta.appendChild(el('div',{class:'title'}, item.title));
      meta.appendChild(el('div',{class:'meta-muted'}, `${item.year || ''} • ${item.length || ''}`));
      card.appendChild(thumb);
      card.appendChild(meta);
      // context menu: add to watchlist
      const wl = el('button',{class:'btn', onclick:(e)=>{ e.stopPropagation(); toggleWatch(item); }}, state.watchlist.has(item.id)?'Remove':'Watch');
      wl.style.position='absolute'; wl.style.right='8px'; wl.style.top='8px'; wl.style.fontSize='12px';
      card.style.position='relative';
      card.appendChild(wl);
      strip.appendChild(card);
    });
    rDiv.appendChild(strip);
    rowsEl.appendChild(rDiv);
  });
}

/* ============================
   Placeholders (SVG posters)
   ============================ */
function placeholderPosterSVG(title, color='#444', w=160, h=210){
  const ns = 'http://www.w3.org/2000/svg';
  const svg = document.createElementNS(ns,'svg');
  svg.setAttribute('width', w);
  svg.setAttribute('height', h);
  svg.setAttribute('viewBox', `0 0 ${w} ${h}`);
  svg.setAttribute('aria-hidden','true');
  const rect = document.createElementNS(ns,'rect');
  rect.setAttribute('x',0); rect.setAttribute('y',0); rect.setAttribute('width',w); rect.setAttribute('height',h);
  rect.setAttribute('rx',8);
  rect.setAttribute('fill', color);
  rect.setAttribute('opacity',0.9);
  svg.appendChild(rect);
  const g = document.createElementNS(ns,'g');
  const text = document.createElementNS(ns,'text');
  text.setAttribute('x',w/2); text.setAttribute('y',h/2);
  text.setAttribute('fill','#fff'); text.setAttribute('font-size',14); text.setAttribute('font-weight',700);
  text.setAttribute('text-anchor','middle'); text.setAttribute('dominant-baseline','middle');
  text.textContent = title.length>18? title.slice(0,16)+'…' : title;
  g.appendChild(text);
  svg.appendChild(g);
  return svg;
}

/* ============================
   Modal / Player
   ============================ */
function openModal(item){
  const modal = document.getElementById('modal');
  const title = document.getElementById('modalTitle');
  const overview = document.getElementById('modalOverview');
  const playerWrap = document.getElementById('playerWrap');
  title.textContent = item.title || item.name || '';
  overview.textContent = item.desc || DB.featured.overview || '';
  playerWrap.innerHTML = '';
  // prefer trailer (youtube embed URL), otherwise placeholder
  if(item.trailer){
    const iframe = document.createElement('iframe');
    iframe.setAttribute('src', item.trailer + '?autoplay=1&rel=0');
    iframe.setAttribute('allow','autoplay; encrypted-media');
    iframe.setAttribute('title', item.title + ' trailer');
    playerWrap.appendChild(iframe);
  } else {
    // fallback: simple video black box
    const p = document.createElement('div');
    p.style.background='linear-gradient(180deg,#000,#111)'; p.style.width='100%'; p.style.height='100%';
    p.style.display='grid'; p.style.placeItems='center'; p.textContent='No Trailer — Play Sample';
    playerWrap.appendChild(p);
  }
  modal.style.display='flex';
  modal.setAttribute('aria-hidden','false');
  // trap focus simply
  document.getElementById('closeModal').focus();
}

function closeModal(){
  const modal = document.getElementById('modal');
  modal.style.display='none';
  modal.setAttribute('aria-hidden','true');
  document.getElementById('app').focus();
}

document.getElementById('closeModal').addEventListener('click', closeModal);
document.getElementById('modal').addEventListener('click', (e)=>{ if(e.target === e.currentTarget) closeModal(); });
window.addEventListener('keydown', (e)=>{ if(e.key==='Escape'){ closeModal(); }});

/* ============================
   Watchlist
   ============================ */
function toggleWatch(item){
  if(state.watchlist.has(item.id)){
    state.watchlist.delete(item.id);
  } else state.watchlist.add(item.id);
  saveWatch();
  // re-render rows to update buttons (cheap approach for demo)
  renderRows(document.getElementById('search').value);
}
document.getElementById('watchlistBtn').addEventListener('click', ()=>{
  const ids = Array.from(state.watchlist);
  if(!ids.length){ alert('Watchlist is empty'); return; }
  const titles = ids.map(id=>{
    for(const r of DB.rows) for(const it of r.items) if(it.id===id) return `${it.title} (${it.year||''})`;
    return id;
  });
  alert('Your Watchlist:\n\n' + titles.join('\n'));
});

/* ============================
   Search, random, and init
   ============================ */
document.getElementById('search').addEventListener('input', (e)=>{
  renderRows(e.target.value);
});
document.getElementById('randBtn').addEventListener('click', ()=>{
  const all = DB.rows.flatMap(r=>r.items);
  const pick = all[Math.floor(Math.random()*all.length)];
  openModal(pick);
});
document.getElementById('homeBtn').addEventListener('click', ()=> window.scrollTo({top:0, behavior:'smooth'}));
document.getElementById('signinBtn').addEventListener('click', ()=> alert('Demo — sign-in stub'));

function init(){
  // restore watch
  const stored = JSON.parse(localStorage.getItem('miniflix_watch')||'[]');
  stored.forEach(id=>state.watchlist.add(id));
  updateWatchBadge();
  renderHero();
  renderRows();
}
init();

/* ============================
   Keyboard navigation (demo)
   - Left/Right to move focused strip children
   - Enter opens focused card
   ============================ */
document.addEventListener('keydown', (e)=>{
  // if focus is inside a strip, move it
  const active = document.activeElement;
  if(active && active.classList && active.classList.contains('strip')){
    if(e.key === 'ArrowRight'){ active.querySelector('.card:focusable, .card')?.scrollIntoView({behavior:'smooth', inline:'center'}); active.scrollBy({left:220, behavior:'smooth'}); e.preventDefault(); }
    if(e.key === 'ArrowLeft'){ active.scrollBy({left:-220, behavior:'smooth'}); e.preventDefault(); }
  }
  if(e.key === 'k'){ // quick open watchlist with K
    document.getElementById('watchlistBtn').click();
  }
});

/* tiny polyfill for query of first focusable card (not necessary) */
(function addFocusableHelper(){
  if(!Element.prototype.matches) return;
  Element.prototype.querySelectorFocusable = function(selector){
    const items = this.querySelectorAll(selector);
    return items[0];
  };
})();
</script>
</body>
</html>
<footer>
  <div style="display:flex; gap:10px; justify-content:center; align-items:center; flex-wrap:wrap">
    <div style="color:rgba(255,255,255,0.06)">&copy; Code × Ninja</div>
    <div style="color:rgba(255,255,255,0.06)">—</div>
    <div style="color:rgba(255,255,255,0.06)">Designed for precision</div>
  </div>
  <div style="margin-top:8px; font-size:13px; color:rgba(255,255,255,0.3); text-align:center;">
    Rudra Pratap Singh • Reg No: RA2511003011539 • SRM Institute of Science and Technology
  </div>
</footer>
