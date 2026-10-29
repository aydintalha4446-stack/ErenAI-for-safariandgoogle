<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EREN AI - Autonomous Cyber Core [Enterprise Edition]</title>
    <!-- WebLLM Core Library -->
    <script type="module">
        import * as webllm from "https://esm.run/@mlc-ai/web-llm";
        window.webllm = webllm;
    </script>
    <style>
        :root {
            --bg-deep: #030712;
            --panel-bg: #0b0f19;
            --accent-blue: #38bdf8;
            --accent-green: #22c55e;
            --accent-purple: #818cf8;
            --text-main: #f3f4f6;
            --text-sub: #9ca3af;
            --border: #1f2937;
            --danger: #ef4444;
            --warning: #f59e0b;
        }
        * { box-sizing: border-box; font-family: 'Fira Code', Consolas, monospace; }
        body {
            background-color: var(--bg-deep);
            color: var(--text-main);
            margin: 0;
            padding: 16px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            width: 100%;
            max-width: 1050px;
            height: 94vh;
            background: var(--panel-bg);
            border: 1px solid var(--border);
            border-radius: 14px;
            display: flex;
            flex-direction: column;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.7);
            overflow: hidden;
        }
        .header {
            padding: 16px 24px;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #0f172a;
        }
        .brand { display: flex; align-items: center; gap: 12px; }
        .status-dot { width: 12px; height: 12px; background: var(--warning); border-radius: 50%; box-shadow: 0 0 10px var(--warning); transition: 0.3s; }
        .telemetry { display: flex; gap: 12px; font-size: 0.8rem; color: var(--text-sub); }
        .metric-badge { background: #1e293b; padding: 5px 10px; border-radius: 6px; border: 1px solid #334155; }
        
        .loading-banner {
            background: #1e293b;
            color: var(--warning);
            padding: 12px 24px;
            font-size: 0.85rem;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .chat-container {
            flex: 1;
            padding: 24px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 18px;
            background: #050811;
        }
        .message {
            max-width: 82%;
            padding: 14px 18px;
            border-radius: 10px;
            line-height: 1.6;
            font-size: 0.95rem;
            animation: fadeIn 0.3s ease-in-out;
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(6px); } to { opacity: 1; transform: translateY(0); } }
        
        .user-msg {
            align-self: flex-end;
            background: #0284c7;
            color: white;
            border-bottom-right-radius: 2px;
        }
        .eren-msg {
            align-self: flex-start;
            background: #111827;
            color: var(--text-main);
            border: 1px solid var(--border);
            border-bottom-left-radius: 2px;
        }
        
        .msg-footer {
            display: flex;
            justify-content: flex-end;
            margin-top: 8px;
            gap: 8px;
        }
        .speak-btn {
            background: transparent;
            border: 1px solid #374151;
            color: var(--accent-blue);
            padding: 2px 8px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.75rem;
        }
        .speak-btn:hover { background: #1f2937; }

        .control-bar {
            padding: 16px 24px;
            border-top: 1px solid var(--border);
            display: flex;
            gap: 12px;
            background: #0f172a;
            align-items: center;
        }
        select, input, button {
            background: #1e293b;
            border: 1px solid #334155;
            color: white;
            padding: 12px 16px;
            border-radius: 8px;
            outline: none;
            font-size: 0.95rem;
        }
        input { flex: 1; }
        input:focus { border-color: var(--accent-blue); }
        button.action-btn { background: #16a34a; font-weight: bold; cursor: pointer; transition: 0.2s; }
        button.action-btn:hover { background: #15803d; }
        button:disabled { opacity: 0.4; cursor: not-allowed; }
        a { color: var(--accent-blue); text-decoration: none; }
        a:hover { text-decoration: underline; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <div class="brand">
            <div class="status-dot" id="statusDot"></div>
            <div>
                <h3 style="margin:0; font-size: 1.1rem;">EREN AI <span style="font-size:0.65em; color:var(--accent-purple);">Enterprise Core</span></h3>
            </div>
        </div>
        <div class="telemetry">
            <div class="metric-badge">ENGINE: <span id="engineStatus" style="color:var(--warning);">Initializing...</span></div>
            <div class="metric-badge">SECURITY: BLACKHOLE</div>
        </div>
    </div>

    <div class="loading-banner" id="loadingBanner">
        <span>⚙️ WebLLM tarayıcı motoru yükleniyor. İlk açılışta model önbelleğe alınacaktır, lütfen bekleyin...</span>
        <span id="progressText" style="font-weight: bold;">0%</span>
    </div>

    <div class="chat-container" id="chatContainer">
        <div class="message eren-msg">
            <strong>[EREN AI]:</strong> Sistem protokolleri devreye alındı. BlackHole koruması aktif. Siber güvenlik, otonom sistemler ve projeler hakkında komut vermeye hazırım.
            <div class="msg-footer">
                <button class="speak-btn" onclick="speakText(this.parentElement.previousSibling.nodeValue)">🔊 Sesli Dinle</button>
            </div>
        </div>
    </div>

    <div class="control-bar">
        <select id="langSelect">
            <option value="tr">TR 🇹🇷</option>
            <option value="en">EN 🇬🇧</option>
            <option value="de">DE 🇩🇪</option>
            <option value="es">ES 🇪🇸</option>
        </select>
        <input type="text" id="userInput" placeholder="Motor yükleniyor..." disabled onkeydown="if(event.key==='Enter') handleUserSubmit()">
        <button class="action-btn" id="sendBtn" disabled onclick="handleUserSubmit()">Komut Gönder</button>
    </div>
</div>

<script>
    let mlcEngine = null;

    const SYSTEM_INSTRUCTIONS = `
You are Eren, an elite cybersecurity expert AI and autonomous systems core running via browser WebLLM.
Your responses must be concise, extremely sharp, intelligent, and technically accurate.
Core domains:
- Cybersecurity research, network automation, Django Debug Mode configuration audits.
- Autonomous robotics (Agrobot project).
- Local security isolation protocols (BlackHole Mode).
Never output personal identifiers or sensitive data. Always maintain a professional, high-tech tone. Match the user's selected language.
`;

    async function initializeEngine() {
        const progressText = document.getElementById("progressText");
        const loadingBanner = document.getElementById("loadingBanner");
        const engineStatus = document.getElementById("engineStatus");
        const userInput = document.getElementById("userInput");
        const sendBtn = document.getElementById("sendBtn");
        const statusDot = document.getElementById("statusDot");

        try {
            const modelName = "Gemma-2-2b-it-q4f32_1-MLC";
            
            mlcEngine = await window.webllm.CreateMLCEngine(modelName, {
                initProgressCallback: (report) => {
                    progressText.innerText = report.text;
                }
            });

            // Başarılı Yükleme
            loadingBanner.style.background = "#064e3b";
            loadingBanner.style.color = "#34d399";
            loadingBanner.innerHTML = "<span>🚀 Eren AI Motoru Tamamen Aktif! Doğrudan tarayıcınızda çalışıyor.</span>";
            
            engineStatus.innerText = "ONLINE (GPU)";
            engineStatus.style.color = "var(--accent-green)";
            statusDot.style.background = "var(--accent-green)";
            statusDot.style.boxShadow = "0 0 10px var(--accent-green)";

            userInput.disabled = false;
            sendBtn.disabled = false;
            userInput.placeholder = "Eren'e teknik bir komut veya soru yazın...";

        } catch (error) {
            loadingBanner.style.background = "#7f1d1d";
            loadingBanner.style.color = "#fca5a5";
            loadingBanner.innerHTML = "<span>❌ Hata: Tarayıcınız WebGPU hızlandırmasını desteklemiyor olabilir.</span>";
            engineStatus.innerText = "FAILED";
            console.error(error);
        }
    }

    async function handleUserSubmit() {
        const input = document.getElementById("userInput");
        const query = input.value.trim();
        if (!query || !mlcEngine) return;

        const container = document.getElementById("chatContainer");
        container.innerHTML += `<div class="message user-msg"><strong>[Operator]:</strong> ${escapeHtml(query)}</div>`;
        input.value = "";
        container.scrollTop = container.scrollHeight;

        const loadingId = "response_" + Date.now();
        container.innerHTML += `
            <div class="message eren-msg" id="${loadingId}">
                <strong>[EREN AI]:</strong> Analiz ediliyor...
            </div>`;
        container.scrollTop = container.scrollHeight;

        try {
            const lang = document.getElementById("langSelect").value;
            const messages = [
                { role: "system", content: SYSTEM_INSTRUCTIONS + ` Active Language: ${lang}` },
                { role: "user", content: query }
            ];

            const completion = await mlcEngine.chat.completions.create({ messages });
            const replyText = completion.choices[0].message.content;

            const targetElement = document.getElementById(loadingId);
            targetElement.innerHTML = `
                <strong>[EREN AI]:</strong> ${escapeHtml(replyText)}
                <div class="msg-footer">
                    <button class="speak-btn" onclick="speakText(this.parentElement.previousSibling.nodeValue.trim())">🔊 Sesli Dinle</button>
                </div>`;
        } catch (err) {
            document.getElementById(loadingId).innerHTML = `<strong>[EREN AI]:</strong> [Kritik Hata]: Yanıt işlenemedi.`;
        }
        container.scrollTop = container.scrollHeight;
    }

    // Web Speech API (Metni Sesli Okuma Özelliği)
    function speakText(text) {
        if (!('speechSynthesis' in window)) {
            alert("Tarayıcınız ses sentezleme özelliğini desteklemiyor.");
            return;
        }
        window.speechSynthesis.cancel(); // Önceki sesleri durdur
        const utterance = new SpeechSynthesisUtterance(text);
        const lang = document.getElementById("langSelect").value;
        
        // Dil eşleştirmesi
        const langMap = { "tr": "tr-TR", "en": "en-US", "de": "de-DE", "es": "es-ES" };
        utterance.lang = langMap[lang] || "tr-TR";
        utterance.rate = 1.0;
        
        window.speechSynthesis.speak(utterance);
    }

    function escapeHtml(text) {
        return text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    }

    window.addEventListener("DOMContentLoaded", () => {
        initializeEngine();
    });
</script>

</body>
</html>
