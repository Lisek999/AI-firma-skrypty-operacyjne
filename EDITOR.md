#!/bin/bash
# SKRYPT: CAŁKOWITA NAPRAWA DASHBOARD - nowy poprawny JavaScript
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.complete_rewrite_$(date +%Y%m%d_%H%M%S)"
TEMP_FILE="/tmp/new_dashboard_js.html"

echo "=== CAŁKOWITA NAPRAWA DASHBOARD ==="
echo "1. Backup: $BACKUP"
cp "$FILE" "$BACKUP"

echo "2. Tworzenie nowego poprawnego JavaScript..."

# Kopiujemy wszystko DO znacznika <script> (zachowujemy HTML i CSS)
head -n $(grep -n "<script>" "$FILE" | cut -d: -f1) "$FILE" > "$TEMP_FILE"

# Dodajemy NOWY, POPRAWNY JavaScript
cat >> "$TEMP_FILE" << 'JS_EOF'
    <script>
        // ==================== NOWY POPRAWNY JAVASCRIPT ====================
        // Usunięte wszystkie odwołania do nieistniejących elementów Gold Image
        // Prosty, działający dashboard
        
        async function loadData() {
            try {
                const response = await fetch('/api/health');
                if (!response.ok) throw new Error(`HTTP ${response.status}`);
                const data = await response.json();
                
                // Aktualizuj czas
                const updateTime = document.getElementById('updateTime');
                if (updateTime) updateTime.textContent = new Date().toLocaleTimeString();
                
                // Renderuj dane
                renderHealth(data);
                renderServices(data.services);
                renderResources(data.memory, data.disk, data.system);
                
            } catch (error) {
                console.error('Błąd ładowania danych:', error);
                const healthEl = document.getElementById('healthMetrics');
                if (healthEl) {
                    healthEl.innerHTML = `<p style="color:red;">Błąd ładowania: ${error.message}</p>`;
                }
            }
        }
        
        function renderHealth(data) {
            const el = document.getElementById('healthMetrics');
            if (!el) return;
            
            el.innerHTML = `
                <div class="metric">
                    <span>Status:</span>
                    <span class="value status-online">${data.status.toUpperCase()}</span>
                </div>
                <div class="metric">
                    <span>Uptime:</span>
                    <span class="value">${data.system.uptime}</span>
                </div>
                <div class="metric">
                    <span>Load Avg:</span>
                    <span class="value">${data.system.load_avg.join(', ')}</span>
                </div>
                <div class="metric">
                    <span>CPU:</span>
                    <span class="value">${data.system.cpu_percent}%</span>
                </div>
            `;
        }
        
        function renderServices(services) {
            const el = document.getElementById('serviceList');
            if (!el) return;
            
            const servicesHtml = Object.entries(services).map(([name, isUp]) => `
                <div class="service">
                    <strong>${name}</strong><br>
                    <span class="${isUp ? 'status-online' : 'status-offline'}">
                        ${isUp ? '✅ Aktywna' : '❌ Nieaktywna'}
                    </span>
                </div>
            `).join('');
            
            el.innerHTML = servicesHtml;
        }
        
        function renderResources(mem, disk, sys) {
            const el = document.getElementById('resourceMetrics');
            if (!el) return;
            
            el.innerHTML = `
                <div class="metric">
                    <span>Pamięć RAM:</span>
                    <span class="value">${mem.percent_used}% (${mem.available_gb} GB wolne)</span>
                </div>
                <div class="metric">
                    <span>Dysk (/) :</span>
                    <span class="value">${disk.percent_used}% (${disk.free_gb} GB wolne)</span>
                </div>
            `;
        }
        
        // Inicjalizacja - tylko dashboard, bez Gold Image
        function initDashboard() {
            console.log('Dashboard initialized');
            // Funkcja pusta - usunięte odwołania do nieistniejących elementów
        }
        
        // Start po załadowaniu DOM
        document.addEventListener('DOMContentLoaded', function() {
            initDashboard();
            loadData(); // Pierwsze ładowanie
            setInterval(loadData, 10000); // Co 10 sekund
        });
        // ==================== KONIEC NOWEGO JAVASCRIPT ====================
    </script>
</body>
</html>
JS_EOF

echo "3. Zastępowanie pliku..."
mv "$TEMP_FILE" "$FILE"

echo "4. Sprawdzanie poprawności..."
echo "   Czy loadData istnieje?" $(grep -q "function loadData" "$FILE" && echo "✓" || echo "✗")
echo "   Czy setInterval istnieje?" $(grep -q "setInterval(loadData" "$FILE" && echo "✓" || echo "✗")

echo "5. Restart Gunicorn..."
sudo pkill gunicorn
cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon
sleep 3

echo "6. Weryfikacja..."
echo "   Endpoint /api/health:" $(curl -s http://localhost:5000/api/health | grep -o '"status":"[^"]*"' || echo "błąd")

echo "=== NAPRAWA ZAKOŃCZONA ==="
echo "✓ Nowy, poprawny JavaScript został wgrany."
echo "✓ Sprawdź dashboard w przeglądarce."
