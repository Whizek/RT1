# RT1


<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Organizador de Cores - Estoque de Linhas de Crochê</title>
  <style>
    :root{--bg:#f6f7fb;--card:#ffffff;--muted:#6b7280;--accent:#0f172a;--danger:#ef4444;--ok:#10b981}
    *{box-sizing:border-box;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{margin:0;background:var(--bg);color:var(--accent);padding:20px}
    .container{max-width:1100px;margin:0 auto}
    header{display:flex;gap:16px;align-items:center;margin-bottom:18px}
    header h1{font-size:20px;margin:0}
    .grid{display:grid;grid-template-columns:360px 1fr;gap:18px}
    .card{background:var(--card);border-radius:12px;padding:14px;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
    label{display:block;margin:8px 0 6px;font-size:13px;color:var(--muted)}
    input[type=text], input[type=number]{width:100%;padding:8px;border-radius:8px;border:1px solid #e6e9ef}
    .row{display:flex;gap:8px;align-items:center}
    button{background:#111827;color:white;border:0;padding:8px 10px;border-radius:8px;cursor:pointer}
    button.ghost{background:transparent;color:var(--accent);border:1px solid #e6e9ef}
    .controls{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
    .list-controls{display:flex;gap:8px;align-items:center}
    .search{flex:1}
    .table{margin-top:12px}
    .item{display:grid;grid-template-columns:56px 1fr 120px 115px;gap:10px;align-items:center;padding:10px;border-radius:8px;border:1px solid #f1f3f5;margin-bottom:8px}
    .swatch{width:48px;height:48px;border-radius:6px;border:1px solid rgba(0,0,0,0.06)}
    .name{font-weight:600}
    .muted{color:var(--muted);font-size:13px}
    .qty{font-weight:700}
    .low{color:var(--danger)}
    .small{font-size:12px;padding:6px 8px;border-radius:8px}
    .actions{display:flex;gap:6px}
    .summary{display:flex;gap:12px;align-items:center}
    .import-export{display:flex;gap:6px;align-items:center}
    footer{margin-top:18px;color:var(--muted);font-size:13px}
    @media (max-width:880px){.grid{grid-template-columns:1fr}.table .item{grid-template-columns:56px 1fr auto}
      .item .actions{grid-column:3}
    }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>Organizador de Cores — Estoque de Linhas de Crochê</h1>
      <div style="margin-left:auto" class="summary">
        <div class="muted">Itens: <span id="totalCount">0</span></div>
        <div class="muted">Abaixo do mínimo: <span id="lowCount">0</span></div>
      </div>
    </header>

    <div class="grid">
      <div class="card">
        <h3>Adicionar / Editar cor</h3>
        <form id="form">
          <label>Nome da cor</label>
          <input type="text" id="name" placeholder="ex: Vermelho vivo" required>

          <label>Hex ou cor (opcional)</label>
          <input type="text" id="hex" placeholder="#e11d48 ou nome da cor">

          <label>Quantidade (unidades)</label>
          <input type="number" id="qty" min="0" value="0">

          <label>Quantidade mínima (alerta)</label>
          <input type="number" id="min" min="0" value="2">

          <div style="display:flex;gap:8px;margin-top:12px">
            <button type="submit">Salvar</button>
            <button type="button" class="ghost" id="clearBtn">Limpar</button>
            <button type="button" class="ghost" id="fillDemo">Preencher demo</button>
          </div>
        </form>

        <hr style="margin:12px 0;border:none;border-top:1px solid #f1f3f5">
        <div class="muted">Importar / Exportar</div>
        <div class="import-export" style="margin-top:8px">
          <button id="exportJSON">Exportar JSON</button>
          <button id="exportCSV" class="ghost">Exportar CSV</button>
          <input type="file" id="importFile" accept="application/json,text/csv" style="display:none">
          <button id="importBtn" class="ghost">Importar</button>
        </div>

        <div style="margin-top:12px" class="muted">Dica: os dados ficam salvos no navegador (localStorage)</div>
      </div>

      <div class="card">
        <div style="display:flex;align-items:center;gap:10px">
          <div style="flex:1">
            <div class="list-controls">
              <input id="search" class="search" type="text" placeholder="Pesquisar por nome ou hex...">
              <select id="filter">
                <option value="all">Todos</option>
                <option value="low">Abaixo do mínimo</option>
                <option value="zero">Zerados</option>
              </select>
              <select id="sort">
                <option value="name">Ordenar: Nome</option>
                <option value="qty_desc">Quantidade (maior)</option>
                <option value="qty_asc">Quantidade (menor)</option>
              </select>
            </div>
          </div>
          <div class="controls">
            <button id="bulkZero" class="ghost">Zerar selecionados</button>
            <button id="clearAll" class="ghost">Limpar tudo</button>
          </div>
        </div>

        <div class="table" id="list"></div>
      </div>
    </div>

    <footer>Desenvolvido para organizar rapidamente quais cores estão em falta no estoque — ajuste os mínimos conforme sua rotina - Rainha Tecidos.</footer>
  </div>

  <script>
    // -- Modelo dos itens
    // { id, name, hex, qty, min }
    const STORAGE_KEY = 'croche_colors_v1'

    let items = []
    let editingId = null

    // DOM
    const form = document.getElementById('form')
    const nameInput = document.getElementById('name')
    const hexInput = document.getElementById('hex')
    const qtyInput = document.getElementById('qty')
    const minInput = document.getElementById('min')
    const listEl = document.getElementById('list')
    const searchEl = document.getElementById('search')
    const filterEl = document.getElementById('filter')
    const sortEl = document.getElementById('sort')
    const totalCount = document.getElementById('totalCount')
    const lowCount = document.getElementById('lowCount')

    // load
    function load(){
      const raw = localStorage.getItem(STORAGE_KEY)
      try{
        items = raw ? JSON.parse(raw) : []
      }catch(e){items = []}
      render()
    }

    function save(){
      localStorage.setItem(STORAGE_KEY, JSON.stringify(items))
      render()
    }

    // helpers
    function uid(){return Math.random().toString(36).slice(2,9)}
    function isLow(item){return item.qty <= (item.min||0)}

    // form submit
    form.addEventListener('submit', e=>{
      e.preventDefault()
      const name = nameInput.value.trim()
      if(!name) return alert('Preencha o nome da cor')
      const hex = hexInput.value.trim()
      const qty = Number(qtyInput.value) || 0
      const min = Number(minInput.value) || 0

      if(editingId){
        const it = items.find(i=>i.id===editingId)
        Object.assign(it,{name,hex,qty,min})
        editingId = null
      }else{
        items.push({id:uid(),name,hex,qty,min})
      }
      form.reset()
      qtyInput.value = 0
      minInput.value = 2
      save()
    })

    document.getElementById('clearBtn').addEventListener('click', ()=>{form.reset();editingId=null})
    document.getElementById('fillDemo').addEventListener('click', ()=>{
      items = [
        {id:uid(),name:'Vermelho vivo',hex:'#e11d48',qty:1,min:3},
        {id:uid(),name:'Branco puro',hex:'#ffffff',qty:6,min:2},
        {id:uid(),name:'Azul bebê',hex:'#60a5fa',qty:0,min:2},
        {id:uid(),name:'Areia',hex:'#f5f0e1',qty:2,min:2}
      ]
      save()
    })

    // import/export
    document.getElementById('exportJSON').addEventListener('click', ()=>{
      const blob = new Blob([JSON.stringify(items, null, 2)],{type:'application/json'})
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url; a.download = 'cores_estoque.json'; a.click(); URL.revokeObjectURL(url)
    })

    document.getElementById('exportCSV').addEventListener('click', ()=>{
      const rows = [['id','name','hex','qty','min']]
      items.forEach(i=>rows.push([i.id,i.name,(i.hex||''),i.qty,i.min]))
      const csv = rows.map(r=>r.map(c=>'"'+String(c).replace(/"/g,'""')+'"').join(',')).join('\n')
      const blob = new Blob([csv],{type:'text/csv'})
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url; a.download = 'cores_estoque.csv'; a.click(); URL.revokeObjectURL(url)
    })

    const importFile = document.getElementById('importFile')
    document.getElementById('importBtn').addEventListener('click', ()=>importFile.click())
    importFile.addEventListener('change', async (e)=>{
      const file = e.target.files[0]
      if(!file) return
      const text = await file.text()
      try{
        if(file.name.endsWith('.csv')){
          const lines = text.split('\n').filter(Boolean)
          const arr = lines.slice(1).map(l=>{
            const cols = l.split(',')
            return {id:cols[0].replace(/"/g,''),name:cols[1].replace(/"/g,''),hex:cols[2].replace(/"/g,''),qty:Number(cols[3]||0),min:Number(cols[4]||0)}
          })
          items = arr
        }else{
          items = JSON.parse(text)
        }
        save()
        importFile.value = ''
      }catch(err){alert('Erro ao importar: '+err.message)}
    })

    // list rendering
    function render(){
      const q = searchEl.value.trim().toLowerCase()
      let out = items.slice()
      // filter
      if(filterEl.value === 'low') out = out.filter(isLow)
      if(filterEl.value === 'zero') out = out.filter(i=>i.qty===0)
      // search
      if(q) out = out.filter(i=> (i.name||'').toLowerCase().includes(q) || (i.hex||'').toLowerCase().includes(q) )
      // sort
      if(sortEl.value === 'name') out.sort((a,b)=>a.name.localeCompare(b.name))
      if(sortEl.value === 'qty_desc') out.sort((a,b)=>b.qty - a.qty)
      if(sortEl.value === 'qty_asc') out.sort((a,b)=>a.qty - b.qty)

      listEl.innerHTML = ''
      out.forEach(i=>{
        const div = document.createElement('div')
        div.className = 'item'
        const sw = document.createElement('div')
        sw.className = 'swatch'
        sw.style.background = i.hex || '#ddd'

        const info = document.createElement('div')
        info.innerHTML = `<div class="name">${escapeHtml(i.name)}</div><div class="muted">${i.hex||''}</div>`

        const qtyBox = document.createElement('div')
        qtyBox.innerHTML = `<div class="muted">Quantidade</div><div class="qty ${isLow(i)?'low':''}">${i.qty}</div>`

        const actions = document.createElement('div')
        actions.className = 'actions'

        // decrease
        const dec = document.createElement('button')
        dec.textContent = '-'
        dec.title = 'Usar / retirar 1 unidade'
        dec.className = 'small'
        dec.addEventListener('click', ()=>{i.qty = Math.max(0,i.qty-1); save()})

        const inc = document.createElement('button')
        inc.textContent = '+'
        inc.title = 'Adicionar 1 unidade'
        inc.className = 'small'
        inc.addEventListener('click', ()=>{i.qty = Number(i.qty||0)+1; save()})

        const edit = document.createElement('button')
        edit.textContent = 'Editar'
        edit.className = 'ghost small'
        edit.addEventListener('click', ()=>{
          editingId = i.id
          nameInput.value = i.name
          hexInput.value = i.hex||''
          qtyInput.value = i.qty
          minInput.value = i.min
          window.scrollTo({top:0,behavior:'smooth'})
        })

        const del = document.createElement('button')
        del.textContent = 'Excluir'
        del.className = 'small'
        del.addEventListener('click', ()=>{
          if(confirm('Excluir '+i.name+'?')){ items = items.filter(x=>x.id!==i.id); save() }
        })

        actions.appendChild(dec)
        actions.appendChild(inc)
        actions.appendChild(edit)
        actions.appendChild(del)

        div.appendChild(sw)
        div.appendChild(info)
        div.appendChild(qtyBox)
        div.appendChild(actions)

        listEl.appendChild(div)
      })

      // summary
      totalCount.textContent = items.length
      lowCount.textContent = items.filter(isLow).length
    }

    // utilities
    function escapeHtml(s){return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}

    // events
    searchEl.addEventListener('input', render)
    filterEl.addEventListener('change', render)
    sortEl.addEventListener('change', render)

    document.getElementById('clearAll').addEventListener('click', ()=>{
      if(confirm('Limpar todo o estoque? Esta ação não pode ser desfeita.')){items=[];save()}
    })

    document.getElementById('bulkZero').addEventListener('click', ()=>{
      if(items.length===0) return alert('Nenhum item.')
      if(confirm('Zerar todas as quantidades?')){items.forEach(i=>i.qty=0);save()}
    })

    // init
    load()
  </script>
</body>
</html>
