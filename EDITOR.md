#!/bin/bash
# SKRYPT 5: Ostateczna naprawa frontendu - podmiana skryptu w archive.html
# Dostarczany przez system EDITOR.md/getscript

cd /opt/ai_firma_dashboard || exit 1

# 1. Backup obecnego stanu (po przywróceniu)
cp templates/archive.html templates/archive.html.backup_final_fix

# 2. Nowy, prosty i pewny skrypt JavaScript
NEW_SCRIPT='    // ========== OBSŁUGA API BACKUPÓW I GOLD IMAGE ==========
    document.addEventListener(\"DOMContentLoaded\", function() {
        const endpoints = {
            status: { url: \"/api/backup/status\", method: \"GET\", name: \"Status backupów\" },
            run_manual: { url: \"/api/backup/run_manual\", method: \"POST\", name: \"Ręczny backup\" },
            restore: { url: \"/api/backup/restore\", method: \"POST\", name: \"Przywracanie\" },
            configure: { url: \"/api/backup/configure\", method: \"POST\", name: \"Konfiguracja\" },
            manage_gold: { url: \"/api/gold_image/manage\", method: \"POST\", name: \"Zarządzaj Gold Image\" }
        };

        // Funkcja pokazująca wynik
        function showResult(message, type = \"info\") {
            alert(\"[\" + type.toUpperCase() + \"] \" + message);
            console.log(\"[\" + type + \"] \" + message);
        }

        // Obsługa kliknięcia przycisku
        async function handleButtonClick(action, event) {
            const endpoint = endpoints[action];
            if (!endpoint) return;
            
            const button = event.target.closest(\".action-btn\");
            if (!button) return;
            
            const originalText = button.innerHTML;
            button.innerHTML = \"⏳ Przetwarzanie...\";
            button.disabled = true;
            
            try {
                const response = await fetch(endpoint.url, { method: endpoint.method });
                const result = await response.json();
                
                if (response.ok) {
                    showResult(\"✅ \" + result.message, \"success\");
                } else {
                    showResult(\"❌ \" + (result.message || \"Błąd serwera\"), \"error\");
                }
            } catch (error) {
                showResult(\"⚠️ Błąd połączenia: \" + error.message, \"error\");
            } finally {
                button.innerHTML = originalText;
                button.disabled = false;
            }
        }

        // Podpięcie przycisków
        document.querySelectorAll(\".action-btn[data-action]\").forEach(btn => {
            btn.addEventListener(\"click\", (e) => handleButtonClick(btn.dataset.action, e));
        });
        document.getElementById(\"btnManageGoldImage\")?.addEventListener(\"click\", (e) => handleButtonClick(\"manage_gold\", e));
    });'

# 3. Znajdź numer linii początku bloku <script> i końca
START_LINE=$(grep -n "<script>" templates/archive.html | head -1 | cut -d: -f1)
END_LINE=$(grep -n "</script>" templates/archive.html | head -1 | cut -d: -f1)

if [ -n "$START_LINE" ] && [ -n "$END_LINE" ]; then
    # Usuń stary skrypt (zachowując linie <script> i </script>)
    sed -i "$((START_LINE+1)),$((END_LINE-1))d" templates/archive.html
    # Wstaw nowy skrypt
    echo "$NEW_SCRIPT" | sed -i "${START_LINE}r /dev/stdin" templates/archive.html
    echo "Skrypt JavaScript został podmieniony na działającą wersję."
else
    echo "BŁĄD: Nie znaleziono tagów <script> w pliku."
    exit 1
fi
