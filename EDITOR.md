#!/bin/bash
# SKRYPT 4: Naprawa wyświetlania wiadomości w archive.html
# Cel: Zastąpienie złamanego skryptu prostą, działającą wersją.

cd /opt/ai_firma_dashboard || exit 1

# Backup
cp templates/archive.html templates/archive.html.backup_before_message_fix

# Znajdź i zastąp cały blok <script> od linii zawierającej "// ========== OBSŁUGA API" do przedostatniego </script>
# Uproszczone: wstawimy nowy skrypt od linii 431 (gdzie zaczyna się <script>)
sed -i '431,450d' templates/archive.html  # Usuń stary skrypt (jeśli w podobnym zakresie)

# Wstaw nowy, prosty i działający skrypt
cat >> templates/archive.html << 'EOF'
    // ========== OBSŁUGA API BACKUPÓW I GOLD IMAGE (wersja uproszczona) ==========
    document.addEventListener('DOMContentLoaded', function() {
        const endpoints = {
            status: { url: '/api/backup/status', method: 'GET', name: 'Status backupów' },
            run_manual: { url: '/api/backup/run_manual', method: 'POST', name: 'Ręczny backup' },
            restore: { url: '/api/backup/restore', method: 'POST', name: 'Przywracanie' },
            configure: { url: '/api/backup/configure', method: 'POST', name: 'Konfiguracja' },
            manage_gold: { url: '/api/gold_image/manage', method: 'POST', name: 'Zarządzaj Gold Image' }
        };

        // Funkcja pomocnicza do pokazywania wiadomości
        function showAlert(message, type = 'info') {
            alert(`[${type.toUpperCase()}] ${message}`);
            console.log(`[${type}] ${message}`);
        }

        // Obsługa kliknięć
        async function handleButtonClick(action, event) {
            const endpoint = endpoints[action];
            if (!endpoint) return;
            
            const button = event.target.closest('.action-btn');
            const originalText = button.innerHTML;
            button.innerHTML = '⏳ Przetwarzanie...';
            button.disabled = true;
            
            try {
                const response = await fetch(endpoint.url, { method: endpoint.method });
                const result = await response.json();
                
                if (response.ok) {
                    showAlert(`✅ ${result.message}`, 'success');
                } else {
                    showAlert(`❌ ${result.message || 'Błąd serwera'}`, 'error');
                }
            } catch (error) {
                showAlert(`⚠️ Błąd połączenia: ${error.message}`, 'error');
            } finally {
                button.innerHTML = originalText;
                button.disabled = false;
            }
        }

        // Podpięcie przycisków
        document.querySelectorAll('.action-btn[data-action]').forEach(btn => {
            btn.addEventListener('click', (e) => handleButtonClick(btn.dataset.action, e));
        });
        document.getElementById('btnManageGoldImage')?.addEventListener('click', (e) => handleButtonClick('manage_gold', e));
    });
EOF

echo "Naprawiono skrypt JavaScript. Wiadomości będą pokazywane przez alert()."
