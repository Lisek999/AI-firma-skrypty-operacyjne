#!/bin/bash
# SKRYPT 3: Integracja frontendu archive.html z endpointami API
# Autor: Wojtek (AI)
# Cel: Podpięcie przycisków pod endpointy, usunięcie placeholderów.

cd /opt/ai_firma_dashboard || exit 1

# Backup szablonu
cp templates/archive.html templates/archive.html.backup_before_integration

# 1. Usunięcie klasy 'disabled' z przycisku "Zarządzaj Gold Image"
sed -i 's/<button class="action-btn disabled" title="Funkcja w przygotowaniu">/<button class="action-btn" id="btnManageGoldImage">/' templates/archive.html

# 2. Zamiana onclick na atrybuty data-action dla przycisków (zachowując oryginalny tekst)
# Przycisk 1: "Sprawdź Status Backupów" -> endpoint GET /api/backup/status
sed -i 's/onclick="showComingSoon(\x27Status backupów\x27)"/data-action="status"/' templates/archive.html
# Przycisk 2: "Uruchom Backup Ręczny" -> endpoint POST /api/backup/run_manual
sed -i 's/onclick="showComingSoon(\x27Ręczny backup\x27)"/data-action="run_manual"/' templates/archive.html
# Przycisk 3: "Przywróć z Backupu" -> endpoint POST /api/backup/restore
sed -i 's/onclick="showComingSoon(\x27Przywracanie systemu\x27)"/data-action="restore"/' templates/archive.html
# Przycisk 4: "Konfiguruj System Backupów" -> endpoint POST /api/backup/configure
sed -i 's/onclick="showComingSoon(\x27Konfiguracja\x27)"/data-action="configure"/' templates/archive.html

# 3. Zamiana całej funkcji showComingSoon na nową logikę obsługi API
# Znajdź linię z "function showComingSoon" i zastąp cały blok aż do zamknięcia </script> (uproszczone)
# Stworzenie tymczasowego pliku z nowym skryptem
cat > /tmp/new_script.js << 'EOF'
    // ========== OBSŁUGA API BACKUPÓW I GOLD IMAGE ==========
    const API_BASE = '/api';
    const endpoints = {
        status: { url: `${API_BASE}/backup/status`, method: 'GET' },
        run_manual: { url: `${API_BASE}/backup/run_manual`, method: 'POST' },
        restore: { url: `${API_BASE}/backup/restore`, method: 'POST' },
        configure: { url: `${API_BASE}/backup/configure`, method: 'POST' },
        manage_gold: { url: `${API_BASE}/gold_image/manage`, method: 'POST' }
    };

    // Główna funkcja obsługi kliknięć
    async function handleAction(action) {
        const endpoint = endpoints[action];
        if (!endpoint) {
            showMessage(`Nieznana akcja: ${action}`, 'error');
            return;
        }

        const button = event?.target.closest('.action-btn');
        if (button) button.classList.add('loading');

        try {
            const options = {
                method: endpoint.method,
                headers: { 'Content-Type': 'application/json' }
            };
            
            showMessage(`Wysyłanie żądania: ${action}...`, 'info');
            
            const response = await fetch(endpoint.url, options);
            const result = await response.json();
            
            if (response.ok) {
                showMessage(`✅ ${result.message}`, 'success');
                if (action === 'status' && result.data) {
                    console.log('Status backupów:', result.data);
                }
            } else {
                showMessage(`❌ Błąd: ${result.message || response.statusText}`, 'error');
            }
        } catch (error) {
            showMessage(`⚠️ Błąd połączenia: ${error.message}`, 'error');
            console.error('API Error:', error);
        } finally {
            if (button) button.classList.remove('loading');
        }
    }

    // Funkcja pokazująca wiadomość (zamiast alertu)
    function showMessage(text, type = 'info') {
        // Tworzymy lub znajdujemy kontener na wiadomości
        let messageContainer = document.getElementById('action-messages');
        if (!messageContainer) {
            messageContainer = document.createElement('div');
            messageContainer.id = 'action-messages';
            messageContainer.style.cssText = 'position: fixed; top: 20px; right: 20px; max-width: 400px; z-index: 1000;';
            document.body.appendChild(messageContainer);
        }
        
        const messageEl = document.createElement('div');
        messageEl.className = `message ${type}`;
        messageEl.textContent = text;
        messageEl.style.cssText = 'margin-bottom: 10px; padding: 15px; border-radius: 8px; background: var(--bg-secondary); border-left: 4px solid; color: white;';
        
        const colors = { info: '#3498db', success: '#27ae60', error: '#e74c3c', warning: '#f39c12' };
        messageEl.style.borderLeftColor = colors[type] || colors.info;
        
        messageContainer.appendChild(messageEl);
        setTimeout(() => messageEl.remove(), 5000);
    }

    // Podpięcie event listenerów po załadowaniu DOM
    document.addEventListener('DOMContentLoaded', function() {
        // Przyciski z data-action
        document.querySelectorAll('.action-btn[data-action]').forEach(btn => {
            btn.addEventListener('click', () => handleAction(btn.dataset.action));
        });
        // Specjalny przycisk Gold Image (ma id)
        const goldBtn = document.getElementById('btnManageGoldImage');
        if (goldBtn) {
            goldBtn.addEventListener('click', () => handleAction('manage_gold'));
        }
    });
EOF

# Wstawienie nowego skryptu do archive.html (zastąpienie starego bloku <script>...</script>)
# Używamy prostego podejścia: zastąp od linii 431 do końca bloku </script> (ale przed końcem bloku extra_scripts)
sed -i '431,450d' templates/archive.html  # Usuń stary skrypt (od linii 431 do 450)
sed -i '431r /tmp/new_script.js' templates/archive.html  # Wstaw nowy skrypt od linii 431

echo "Zaktualizowano szablon archive.html."
echo "Przyciski są teraz podpięte pod endpointy API."
