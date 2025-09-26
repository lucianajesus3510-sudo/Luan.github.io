# Luan.github.io
Site
<!DOCTYPE html><html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Roblox Keys Manager</title>
  <style>
    :root {
      --bg: #0f172a;
      --card: #1e293b;
      --text: #f1f5f9;
      --accent: #38bdf8;
      --danger: #ef4444;
      --success: #22c55e;
    }
    * { box-sizing: border-box; }
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      background: var(--bg);
      color: var(--text);
      display: flex;
      justify-content: center;
      align-items: flex-start;
      padding: 2rem;
      min-height: 100vh;
    }
    .container {
      width: 100%;
      max-width: 900px;
      background: var(--card);
      padding: 2rem;
      border-radius: 1rem;
      box-shadow: 0 8px 24px rgba(0,0,0,.4);
    }
    h1 { color: var(--accent); text-align: center; }
    button {
      background: var(--accent);
      border: none;
      color: white;
      padding: .6rem 1.2rem;
      border-radius: .5rem;
      cursor: pointer;
      transition: .3s;
    }
    button:hover { opacity: .8; }
    .danger { background: var(--danger); }
    .success { background: var(--success); }
    input, select {
      width: 100%;
      padding: .5rem;
      margin: .3rem 0 1rem;
      border-radius: .4rem;
      border: none;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
    }
    th, td {
      padding: .6rem;
      border-bottom: 1px solid #334155;
      text-align: left;
    }
    th { color: var(--accent); }
    .actions button { margin-right: .3rem; }
    .notification {
      margin-top: 1rem;
      padding: .5rem;
      border-radius: .5rem;
      display: none;
      text-align: center;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Roblox Keys Manager</h1>
    <form id="keyForm">
      <label>Nome da Key</label>
      <input type="text" id="keyName" required>
      <label>Duração (minutos)</label>
      <input type="number" id="duration" min="1" required>
      <button type="submit">Gerar Key</button>
    </form><div id="notification" class="notification"></div>

<table>
  <thead>
    <tr>
      <th>Key</th>
      <th>Nome</th>
      <th>Expira em</th>
      <th>Ações</th>
    </tr>
  </thead>
  <tbody id="keysTable"></tbody>
</table>

<div style="margin-top:1rem; text-align:center;">
  <button id="exportBtn">Exportar</button>
  <button id="importBtn">Importar</button>
  <input type="file" id="fileInput" style="display:none" accept="application/json">
</div>

  </div>  <script>
    const form = document.getElementById('keyForm');
    const tableBody = document.getElementById('keysTable');
    const notification = document.getElementById('notification');
    const exportBtn = document.getElementById('exportBtn');
    const importBtn = document.getElementById('importBtn');
    const fileInput = document.getElementById('fileInput');

    let keys = JSON.parse(localStorage.getItem('robloxKeys')) || [];

    function generateKey() {
      return 'RBX-' + Math.random().toString(36).substring(2, 10).toUpperCase();
    }

    function showNotification(msg, type='success') {
      notification.textContent = msg;
      notification.style.display = 'block';
      notification.style.background = type === 'success' ? 'var(--success)' : 'var(--danger)';
      setTimeout(()=> notification.style.display = 'none', 2500);
    }

    function renderKeys() {
      tableBody.innerHTML = '';
      keys.forEach((k, i) => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${k.code}</td>
          <td>${k.name}</td>
          <td>${new Date(k.expires).toLocaleString()}</td>
          <td class="actions">
            <button onclick="copyKey(${i})">Copiar</button>
            <button class="danger" onclick="deleteKey(${i})">Excluir</button>
          </td>`;
        tableBody.appendChild(tr);
      });
    }

    function saveKeys() {
      localStorage.setItem('robloxKeys', JSON.stringify(keys));
    }

    form.addEventListener('submit', e => {
      e.preventDefault();
      const name = document.getElementById('keyName').value;
      const duration = parseInt(document.getElementById('duration').value);
      const code = generateKey();
      const expires = Date.now() + duration * 60000;
      keys.push({ code, name, expires });
      saveKeys();
      renderKeys();
      showNotification('Key gerada com sucesso!');
      form.reset();
    });

    function copyKey(index) {
      navigator.clipboard.writeText(keys[index].code);
      showNotification('Key copiada!');
    }

    function deleteKey(index) {
      keys.splice(index, 1);
      saveKeys();
      renderKeys();
      showNotification('Key removida', 'danger');
    }

    exportBtn.addEventListener('click', () => {
      const blob = new Blob([JSON.stringify(keys)], { type: 'application/json' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = 'roblox_keys.json';
      a.click();
    });

    importBtn.addEventListener('click', ()=> fileInput.click());
    fileInput.addEventListener('change', e => {
      const file = e.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = ev => {
        try {
          keys = JSON.parse(ev.target.result);
          saveKeys();
          renderKeys();
          showNotification('Backup importado com sucesso!');
        } catch {
          showNotification('Erro ao importar backup', 'danger');
        }
      };
      reader.readAsText(file);
    });

    renderKeys();
  </script></body>
</html>
