#!/bin/bash
# SKRYPT 6: Ostateczna naprawa struktury archive.html
# Przywraca poprawny blok {% block extra_scripts %} z działającym skryptem.

cd /opt/ai_firma_dashboard || exit 1

# Backup
cp templates/archive.html templates/archive.html.backup_before_final_fix

# Znajdź linie graniczne
START_LINE=$(grep -n "{% block extra_scripts %}" templates/archive.html | cut -d: -f1)
END_BLOCK_LINE=$(grep -n "{% endblock %}" templates/archive.html | tail -1 | cut -d: -f1)

if [ -z "$START_LINE" ] || [ -z "$END_BLOCK_LINE" ]; then
    echo "BŁĄD: Nie znaleziono bloków szablonu. Przywracam z backupu."
    cp templates/archive.html.backup_before_integration templates/archive.html
    START_LINE=$(grep -n "{% block extra_scripts %}" templates/archive.html | cut -d: -f1)
    END_BLOCK_LINE=$(grep -n "{% endblock %}" templates/archive.html | tail -1 | cut -d: -f1)
fi

# Nowa zawartość bloku extra_scripts (ZACHOWUJEMY przyciski i HTML, tylko zmieniamy skrypt)
NEW_CONTENT='{% block extra_scripts %}
<script>
    // ========== OBSŁUGA API BACKUPÓW I GOLD IMAGE ==========
    document.addEventListener("DOMContentLoaded", function() {
        const endpoints = {
            status: { url: "/api/backup/status", method: "GET", name: "Status backupów" },
            run_manual: { url: "/api/backup/run_manual", method: "POST", name: "Ręczny backup" },
            restore: { url: "/api/backup/restore", method: "POST", name: "Przywracanie" },
            configure: { url: "/api/backup/configure", method: "POST", name: "Konfiguracja" },
            manage_gold: { url: "/api/gold_image/manage", method: "POST", name: "Zarządzaj Gold Image" }
        };

        // Funkcja pokazująca wynik
        function showResult(message, type = "info") {
            alert("[" + type.toUpperCase() + "] " + message);
            console.log("[" + type + "] " + message);
        }

        // Obsługa kliknięcia przycisku
        async function handleButtonClick(action, event) {
            const endpoint = endpoints[action];
            if (!endpoint) return;
            
            const button = event.target.closest(".action-btn");
            if (!button) return;
            
            const originalText = button.innerHTML;
            button.innerHTML = "⏳ Przetwarzanie...";
            button.disabled = true;
            
            try {
                const response = await fetch(endpoint.url, { method: endpoint.method });
                const result = await response.json();
                
                if (response.ok) {
                    showResult("✅ " + result.message, "success");
                } else {
                    showResult("❌ " + (result.message || "Błąd serwera"), "error");
                }
            } catch (error) {
                showResult("⚠️ Błąd połączenia: " + error.message, "error");
            } finally {
                button.innerHTML = originalText;
                button.disabled = false;
            }
        }

        // Podpięcie przycisków
        document.querySelectorAll(".action-btn[data-action]").forEach(btn => {
            btn.addEventListener("click", (e) => handleButtonClick(btn.dataset.action, e));
        });
        document.getElementById("btnManageGoldImage")?.addEventListener("click", (e) => handleButtonClick("manage_gold", e));
    });
</script>
{% endblock %}'

# Zastąp cały blok (od START_LINE do END_BLOCK_LINE) nową zawartością
TEMP_FILE=$(mktemp)
head -n $((START_LINE - 1)) templates/archive.html > "$TEMP_FILE"
echo "$NEW_CONTENT" >> "$TEMP_FILE"
tail -n +$((END_BLOCK_LINE + 1)) templates/archive.html >> "$TEMP_FILE"

# Bezpieczna podmiana
mv "$TEMP_FILE" templates/archive.html

echo "Przywrócono poprawną strukturę archive.html z działającym skryptem."
echo "Odśwież stronę /archive i przetestuj przyciski."
