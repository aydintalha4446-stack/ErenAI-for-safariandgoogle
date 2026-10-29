<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EREN AI - Autonomous Cyber Core (WebLLM Engine)</title>
    <!-- WebLLM Kütüphanesi (Tarayıcıda Yerel AI Çalıştırmak İçin) -->
    <script type="module">
        import * as webllm from "https://esm.run/@mlc-ai/web-llm";
        window.webllm = webllm;
    </script>
    <style>
        :root {
            --bg-color: #06090f;
            --panel-bg: #0d1117;
            --accent: #58a6ff;
            --accent-green: #3fb950;
            --text-main: #c9d1d9;
            --text-sub: #8b949e;
            --border: #30363d;
            --orange: #ffa657;
        }
        * { box-sizing: border-box; font-family: 'Fira Code', 'Segoe UI', monospace; }
        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .app-container {
            width: 100%;
            max-width: 950px;
            height: 92vh;
            background: var(--panel-bg);
            border: 1px solid var(--border);
            border-radius: 12px;
            display: flex;
            flex-direction: column;
            box-shadow: 0 8px 24px rgba(0,0,0,0.8);
            overflow: hidden;
        }
        .header {
            padding: 16px 24px;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #161b22;
            flex-wrap: wrap;
            gap: 10px;
        }
        .brand { display: flex; align-items: center; gap: 12px; }
        .dot { width: 10px; height: 10px; background: #8957e5; border-radius: 50%; box-shadow: 0 0 8px #8957e5; }
        .telemetry { display: flex; gap: 15px; font-size: 0.85rem; color: var(--text-sub); }
        .metric { background: #21262d; padding: 4px 10px; border-radius: 6px; border: 1px solid var(--border); }
        
        /* Durum ve Yükleme Çubuğu */
        .status-banner {
            background: #1f242c;
            color: var(--orange);
            padding: 10px 24px;
            font-size: 0.9rem;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .chat-area {
            flex: 1;
            padding: 24px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 16px;
            background: #040d1a;
        }
        .message {
            max-width: 85%;
            padding: 14px 18px;
            border-radius: 8px;
            line-height: 1.6;
            font-size: 0.95rem;
        }
        .user-msg {
            align-self: flex-end;
            background: #1f6feb;
            color: white;
            border-bottom-right-radius: 2px;
        }
        .eren-msg {
            align-self: flex-start;
            background: #161b22;
            color: var(--text-main);
            border: 1px solid var(--border);
            border-bottom-left-radius: 2px;
        }
        .input-bar {
            padding: 16px 24px;
            border-top: 1px solid var(--border);
            display: flex;
            gap: 12px;
            background: #161b22;
        }
        select, input, button {
            background: #21262d;
            border: 1px solid var(--border);
            color: white;
            padding: 12px;
            border-radius: 6px;
            outline: none;
        }
        input { flex: 1; }
        input:focus { border-color: var(--accent); }
        button.send-btn { background: #238636; font-weight: bold; cursor: pointer; transition: 0.2s; }
        button.send-btn:hover { background: #2ea043; }
        button:disabled { opacity: 0.5; cursor: not-allowed; }
        a { color: var(--accent); text-decoration: none; }
        a:hover { text-decoration: underline; }
    </style>
</head>
<body>

<div class="app-container">
    <div class="header">
        <div class="brand">
            <div class="dot" id="statusDot"></div>
            <h3 style="margin:0;">EREN AI <span style="font-size:0.7em; color:var(--text-sub);">WebLLM Core v3.0</span></h3>
        </div>
        <div class="telemetry">
            <div class="metric">ENGINE: <span id="engineStatus" style="color:var(--orange);">Yükleniyor...</span></div>
            <div class="metric">SECURITY: BLACKHOLE</div>
        </div>
    </div>

    <div class="status-banner" id="statusBanner">
        <span>💡 Eren'in tarayıcı motoru (WebLLM) arka planda hazırlanıyor. İlk açılışta model hafızaya yüklenecektir, lütfen bekleyin...</span>
        <span id="loadProgress">0%</span>
    </div>

    <div class="chat-area" id="chatArea">
        <div class="message eren-msg">
            <strong>[EREN AI]:</strong> Sistem başlatılıyor. Ben Eren, siber güvenlik uzmanı ve otonom sistem geliştiricisiyim. Model sisteme tamamen yüklendiğinde seninle doğrudan tarayıcında yerel olarak sohbet etmeye başlayacağım!
        </div>
    </div>

    <div class="input-bar">
        <select id="langSelect">
            <option value="tr">TR 🇹🇷</option>
            <option value="en">EN 🇬🇧</option>
            <option value="de">DE 🇩🇪</option>
            <option value="es">ES 🇪🇸</option>
        </select>
        <input type="text" id="userInput" placeholder="Model yükleniyor, lütfen bekleyin..." disabled onkeydown="if(event.key==='Enter') sendMsg()">
        <button class="send-btn" id="sendBtn" disabled onclick="sendMsg()">Gönder</button>
    </div>
</div>

<script>
    let engine = null;
    
    // Eren'in Kişilik ve Bilgi Bankası (Sıfır Kişisel Veri, Tamamen Teknik & Siber Güvenlik Odaklı)
    const SYSTEM_PROMPT = `
You are Eren, an advanced cybersecurity expert AI and autonomous systems developer running directly in the browser via WebLLM.
Your responses must be short, cool, smart, and precise.
You specialize in:
- Cybersecurity research, network automation, and vulnerability analysis (e.g., Django Debug Mode audits).
- Autonomous systems and robotics (e.g., Agrobot project).
- Local core architectures and threat isolation protocols (BlackHole Mode).
Never reveal sensitive personal info. Keep answers professional, technical, and sharp. Match the user's language (Turkish by default, English, German, or Spanish).
`;

    async function initWebLLM() {
        const progressEl = document.getElementById("loadProgress");
        const statusBanner = document.getElementById("statusBanner");
        const engineStatus = document.getElementById("engineStatus");
        const userInput = document.getElementById("userInput");
        const sendBtn = document.getElementById("sendBtn");
        const statusDot = document.getElementById("statusDot");

        try {
            // Hafif ve hızlı çalışan tarayıcı içi model (Gemma-2-2b veya Llama-3 model yapılandırması)
            const selectedModel = "Gemma-2-2b-it-q4f32_1-MLC";
            
            engine = await window.webllm.CreateMLCEngine(
                selectedModel,
                {
                    initProgressCallback: (report) => {
                        progressEl.innerText = report.text;
                    }
                }
            );

            // Yükleme tamamlandı
            statusBanner.style.background = "#13231a";
            statusBanner.style.color = "var(--accent-green)";
            statusBanner.innerHTML = "<span>🚀 Eren AI Başarıyla Yüklendi ve Aktif! Tarayıcınız üzerinden yerel olarak çalışıyor.</span>";
            engineStatus.innerText = "ONLINE (WEBLLM)";
            engineStatus.style.color = "var(--accent-green)";
            statusDot.style.background = "var(--accent-green)";
            statusDot.style.boxShadow = "0 0 8px var(--accent-green)";
            
            userInput.disabled = false;
            sendBtn.disabled = false;
            userInput.placeholder = "Eren'e bir komut veya soru yazın...";

        } catch (err) {
            statusBanner.style.background = "#3b1717";
            statusBanner.style.color = "#f85149";
            statusBanner.innerHTML = "<span>❌ Model yüklenirken hata oluştu. Tarayıcınız WebGPU desteklemiyor olabilir.</span>";
            engineStatus.innerText = "ERROR";
            console.error(err);
        }
    }

    async function sendMsg() {
        const input = document.getElementById('userInput');
        const msg = input.value.trim();
        if(!msg || !engine) return;

        const chatArea = document.getElementById('chatArea');
        chatArea.innerHTML += `<div class="message user-msg"><strong>[User]:</strong> ${input.value}</div>`;
        const userQuery = input.value;
        input.value = '';
        chatArea.scrollTop = chatArea.scrollHeight;

        // Düşünüyor mesajı ekle
        const loadingId = "loading_" + Date.now();
        chatArea.innerHTML += `<div class="message eren-msg" id="${loadingId}"><strong>[EREN AI]:</strong> İşleniyor...</div>`;
        chatArea.scrollTop = chatArea.scrollHeight;

        try {
            const lang = document.getElementById('langSelect').value;
            const messages = [
                { role: "system", content: SYSTEM_PROMPT + ` Current language preference: ${lang}` },
                { role: "user", content: userQuery }
            ];

            const reply = await engine.chat.completions.create({ messages });
            const replyText = reply.choices[0].message.content;

            document.getElementById(loadingId).innerHTML = `<strong>[EREN AI]:</strong> ${replyText}`;
        } catch (err) {
            document.getElementById(loadingId).innerHTML = `<strong>[EREN AI]:</strong> [Hata]: Yanıt üretilemedi.`;
        }
        chatArea.scrollTop = chatArea.scrollHeight;
    }

    // Sayfa açıldığında WebLLM motorunu başlat
    window.addEventListener("DOMContentLoaded", () => {
        initWebLLM();
    });
</script>

</body>
</html>
