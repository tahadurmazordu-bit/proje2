<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <title>Yapay Zeka Web Sitesi</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    :root {
      --bg: #f4f4f4;
      --text: #222;
      --card: #ffffff;
      --primary: #4f46e5;
    }
    body.dark {
      --bg: #0f172a;
      --text: #e5e7eb;
      --card: #1e293b;
      --primary: #6366f1;
    }
    body {
      margin: 0;
      font-family: Arial, Helvetica, sans-serif;
      background: var(--bg);
      color: var(--text);
    }
    .container {
      max-width: 420px;
      margin: 60px auto;
      background: var(--card);
      padding: 24px;
      border-radius: 12px;
      box-shadow: 0 10px 25px rgba(0,0,0,.1);
    }
    h2 { text-align: center; }
    input, button {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      border-radius: 8px;
      border: 1px solid #ccc;
    }
    button {
      background: var(--primary);
      color: white;
      border: none;
      cursor: pointer;
    }
    button.secondary {
      background: transparent;
      color: var(--primary);
    }
    .link {
      text-align: center;
      margin-top: 10px;
      cursor: pointer;
      color: var(--primary);
    }
    .hidden { display: none; }
    .topbar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
    }
    .chat {
      background: var(--bg);
      padding: 12px;
      border-radius: 8px;
      min-height: 120px;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>

  <!-- GİRİŞ -->
  <div class="container" id="loginBox">
    <h2>Giriş Yap</h2>
    <input id="loginUser" placeholder="Kullanıcı Adı" />
    <input id="loginPass" type="password" placeholder="Şifre" />
    <button onclick="login()">Giriş Yap</button>
    <div class="link" onclick="showRegister()">Kayıt Ol</div>
  </div>

  <!-- KAYIT -->
  <div class="container hidden" id="registerBox">
    <h2>Kayıt Ol</h2>
    <input id="regUser" placeholder="Kullanıcı Adı" />
    <input id="regPass" type="password" placeholder="Şifre" />
    <button onclick="register()">Kayıt Ol</button>
    <div class="link" onclick="showLogin()">Girişe Dön</div>
  </div>

  <!-- PANEL -->
  <div class="container hidden" id="panelBox">
    <div class="topbar">
      <strong id="welcome"></strong>
      <button class="secondary" onclick="openSettings()">⚙️ Ayarlar</button>
    </div>

    <h3>🧠 Emergent Tarzı Yapay Zeka</h3>
    <p><b>Mod:</b> AI siteyi kendi kursun (tam otomatik)</p>

    <div class="chat" id="messages"></div>

    <!-- AYARLAR -->
    <div id="settings" class="hidden">
      <h4>⚙️ Ayarlar</h4>
      <button onclick="toggleTheme()">🌙 Tema Değiştir</button>
      <button onclick="closeSettings()">Kapat</button>
    </div>

    <input id="aiInput" placeholder="Görev veya soru yaz (agent kullanılır)..." />
    <button onclick="askAI()">Gönder</button>
    <button onclick="autoBuildSite()" style="margin-top:10px">🌐 Siteyi AI Kursun</button>
    <button onclick="autoBuildSite()" style="margin-top:10px">🌐 Siteyi Otomatik Kur</button>
    <button onclick="autoBuildApp()" style="margin-top:10px">📱 Uygulamayı Otomatik Kur</button>

    <hr />

    <h4>🤖 Agent Seçimi</h4>
    <select id="agent">
      <option value="research">Araştırmacı Agent</option>
      <option value="coder">Yazılımcı Agent</option>
      <option value="planner">Planlayıcı Agent</option>
    </select>

    <h4>📋 Görevler</h4>
    <ul id="tasks"></ul>

    <input id="newTask" placeholder="Yeni görev ekle" />
    <button onclick="addTask()">Görev Ekle</button>

    <hr />

    <input id="newPass" type="password" placeholder="Yeni Şifre" />
    <button onclick="changePassword()">Şifre Değiştir</button>

    <button onclick="logout()" style="margin-top:15px">Çıkış Yap</button>
  </div>

<script>
  /* ==== KULLANICI SİSTEMİ (DÜZELTİLDİ) ==== */
  const users = JSON.parse(localStorage.getItem('users') || '{}');
  let currentUser = null;

  function saveUsers() {
    localStorage.setItem('users', JSON.stringify(users));
  }

  function showRegister() {
    loginBox.classList.add('hidden');
    registerBox.classList.remove('hidden');
  }

  function showLogin() {
    registerBox.classList.add('hidden');
    loginBox.classList.remove('hidden');
  }

  function register() {
    const u = regUser.value.trim();
    const p = regPass.value;
    if (!u || !p) return alert('Bilgiler eksik');
    if (users[u]) return alert('Bu kullanıcı zaten var');
    users[u] = p;
    saveUsers();
    alert('Kayıt başarılı');
    showLogin();
  }

  function login() {
    const u = loginUser.value.trim();
    const p = loginPass.value;

    if (!users[u]) {
      alert('❌ Kullanıcı bulunamadı');
      return;
    }
    if (users[u] !== p) {
      alert('❌ Şifre hatalı');
      return;
    }

    currentUser = u;
    loginUser.value = '';
    loginPass.value = '';
    loginBox.classList.add('hidden');
    panelBox.classList.remove('hidden');
    welcome.innerText = 'Hoşgeldin ' + u;
  }

  function logout() {
    currentUser = null;
    panelBox.classList.add('hidden');
    loginBox.classList.remove('hidden');
  }

  function changePassword() {
    const np = newPass.value;
    if (!np) return alert('Yeni şifre gir');
    users[currentUser] = np;
    saveUsers();
    alert('Şifre değiştirildi');
    newPass.value = '';
  }

  /* ==== EMERGENT TARZI AI ==== */
  function askAI() {
    const q = aiInput.value.trim();
    const ag = agent.value;
    if (!q) return;

    messages.innerHTML += `<div><b>👤 Sen:</b> ${q}</div>`;

    // Normal sohbet mi kontrol et
    if (!q.toLowerCase().includes('site') && !q.toLowerCase().includes('uygulama')) {
      messages.innerHTML += `<div><b>🤖 AI:</b> ${normalChatResponse(q)}</div>`;
      aiInput.value = '';
      return;
    }

    messages.innerHTML += `<div><b>🤖 ${ag} agent:</b> Görev algılandı. Otomatik işlem yapabilirim.</div>`;
    aiInput.value = '';
  }

  function addTask() {
    const t = newTask.value.trim();
    if (!t) return;
    const li = document.createElement('li');
    li.textContent = t;
    tasks.appendChild(li);
    newTask.value = '';
  }

  /* ==== GERÇEK AKSİYON: DOSYA OLUŞTURMA ==== */
  function downloadFile(filename, content) {
    const blob = new Blob([content], { type: 'text/plain' });
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = filename;
    a.click();
    URL.revokeObjectURL(a.href);
  }

  function autoBuildSite() {
    const prompt = aiInput.value || 'Basit tanıtım sitesi';
    const html = `<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<title>AI Tarafından Kurulan Site</title>
<style>
body{font-family:Arial;background:#f4f4f4;margin:0;padding:0}
header{background:#4f46e5;color:#fff;padding:20px;text-align:center}
section{padding:20px}
.card{background:#fff;padding:15px;border-radius:10px;margin-bottom:10px}
footer{text-align:center;padding:10px;color:#666}
</style>
</head>
<body>
<header>
<h1>AI Web Sitesi</h1>
<p>${prompt}</p>
</header>
<section>
<div class="card">Bu site yapay zeka tarafından otomatik oluşturuldu.</div>
<div class="card">Kullanıcı: ${currentUser}</div>
</section>
<footer>© AI System</footer>
</body>
</html>`;
    downloadFile('ai-website.html', html);
    messages.innerHTML += `<div><b>🌐 Sistem:</b> Site kuruldu ve indirildi.</div>`;
}

  function autoBuildApp() {
    const prompt = aiInput.value || 'Basit mobil uygulama';
    const app = `import React from 'react';
import { Text, View } from 'react-native';

export default function App() {
  return (
    <View style={{flex:1,alignItems:'center',justifyContent:'center'}}>
      <Text>AI Mobil Uygulama</Text>
      <Text>${prompt}</Text>
    </View>
  );
}`;
    downloadFile('AIApp.jsx', app);
    messages.innerHTML += `<div><b>📱 Sistem:</b> Uygulama kuruldu ve indirildi.</div>`;
}

  /* ==== TEMA ==== */
  function autoBuildSite() {
    messages.innerHTML += `<div><b>🤖 Planner:</b> Site yapısı planlanıyor...</div>`;
    setTimeout(() => {
      messages.innerHTML += `<div><b>🤖 Researcher:</b> Gerekli sayfalar belirlendi (Ana sayfa, Hakkında, İletişim)</div>`;
    }, 800);
    setTimeout(() => {
      messages.innerHTML += `<div><b>🤖 Coder:</b> HTML/CSS oluşturuluyor...</div>`;
      const html = `<!DOCTYPE html>
<html lang='tr'>
<head>
<meta charset='UTF-8'>
<title>AI Tarafından Oluşturulan Site</title>
<style>
body{font-family:Arial;background:#f4f4f4;padding:40px}
.card{background:white;padding:20px;border-radius:12px}
</style>
</head>
<body>
<div class='card'>
<h1>🚀 AI Tarafından Yapıldı</h1>
<p>Kullanıcı: ${currentUser}</p>
<p>Bu site Emergent-style AI tarafından otomatik oluşturuldu.</p>
</div>
</body>
</html>`;
      downloadFile('ai-site.html', html);
      messages.innerHTML += `<div><b>✅ Sistem:</b> Site oluşturuldu ve indirildi.</div>`;
    }, 1600);
  }

  function openSettings() {
    settings.classList.remove('hidden');
  }

  function closeSettings() {
    settings.classList.add('hidden');
  }

  function normalChatResponse(q) {
    return "Anladım. " + q + " hakkında sana yardımcı olmaya çalışıyorum.";
  }

  function toggleTheme() {
    document.body.classList.toggle('dark');
  }
</script>
</body>
</html>
