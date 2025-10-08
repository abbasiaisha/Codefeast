# Codefeast
the online event website to reducing the food wastage
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>CodeFeast — Online Event Check-in</title>
<meta name="description" content="CodeFeast — Register, generate QR, and check-in guests (online)"/>
<link rel="icon" href="data:;base64,iVBORw0KGgo=">
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --glass: rgba(255,255,255,0.04);
    --accent:#7c3aed; --accent2:#06b6d4; --muted:#9aa4b2; --text:#e6eef8;
    --radius:12px;
  }
  *{box-sizing:border-box}
  body{margin:0;font-family:Inter,system-ui,Segoe UI,Arial;background:
    radial-gradient(800px 300px at 10% 10%, rgba(124,58,237,0.06), transparent),
    var(--bg); color:var(--text); min-height:100vh; padding:28px;}
  .wrap{max-width:1100px;margin:0 auto}
  header{display:flex;align-items:center;gap:16px;margin-bottom:18px}
  .logo{width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent),var(--accent2));display:flex;align-items:center;justify-content:center;font-weight:800;color:white}
  h1{margin:0;font-size:20px}
  .muted{color:var(--muted);font-size:13px}
  .grid{display:grid;grid-template-columns:1fr 420px;gap:18px}
  @media(max-width:980px){ .grid{grid-template-columns:1fr} }
  .card{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03);padding:16px;border-radius:var(--radius)}
  label{display:block;font-weight:700;color:var(--muted);margin-bottom:6px}
  input,select,textarea{width:100%;padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--text)}
  button{background:linear-gradient(90deg,var(--accent),var(--accent2));color:white;padding:10px 12px;border-radius:10px;border:none;cursor:pointer;font-weight:700}
  button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.04)}
  .small{font-size:13px;color:var(--muted)}
  .qr-canvas{width:140px;height:140px;border-radius:12px;background:white;padding:6px}
  table{width:100%;border-collapse:collapse;margin-top:10px}
  th,td{padding:8px;border-bottom:1px solid rgba(255,255,255,0.03);font-size:13px;text-align:left}
  .pill{background:rgba(255,255,255,0.02);padding:6px 10px;border-radius:999px;font-weight:700;color:var(--muted)}
  .row{display:flex;gap:10px;align-items:center;flex-wrap:wrap}
  footer{color:var(--muted);font-size:13px;text-align:center;margin-top:18px}
  .danger{background:#dc2626}
  .hide{display:none !important}
</style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="logo">CF</div>
      <div>
        <h1>CodeFeast</h1>
        <div class="muted">Online — Register guests, generate scannable QR & check-in</div>
      </div>
      <div style="margin-left:auto" class="row small">
        <div class="pill" id="capacityPill">Capacity: 300</div>
      </div>
    </header>

    <div class="grid">
      <!-- left: main -->
      <div>
        <!-- Tabs -->
        <div style="display:flex;gap:8px;margin-bottom:12px">
          <button id="tabRegister" class="tab active">Register</button>
          <button id="tabScan" class="tab">Scan / Check-in</button>
          <button id="tabAdmin" class="tab">Admin</button>
        </div>

        <!-- Register -->
        <div class="card" id="panelRegister">
          <h3>Register Family / Group</h3>
          <div class="small">Family provides details and gets a downloadable QR code.</div>

          <div style="margin-top:12px">
            <label>Family name / Lead</label>
            <input id="r_name" placeholder="e.g. Ahmed family">
          </div>

          <div class="row" style="margin-top:6px">
            <div style="width:150px">
              <label>Persons (1–8)</label>
              <select id="r_persons"></select>
            </div>
            <div style="flex:1">
              <label>Mobile / Email (optional)</label>
              <input id="r_contact" placeholder="phone or email">
            </div>
          </div>

          <div style="margin-top:8px">
            <label>Menu platters (choose 1–8)</label>
            <div id="r_platters" style="display:flex;flex-wrap:wrap;gap:8px;margin-top:6px"></div>
          </div>

          <div style="margin-top:10px" class="row">
            <button id="btnRegister">Register & Generate QR</button>
            <button id="btnDownloadQr" class="ghost" style="display:none">Download QR</button>
            <div class="small" style="margin-left:auto">Registered: <strong id="registeredCount">0</strong></div>
          </div>

          <div style="margin-top:12px" id="lastQrWrap" class="hide">
            <label>QR (download / screenshot)</label>
            <div class="row">
              <canvas id="lastQR" class="qr-canvas"></canvas>
              <div>
                <div><strong id="lastName">—</strong></div>
                <div class="small" id="lastContact">—</div>
                <div style="margin-top:8px"><button id="btnCloseQr" class="ghost">Close</button></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Scan/Check-in -->
        <div class="card hide" id="panelScan">
          <h3>Scan & Check-in</h3>
          <div class="small">Use camera to scan guest QR (mobile recommended) or paste payload.</div>

          <div style="margin-top:10px" class="row">
            <button id="btnOpenCam">Open Camera</button>
            <button id="btnStopCam" class="ghost" style="display:none">Stop Camera</button>
            <input id="scanInput" placeholder="Or paste QR payload" style="flex:1">
            <button id="btnCheckPaste" class="ghost">Validate</button>
          </div>

          <div id="cameraWrap" style="margin-top:12px;display:none">
            <video id="video" autoplay playsinline style="width:100%;border-radius:10px;border:1px solid rgba(255,255,255,0.04)"></video>
            <canvas id="scanCanvas" style="display:none"></canvas>
            <div id="scanResult" class="small" style="margin-top:10px"></div>
          </div>
        </div>

        <!-- Admin panel -->
        <div class="card hide" id="panelAdmin">
          <h3>Admin Login</h3>
          <div class="small">Admin can view, delete entries, export CSV, or reset. Login persists until logout.</div>
          <div style="margin-top:10px" class="row">
            <input id="adminPass" type="password" placeholder="Admin password" style="flex:1">
            <button id="btnAdminLogin">Login</button>
          </div>

          <div id="adminArea" style="margin-top:12px;display:none">
            <div class="row" style="align-items:center;gap:8px">
              <button id="btnExport" class="ghost">Export CSV</button>
              <button id="btnReset" class="danger">Reset All</button>
              <button id="btnLogout" class="ghost" style="margin-left:auto">Logout</button>
            </div>

            <div style="margin-top:12px;overflow:auto;max-height:380px">
              <table>
                <thead><tr><th>#</th><th>Family</th><th>P</th><th>Contact</th><th>Menu</th><th>QR</th><th>Status</th><th>Action</th></tr></thead>
                <tbody id="adminTable"></tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <!-- right column: info -->
      <aside>
        <div class="card">
          <h4>Quick Info</h4>
          <div class="small">Website: <strong>CodeFeast</strong></div>
          <div style="margin-top:8px" class="small">Capacity: <strong id="cap">300</strong></div>
          <div style="margin-top:8px">
            <label>Search family</label>
            <input id="searchInput" placeholder="type name and press Enter">
          </div>
        </div>

        <div class="card" style="margin-top:12px">
          <h4>Platters (1–8)</h4>
          <ol id="platList" class="small"></ol>
        </div>
      </aside>
    </div>

    <footer><div class="small" style="text-align:center;margin-top:18px">Made with ♥ — CodeFeast</div></footer>
  </div>

  <!-- libs: QR generation + QR decode -->
  <script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.1/build/qrcode.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/jsqr/dist/jsQR.min.js"></script>

  <script>
  /* ----------------------
    CodeFeast — Online-ready single file
     - LocalStorage by default
     - Optional Firebase hooks commented
     - Admin password persistent in localStorage
  ------------------------*/
  const DEFAULT_ADMIN_PASS = 'codefeast123';
  const STORAGE_KEY = 'codefeast_online_v1';
  const ADMIN_LOGGED_KEY = 'codefeast_admin_logged';
  const CAP = 300;

  // UI refs
  const r_name = document.getElementById('r_name');
  const r_persons = document.getElementById('r_persons');
  const r_contact = document.getElementById('r_contact');
  const r_platters = document.getElementById('r_platters');
  const registeredCount = document.getElementById('registeredCount');
  const lastQR = document.getElementById('lastQR');
  const lastName = document.getElementById('lastName');
  const lastContact = document.getElementById('lastContact');
  const lastQrWrap = document.getElementById('lastQrWrap');
  const btnDownloadQr = document.getElementById('btnDownloadQr');

  const tabRegister = document.getElementById('tabRegister');
  const tabScan = document.getElementById('tabScan');
  const tabAdmin = document.getElementById('tabAdmin');

  const panelRegister = document.getElementById('panelRegister');
  const panelScan = document.getElementById('panelScan');
  const panelAdmin = document.getElementById('panelAdmin');

  const btnRegister = document.getElementById('btnRegister');
  const btnOpenCam = document.getElementById('btnOpenCam');
  const btnStopCam = document.getElementById('btnStopCam');
  const video = document.getElementById('video');
  const scanCanvas = document.getElementById('scanCanvas');
  const scanResult = document.getElementById('scanResult');
  const scanInput = document.getElementById('scanInput');
  const btnCheckPaste = document.getElementById('btnCheckPaste');

  const adminPass = document.getElementById('adminPass');
  const btnAdminLogin = document.getElementById('btnAdminLogin');
  const adminArea = document.getElementById('adminArea');
  const adminTable = document.getElementById('adminTable');
  const btnExport = document.getElementById('btnExport');
  const btnReset = document.getElementById('btnReset');
  const btnLogout = document.getElementById('btnLogout');
  const searchInput = document.getElementById('searchInput');

  // state
  let families = loadData();
  let nextId = families.length ? Math.max(...families.map(f=>f.id))+1 : 1;
  let videoStream = null;
  const plattersList = ['Grilled Chicken','Veggie Delight','Seafood Special','BBQ Mixed','Paneer Tikka','Salad & Dips','Rice & Biryani','Dessert Assortment'];

  // populate persons and platters
  for(let i=1;i<=8;i++){ const o=document.createElement('option'); o.value=i; o.textContent=i; r_persons.appendChild(o); }
  plattersList.forEach((p,i)=>{
    const btn = document.createElement('button');
    btn.className='ghost';
    btn.style.borderRadius='10px';
    btn.textContent = `${i+1}) ${p}`;
    btn.onclick = (e)=> {
      // toggle a hidden checkbox
      const existing = document.getElementById('chk_pl_'+i);
      if(!existing){
        const chk = document.createElement('input'); chk.type='checkbox'; chk.id='chk_pl_'+i; chk.value=p; chk.style.display='none';
        r_platters.appendChild(chk);
        chk.checked = true;
        btn.style.background = 'linear-gradient(90deg,var(--accent),var(--accent2))';
        btn.style.color = 'white';
      } else {
        // remove checkbox
        existing.remove();
        btn.style.background='';
        btn.style.color='';
      }
    };
    // style as small pill
    btn.style.padding='8px 10px';
    btn.style.margin='4px';
    btn.style.border='1px solid rgba(255,255,255,0.04)';
    btn.style.background='transparent';
    r_platters.appendChild(btn);

    // also list in right column
    const li = document.createElement('li'); li.textContent=(i+1)+') '+p; document.getElementById('platList').appendChild(li);
  });

  // tabs
  function showPanel(name){
    panelRegister.classList.add('hide'); panelScan.classList.add('hide'); panelAdmin.classList.add('hide');
    tabRegister.classList.remove('active'); tabScan.classList.remove('active'); tabAdmin.classList.remove('active');
    if(name==='register'){ panelRegister.classList.remove('hide'); tabRegister.classList.add('active'); }
    if(name==='scan'){ panelScan.classList.remove('hide'); tabScan.classList.add('active'); }
    if(name==='admin'){ panelAdmin.classList.remove('hide'); tabAdmin.classList.add('active'); }
  }
  tabRegister.onclick = ()=> showPanel('register');
  tabScan.onclick = ()=> showPanel('scan');
  tabAdmin.onclick = ()=> showPanel('admin');

  // default
  document.getElementById('registeredCount').textContent = families.length;
  document.getElementById('cap').textContent = CAP;
  document.getElementById('capacityPill').textContent = `Capacity: ${CAP}`;

  // register
  btnRegister.onclick = async ()=>{
    if(families.length >= CAP){ alert('Capacity reached'); return; }
    const name = (r_name.value || ('Family #'+nextId)).trim();
    const persons = Number(r_persons.value);
    const contact = r_contact.value.trim();
    const chosen = Array.from(r_platters.querySelectorAll('input[type=checkbox]')).map(c=>c.value);
    if(chosen.length < 1 || chosen.length > 8){ alert('Please select 1–8 platters'); return; }
    const payload = `CODEFEAST|FAMILY|${nextId}`; // payload encoded in QR
    // generate QR canvas (uses qrcode lib)
    try {
      await QRCode.toCanvas(lastQR, payload, {width:140});
    } catch(e){ console.error(e); }
    lastName.textContent = name;
    lastContact.textContent = contact || '—';
    lastQrWrap.classList.remove('hide');
    btnDownloadQr.style.display = 'inline-block';
    btnDownloadQr.onclick = ()=> {
      const data = lastQR.toDataURL('image/png');
      const a = document.createElement('a'); a.href = data; a.download = `${name.replace(/\s+/g,'_')}_codefeast_qr.png`; a.click();
    };

    const record = { id: nextId, name, persons, contact, platters: chosen, payload, checkedIn:false, created: Date.now() };
    families.push(record); nextId++;
    saveData();
    r_name.value=''; r_contact.value='';
    // clear platters checkboxes (remove inputs)
    Array.from(r_platters.querySelectorAll('input[type=checkbox]')).forEach(n=>n.remove());
    // reset buttons style
    Array.from(r_platters.querySelectorAll('button')).forEach(b=>{ b.style.background='transparent'; b.style.color=''; });
    document.getElementById('registeredCount').textContent = families.length;
    renderAdminTable();
    alert('Registered successfully — QR generated below.');
  };

  // search/filter helper
  searchInput.addEventListener('keydown', (e)=> { if(e.key==='Enter') renderAdminTable(searchInput.value.trim()); });

  // Admin login/area
  btnAdminLogin.onclick = ()=>{
    const pass = adminPass.value || '';
    const stored = localStorage.getItem('cf_admin_pass') || DEFAULT_ADMIN_PASS;
    if(pass === stored){
      localStorage.setItem(ADMIN_LOGGED_KEY, 'true');
      adminArea.style.display='block';
      adminPass.value='';
      renderAdminTable();
      alert('Admin logged in. You may manage entries.');
    } else alert('Incorrect password');
  };

  // auto-login if flag
  if(localStorage.getItem(ADMIN_LOGGED_KEY) === 'true'){ adminArea.style.display='block'; }

  function renderAdminTable(filter=''){
    adminTable.innerHTML = '';
    const list = (filter ? families.filter(f=>f.name.toLowerCase().includes(filter.toLowerCase())) : families).slice().sort((a,b)=>a.id-b.id);
    list.forEach((f, idx)=>{
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${f.id}</td>
        <td style="max-width:180px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis">${escapeHtml(f.name)}</td>
        <td>${f.persons}</td>
        <td style="max-width:180px;overflow:hidden;text-overflow:ellipsis">${escapeHtml(f.contact||'—')}</td>
        <td style="max-width:220px">${escapeHtml(f.platters.join(', '))}</td>
        <td><canvas id="qr_${f.id}" width="96" height="96" style="border-radius:6px;background:white"></canvas></td>
        <td>${f.checkedIn ? '<span style="color:#10b981;font-weight:700">Checked-in</span>' : 'Pending'}</td>
        <td><button data-id="${f.id}" class="btnDelete ghost">Delete</button></td>`;
      adminTable.appendChild(tr);
      // generate QR
      QRCode.toCanvas(document.getElementById('qr_'+f.id), f.payload, {width:96}).catch(()=>{});
    });
    // attach delete handlers
    Array.from(document.querySelectorAll('.btnDelete')).forEach(btn=>{
      btn.onclick = ()=> {
        const id = Number(btn.dataset.id);
        if(!confirm('Delete record #'+id+'?')) return;
        families = families.filter(x=>x.id!==id);
        saveData(); renderAdminTable(filter);
      };
    });
  }

  // export CSV
  btnExport.onclick = ()=>{
    if(!families.length){ alert('No registrations to export'); return; }
    const rows = [['id','name','persons','contact','platters','payload','checkedIn','created']];
    families.forEach(f=> rows.push([f.id,f.name,f.persons,f.contact||'',f.platters.join('|'),f.payload,f.checkedIn,f.created]));
    const csv = rows.map(r=> r.map(c=> `"${String(c).replace(/"/g,'""')}"`).join(',')).join('\n');
    const blob = new Blob([csv], {type:'text/csv'}); const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href=url; a.download='codefeast_registrations.csv'; a.click(); URL.revokeObjectURL(url);
  };

  // reset
  btnReset.onclick = ()=> {
    if(!confirm('Reset all data? This will remove all registrations.')) return;
    families = []; nextId = 1; saveData(); renderAdminTable(); document.getElementById('registeredCount').textContent = 0;
  };

  btnLogout.onclick = ()=> {
    localStorage.removeItem(ADMIN_LOGGED_KEY); adminArea.style.display='none'; alert('Admin logged out');
  };

  // scan using camera & jsQR
  btnOpenCam.onclick = async ()=>{
    if(!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia){ alert('Camera not supported in this browser'); return; }
    try {
      videoStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' }});
      video.srcObject = videoStream;
      document.getElementById('cameraWrap').style.display='block';
      btnStopCam.style.display='inline-block';
      btnOpenCam.style.display='none';
      tickScan();
    } catch(err){ alert('Camera permission denied or not available'); console.error(err); }
  };
  btnStopCam.onclick = ()=>{
    if(videoStream){ videoStream.getTracks().forEach(t=>t.stop()); videoStream=null; }
    document.getElementById('cameraWrap').style.display='none';
    btnStopCam.style.display='none';
    btnOpenCam.style.display='inline-block';
    scanResult.textContent='';
  };

  function tickScan(){
    if(!videoStream) return;
    const v = video;
    const w = v.videoWidth, h = v.videoHeight;
    if(w && h){
      scanCanvas.width = w; scanCanvas.height = h;
      const ctx = scanCanvas.getContext('2d'); ctx.drawImage(v,0,0,w,h);
      const img = ctx.getImageData(0,0,w,h);
      const code = jsQR(img.data, img.width, img.height, { inversionAttempts: 'dontInvert' });
      if(code){
        handleScannedPayload(code.data);
        // stop camera after successful scan to avoid double-scans
        btnStopCam.onclick();
        return;
      }
    }
    requestAnimationFrame(tickScan);
  }

  // manual paste validate
  btnCheckPaste.onclick = ()=> {
    const payload = scanInput.value.trim();
    if(!payload) return alert('Paste payload from QR or type it');
    handleScannedPayload(payload);
  };

  function handleScannedPayload(payload){
    const f = families.find(x=>x.payload === payload);
    if(!f) return alert('No registration found for this QR/payload');
    if(f.checkedIn) return alert('This family already checked-in: '+f.name);
    f.checkedIn = true; saveData(); renderAdminTable(); alert('Checked-in: ' + f.name);
  }

  // helpers
  function saveData(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(families)); }
  function loadData(){ try{ return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'); }catch(e){ return []; } }
  function escapeHtml(s){ if(!s) return ''; return String(s).replace(/[&<>"]/g, c=> ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c])); }

  function renderAdminTableDelayed(){ if(localStorage.getItem(ADMIN_LOGGED_KEY)==='true') adminArea.style.display='block'; renderAdminTable(); }

  renderAdminTableDelayed();

  // optional: auto open admin panel if logged in
  if(localStorage.getItem(ADMIN_LOGGED_KEY) === 'true'){ showPanel('admin'); adminArea.style.display='block'; }

  // storage helper & initial UI counts
  function loadData(){ try{ return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'); } catch(e){return [];} }
  function saveData(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(families)); document.getElementById('registeredCount').textContent = families.length; }
  // ensure correct variable shadowing fixed (re-assign functions above)
  // (we replaced earlier simple ones to avoid double-definition)

  // override previous definitions to ensure save/load work
  saveData(); document.getElementById('registeredCount').textContent = families.length;

  // small utility: close QR preview
  document.getElementById('btnCloseQr').onclick = ()=> { lastQrWrap.classList.add('hide'); btnDownloadQr.style.display='none'; };

  // Allow Enter on admin pass to login
  adminPass.addEventListener('keydown', (e)=> { if(e.key==='Enter') btnAdminLogin.click(); });
  searchInput.addEventListener('keydown', (e)=> { if(e.key==='Enter') renderAdminTable(searchInput.value.trim()); });

  // Optional Firebase integration (cloud) — commented, extra steps required:
  /*
    // 1) create Firebase project, enable Firestore or Realtime DB
    // 2) add Firebase SDK scripts and initialize with your config
    // 3) replace loadData/saveData with cloud reads/writes
    // Example SDK scripts:
    // <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-app-compat.js"></script>
    // <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-firestore-compat.js"></script>
    // Then use firestore().collection('registrations')...
  */

  </script>
</body>
</html>
