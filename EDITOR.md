#!/bin/bash
# Skrypt: Instalacja panelu Gold Image w dashboardzie AI Firma
# Cel: Dodanie nowej sekcji frontendowej "Zarządzanie Gold Image"
# Użycie: getscript gold_image_install -> ~/skrypty/gold_image_install.sh

echo "=== INSTALATOR PANELU GOLD IMAGE ==="
echo "Data: $(date)"
echo "Autor: CEO Tomek via Wojtek"
echo ""

# KROK 1: Backup obecnego dashboardu
BACKUP_FILE="/opt/ai_firma_dashboard/static/index.html.backup_$(date +%Y%m%d_%H%M%S)"
INDEX_FILE="/opt/ai_firma_dashboard/static/index.html"

echo "1. Tworzenie backupu obecnego index.html..."
if sudo cp "$INDEX_FILE" "$BACKUP_FILE"; then
    echo "   ✓ Backup utworzony: $BACKUP_FILE"
else
    echo "   ✗ Błąd przy tworzeniu backupu!"
    exit 1
fi

# KROK 2: Sprawdzenie/ustawienie klucza API w backendzie
echo "2. Konfiguracja klucza API w backendzie..."
if sudo grep -q "os.environ.get('API_KEY')" /opt/ai_firma_dashboard/app.py; then
    echo "   Sprawdzam czy klucz ma domyślną wartość..."
    if ! sudo grep -q "AI_FIRMA_GOLD_IMAGE_KEY" /opt/ai_firma_dashboard/app.py; then
        echo "   Dodaję domyślny klucz API (AI_FIRMA_GOLD_IMAGE_KEY_2024)..."
        sudo sed -i "s/os.environ.get('API_KEY')/os.environ.get('API_KEY', 'AI_FIRMA_GOLD_IMAGE_KEY_2024')/" /opt/ai_firma_dashboard/app.py
        echo "   ✓ Domyślny klucz API dodany"
    else
        echo "   ✓ Klucz API już skonfigurowany"
    fi
else
    echo "   ⚠ Nie znaleziono endpointu API_KEY w app.py"
fi

# KROK 3: Tworzenie nowego index.html z panelem Gold Image
echo "3. Tworzenie nowego dashboardu z panelem Gold Image..."

# Utwórz tymczasowy plik HTML
TEMP_HTML=$(mktemp)
cat > "$TEMP_HTML" << 'HTML_EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Firma - Status Serwera</title>
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
        
        /* Styl dla panelu Gold Image */
        .gold-panel { margin-top: 15px; }
        .gold-btn {
            background: #27ae60;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            margin: 10px 0;
            display: inline-block;
        }
        .gold-btn:hover { background: #219653; }
        .gold-btn:disabled { background: #95a5a6; cursor: not-allowed; }
        
        .gold-message {
            padding: 12px;
            border-radius: 6px;
            margin: 10px 0;
            display: none;
            border-left: 4px solid #3498db;
            background: #f8f9fa;
        }
        .gold-message.show { display: block; }
        .gold-message.success { border-left-color: #27ae60; background: #d5f4e6; }
        .gold-message.error { border-left-color: #e74c3c; background: #fadbd8; }
        .gold-message.info { border-left-color: #3498db; background: #e8f4fc; }
        
        .key-input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 6px;
            margin: 10px 0;
            font-family: monospace;
        }
        
        @media (max-width: 600px) { .service-list { grid-template-columns: 1fr; } }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>🖥️ Status Serwera AI Firma</h1>
            <p>Ostatnia aktualizacja: <span id="updateTime">-</span></p>
        </header>
        
        <div class="card">
            <h2>Zdrowie Systemu</h2>
            <div id="healthMetrics"></div>
        </div>
        
        <div class="card">
            <h2>Usługi</h2>
            <div class="service-list" id="serviceList"></div>
        </div>
        
        <div class="card">
            <h2>Zasoby</h2>
            <div id="resourceMetrics"></div>
        </div>
        
        <!-- NOWY PANEL: Zarządzanie Gold Image -->
        <div class="card">
            <h2>🛡️ Zarządzanie Gold Image</h2>
            <div class="gold-panel">
                <p><strong>Status:</strong> <span id="goldStatus">Nie uwierzytelniony</span></p>
                <p><strong>Ostatni backup:</strong> <span id="lastBackup">Brak</span></p>
                
                <div id="authSection">
                    <p><label for="apiKeyInput">Klucz API:</label></p>
                    <input type="password" id="apiKeyInput" class="key-input" placeholder="Wprowadź klucz API...">
                    <button id="saveKeyBtn" class="gold-btn">Zapisz klucz</button>
                    <p style="font-size: 12px; color: #666;">Klucz jest przechowywany tylko w Twojej przeglądarce</p>
                </div>
                
                <button id="createGoldBtn" class="gold-btn" style="display:none;">🔄 Utwórz nowy Gold Image</button>
                
                <div id="goldMessage" class="gold-message"></div>
            </div>
        </div>
    </div>

    <script>
        // Globalne zmienne
        let apiKey = localStorage.getItem('ai_firma_api_key') || '';
        let isAuthenticated = false;
        
        // Inicjalizacja
        function initGoldPanel() {
            const goldStatus = document.getElementById('goldStatus');
            const authSection = document.getElementById('authSection');
            const createBtn = document.getElementById('createGoldBtn');
            
            if (apiKey) {
                // Mamy klucz - ukryj sekcję autoryzacji, pokaż przycisk
                authSection.style.display = 'none';
                createBtn.style.display = 'inline-block';
                goldStatus.textContent = 'Gotowy';
                goldStatus.className = 'status-online';
                isAuthenticated = true;
            } else {
                // Brak klucza - pokaż sekcję autoryzacji
                authSection.style.display = 'block';
                createBtn.style.display = 'none';
                goldStatus.textContent = 'Wymaga klucza';
                goldStatus.className = 'status-offline';
            }
        }
        
        // Zapisz klucz API
        document.getElementById('saveKeyBtn').addEventListener('click', function() {
            const keyInput = document.getElementById('apiKeyInput');
            const key = keyInput.value.trim();
            
            if (!key) {
                showMessage('Proszę wprowadzić klucz API', 'error');
                return;
            }
            
            localStorage.setItem('ai_firma_api_key', key);
            apiKey = key;
            showMessage('Klucz API zapisany pomyślnie', 'success');
            initGoldPanel();
        });
        
        // Utwórz Gold Image
        document.getElementById('createGoldBtn').addEventListener('click', async function() {
            const btn = this;
            const messageDiv = document.getElementById('goldMessage');
            
            // Przygotuj UI
            btn.disabled = true;
            btn.textContent = 'Tworzenie...';
            showMessage('⌛ Rozpoczynam tworzenie Gold Image. To może potrwać do 2 minut...', 'info');
            
            try {
                const response = await fetch('/api/gold_image/create', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'X-API-Key': apiKey
                    },
                    body: JSON.stringify({ description: 'Utworzono z dashboardu' })
                });
                
                const data = await response.json();
                
                if (response.status === 401) {
                    // Nieautoryzowany - usuń klucz
                    localStorage.removeItem('ai_firma_api_key');
                    apiKey = '';
                    showMessage('❌ Nieprawidłowy klucz API. Wprowadź poprawny klucz.', 'error');
                    initGoldPanel();
                } else if (data.success) {
                    showMessage(`✅ ${data.message}${data.tag ? ' (Tag: ' + data.tag + ')' : ''}`, 'success');
                    if (data.tag) {
                        document.getElementById('lastBackup').textContent = data.tag;
                    }
                } else {
                    showMessage(`❌ ${data.message}`, 'error');
                }
            } catch (error) {
                showMessage(`❌ Błąd sieciowy: ${error.message}`, 'error');
            } finally {
                // Przywróć UI
                btn.disabled = false;
                btn.textContent = '🔄 Utwórz nowy Gold Image';
            }
        });
        
        // Funkcja pomocnicza do wyświetlania wiadomości
        function showMessage(text, type) {
            const messageDiv = document.getElementById('goldMessage');
            messageDiv.textContent = text;
            messageDiv.className = `gold-message show ${type}`;
            
            // Auto-ukrywanie po 8 sekundach (oprócz błędów)
            if (type !== 'error') {
                setTimeout(() => {
                    messageDiv.className = 'gold-message';
                }, 8000);
            }
        }
        
        // Oryginalne funkcje dashboardu (bez zmian)
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
        
        // Start aplikacji
        document.addEventListener('DOMContentLoaded', function() {
            initGoldPanel();
            loadData();
            setInterval(loadData, 10000);
            
            // Auto-focus na polu klucza jeśli puste
            if (!apiKey) {
                document.getElementById('apiKeyInput').focus();
            }
        });
    </script>
</body>
</html>
HTML_EOF

# Skopiuj nowy HTML do docelowej lokalizacji
if sudo cp "$TEMP_HTML" "$INDEX_FILE"; then
    sudo chown www-data:www-data "$INDEX_FILE"
    sudo chmod 644 "$INDEX_FILE"
    echo "   ✓ Nowy index.html został zainstalowany"
    rm -f "$TEMP_HTML"
else
    echo "   ✗ Błąd przy kopiowaniu nowego index.html!"
    rm -f "$TEMP_HTML"
    exit 1
fi

# KROK 4: Restart backendu (jeśli zmienialiśmy klucz API)
echo "4. Restart backendu Flask..."
if sudo supervisorctl restart dashboard-api > /dev/null 2>&1; then
    echo "   ✓ Backend zrestartowany pomyślnie"
    sleep 2
    echo "   Status backendu: $(sudo supervisorctl status dashboard-api | awk '{print $2}')"
else
    echo "   ⚠ Nie udało się zrestartować backendu (może już działa?)"
fi

# KROK 5: Podsumowanie
echo ""
echo "================================================"
echo "INSTALACJA ZAKOŃCZONA POMYŚLNIE! 🎉"
echo "================================================"
echo "Co zostało zrobione:"
echo "1. ✓ Backup obecnego dashboardu"
echo "2. ✓ Skonfigurowano domyślny klucz API w backendzie"
echo "3. ✓ Zainstalowano nowy dashboard z panelem Gold Image"
echo ""
echo "Następne kroki:"
echo "1. Otwórz dashboard: http://57.128.247.215"
echo "2. W panelu Gold Image wprowadź klucz API:"
echo "   Domyślny klucz: AI_FIRMA_GOLD_IMAGE_KEY_2024"
echo "3. Kliknij 'Zapisz klucz', a następnie 'Utwórz nowy Gold Image'"
echo ""
echo "Test endpointu backendu:"
echo "curl -X POST http://localhost:5000/api/verify_key \\"
echo "  -H 'Content-Type: application/json' \\"
echo "  -d '{\"api_key\":\"AI_FIRMA_GOLD_IMAGE_KEY_2024\"}'"
echo "================================================"
