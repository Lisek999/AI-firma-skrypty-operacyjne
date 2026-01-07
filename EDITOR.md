#!/bin/bash
# SKRYPT: Tworzenie strony backup_management.html
BACKUP_HTML="/opt/ai_firma_dashboard/static/backup_management.html"

echo "Tworzenie strony zarządzania backupami..."
cat > "$BACKUP_HTML" << 'HTML_EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Firma - Zarządzanie Backupami</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { background: #f5f5f5; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 800px; margin: 0 auto; }
        header { text-align: center; margin-bottom: 30px; }
        h1 { color: #2c3e50; }
        .card { background: white; border-radius: 10px; padding: 25px; margin-bottom: 20px; box-shadow: 0 3px 10px rgba(0,0,0,0.08); }
        .btn {
            background: #3498db;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            margin: 10px 5px;
            display: inline-block;
        }
        .btn:hover { background: #2980b9; }
        .btn-danger { background: #e74c3c; }
        .btn-danger:hover { background: #c0392b; }
        .btn-success { background: #27ae60; }
        .btn-success:hover { background: #219653; }
        .btn:disabled { background: #95a5a6; cursor: not-allowed; }
        .message {
            padding: 15px;
            border-radius: 6px;
            margin: 15px 0;
            display: none;
            border-left: 4px solid #3498db;
            background: #f8f9fa;
        }
        .message.show { display: block; }
        .message.success { border-left-color: #27ae60; background: #d5f4e6; }
        .message.error { border-left-color: #e74c3c; background: #fadbd8; }
        .message.warning { border-left-color: #f39c12; background: #fef5e7; }
        .back-link { margin-top: 20px; }
        .back-link a {
            color: #3498db;
            text-decoration: none;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>🔧 Zarządzanie Backupami i Gold Image</h1>
            <p>Strona testowa i zarządzania kopiami zapasowymi</p>
        </header>

        <!-- Panel informacyjny -->
        <div class="card">
            <h2>ℹ️ Informacja</h2>
            <p>Ta strona jest <strong>środowiskiem testowym</strong> dla funkcji Gold Image i backup.</p>
            <p>Wszystkie operacje są wykonywane w izolowanym środowisku testowym (<code>/tmp/</code>).</p>
        </div>

        <!-- Panel Gold Image - TEST -->
        <div class="card">
            <h2>🛡️ Gold Image (ŚRODOWISKO TESTOWE)</h2>
            
            <div id="testGoldPanel">
                <p><strong>Status środowiska testowego:</strong> <span id="testEnvStatus">Nieskonfigurowane</span></p>
                
                <div style="margin: 20px 0;">
                    <button id="btnSetupTestEnv" class="btn">1. Skonfiguruj środowisko testowe</button>
                    <p style="font-size: 12px; color: #666; margin-top: 5px;">
                        Tworzy strukturę katalogów w /tmp/gold_image_test/
                    </p>
                </div>

                <div style="margin: 20px 0; border-top: 1px solid #eee; padding-top: 20px;">
                    <h3>Operacje testowe:</h3>
                    
                    <button id="btnUpdateGoldTest" class="btn btn-success" disabled>
                        2. Aktualizuj Gold Image (TEST)
                    </button>
                    <p style="font-size: 12px; color: #666; margin-top: 5px;">
                        Kopiuje pliki testowe do katalogu gold_image
                    </p>

                    <button id="btnRestoreGoldTest" class="btn btn-danger" disabled>
                        3. Przywróć Gold Image (TEST)
                    </button>
                    <p style="font-size: 12px; color: #666; margin-top: 5px;">
                        Uruchamia testową wersję deploy_gold_TEST.sh
                    </p>

                    <button id="btnRunFullTest" class="btn" disabled>
                        4. Przeprowadź pełny test end-to-end
                    </button>
                    <p style="font-size: 12px; color: #666; margin-top: 5px;">
                        Wykonuje wszystkie kroki automatycznie
                    </p>
                </div>

                <div id="testMessage" class="message"></div>
                
                <div id="testResults" style="display:none; margin-top: 20px; padding: 15px; background: #f8f9fa; border-radius: 6px;">
                    <h3>📊 Wyniki testu:</h3>
                    <pre id="testOutput" style="background: white; padding: 10px; border-radius: 4px; overflow: auto; max-height: 200px;"></pre>
                </div>
            </div>
        </div>

        <!-- Panel przyszłych funkcji -->
        <div class="card">
            <h2>📋 Planowane funkcje</h2>
            <ul>
                <li>Integracja z istniejącym systemem backupów</li>
                <li>Panel statusu backupów z /var/log/backup_status.json</li>
                <li>Automatyczne testy bezpieczeństwa</li>
                <li>Ręczne przywracanie z pre-backupów</li>
            </ul>
        </div>

        <div class="back-link">
            <a href="/">← Powrót do głównego dashboardu</a>
        </div>
    </div>

    <script>
        // Prosta obsługa UI dla strony testowej
        document.addEventListener('DOMContentLoaded', function() {
            const testEnvStatus = document.getElementById('testEnvStatus');
            const btnSetupTestEnv = document.getElementById('btnSetupTestEnv');
            const btnUpdateGoldTest = document.getElementById('btnUpdateGoldTest');
            const btnRestoreGoldTest = document.getElementById('btnRestoreGoldTest');
            const btnRunFullTest = document.getElementById('btnRunFullTest');
            const testMessage = document.getElementById('testMessage');
            const testResults = document.getElementById('testResults');
            const testOutput = document.getElementById('testOutput');

            // Prosta symulacja - w przyszłości podłączymy do backendu
            btnSetupTestEnv.addEventListener('click', function() {
                showMessage('Konfigurowanie środowiska testowego w /tmp/gold_image_test/...', 'info');
                testEnvStatus.textContent = 'Konfigurowane';
                testEnvStatus.style.color = '#f39c12';
                
                // Symulacja opóźnienia
                setTimeout(() => {
                    testEnvStatus.textContent = 'Gotowe';
                    testEnvStatus.style.color = '#27ae60';
                    btnUpdateGoldTest.disabled = false;
                    btnRestoreGoldTest.disabled = false;
                    btnRunFullTest.disabled = false;
                    showMessage('Środowisko testowe gotowe! Możesz teraz wykonywać operacje testowe.', 'success');
                }, 1000);
            });

            btnUpdateGoldTest.addEventListener('click', function() {
                showMessage('Uruchamianie testowej aktualizacji Gold Image...', 'info');
                // W przyszłości: wywołanie endpointu /api/gold_image/update_test
            });

            btnRestoreGoldTest.addEventListener('click', function() {
                showMessage('Uruchamianie testowego przywracania Gold Image...', 'warning');
                // W przyszłości: wywołanie endpointu /api/gold_image/restore_test
            });

            btnRunFullTest.addEventListener('click', function() {
                showMessage('Rozpoczynanie pełnego testu end-to-end...', 'info');
                testResults.style.display = 'block';
                testOutput.textContent = 'Test w toku...\nTa funkcja będzie zaimplementowana w backendzie.';
            });

            function showMessage(text, type) {
                testMessage.textContent = text;
                testMessage.className = 'message show ' + type;
                testMessage.style.display = 'block';
            }
        });
    </script>
</body>
</html>
HTML_EOF

echo "Stworzono: $BACKUP_HTML"
echo "Sprawdzanie czy plik istnieje:"
ls -la "$BACKUP_HTML"

echo "Możesz teraz otworzyć: http://twoj-vps:5000/static/backup_management.html"
