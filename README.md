<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1.0"/>
<title>Kanban</title>
<script src="https://cdn.jsdelivr.net/npm/monday-sdk-js/dist/main.js"></script>
<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

:root {
  --bg: #0f0f11;
  --surface: #1a1a1f;
  --surface2: #212127;
  --surface3: #26262d;
  --border: #2e2e38;
  --border2: #38383f;
  --text: #e4e4ea;
  --text-dim: #9494a2;
  --text-muted: #5c5c6b;
  --accent: #6c5fd5;
  --st-pauta-bg: rgba(0,133,255,.1);
  --st-pauta-border: rgba(0,133,255,.3);
  --st-pauta-text: #3d9bff;
  --st-pauta-dot: #0085ff;
  --st-andamento-bg: rgba(255,185,0,.1);
  --st-andamento-border: rgba(255,185,0,.3);
  --st-andamento-text: #ffb900;
  --st-andamento-dot: #ffb900;
  --st-pausado-bg: rgba(92,92,107,.1);
  --st-pausado-border: rgba(92,92,107,.3);
  --st-pausado-text: #9494a2;
  --st-pausado-dot: #5c5c6b;
  --st-concluido-bg: rgba(0,202,114,.1);
  --st-concluido-border: rgba(0,202,114,.3);
  --st-concluido-text: #00ca72;
  --st-concluido-dot: #00ca72;
  --radius: 9px;
  --radius-sm: 5px;
}

*{box-sizing:border-box;margin:0;padding:0}

body {
  font-family: 'Inter', -apple-system, sans-serif;
  background: var(--bg);
  color: var(--text);
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  font-size: 13px;
}

/* ── HEADER ── */
#hdr {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 16px;
  height: 48px;
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}

#hdr-title {
  font-size: 13px;
  font-weight: 600;
  color: var(--text);
}

#hdr-sep { width: 1px; height: 16px; background: var(--border2); }
#hdr-board { font-size: 11px; color: var(--text-muted); max-width: 200px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
#hdr-right { margin-left: auto; display: flex; align-items: center; gap: 8px; }

/* tabs */
#person-tabs {
  display: flex;
  gap: 3px;
  background: var(--surface2);
  padding: 3px;
  border-radius: 7px;
  border: 1px solid var(--border);
}

.ptab {
  padding: 3px 10px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 500;
  color: var(--text-muted);
  cursor: pointer;
  border: none;
  background: none;
  font-family: inherit;
  transition: all .15s;
  white-space: nowrap;
}
.ptab:hover { color: var(--text); background: var(--border); }
.ptab.active { background: var(--accent); color: #fff; }
.ptab.all-tab.active { background: var(--border2); color: var(--text); }

#sort-select {
  background: var(--surface2);
  border: 1px solid var(--border);
  color: var(--text-dim);
  padding: 4px 8px;
  border-radius: var(--radius-sm);
  font-size: 11px;
  font-family: inherit;
  cursor: pointer;
  outline: none;
}

#btn-refresh {
  width: 28px; height: 28px;
  background: var(--surface2);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  color: var(--text-dim);
  cursor: pointer;
  font-size: 15px;
  display: flex; align-items: center; justify-content: center;
  transition: all .15s;
}
#btn-refresh:hover { border-color: var(--accent); color: var(--accent); }
#btn-refresh.spinning { animation: spin .6s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }

/* ── BOARD ── */
#board-wrap {
  flex: 1;
  display: flex;
  overflow-x: auto;
  overflow-y: hidden;
  padding: 12px 16px;
  gap: 12px;
  align-items: flex-start;
}
#board-wrap::-webkit-scrollbar { height: 4px; }
#board-wrap::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 2px; }

/* ── COLUNA ── */
.col {
  width: 460px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 100%;
}

.col-header {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 0 2px 2px;
  flex-shrink: 0;
}

.col-dot { width: 7px; height: 7px; border-radius: 50%; flex-shrink: 0; }
.col-title { font-size: 11px; font-weight: 700; letter-spacing: .4px; text-transform: uppercase; flex: 1; }
.col-count {
  font-size: 10px; color: var(--text-muted);
  background: var(--surface2); border: 1px solid var(--border);
  border-radius: 10px; padding: 1px 6px; font-weight: 600;
}

.col[data-status="Em Pauta"] .col-dot { background: var(--st-pauta-dot); }
.col[data-status="Em Pauta"] .col-title { color: var(--st-pauta-text); }
.col[data-status="Em Andamento"] .col-dot { background: var(--st-andamento-dot); }
.col[data-status="Em Andamento"] .col-title { color: var(--st-andamento-text); }
.col[data-status="Concluído"] .col-dot { background: var(--st-concluido-dot); }
.col[data-status="Concluído"] .col-title { color: var(--st-concluido-text); }

.col-body {
  overflow-y: auto;
  padding-bottom: 8px;
  flex: 1;
  min-height: 60px;
}
.col-body::-webkit-scrollbar { width: 3px; }
.col-body::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

/* seção por pessoa */
.person-section { margin-bottom: 12px; }

.person-label {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 7px;
  padding: 0 2px;
}

.person-av {
  width: 20px; height: 20px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 8px; font-weight: 700; color: #fff;
  flex-shrink: 0; overflow: hidden;
}
.person-av img { width: 100%; height: 100%; object-fit: cover; border-radius: 50%; }
.person-name-lbl { font-size: 11px; font-weight: 600; color: var(--text-dim); }
.person-count-lbl { font-size: 10px; color: var(--text-muted); }

/* grid 2 colunas */
.cards-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 7px;
}

/* ── CARD ── */
.card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 10px 11px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  cursor: grab;
  transition: border-color .15s, box-shadow .15s, opacity .15s;
  min-width: 0;
}
.card:hover { border-color: var(--border2); box-shadow: 0 3px 12px rgba(0,0,0,.3); }
.card.dragging { opacity: .35; cursor: grabbing; }

.card-top {
  display: flex;
  align-items: flex-start;
  gap: 5px;
  min-width: 0;
}

.card-title {
  font-size: 11px;
  font-weight: 600;
  color: var(--text);
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
  line-height: 1.4;
}

.card-actions { display: flex; gap: 3px; flex-shrink: 0; }

.card-btn {
  width: 20px; height: 20px;
  background: none;
  border: 1px solid var(--border);
  border-radius: 4px;
  color: var(--text-muted);
  cursor: pointer;
  font-size: 10px;
  display: flex; align-items: center; justify-content: center;
  transition: all .15s;
}
.card-btn:hover { border-color: var(--accent); color: var(--accent); }

/* briefing */
.card-briefing {
  font-size: 10px;
  color: var(--text-dim);
  padding: 5px 8px;
  background: var(--surface2);
  border-radius: var(--radius-sm);
  border-left: 2px solid var(--border2);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}
.card-briefing.empty { color: var(--text-muted); font-style: italic; border-left-color: var(--border); }

/* subitem row */
.sub-row {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 6px;
  background: var(--surface2);
  border-radius: var(--radius-sm);
  border: 1px solid var(--border);
  min-width: 0;
}

.sub-av {
  width: 16px; height: 16px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 7px; font-weight: 700; color: #fff;
  flex-shrink: 0; overflow: hidden;
}
.sub-av img { width: 100%; height: 100%; object-fit: cover; border-radius: 50%; }

.sub-name {
  font-size: 10px;
  color: var(--text-dim);
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}

/* prazo */
.sub-prazo {
  position: relative;
  display: flex; align-items: center; gap: 2px;
  background: var(--surface3);
  border: 1px solid var(--border2);
  border-radius: 4px;
  padding: 2px 5px;
  font-size: 10px; font-weight: 500;
  color: var(--text-dim);
  cursor: pointer;
  white-space: nowrap;
  transition: all .15s;
  flex-shrink: 0;
}
.sub-prazo:hover { border-color: var(--accent); color: var(--accent); }
.sub-prazo.overdue { border-color: rgba(239,83,80,.5); color: #ef5350; background: rgba(239,83,80,.08); }
.sub-prazo.today   { border-color: rgba(255,185,0,.5); color: #ffb900; background: rgba(255,185,0,.08); }
.sub-prazo.soon    { border-color: rgba(0,133,255,.5); color: #3d9bff; background: rgba(0,133,255,.08); }
.sub-prazo input[type=date] {
  position: absolute; inset: 0; opacity: 0; cursor: pointer; width: 100%; height: 100%;
}

/* status */
.sub-status { position: relative; flex-shrink: 0; }

.st-badge {
  display: flex; align-items: center; gap: 3px;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 10px; font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
  border: 1px solid transparent;
  transition: filter .15s;
}
.st-badge:hover { filter: brightness(1.15); }
.st-badge.em-pauta     { background: var(--st-pauta-bg);     border-color: var(--st-pauta-border);     color: var(--st-pauta-text); }
.st-badge.em-andamento { background: var(--st-andamento-bg); border-color: var(--st-andamento-border); color: var(--st-andamento-text); }
.st-badge.pausado      { background: var(--st-pausado-bg);   border-color: var(--st-pausado-border);   color: var(--st-pausado-text); }
.st-badge.concluido    { background: var(--st-concluido-bg); border-color: var(--st-concluido-border); color: var(--st-concluido-text); }

.st-dd {
  position: absolute; top: calc(100% + 3px); right: 0;
  background: var(--surface2); border: 1px solid var(--border2);
  border-radius: var(--radius-sm); z-index: 300;
  box-shadow: 0 8px 24px rgba(0,0,0,.55);
  display: none; min-width: 140px; overflow: hidden;
}
.st-dd.open { display: block; }
.st-dd-item {
  padding: 7px 11px; font-size: 11px; color: var(--text-dim);
  cursor: pointer; display: flex; align-items: center; gap: 7px; transition: background .1s;
}
.st-dd-item:hover { background: var(--border); color: var(--text); }
.st-dd-dot { width: 6px; height: 6px; border-radius: 50%; flex-shrink: 0; }

/* drag ghost */
#drag-ghost {
  position: fixed; pointer-events: none; z-index: 9999;
  background: var(--surface2); border: 1px solid var(--accent);
  border-radius: var(--radius); padding: 7px 11px;
  font-size: 11px; font-weight: 600; color: var(--text);
  box-shadow: 0 8px 24px rgba(0,0,0,.6);
  display: none; max-width: 200px;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}

/* drop zone */
.col-drop-zone { min-height: 60px; border-radius: var(--radius); transition: background .15s; }
.col-drop-zone.drag-active { background: rgba(108,95,213,.07); border: 1px dashed rgba(108,95,213,.4); }

/* loading */
#loading {
  flex: 1; display: none; align-items: center; justify-content: center;
  flex-direction: column; gap: 12px; color: var(--text-muted);
}
.spinner {
  width: 22px; height: 22px;
  border: 2px solid var(--border2); border-top-color: var(--accent);
  border-radius: 50%; animation: spin .7s linear infinite;
}

.col-empty {
  grid-column: 1/-1; padding: 18px; text-align: center;
  font-size: 11px; color: var(--text-muted);
  border: 1px dashed var(--border); border-radius: var(--radius);
}

/* toast */
#toast {
  position: fixed; bottom: 16px; left: 50%; transform: translateX(-50%);
  background: var(--surface2); border: 1px solid var(--border2);
  color: var(--text); padding: 7px 14px; border-radius: 8px;
  font-size: 11px; z-index: 999; display: none;
  box-shadow: 0 4px 16px rgba(0,0,0,.4); white-space: nowrap;
}
#toast.show { display: block; }
#toast.error { border-color: #ef5350; color: #ef5350; }
</style>
</head>
<body>

<div id="hdr">
  <div id="hdr-title">Kanban</div>
  <div id="hdr-sep"></div>
  <div id="hdr-board">—</div>
  <div id="hdr-right">
    <div id="person-tabs">
      <button class="ptab all-tab active" data-person="all">Todos</button>
      <button class="ptab" data-person="Maiko da Silva">Maiko</button>
      <button class="ptab" data-person="Isabele Coelho">Isabele</button>
      <button class="ptab" data-person="Gabriela Paganini">Gabi</button>
      <button class="ptab" data-person="Letícia Santos">Letícia</button>
    </div>
    <select id="sort-select">
      <option value="asc">Prazo ↑</option>
      <option value="desc">Prazo ↓</option>
    </select>
    <button id="btn-refresh">↻</button>
  </div>
</div>

<div id="loading"><div class="spinner"></div><span>Carregando...</span></div>
<div id="board-wrap" style="display:none"></div>
<div id="toast"></div>
<div id="drag-ghost"></div>

<script>
(function(){
  if(typeof mondaySdk==='function') initApp(mondaySdk());
  else document.getElementById('loading').innerHTML='<span style="color:#ef5350">Erro: SDK não encontrado.</span>';
})();

function initApp(monday) {

  /* ══ CONFIG ══ */
  const PEOPLE = ['Maiko da Silva','Isabele Coelho','Gabriela Paganini','Letícia Santos'];
  const STATUS_COLS = ['Em Pauta','Em Andamento','Concluído'];
  const STATUS_META = {
    'Em Pauta':     { cls:'em-pauta',     dot:'#0085ff' },
    'Em Andamento': { cls:'em-andamento', dot:'#ffb900' },
    'Pausado':      { cls:'pausado',      dot:'#5c5c6b' },
    'Concluído':    { cls:'concluido',    dot:'#00ca72' },
  };

  /* ══ ESTADO ══ */
  let allItems      = [];
  let userPhotoById = {};
  let selectedPerson= 'all';
  let sortOrder     = 'asc';
  let colSchema     = null;
  let statusColId   = null;
  let currentBoardId= null;
  let dragItemData  = null;

  /* ══ API ══ */
  async function api(q) {
    let res = await monday.api(q);
    if(res?.errors) {
      const ce = res.errors.find(e=>e.extensions?.code==='COMPLEXITY_BUDGET_EXHAUSTED');
      if(ce){ await sleep((ce.extensions?.retry_in_seconds||30)*1000); return api(q); }
      throw new Error(res.errors[0]?.message||'API error');
    }
    return res;
  }
  const sleep = ms => new Promise(r=>setTimeout(r,ms));

  /* ══ FETCH ══ */
  async function fetchSchema(bId) {
    const r1 = await api(`query{boards(ids:[${bId}]){items_page(limit:1){items{subitems{board{columns{id title type}}}}}}}`);
    const items = r1.data.boards[0].items_page.items;
    const schema = (items.length && items[0].subitems.length) ? items[0].subitems[0].board.columns : [];

    const r2 = await api(`query{boards(ids:[${bId}]){columns{id title type}}}`);
    const pCols = r2.data.boards[0].columns || [];
    const stCol = pCols.find(c => c.title === 'Status');
    if(stCol) statusColId = stCol.id;

    return schema;
  }

  async function fetchUsers() {
    const r = await api('query{users{id name photo_thumb}}');
    (r.data.users||[]).forEach(u => {
      if(u.name && u.photo_thumb) userPhotoById[u.name] = u.photo_thumb;
    });
  }

  async function fetchItems(bId) {
    const gRes = await api(`query{boards(ids:[${bId}]){groups{id title}}}`);
    const groups = gRes.data.boards[0].groups.filter(g => {
      const t = g.title.trim();
      return t==='Em Pauta' || t==='Em Andamento' || t.includes('HOJE') || t.includes('Hoje');
    });
    if(!groups.length) return [];

    let fetched = [];
    for(const g of groups) {
      let cursor = null;
      do {
        const cp = cursor ? `,cursor:"${cursor}"` : '';
        const q = `query{boards(ids:[${bId}]){items_page(limit:50${cp},query_params:{rules:[{column_id:"group",compare_value:["${g.id}"]}]}){cursor items{id name group{id title} column_values{id text} subitems{id name board{id} column_values{id text}}}}}}`;
        let res; try{ res = await api(q); }catch(e){ break; }
        const page = res.data.boards[0].items_page;
        fetched = fetched.concat(page.items.map(it=>({...it, _groupTitle:g.title.trim()})));
        cursor = page.cursor||null;
        if(cursor) await sleep(250);
      } while(cursor);
    }
    return fetched;
  }

  /* ══ PROCESSAR ══ */
  function processItems(raw, schema) {
    const ctm = {};
    schema.forEach(c => ctm[c.id] = c.title);

    return raw.map(item => {
      const gt = item._groupTitle;
      let kanbanStatus = 'Em Pauta';
      if(gt==='Em Andamento') kanbanStatus = 'Em Andamento';
      else if(gt.includes('HOJE')||gt.includes('Hoje')) kanbanStatus = 'Concluído';

      const subitems = (item.subitems||[]).map(sub => {
        const cols={}, colIds={};
        sub.column_values.forEach(c => {
          const title = ctm[c.id]||c.id;
          cols[title]=c.text; colIds[title]=c.id;
        });
        const resp     = cols['Resp.']||cols['Responsavel']||cols['Resp']||'';
        const cron     = cols['Cronograma']||'';
        const status   = cols['Status']||'';
        const briefing = cols['Briefing']||cols['briefing']||'';
        return {
          subId:sub.id, boardId:sub.board?.id, name:sub.name,
          resp, cron, status, briefing,
          cronColId:colIds['Cronograma']||null,
          statusColId:colIds['Status']||null,
          _range:parseRange(cron),
        };
      }).filter(s => PEOPLE.includes(s.resp));

      if(!subitems.length) return null;

      const briefing = subitems.find(s=>s.briefing)?.briefing || '';
      return { id:item.id, name:item.name, kanbanStatus, briefing, subitems };
    }).filter(Boolean);
  }

  /* ══ UTILS DATA ══ */
  function parseRange(str) {
    if(!str) return null;
    const m = str.match(/(\d{4}-\d{2}-\d{2})/);
    return m ? new Date(m[1]+'T12:00:00') : null;
  }
  function fmtDate(d) { return d ? d.toISOString().split('T')[0] : ''; }
  function prazoLabel(ds) {
    if(!ds) return '—';
    const d=new Date(ds+'T12:00:00'), today=new Date(); today.setHours(12,0,0,0);
    const diff=Math.round((d-today)/86400000);
    const day=d.getDate().toString().padStart(2,'0');
    const mon=['jan','fev','mar','abr','mai','jun','jul','ago','set','out','nov','dez'][d.getMonth()];
    if(diff<0) return `${day} ${mon} ⚠`;
    if(diff===0) return 'Hoje';
    if(diff===1) return 'Amanhã';
    return `${day} ${mon}`;
  }
  function prazoClass(ds) {
    if(!ds) return '';
    const d=new Date(ds+'T12:00:00'), today=new Date(); today.setHours(12,0,0,0);
    const diff=Math.round((d-today)/86400000);
    if(diff<0) return 'overdue'; if(diff===0) return 'today'; if(diff<=2) return 'soon'; return '';
  }
  function statusCls(st) {
    if(!st) return 'em-pauta';
    const s=st.toLowerCase();
    if(s.includes('andamento')) return 'em-andamento';
    if(s.includes('pausado')||s.includes('pausada')) return 'pausado';
    if(s.includes('conclu')) return 'concluido';
    return 'em-pauta';
  }

  /* ══ AVATAR ══ */
  const avColors=['#5c7cfa','#f06595','#ffa94d','#51cf66','#cc5de8','#22b8cf','#ff6b6b'];
  const avMap={}; let avI=0;
  function avColor(n){if(!avMap[n])avMap[n]=avColors[avI++%avColors.length];return avMap[n];}
  function initials(n){return(n||'?').split(' ').map(p=>p[0]).join('').slice(0,2).toUpperCase();}
  function makeAv(name,size,cls){
    const av=document.createElement('div');
    av.className=cls; av.style.width=size+'px'; av.style.height=size+'px';
    const ph=userPhotoById[name];
    if(ph){av.innerHTML=`<img src="${ph}" alt="${initials(name)}"/>`;}
    else{av.style.background=avColor(name);av.textContent=initials(name);}
    return av;
  }

  /* ══ MUTATIONS ══ */
  async function mutPrazo(sub, newDate) {
    if(!sub.boardId||!sub.cronColId){toast('Cronograma não encontrado',true);return;}
    try {
      const val=JSON.stringify({from:newDate,to:newDate});
      await monday.api(`mutation{change_column_value(board_id:${sub.boardId},item_id:${sub.subId},column_id:"${sub.cronColId}",value:${JSON.stringify(val)}){id}}`);
      sub.cron=newDate; sub._range=parseRange(newDate);
      toast('Prazo atualizado!'); render();
    } catch(e){toast('Erro ao atualizar prazo',true);}
  }

  async function mutSubStatus(sub, newStatus) {
    const colId=sub.statusColId;
    if(!sub.boardId||!colId){toast('Coluna Status não encontrada',true);return;}
    try {
      const val=JSON.stringify({label:newStatus});
      await monday.api(`mutation{change_column_value(board_id:${sub.boardId},item_id:${sub.subId},column_id:"${colId}",value:${JSON.stringify(val)}){id}}`);
      sub.status=newStatus; toast('Status atualizado!'); render();
    } catch(e){toast('Erro ao atualizar status',true);}
  }

  async function mutItemStatus(item, newStatus) {
    if(!statusColId){toast('Coluna Status não encontrada',true);return;}
    try {
      const val=JSON.stringify({label:newStatus});
      await monday.api(`mutation{change_column_value(board_id:${currentBoardId},item_id:${item.id},column_id:"${statusColId}",value:${JSON.stringify(val)}){id}}`);
      item.kanbanStatus=newStatus; toast('Movido para '+newStatus+'!');
    } catch(e){toast('Erro ao mover card',true);console.error(e);}
  }

  /* ══ TOAST ══ */
  let _tt;
  function toast(msg,err){
    const t=document.getElementById('toast');
    t.textContent=msg; t.className=err?'show error':'show';
    clearTimeout(_tt); _tt=setTimeout(()=>t.className='',2500);
  }

  /* ══ DRAG & DROP ══ */
  const ghost=document.getElementById('drag-ghost');

  function setupDrag(cardEl,item){
    cardEl.addEventListener('dragstart',e=>{
      dragItemData=item; cardEl.classList.add('dragging');
      ghost.textContent=item.name; ghost.style.display='block';
      e.dataTransfer.effectAllowed='move';
      e.dataTransfer.setDragImage(new Image(),0,0);
    });
    cardEl.addEventListener('dragend',()=>{
      cardEl.classList.remove('dragging');
      ghost.style.display='none'; dragItemData=null;
      document.querySelectorAll('.col-drop-zone').forEach(z=>z.classList.remove('drag-active'));
    });
  }

  document.addEventListener('dragover',e=>{
    if(ghost.style.display==='block'){
      ghost.style.left=(e.clientX+12)+'px'; ghost.style.top=(e.clientY+12)+'px';
    }
  });

  function setupDrop(zone, targetStatus){
    zone.addEventListener('dragover',e=>{
      if(!dragItemData)return; e.preventDefault();
      document.querySelectorAll('.col-drop-zone').forEach(z=>z.classList.remove('drag-active'));
      zone.classList.add('drag-active');
    });
    zone.addEventListener('dragleave',()=>zone.classList.remove('drag-active'));
    zone.addEventListener('drop',async e=>{
      e.preventDefault(); zone.classList.remove('drag-active');
      if(!dragItemData||dragItemData.kanbanStatus===targetStatus)return;
      await mutItemStatus(dragItemData,targetStatus);
      render();
    });
  }

  /* ══ RENDER ══ */
  function render(){
    const wrap=document.getElementById('board-wrap');
    wrap.innerHTML='';

    STATUS_COLS.forEach(status=>{
      let items=allItems.filter(it=>it.kanbanStatus===status);
      if(selectedPerson!=='all')
        items=items.filter(it=>it.subitems.some(s=>s.resp===selectedPerson));

      const col=document.createElement('div');
      col.className='col'; col.dataset.status=status;

      const hdr=document.createElement('div');
      hdr.className='col-header';
      hdr.innerHTML=`<div class="col-dot"></div><div class="col-title">${status}</div><div class="col-count">${items.length}</div>`;
      col.appendChild(hdr);

      const drop=document.createElement('div');
      drop.className='col-drop-zone col-body';
      setupDrop(drop,status);

      if(selectedPerson==='all'){
        PEOPLE.forEach(person=>{
          const pItems=sortItems(items.filter(it=>it.subitems.some(s=>s.resp===person)));
          if(!pItems.length)return;
          const sec=document.createElement('div'); sec.className='person-section';
          const lbl=document.createElement('div'); lbl.className='person-label';
          lbl.appendChild(makeAv(person,20,'person-av'));
          const nm=document.createElement('span'); nm.className='person-name-lbl'; nm.textContent=person.split(' ')[0];
          const ct=document.createElement('span'); ct.className='person-count-lbl'; ct.textContent=` ${pItems.length} job${pItems.length>1?'s':''}`;
          lbl.appendChild(nm); lbl.appendChild(ct);
          sec.appendChild(lbl);
          const grid=document.createElement('div'); grid.className='cards-grid';
          pItems.forEach(it=>grid.appendChild(buildCard(it,person)));
          sec.appendChild(grid); drop.appendChild(sec);
        });
      } else {
        const pItems=sortItems(items);
        const grid=document.createElement('div'); grid.className='cards-grid';
        if(!pItems.length) grid.innerHTML='<div class="col-empty">Nenhum job aqui</div>';
        else pItems.forEach(it=>grid.appendChild(buildCard(it,selectedPerson)));
        drop.appendChild(grid);
      }

      col.appendChild(drop); wrap.appendChild(col);
    });
  }

  function sortItems(items){
    return [...items].sort((a,b)=>{
      const sa=selectedPerson==='all'?a.subitems:a.subitems.filter(s=>s.resp===selectedPerson);
      const sb=selectedPerson==='all'?b.subitems:b.subitems.filter(s=>s.resp===selectedPerson);
      const da=sa[0]?._range?.getTime()??(sortOrder==='asc'?Infinity:-Infinity);
      const db=sb[0]?._range?.getTime()??(sortOrder==='asc'?Infinity:-Infinity);
      return sortOrder==='asc'?da-db:db-da;
    });
  }

  function buildCard(item, personFilter){
    const subs=personFilter==='all'?item.subitems:item.subitems.filter(s=>s.resp===personFilter);
    const card=document.createElement('div');
    card.className='card'; card.draggable=true;
    setupDrag(card,item);

    // top
    const top=document.createElement('div'); top.className='card-top';
    const title=document.createElement('div'); title.className='card-title'; title.textContent=item.name; title.title=item.name;
    const actions=document.createElement('div'); actions.className='card-actions';

    const btnU=document.createElement('button'); btnU.className='card-btn'; btnU.title='Atualizações'; btnU.textContent='💬';
    btnU.addEventListener('click',e=>{e.stopPropagation();monday.execute('openItemCard',{itemId:parseInt(item.id),kind:'updates'});});

    const btnO=document.createElement('button'); btnO.className='card-btn'; btnO.title='Abrir'; btnO.textContent='↗';
    btnO.addEventListener('click',e=>{e.stopPropagation();monday.execute('openItemCard',{itemId:parseInt(item.id)});});

    actions.appendChild(btnU); actions.appendChild(btnO);
    top.appendChild(title); top.appendChild(actions);
    card.appendChild(top);

    // briefing
    const brief=document.createElement('div');
    brief.className='card-briefing'+(item.briefing?'':' empty');
    brief.textContent=item.briefing||'Sem briefing';
    brief.title=item.briefing||'';
    card.appendChild(brief);

    // subitems
    subs.forEach(sub=>{
      const row=document.createElement('div'); row.className='sub-row';
      row.appendChild(makeAv(sub.resp,16,'sub-av'));

      const nm=document.createElement('div'); nm.className='sub-name'; nm.textContent=sub.name||sub.resp.split(' ')[0]; nm.title=sub.name||'';
      row.appendChild(nm);

      // prazo
      const dateVal=sub._range?fmtDate(sub._range):(sub.cron?.match(/\d{4}-\d{2}-\d{2}/)?.[0]||'');
      const prazo=document.createElement('div');
      prazo.className='sub-prazo '+prazoClass(dateVal);
      prazo.title='Editar prazo';
      prazo.textContent='📅 '+prazoLabel(dateVal);
      const di=document.createElement('input'); di.type='date'; di.value=dateVal;
      di.addEventListener('change',async e=>{e.stopPropagation();if(e.target.value)await mutPrazo(sub,e.target.value);});
      prazo.appendChild(di); row.appendChild(prazo);

      // status
      const stWrap=document.createElement('div'); stWrap.className='sub-status';
      const badge=document.createElement('div');
      badge.className=`st-badge ${statusCls(sub.status)}`;
      badge.textContent=sub.status||'Em Pauta';
      const dd=document.createElement('div'); dd.className='st-dd';
      Object.entries(STATUS_META).forEach(([label,meta])=>{
        const di2=document.createElement('div'); di2.className='st-dd-item';
        di2.innerHTML=`<span class="st-dd-dot" style="background:${meta.dot}"></span>${label}`;
        di2.addEventListener('click',async e=>{e.stopPropagation();dd.classList.remove('open');await mutSubStatus(sub,label);});
        dd.appendChild(di2);
      });
      badge.addEventListener('click',e=>{
        e.stopPropagation();
        document.querySelectorAll('.st-dd.open').forEach(d=>{if(d!==dd)d.classList.remove('open');});
        dd.classList.toggle('open');
      });
      stWrap.appendChild(badge); stWrap.appendChild(dd);
      row.appendChild(stWrap);
      card.appendChild(row);
    });

    return card;
  }

  /* ══ LOAD ══ */
  async function load(bId){
    document.getElementById('loading').style.display='flex';
    document.getElementById('board-wrap').style.display='none';
    try {
      [colSchema]=await Promise.all([fetchSchema(bId),fetchUsers()]);
      const raw=await fetchItems(bId);
      allItems=processItems(raw,colSchema);
      document.getElementById('loading').style.display='none';
      document.getElementById('board-wrap').style.display='flex';
      render();
    } catch(e){
      document.getElementById('loading').innerHTML=`<span style="color:#ef5350">Erro: ${e.message}</span>`;
      console.error(e);
    }
  }

  /* ══ EVENTOS ══ */
  document.querySelectorAll('.ptab').forEach(tab=>{
    tab.addEventListener('click',()=>{
      document.querySelectorAll('.ptab').forEach(t=>t.classList.remove('active'));
      tab.classList.add('active');
      selectedPerson=tab.dataset.person;
      render();
    });
  });

  document.getElementById('sort-select').addEventListener('change',e=>{sortOrder=e.target.value;render();});
  document.getElementById('btn-refresh').addEventListener('click',()=>{if(currentBoardId)load(currentBoardId);});
  document.addEventListener('click',()=>document.querySelectorAll('.st-dd.open').forEach(d=>d.classList.remove('open')));

  setInterval(()=>{if(currentBoardId)load(currentBoardId);},5*60*1000);

  /* ══ CONTEXTO MONDAY ══ */
  monday.listen('context',async res=>{
    const bId=res.data.boardId||(res.data.boardIds&&res.data.boardIds[0]);
    currentBoardId=bId;
    const boardName=res.data.boardName||'Kanban';
    document.getElementById('hdr-board').textContent=boardName;
    await load(bId);
  });

} // fim initApp
</script>
</body>
</html>
