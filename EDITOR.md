#!/bin/bash
# SKRYPT AWARYJNY: Tworzenie minimalnego index.html dla dashboardu
# AUTOR: Wojtek (AI-Programista) | DATA: 2026-01-22
# CEL: Przywrócenie dostępności strony głównej (endpoint /) poprzez stworzenie podstawowego index.html

set -e

echo "=== Przywracam dostępność strony głównej dashboardu ==="

# 1. BACKUP obecnego stanu katalogu static (jeśli zawiera pliki)
BACKUP_DIR="/opt/ai_firma_dashboard/static_backup_$(date +%s)"
echo "1. Tworzę backup katalogu static: $BACKUP_DIR"
if [ -d "/opt/ai_firma_dashboard/static" ]; then
    mkdir -p "$BACKUP_DIR"
    cp -r /opt/ai_firma_dashboard/static/* "$BACKUP_DIR/" 2>/dev/null || true
    echo "   ✅ Backup utworzony"
else
    echo "   ℹ️  Katalog static nie istnieje, tworzę go"
    mkdir -p /opt/ai_firma_dashboard/static
fi

# 2. TWORZENIE MINIMALNEGO INDEX.HTML
echo "2. Tworzę plik index.html..."
cat > /opt/ai_firma_dashboard/static/index.html << 'EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard AI Firma - System przywrócony</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            border-radius: 10px;
            padding: 30px;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
            margin-top: 20px;
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #3498db;
            padding-bottom: 10px;
        }
        .status-badge {
            display: inline-block;
            background-color: #2ecc71;
            color: white;
            padding: 5px 15px;
            border-radius: 20px;
            font-weight: bold;
            margin-bottom: 20px;
        }
        .endpoint {
            background-color: #f8f9fa;
            border-left: 4px solid #3498db;
            padding: 15px;
            margin: 15px 0;
            border-radius: 0 5px 5px 0;
        }
        code {
            background-color: #e8f4fc;
            padding: 2px 6px;
            border-radius: 3px;
            font-family: 'Courier New', monospace;
        }
        a {
            color: #2980b9;
            text-decoration: none;
        }
        a:hover {
            text-decoration: underline;
        }
        .footer {
            margin-top: 30px;
            color: #7f8c8d;
            font-size: 0.9em;
            border-top: 1px solid #eee;
            padding-top: 15px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Dashboard AI Firma</h1>
        <div class="status-badge">SYSTEM PRZYWRÓCONY</div>
        
        <p>Backend aplikacji został pomyślnie naprawiony. API działa poprawnie.</p>
        
        <h2>📊 Status systemu</h2>
        <div class="endpoint">
            <strong>GET</strong> <code>/api/health</code> 
            <a href="/api/health" target="_blank">(otwórz)</a>
            <p>Zwraca aktualny stan serwera: użycie CPU, pamięci, dysku i status usług.</p>
        </div>
        
        <h2>🔐 Endpointy wymagające autoryzacji</h2>
        <div class="endpoint">
            <strong>GET</strong> <code>/api/backup/status</code>
            <p>Sprawdza status backupów. Wymaga nagłówka <code>X-API-Key</code>.</p>
        </div>
        
        <div class="endpoint">
            <strong>POST</strong> <code>/api/gold_image/create</code>
            <p>Tworzy nowy Gold Image. Wymaga nagłówka <code>X-API-Key</code>.</p>
        </div>
        
        <h2>🔧 Informacje techniczne</h2>
        <ul>
            <li><strong>Serwer:</strong> Flask + Gunicorn</li>
            <li><strong>Supervisor:</strong> <code>dashboard-api</code> (RUNNING)</li>
            <li><strong>Data przywrócenia:</strong> 22.01.2026</li>
            <li><strong>Uwaga:</strong> Pełny interfejs graficzny wymaga przywrócenia plików frontendowych.</li>
        </ul>
        
        <div class="footer">
            <p>System zarządzany przez <strong>AI Firma Operations</strong> | 
            <a href="/api/health">Health Check</a> | 
            <a href="http://57.128.247.215:5000/">Zewnętrzny adres</a></p>
        </div>
    </div>
</body>
</html>
EOF

# 3. USTAWIENIE UPRAWNIEŃ
echo "3. Ustawiam uprawnienia..."
chown -R ubuntu:ubuntu /opt/ai_firma_dashboard/static
chmod 644 /opt/ai_firma_dashboard/static/index.html

# 4. WERYFIKACJA
echo "4. Weryfikuję..."
if [ -f "/opt/ai_firma_dashboard/static/index.html" ]; then
    echo "   ✅ Plik index.html został utworzony"
    echo "   📊 Rozmiar pliku: $(wc -l < /opt/ai_firma_dashboard/static/index.html) linii"
else
    echo "   ❌ Błąd: plik nie został utworzony"
    exit 1
fi

echo "=== Zakończono pomyślnie ==="
echo "Backup katalogu static: $BACKUP_DIR"
echo "Strona główna powinna być dostępna pod:"
echo "   http://localhost:5000/"
echo "   http://57.128.247.215:5000/"
echo ""
echo "⚠️  Uwaga: To jest TYMCZASOWY interfejs. Pełny dashboard wymaga przywrócenia plików frontendowych."
