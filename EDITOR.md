#!/bin/bash
# Skrypt: Nowy index.html z panelem Gold Image (uproszczona wersja)
# Użycie: Skopiuj całą zawartość tego bloku i wklej do EDITOR.md na GitHubie

cat > /tmp/simple_gold_frontend.sh << 'EOF'
#!/bin/bash
# Backend index.html
BACKUP_FILE="/opt/ai_firma_dashboard/static/index.html.backup_$(date +%Y%m%d_%H%M%S)"
INDEX_FILE="/opt/ai_firma_dashboard/static/index.html"

echo "Tworzenie backupu: $BACKUP_FILE"
sudo cp "$INDEX_FILE" "$BACKUP_FILE"

echo "Tworzenie nowego index.html..."

sudo cat > "$INDEX_FILE" << 'HTML_EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Firma - Status Serwera + Gold Image</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { background: #f5f5f5; margin: 0; padding: 15px; color: #333; }
        .container { max-width: 800px; margin: 0 auto; }
        header { text-align: center; margin-bottom: 25px; }
        h1 { color: #2c3e50; }
        h2 { color: #34495e; margin-top: 0; }
        .card { background: white; border-radius: 10px; padding: 20px; margin-bottom: 15px; box-shadow: 0 3px 10px rgba(0,0,0,0.08); }
        .metric { display: flex; justify-content: space-between; margin: 10px 0; }
        .metric .value { font-weight: bold; }
        .status-online { color: #27ae60; }
        .status-offline { color: #e74c3c; }
        .service-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 10px; }
        .service { padding: 10px; border-radius: 6px; background: #ecf0f1; text-align: center; }
        
        /* Panel Gold Image */
        .gold-panel { border-left: 4px solid #f39c12; padding-left: 15px; margin-top: 15px; }
        .btn {
            background: #3498db;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 16px;
            margin-top: 10px;
        }
        .btn:hover { background: #2980b9; }
        .btn:disabled { background: #95a5a6; cursor: not-allowed; }
        .btn-success { background: #27ae60; }
        .btn-success:hover { background: #219653; }
        
        /* Modal */
        .modal-overlay {
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.5);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }
        .modal {
            background: white;
            border-radius: 10px;
            padding: 25px;
            width: 90%;
            max-width: 500px;
        }
        .modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 20px; }
        textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 6px;
            min-height: 80px;
            margin-top: 10px;
        }
        .api-key-input {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 6px;
            font-family: monospace;
            margin-bottom: 15px;
        }
        .message {
            padding: 10px;
            border-radius: 6px;
            margin-top: 10px;
            display: none;
        }
        .message.success { background: #d5f4e6; border: 1px solid #27ae60; display: block; }
        .message.error { background: #fadbd8; border: 1px solid #e74c3c; display: block; }
        .message.info { background: #e8f4fc; border: 1px solid #3498db; display: block; }
    </style>
</head>
<body>
    <div class="container">
        <header><h1>🖥️ Status Serwera AI Firma</h1><p>Ostatnia aktualizacja: <span id="updateTime">-</span></p></header>
        
        <div class="card"><h2>Zdrowie Systemu</h2><div id="healthMetrics"></div></div>
        <div class="card"><h2>Usługi</h2><div class="service-list" id="serviceList"></div></div>
        <div class="card"><h2>Zasoby</h2><div id="resourceMetrics"></div></div>
        
        <!-- PANEL GOLD IMAGE -->
        <div class="card">
            <h2>🛡️ Zarządzanie Gold Image</h2>
            <div class="gold-panel">
                <p><strong>Status:</strong> <span id="goldStatus">Gotowy</span></p>
                <p><strong>Ostatni backup:</strong> <span id="lastBackup">Nie utworzony</span></p>
                
                <div id="authSection" style="display: none;">
                    <p>Wymagany klucz API:</p>
                    <input type="password" id="apiKey" class="api-key-input" placeholder="Wklej klucz API...">
                    <button id="saveKeyBtn" class="btn">Zapisz klucz</button>
                    <p style="font-size: 12px; color: #7f8c8d;">Klucz zapisze się w przeglądarce</p>
                </div>
                
                <button id="createBtn" class="btn btn-success">Utwórz nowy Gold Image</button>
                
                <div id="goldMessage" class="message"></div>
            </div>
        </div>
    </div>

    <!-- Modal tworzenia -->
    <div id="createModal" class="modal-overlay">
        <div class="modal">
            <h3>📸 Utwórz Gold Image</h3>
            <p>Opis (opcjonalnie):</p>
            <textarea id="description" placeholder="Co zmieniono w systemie..."></textarea>
            <div class="modal-actions">
                <button id="cancelBtn" class="btn">Anuluj</button>
                <button id="executeBtn" class="btn btn-success">Wykonaj</button>
            </div>
        </div>
    </div>

    <script>
        // Zmienne
        let apiKey = localStorage.getItem('gold_api_key') || '';
        
        // Elementy DOM
        const goldStatus = document.getElementById('goldStatus');
        const authSection = document.getElementById('authSection');
        const createBtn = document.getElementById('createBtn');
        const goldMessage = document.getElementById('goldMessage');
        const createModal = document.getElementById('createModal');
        const cancelBtn = document.getElementById('cancelBtn');
        const executeBtn = document.getElementById('executeBtn');
        const apiKeyInput = document.getElementById('apiKey');
        const saveKeyBtn = document.getElementById('saveKeyBtn');
        
        // Inicjalizacja
        function init() {
            if (!apiKey) {
                authSection.style.display = 'block';
                goldStatus.textContent = 'Wymaga klucza API';
                goldStatus.className = 'status-offline';
            } else {
                authSection.style.display = 'none';
                goldStatus.textContent = 'Gotowy (klucz zapisany)';
                goldStatus.className = 'status-online';
            }
            
            // Event listeners
            createBtn.addEventListener('click', () => {
                document.getElementById('description').value = '';
                createModal.style.display = 'flex';
            });
            
            cancelBtn.addEventListener('click', () => {
                createModal.style.display = 'none';
            });
            
            executeBtn.addEventListener('click', createGoldImage);
            
            saveKeyBtn.addEventListener('click', () => {
                const key = apiKeyInput.value.trim();
                if (key) {
                    localStorage.setItem('gold_api_key', key);
                    apiKey = key;
                    authSection.style.display = 'none';
                    goldStatus.textContent = 'Gotowy (klucz zapisany)';
                    goldStatus.className = 'status-online';
                    showMessage('Klucz API zapisany.', 'success');
                }
            });
            
            // Jeśli już mamy klucz, wypełnij pole
            if (apiKey) {
                apiKeyInput.value = '••••••••••••••••';
            }
        }
        
        // Funkcja tworzenia Gold Image
        async function createGoldImage() {
            const description = document.getElementById('description').value.trim();
            
            // Przygotuj UI
            executeBtn.disabled = true;
            executeBtn.textContent = 'Tworzenie...';
            createBtn.disabled = true;
            
            showMessage('⌛ Trwa tworzenie Gold Image... (może potrwać do 2 minut)', 'info');
            
            try {
                const response = await fetch('/api/gold_image/create', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'X-API-Key': apiKey || ''
                    },
                    body: JSON.stringify({ description })
                });
                
                const data = await response.json();
                
                if (response.status === 401) {
                    // Brak autoryzacji - pokaż sekcję z kluczem
                    showMessage('❌ Brak autoryzacji. Wprowadź klucz API powyżej.', 'error');
                    authSection.style.display = 'block';
                    goldStatus.textContent = 'Wymaga klucza API';
                    goldStatus.className = 'status-offline';
                } else if (data.success) {
                    showMessage(`✅ ${data.message}${data.tag ? ' (Tag: ' + data.tag + ')' : ''}`, 'success');
                    if (data.tag) {
                        document.getElementById('lastBackup').textContent = data.tag;
                    }
                } else {
                    showMessage(`❌ ${data.message}`, 'error');
                }
            } catch (error) {
                showMessage(`❌ Błąd: ${error.message}`, 'error');
            } finally {
                // Przywróć UI
                executeBtn.disabled = false;
                executeBtn.textContent = 'Wykonaj';
                createBtn.disabled = false;
                createModal.style.display = 'none';
            }
        }
        
        // Pomocnicza funkcja do komunikatów
        function showMessage(text, type) {
            goldMessage.textContent = text;
            goldMessage.className = `message ${type}`;
            goldMessage.style.display = 'block';
            
            // Auto-ukrywanie po 10s dla sukcesu
            if (type === 'success') {
                setTimeout(() => {
                    goldMessage.style.display = 'none';
                }, 10000);
            }
        }
        
        // Oryginalne funkcje dashboardu
        async function loadData() {
            try {
                const resp = await fetch('/api/health');
                const data = await resp.json();
                document.getElementById('updateTime').textContent = new Date().toLocaleTimeString();
                renderHealth(data);
                renderServices(data.services);
                renderResources(data.memory, data.disk, data.system);
            } catch (err) {
                document.getElementById('healthMetrics').innerHTML = `<p style="color:red;">Błąd ładowania: ${err}</p>`;
            }
        }
        
        function renderHealth(data) {
            const el = document.getElementById('healthMetrics');
            el.innerHTML = `
                <div class="metric"><span>Status:</span><span class="value status-online">${data.status.toUpperCase()}</span></div>
                <div class="metric"><span>Uptime:</span><span class="value">${data.system.uptime}</span></div>
                <div class="metric"><span>Load Avg:</span><span class="value">${data.system.load_avg.join(', ')}</span></div>
                <div class="metric"><span>CPU:</span><span class="value">${data.system.cpu_percent}%</span></div>
            `;
        }
        
        function renderServices(services) {
            const el = document.getElementById('serviceList');
            el.innerHTML = Object.entries(services).map(([name, isUp]) => `
                <div class="service">
                    <strong>${name}</strong><br>
                    <span class="${isUp ? 'status-online' : 'status-offline'}">${isUp ? '✅ Aktywna' : '❌ Nieaktywna'}</span>
                </div>
            `).join('');
        }
        
        function renderResources(mem, disk, sys) {
            const el = document.getElementById('resourceMetrics');
            el.innerHTML = `
                <div class="metric"><span>Pamięć RAM:</span><span class="value">${mem.percent_used}% (${mem.available_gb} GB wolne)</span></div>
                <div class="metric"><span>Dysk (/) :</span><span class="value">${disk.percent_used}% (${disk.free_gb} GB wolne)</span></div>
            `;
        }
        
        // Start
        document.addEventListener('DOMContentLoaded', () => {
            init();
            loadData();
            setInterval(loadData, 10000);
        });
    </script>
</body>
</html>
HTML_EOF

# Ustaw poprawne uprawnienia
sudo chown www-data:www-data "$INDEX_FILE"
sudo chmod 644 "$INDEX_FILE"

echo "✓ Nowy index.html został zainstalowany"
echo "✓ Otwórz dashboard: http://57.128.247.215"
echo "✓ Jeśli nie masz klucza API, wykonaj: echo \$API_KEY"
echo ""
echo "Czy przechodzimy do wykonania tego skryptu?"
EOF

# Nadaj uprawnienia i pokaż instrukcję
chmod +x /tmp/simple_gold_frontend.sh
echo "========================================================"
echo "GOTOWY SKRYPT DO WYKONANIA:"
echo "========================================================"
echo "1. Pobierz skrypt przez getscript:"
echo "   getscript gold_ui_simple"
echo ""
echo "2. Uruchom pobrany skrypt:"
echo "   ~/skrypty/gold_ui_simple.sh"
echo ""
echo "3. Otwórz dashboard i przetestuj:"
echo "   http://57.128.247.215"
echo ""
echo "Czy wykonujemy te kroki?"
