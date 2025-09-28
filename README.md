<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Nice To‑Do List</title>
  <style>
    :root{
      --bg:#0f1724;
      --card:#0b1a2b;
      --accent:#7c3aed;
      --muted:#9aa4b2;
      --glass: rgba(255,255,255,0.04);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0;
      font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      background: radial-gradient(1000px 600px at 10% 10%, rgba(124,58,237,0.12), transparent),
                  linear-gradient(180deg, #071023 0%, #06121b 100%),
                  var(--bg);
      color:#e6eef8;
      display:flex;
      align-items:center;
      justify-content:center;
      padding:40px 20px;
      -webkit-font-smoothing:antialiased;
    }

    .wrap{
      width:100%;
      max-width:760px;
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:14px;
      box-shadow: 0 10px 30px rgba(2,6,23,0.7);
      padding:28px;
      border:1px solid rgba(255,255,255,0.03);
    }

    header{
      display:flex;align-items:center;justify-content:space-between;margin-bottom:18px;
    }
    h1{margin:0;font-size:20px;letter-spacing:-0.2px}
    .subtitle{color:var(--muted);font-size:13px}

    .controls{
      display:flex;gap:10px;align-items:center;margin-bottom:18px;
    }

    .input-row{
      display:flex;gap:10px;flex-wrap:wrap;
    }
    input[type=text]{
      flex:1;
      min-width:180px;
      background:var(--glass);
      border:1px solid rgba(255,255,255,0.03);
      padding:12px 14px;border-radius:10px;color:inherit;font-size:14px;
      outline:none;transition:box-shadow .15s,transform .08s;
    }
    input[type=text]:focus{box-shadow:0 6px 18px rgba(124,58,237,0.12);transform:translateY(-1px)}

    button.btn{
      background:linear-gradient(90deg,var(--accent),#5b21b6);
      border:none;color:white;padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600;
      box-shadow:0 6px 18px rgba(92,33,182,0.18);transition:transform .08s,opacity .12s;
    }
    button.btn:active{transform:translateY(1px)}
    .ghost{background:transparent;border:1px solid rgba(255,255,255,0.05);color:var(--muted);padding:8px 10px}

    .filters{display:flex;gap:8px;align-items:center}
    .filters button{background:transparent;border:none;color:var(--muted);padding:8px 10px;border-radius:8px;cursor:pointer}
    .filters button.active{color:white;background:rgba(255,255,255,0.03)}

    ul.list{list-style:none;padding:0;margin:0;display:flex;flex-direction:column;gap:10px}
    .task{
      display:flex;align-items:center;gap:12px;padding:12px;border-radius:10px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.0));
      border:1px solid rgba(255,255,255,0.02);
    }
    .task .left{display:flex;align-items:center;gap:12px;flex:1}
    .checkbox{width:18px;height:18px;border-radius:6px;border:1px solid rgba(255,255,255,0.08);display:inline-grid;place-items:center;cursor:pointer}
    .checkbox.checked{background:linear-gradient(90deg,var(--accent),#5b21b6);border:none}
    .title{font-size:15px}
    .title.completed{color:var(--muted);text-decoration:line-through}
    .meta{font-size:12px;color:var(--muted)}

    .actions{display:flex;gap:8px}
    .icon-btn{background:transparent;border:none;color:var(--muted);padding:8px;border-radius:8px;cursor:pointer}
    .icon-btn:hover{color:white;background:rgba(255,255,255,0.02)}

    .footer{display:flex;justify-content:space-between;align-items:center;margin-top:14px;color:var(--muted);font-size:13px}
    @media (max-width:520px){.wrap{padding:18px}}
  </style>
</head>
<body>
  <div class="wrap" role="application" aria-label="To Do List">
    <header>
      <div>
        <h1>Nice To‑Do</h1>
        <div class="subtitle">Simple, responsive & saved in your browser</div>
      </div>
      <div class="subtitle" id="dateNow"></div>
    </header>

    <div class="controls">
      <div style="flex:1">
        <div class="input-row">
          <input id="taskInput" type="text" placeholder="Add a new task and press Add or Enter" aria-label="New task">
          <button id="addBtn" class="btn">Add</button>
          <button id="clearBtn" class="ghost">Clear all</button>
        </div>
      </div>
    </div>

    <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
      <div class="filters" role="tablist" aria-label="Filter tasks">
        <button data-filter="all" class="active">All</button>
        <button data-filter="active">Active</button>
        <button data-filter="completed">Completed</button>
      </div>
      <div class="meta" id="count">0 tasks</div>
    </div>

    <ul id="list" class="list" aria-live="polite"></ul>

    <div class="footer">
      <div>Tip: double-click a task to edit it</div>
      <div><button id="clearCompleted" class="ghost">Remove completed</button></div>
    </div>
  </div>

  <script>
    // Simple To-Do with localStorage
    const LS_KEY = 'nice_todo_tasks_v1';
    const listEl = document.getElementById('list');
    const input = document.getElementById('taskInput');
    const addBtn = document.getElementById('addBtn');
    const clearBtn = document.getElementById('clearBtn');
    const clearCompletedBtn = document.getElementById('clearCompleted');
    const countEl = document.getElementById('count');
    const dateNow = document.getElementById('dateNow');
    const filters = document.querySelectorAll('.filters button');

    let tasks = JSON.parse(localStorage.getItem(LS_KEY) || '[]');
    let filter = 'all';

    function formatDate(d){
      return d.toLocaleDateString(undefined,{month:'short',day:'numeric'});
    }
    dateNow.textContent = formatDate(new Date());

    function save(){
      localStorage.setItem(LS_KEY, JSON.stringify(tasks));
    }

    function uid(){return Date.now().toString(36) + Math.random().toString(36).slice(2,7)}

    function addTask(text){
      const trimmed = text.trim(); if(!trimmed) return;
      tasks.unshift({id:uid(),text:trimmed,done:false,created:Date.now()});
      save(); render(); input.value=''; input.focus();
    }

    function toggleDone(id){
      tasks = tasks.map(t => t.id===id?{...t,done:!t.done}:t);
      save(); render();
    }

    function removeTask(id){
      tasks = tasks.filter(t => t.id!==id);
      save(); render();
    }

    function editTask(id,newText){
      tasks = tasks.map(t => t.id===id?{...t,text:newText}:t);
      save(); render();
    }

    function clearAll(){ if(!confirm('Clear ALL tasks?')) return; tasks=[]; save(); render(); }
    function clearCompleted(){ tasks = tasks.filter(t => !t.done); save(); render(); }

    function setFilter(f){ filter = f; filters.forEach(b=>b.classList.toggle('active', b.dataset.filter===f)); render(); }

    function render(){
      // apply filter
      let shown = tasks.slice();
      if(filter==='active') shown = shown.filter(t=>!t.done);
      if(filter==='completed') shown = shown.filter(t=>t.done);

      listEl.innerHTML='';
      if(shown.length===0){
        const empty = document.createElement('div'); empty.className='meta'; empty.style.padding='18px 12px'; empty.textContent='No tasks yet — add something!';
        listEl.appendChild(empty);
      }

      for(const t of shown){
        const li = document.createElement('li'); li.className='task'; li.setAttribute('draggable','false');
        const left = document.createElement('div'); left.className='left';

        const cb = document.createElement('div'); cb.className='checkbox'+(t.done? ' checked':''); cb.setAttribute('role','button'); cb.setAttribute('aria-pressed', String(!!t.done));
        cb.addEventListener('click', ()=> toggleDone(t.id));
        cb.innerHTML = t.done? '&#10003;' : '';

        const title = document.createElement('div'); title.className='title'+(t.done? ' completed':''); title.textContent = t.text;
        title.title = t.text;
        title.addEventListener('dblclick', ()=> startEdit(t, title));

        const meta = document.createElement('div'); meta.className='meta'; meta.textContent = 'Added ' + formatDate(new Date(t.created));

        left.appendChild(cb); left.appendChild(title); left.appendChild(meta);
        li.appendChild(left);

        const actions = document.createElement('div'); actions.className='actions';
        const del = document.createElement('button'); del.className='icon-btn'; del.innerHTML='🗑'; del.title='Delete'; del.addEventListener('click', ()=> removeTask(t.id));
        actions.appendChild(del);
        li.appendChild(actions);

        listEl.appendChild(li);
      }

      countEl.textContent = `${tasks.filter(t=>!t.done).length} active / ${tasks.length} total`;
    }

    function startEdit(task, titleEl){
      const input = document.createElement('input'); input.type='text'; input.value = task.text; input.style.width='100%'; input.style.padding='8px'; input.style.borderRadius='8px';
      titleEl.replaceWith(input); input.focus();
      input.select();
      function finish(saveFlag){
        const val = input.value.trim();
        if(saveFlag && val) editTask(task.id, val);
        else render();
      }
      input.addEventListener('blur', ()=> finish(true));
      input.addEventListener('keydown', (e)=>{
        if(e.key==='Enter'){ finish(true); }
        else if(e.key==='Escape'){ finish(false); }
      });
    }

    // events
    addBtn.addEventListener('click', ()=> addTask(input.value));
    input.addEventListener('keydown', e=>{ if(e.key==='Enter') addTask(input.value); });
    clearBtn.addEventListener('click', clearAll);
    clearCompletedBtn.addEventListener('click', ()=>{ if(confirm('Remove completed tasks?')) clearCompleted(); });
    filters.forEach(b => b.addEventListener('click', ()=> setFilter(b.dataset.filter)));

    // initial
    render();
  </script>
</body>
</html>
