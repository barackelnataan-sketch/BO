<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Branch Daily Tasks & Issue Register</title>
  <style>
    :root{ --bg:#f6f8fb; --card:#fff; --muted:#6b7280; --accent:#2563eb; }
    *{box-sizing:border-box}
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial; margin:0; background:var(--bg); color:#111}
    .wrap{max-width:1100px;margin:18px auto;padding:16px}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px}
    h1{font-size:1.15rem;margin:0}
    p.lead{margin:4px 0 12px;color:var(--muted);font-size:0.95rem}
    .grid{display:grid;grid-template-columns:1fr 380px;gap:14px}
    @media (max-width:920px){.grid{grid-template-columns:1fr}}
    .card{background:var(--card);border-radius:10px;padding:14px;box-shadow:0 6px 20px rgba(15,23,42,0.06)}
    label{display:block;font-weight:600;font-size:0.88rem;margin-bottom:6px}
    .field{margin-bottom:10px}
    input,select,textarea{width:100%;padding:10px;border-radius:8px;border:1px solid #d1dfea;font-size:0.95rem}
    textarea{resize:vertical}
    .btn{display:inline-block;padding:10px 12px;border-radius:8px;border:0;background:var(--accent);color:#fff;font-weight:700;cursor:pointer}
    .btn.ghost{background:transparent;color:var(--accent);border:1px solid #dbeafe}
    .small{font-size:0.9rem;color:var(--muted)}
    .list{margin-top:10px;display:flex;flex-direction:column;gap:8px}
    .item{padding:10px;border-radius:8px;border:1px solid #eef2ff;background:linear-gradient(180deg,#fff,#fbfdff);display:flex;align-items:flex-start;gap:8px}
    .item .meta{font-size:0.82rem;color:var(--muted)}
    .item h3{margin:0;font-size:1rem}
    .item-actions{margin-left:auto;display:flex;gap:6px}
    .badge{padding:4px 8px;border-radius:999px;font-size:0.78rem}
    .priority-high{background:#fee2e2;color:#991b1b}
    .priority-med{background:#fff7ed;color:#92400e}
    .priority-low{background:#ecfdf5;color:#065f46}
    .filters{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
    .muted{color:var(--muted)}
    footer{margin-top:14px;font-size:0.85rem;color:var(--muted)}
    .status-line{display:flex;gap:8px;align-items:center}
    .status-dot{width:10px;height:10px;border-radius:999px;background:#94a3b8;display:inline-block}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1>Branch Daily Tasks & Issue Register</h1>
        <p class="lead">Offline-first: localStorage. Use Sync to push/pull with a local server when available.</p>
      </div>
      <div style="display:flex;gap:8px;align-items:center">
        <div class="status-line"><span class="status-dot" id="netDot"></span><span id="netStatus" class="small muted">Offline</span></div>
        <button class="btn" onclick="syncToServer()">Sync → Server</button>
        <button class="btn ghost" onclick="pullFromServer()">Pull from Server</button>
        <button class="btn" onclick="exportData()">Export JSON</button>
        <button class="btn ghost" onclick="importDataPrompt()">Import JSON</button>
      </div>
    </header>

    <div class="grid">
      <main>
        <section class="card">
          <div style="display:flex;align-items:center;justify-content:space-between">
            <h2 style="font-size:1rem;margin:0">Today's Tasks</h2>
            <div class="small muted">Local only • Auto-saved</div>
          </div>

          <div class="filters">
            <select id="filterStatus" onchange="renderAll()">
              <option value="all">All statuses</option>
              <option value="pending">Pending</option>
              <option value="in-progress">In progress</option>
              <option value="done">Done</option>
            </select>

            <select id="filterPriority" onchange="renderAll()">
              <option value="all">All priorities</option>
              <option value="high">High</option>
              <option value="medium">Medium</option>
              <option value="low">Low</option>
            </select>

            <input id="search" placeholder="Search tasks or issues" oninput="renderAll()" style="padding:10px;border-radius:8px;border:1px solid #dfe8f8;" />
          </div>

          <div id="tasksList" class="list" aria-live="polite"></div>

          <hr style="margin:12px 0" />

          <h3 style="margin:0 0 8px 0">Issue Register</h3>
          <div class="small muted">Log incidents, operational problems, or service issues here.</div>
          <div id="issuesList" class="list"></div>
        </section>

        <section class="card" style="margin-top:12px">
          <h2 style="font-size:1rem;margin:0 0 8px 0">Quick Actions</h2>
          <div style="display:flex;gap:8px;flex-wrap:wrap">
            <button class="btn" onclick="clearAll()">Clear All Data</button>
            <button class="btn ghost" onclick="markAllDone()">Mark All Tasks Done</button>
            <button class="btn ghost" onclick="printReport()">Print Tasks & Issues</button>
          </div>
        </section>
      </main>

      <aside>
        <div class="card">
          <h3 style="margin-top:0">Add / Assign Task</h3>
          <div class="field">
            <label for="taskTitle">Task title</label>
            <input id="taskTitle" placeholder="e.g. Cash balance reconciliation" />
          </div>
          <div class="field">
            <label for="assignee">Assign to (staff)</label>
            <input id="assignee" placeholder="Staff name or initials" />
          </div>
          <div class="field">
            <label for="dueDate">Due date & time</label>
            <input id="dueDate" type="datetime-local" />
          </div>
          <div class="field">
            <label for="priority">Priority</label>
            <select id="priority">
              <option value="medium">Medium</option>
              <option value="high">High</option>
              <option value="low">Low</option>
            </select>
          </div>
          <div class="field">
            <label for="taskNotes">Notes</label>
            <textarea id="taskNotes" rows="3" placeholder="Details or steps to complete"></textarea>
          </div>
          <div style="display:flex;gap:8px">
            <button class="btn" onclick="addTask()">Add Task</button>
            <button class="btn ghost" onclick="clearTaskForm()">Reset</button>
          </div>
        </div>

        <div class="card" style="margin-top:12px">
          <h3 style="margin-top:0">Register an Issue</h3>
          <div class="field">
            <label for="issueTitle">Issue title</label>
            <input id="issueTitle" placeholder="e.g. ATM offline" />
          </div>
          <div class="field">
            <label for="reportedBy">Reported by</label>
            <input id="reportedBy" placeholder="Staff name" />
          </div>
          <div class="field">
            <label for="severity">Severity</label>
            <select id="severity">
              <option value="major">Major</option>
              <option value="minor">Minor</option>
              <option value="critical">Critical</option>
            </select>
          </div>
          <div class="field">
            <label for="issueNotes">Notes / Action taken</label>
            <textarea id="issueNotes" rows="3" placeholder="Describe what happened and immediate steps"></textarea>
          </div>
          <div style="display:flex;gap:8px">
            <button class="btn" onclick="addIssue()">Log Issue</button>
            <button class="btn ghost" onclick="clearIssueForm()">Reset</button>
          </div>
        </div>

        <div class="card" style="margin-top:12px">
          <h3 style="margin-top:0">Tips</h3>
          <ul class="small muted">
            <li>Works offline in the browser — data stored locally.</li>
            <li>Use Sync to send local data to the local server (or pull server copy).</li>
            <li>Export before importing if you want a manual backup.</li>
          </ul>
        </div>
      </aside>
    </div>

    <footer class="small muted">Built for branch operations — simple, offline, and printable.</footer>
  </div>

  <script>
    const STORAGE_KEY = 'branch_tracker_v1';
    let state = loadData();

    // ----- network status UI -----
    function updateNetStatus(){
      const online = navigator.onLine;
      document.getElementById('netStatus').textContent = online ? 'Online' : 'Offline';
      document.getElementById('netDot').style.background = online ? '#10b981' : '#94a3b8';
    }
    window.addEventListener('online', updateNetStatus);
    window.addEventListener('offline', updateNetStatus);
    updateNetStatus();

    // ----- storage helpers -----
    function loadData(){
      try{
        const raw = localStorage.getItem(STORAGE_KEY);
        if(!raw) return {tasks:[], issues:[]};
        return JSON.parse(raw);
      }catch(e){ console.error(e); return {tasks:[], issues:[]}; }
    }
    function saveData(data){ localStorage.setItem(STORAGE_KEY, JSON.stringify(data)); }

    function formatDate(d){ if(!d) return ''; const dt = new Date(d); if(isNaN(dt)) return d; return dt.toLocaleString(); }
    function makeId(prefix='id'){ return prefix + '_' + Math.random().toString(36).slice(2,9); }

    // ----- CRUD (same as before) -----
    function addTask(){ const title = document.getElementById('taskTitle').value.trim(); if(!title){ alert('Enter task title'); return; }
      const task = { id: makeId('task'), title, assignee: document.getElementById('assignee').value.trim() || 'Unassigned', due: document.getElementById('dueDate').value || null, priority: document.getElementById('priority').value || 'medium', notes: document.getElementById('taskNotes').value.trim() || '', status: 'pending', createdAt: new Date().toISOString() };
      state.tasks.unshift(task); saveData(state); clearTaskForm(); renderAll();
    }
    function clearTaskForm(){ document.getElementById('taskTitle').value=''; document.getElementById('assignee').value=''; document.getElementById('dueDate').value=''; document.getElementById('taskNotes').value=''; document.getElementById('priority').value='medium'; }

    function addIssue(){ const title = document.getElementById('issueTitle').value.trim(); if(!title){ alert('Enter issue title'); return; }
      const issue = { id: makeId('issue'), title, reportedBy: document.getElementById('reportedBy').value.trim() || 'Unknown', severity: document.getElementById('severity').value || 'minor', notes: document.getElementById('issueNotes').value.trim() || '', status: 'open', createdAt: new Date().toISOString() };
      state.issues.unshift(issue); saveData(state); clearIssueForm(); renderAll();
    }
    function clearIssueForm(){ document.getElementById('issueTitle').value=''; document.getElementById('reportedBy').value=''; document.getElementById('issueNotes').value=''; document.getElementById('severity').value='major'; }

    // ----- rendering -----
    function renderAll(){ renderTasks(); renderIssues(); }
    function renderTasks(){ const container = document.getElementById('tasksList'); container.innerHTML=''; const statusFilter = document.getElementById('filterStatus').value; const priorityFilter = document.getElementById('filterPriority').value; const q = document.getElementById('search').value.trim().toLowerCase();
      const list = state.tasks.filter(t => {
        if(statusFilter !== 'all' && t.status !== statusFilter) return false;
        if(priorityFilter !== 'all' && t.priority !== priorityFilter) return false;
        if(q) return (t.title + ' ' + t.assignee + ' ' + t.notes).toLowerCase().includes(q);
        return true;
      });
      if(list.length===0){ container.innerHTML = '<div class="small muted">No tasks found.</div>'; return; }
      list.forEach(t => {
        const el = document.createElement('div'); el.className='item';
        const left = document.createElement('div'); left.style.flex='1';
        const title = document.createElement('h3'); title.textContent = t.title;
        const meta = document.createElement('div'); meta.className='meta';
        meta.innerHTML = `<strong>${t.assignee}</strong> • ${formatDate(t.due)} • <span class="small">${t.status}</span>`;
        const notes = document.createElement('div'); notes.textContent = t.notes; notes.className='small muted'; notes.style.marginTop='6px';
        left.appendChild(title); left.appendChild(meta); if(t.notes) left.appendChild(notes);
        const actions = document.createElement('div'); actions.className='item-actions';
        const badge = document.createElement('span'); badge.className='badge ' + (t.priority==='high'?'priority-high':t.priority==='medium'?'priority-med':'priority-low'); badge.textContent = t.priority.toUpperCase(); actions.appendChild(badge);
        const nextBtn = document.createElement('button'); nextBtn.className='btn ghost'; nextBtn.textContent = t.status==='done' ? 'Reopen' : 'Next'; nextBtn.onclick = () => { toggleTaskStatus(t.id); };
        actions.appendChild(nextBtn);
        const editBtn = document.createElement('button'); editBtn.className='btn ghost'; editBtn.textContent='Edit'; editBtn.onclick = () => { editTask(t.id); };
        actions.appendChild(editBtn);
        const delBtn = document.createElement('button'); delBtn.className='btn ghost'; delBtn.textContent='Delete'; delBtn.onclick = () => { deleteTask(t.id); };
        actions.appendChild(delBtn);
        el.appendChild(left); el.appendChild(actions); container.appendChild(el);
      });
    }
    function renderIssues(){ const container = document.getElementById('issuesList'); container.innerHTML=''; const q = document.getElementById('search').value.trim().toLowerCase();
      const list = state.issues.filter(i => { if(q) return (i.title + ' ' + i.reportedBy + ' ' + i.notes).toLowerCase().includes(q); return true; });
      if(list.length===0){ container.innerHTML = '<div class="small muted">No issues logged.</div>'; return; }
      list.forEach(i => {
        const el = document.createElement('div'); el.className='item';
        const left = document.createElement('div'); left.style.flex='1';
        const title = document.createElement('h3'); title.textContent = i.title;
        const meta = document.createElement('div'); meta.className='meta';
        meta.innerHTML = `<strong>${i.reportedBy}</strong> • ${i.severity} • <span class="small">${i.status}</span>`;
        const notes = document.createElement('div'); notes.textContent = i.notes; notes.className='small muted'; notes.style.marginTop='6px';
        left.appendChild(title); left.appendChild(meta); if(i.notes) left.appendChild(notes);
        const actions = document.createElement('div'); actions.className='item-actions';
        const resolveBtn = document.createElement('button'); resolveBtn.className='btn ghost'; resolveBtn.textContent = i.status==='closed'?'Reopen':'Close'; resolveBtn.onclick = () => { toggleIssueStatus(i.id); };
        actions.appendChild(resolveBtn);
        const del = document.createElement('button'); del.className='btn ghost'; del.textContent='Delete'; del.onclick = () => { deleteIssue(i.id); };
        actions.appendChild(del);
        el.appendChild(left); el.appendChild(actions); container.appendChild(el);
      });
    }

    // ----- task/issue actions -----
    function toggleTaskStatus(id){ const t = state.tasks.find(x=>x.id===id); if(!t) return; if(t.status==='done') t.status='pending'; else if(t.status==='pending') t.status='in-progress'; else if(t.status==='in-progress') t.status='done'; saveData(state); renderAll(); }
    function editTask(id){ const t = state.tasks.find(x=>x.id===id); if(!t) return; const newTitle = prompt('Edit task title', t.title); if(newTitle===null) return; t.title = newTitle.trim() || t.title; const newAssignee = prompt('Assignee', t.assignee); if(newAssignee!==null) t.assignee = newAssignee.trim() || t.assignee; const newDue = prompt('Due date (YYYY-MM-DDTHH:mm) or empty', t.due||''); if(newDue!==null) t.due = newDue || null; const newNotes = prompt('Notes', t.notes); if(newNotes!==null) t.notes = newNotes.trim(); saveData(state); renderAll(); }
    function deleteTask(id){ if(!confirm('Delete this task?')) return; state.tasks = state.tasks.filter(x=>x.id!==id); saveData(state); renderAll(); }
    function toggleIssueStatus(id){ const it = state.issues.find(x=>x.id===id); if(!it) return; it.status = it.status==='closed'?'open':'closed'; saveData(state); renderAll(); }
    function deleteIssue(id){ if(!confirm('Delete this issue?')) return; state.issues = state.issues.filter(x=>x.id!==id); saveData(state); renderAll(); }

    // ----- utilities -----
    function clearAll(){ if(!confirm('This will remove all tasks and issues. Continue?')) return; state = {tasks:[], issues:[]}; saveData(state); renderAll(); }
    function markAllDone(){ state.tasks.forEach(t=> t.status='done'); saveData(state); renderAll(); }
    function exportData(){ const dataStr = JSON.stringify(state, null, 2); const blob = new Blob([dataStr], {type:'application/json'}); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href=url; a.download = 'branch-tracker-export.json'; a.click(); URL.revokeObjectURL(url); }
    function importDataPrompt(){ const input = document.createElement('input'); input.type='file'; input.accept='application/json'; input.onchange = e => { const file = e.target.files[0]; if(!file) return; const reader = new FileReader(); reader.onload = ev => { try{ const imported = JSON.parse(ev.target.result); if(!imported.tasks || !imported.issues) throw new Error('Invalid format'); if(confirm('Import will replace current data. Continue?')){ state = imported; saveData(state); renderAll(); } }catch(err){ alert('Failed to import: ' + err.message); } }; reader.readAsText(file); }; input.click(); }
    function importDataFromString(str){ try{ const imported = JSON.parse(str); if(!imported.tasks || !imported.issues) throw new Error('Invalid format'); state = imported; saveData(state); renderAll(); alert('Imported successfully'); }catch(e){ alert('Import failed: '+e.message); } }
    function printReport(){ const w = window.open('','_blank'); const html = `<!doctype html><html><head><meta charset="utf-8"><title>Tasks & Issues Report</title><style>body{font-family:Arial,Helvetica,sans-serif;padding:20px}h1{font-size:18px}h2{font-size:14px;margin-top:14px}table{width:100%;border-collapse:collapse}th,td{border:1px solid #ddd;padding:8px;text-align:left}th{background:#f2f4f8}</style></head><body>`; const tasksHtml = state.tasks.map(t=>`<tr><td>${t.title}</td><td>${t.assignee}</td><td>${t.priority}</td><td>${t.status}</td><td>${formatDate(t.due)}</td></tr>`).join(''); const issuesHtml = state.issues.map(i=>`<tr><td>${i.title}</td><td>${i.reportedBy}</td><td>${i.severity}</td><td>${i.status}</td><td>${i.notes}</td></tr>`).join(''); w.document.write(html + '<h1>Branch Tasks & Issues Report</h1>' + '<h2>Tasks</h2>' + '<table><tr><th>Task</th><th>Assignee</th><th>Priority</th><th>Status</th><th>Due</th></tr>' + tasksHtml + '</table>' + '<h2>Issues</h2>' + '<table><tr><th>Issue</th><th>Reported by</th><th>Severity</th><th>Status</th><th>Notes</th></tr>' + issuesHtml + '</table>' + '</body></html>'); w.document.close(); w.print(); }

    // ----- sync helpers (front->server and pull) -----
    // server base URL (if running locally with server.js defaults)
    const SERVER_BASE = '';

    // push local state to server (replaces remote data with local copy)
    async function syncToServer(){
      if(!navigator.onLine){ alert('You are offline — cannot sync.'); return; }
      try{
        showBusy(true, 'Syncing to server...');
        // POST /api/replace (we will implement replace in server.js)
        const res = await fetch('/api/replace', {
          method: 'POST',
          headers: {'Content-Type':'application/json'},
          body: JSON.stringify(state)
        });
        const json = await res.json();
        showBusy(false);
        if(res.ok) { alert('Sync successful. Server now has latest copy.'); }
        else { alert('Sync failed: ' + (json.message || res.statusText)); }
      }catch(err){ showBusy(false); alert('Sync error: ' + err.message); }
    }

    // pull server copy and replace local (prompt before replacing)
    async function pullFromServer(){
      if(!navigator.onLine){ alert('You are offline — cannot pull.'); return; }
      if(!confirm('Pulling will replace your local data with the server copy. Continue?')) return;
      try{
        showBusy(true, 'Fetching from server...');
        const res = await fetch('/api/all');
        const json = await res.json();
        showBusy(false);
        if(res.ok && json){ state = json; saveData(state); renderAll(); alert('Pulled data from server successfully'); }
        else alert('Failed to pull: ' + (json.message || res.statusText));
      }catch(err){ showBusy(false); alert('Pull error: ' + err.message); }
    }

    // small UI busy indicator
    function showBusy(on, message){
      if(on){
        const b = document.createElement('div'); b.id='busyDiv'; b.style.position='fixed'; b.style.right='12px'; b.style.bottom='12px'; b.style.background='#111'; b.style.color='#fff'; b.style.padding='8px 12px'; b.style.borderRadius='8px'; b.style.boxShadow='0 6px 20px rgba(0,0,0,0.15)'; b.textContent = message || 'Working...'; document.body.appendChild(b);
      } else { const ex = document.getElementById('busyDiv'); if(ex) ex.remove(); }
    }

    // expose a quick "push single changes" method if you prefer incremental pushes
    async function pushChangesToServer(){
      if(!navigator.onLine){ alert('Offline'); return; }
      // example: push each task and issue individually (server accepts /api/tasks, /api/issues)
      try{
        showBusy(true, 'Pushing changes...');
        for(const t of state.tasks){ await fetch('/api/tasks', {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify(t)}); }
        for(const i of state.issues){ await fetch('/api/issues', {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify(i)}); }
        showBusy(false); alert('Push complete (items posted).');
      }catch(err){ showBusy(false); alert('Push error: '+err.message); }
    }

    // initial render
    renderAll();
    updateNetStatus();

    // expose the import helper for advanced users
    window.importDataFromString = importDataFromString;
  </script>
</body>
</html>
