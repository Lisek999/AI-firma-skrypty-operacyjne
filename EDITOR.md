#!/bin/bash
# ============================================================================
# SKRYPT: aktualizacja_app_py.sh
# CEL: Aktualizacja app.py o system statusu i nowy routing
# UWAGA: To jest INCREMENTALNA zmiana - dodajemy do istniejącego pliku
# ============================================================================

echo "=== AKTUALIZUJĘ APP.PY ==="

cd /opt/ai_firma_dashboard || exit 1

# Tworzymy backup przed zmianą
BACKUP_NAME="app.py.backup_before_system_status_$(date +%Y%m%d_%H%M%S)"
cp app.py "$BACKUP_NAME"
echo "✅ Utworzono backup: $BACKUP_NAME"

# Dodajemy funkcję get_system_status() po istniejącej get_service_status()
# Szukamy linii z definicją get_service_status
LINE_NUMBER=$(grep -n "def get_service_status" app.py | head -1 | cut -d: -f1)

if [ -z "$LINE_NUMBER" ]; then
    echo "❌ Nie znaleziono funkcji get_service_status"
    exit 1
fi

# Obliczamy gdzie wstawić nową funkcję (po get_service_status)
INSERT_LINE=$((LINE_NUMBER + 7))  # get_service_status ma około 7 linii

# Tworzymy tymczasowy plik z nową funkcją
cat > /tmp/system_status_function.py << 'EOF'
def get_system_status():
    """Zwraca status systemu dla szablonów HTML"""
    try:
        # Sprawdzamy kluczowe usługi
        nginx_ok = get_service_status('nginx')
        supervisor_ok = get_service_status('supervisor')
        
        # Można dodać więcej checków:
        # - Czy backup_status.json istnieje i ma ostatni status OK?
        # - Czy dysk nie jest pełny?
        # - Czy jest połączenie z internetem?
        
        if nginx_ok and supervisor_ok:
            return {
                'status': 'ok',
                'message': 'Systemy sprawne',
                'details': {
                    'nginx': 'działa',
                    'supervisor': 'działa',
                    'timestamp': time.time()
                }
            }
        else:
            error_msg = []
            if not nginx_ok:
                error_msg.append("nginx")
            if not supervisor_ok:
                error_msg.append("supervisor")
            
            return {
                'status': 'error',
                'message': f'Awaria: {", ".join(error_msg)}',
                'details': {
                    'nginx': 'awaria' if not nginx_ok else 'działa',
                    'supervisor': 'awaria' if not supervisor_ok else 'działa',
                    'timestamp': time.time()
                }
            }
    except Exception as e:
        return {
            'status': 'error',
            'message': f'Błąd sprawdzania statusu: {str(e)}',
            'details': {'error': str(e)}
        }
EOF

# Wstawiamy nową funkcję do app.py
sed -i "${INSERT_LINE}r /tmp/system_status_function.py" app.py
echo "✅ Dodano funkcję get_system_status()"

# Dodajemy kontekstowy procesor - szukamy miejsca po definicji app
APP_LINE=$(grep -n "app = Flask" app.py | head -1 | cut -d: -f1)
CONTEXT_LINE=$((APP_LINE + 2))

# Tworzymy kontekstowy procesor
cat > /tmp/context_processor.py << 'EOF'
# Kontekstowy procesor - status systemu dostępny we wszystkich szablonach
@app.context_processor
def inject_system_status():
    """Wstrzykuje status systemu do wszystkich szablonów"""
    status_data = get_system_status()
    return {
        'status': status_data['status'],
        'status_message': status_data['message']
    }
EOF

# Wstawiamy kontekstowy procesor
sed -i "${CONTEXT_LINE}r /tmp/context_processor.py" app.py
echo "✅ Dodano kontekstowy procesor"

# Aktualizujemy istniejącą route '/' aby używała szablonu
echo ""
echo "=== AKTUALIZUJĘ ROUTING =="

# Tworzymy nową wersję funkcji głównej
cat > /tmp/main_route.py << 'EOF'
from flask import render_template

@app.route('/')
def index():
    """Główna strona z kafelkami"""
    return render_template('index.html')

@app.route('/archive')
def archive():
    """Strona archiwum/backupów"""
    return render_template('archive.html')

@app.route('/terminal')
def terminal():
    """Terminal serwera"""
    return render_template('terminal.html')

@app.route('/explorer')
def explorer():
    """Eksplorator plików"""
    return render_template('explorer.html')

@app.route('/config')
def config():
    """Konfiguracja systemu"""
    return render_template('config.html')

@app.route('/pulse')
def pulse():
    """Puls systemu - monitoring"""
    return render_template('pulse.html')

@app.route('/tools')
def tools():
    """Narzędzia systemowe"""
    return render_template('tools.html')
EOF

# Znajdujemy i zastępujemy starą route '/'
START_LINE=$(grep -n "@app.route('/')" app.py | head -1 | cut -d: -f1)
if [ -n "$START_LINE" ]; then
    END_LINE=$((START_LINE + 10))  # Szukamy końca funkcji
    # Tworzymy nowy plik bez starej route
    head -n $((START_LINE - 1)) app.py > /tmp/app_part1.py
    cat /tmp/main_route.py >> /tmp/app_part1.py
    tail -n +$((END_LINE + 1)) app.py >> /tmp/app_part1.py
    
    # Zastępujemy oryginalny plik
    mv /tmp/app_part1.py app.py
    echo "✅ Zaktualizowano routing główny"
else
    echo "⚠️ Nie znaleziono istniejącej route '/', dodaję nowe"
    echo "" >> app.py
    cat /tmp/main_route.py >> app.py
    echo "✅ Dodano nowy routing"
fi

# Weryfikacja
echo ""
echo "=== WERYFIKACJA ZMIAN ==="
echo "Nowe funkcje:"
grep -n "def get_system_status\|def inject_system_status" app.py
echo ""
echo "Nowy routing:"
grep -n "@app.route" app.py
echo ""
echo "=== IMPORT RENDER_TEMPLATE ==="
grep -n "from flask import" app.py | head -5
echo ""
echo "✅ AKTUALIZACJA APP.PY ZAKOŃCZONA"
echo ""
echo "=== NASTĘPNE KROKI ==="
echo "1. Sprawdź czy app.py się uruchamia: python3 -c 'import app'"
echo "2. Zrestartuj usługę: sudo systemctl restart ai-firma-dashboard"
echo "3. Stwórz szablony index.html i archive.html"
