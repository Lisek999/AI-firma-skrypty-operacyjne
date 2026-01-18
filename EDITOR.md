#!/bin/bash
# ============================================================================
# SKRYPT: bezposrednia_naprawa_app_py.sh
# CEL: Bezpośrednia naprawa app.py NA VPS (bez GitHub cache)
# ============================================================================

echo "=== BEZPOŚREDNIA NAPRAWA APP.PY ==="

cd /opt/ai_firma_dashboard || exit 1

# 1. Tworzymy backup obecnego bałaganu
BACKUP_NAME="app.py.chaos_before_fix_$(date +%Y%m%d_%H%M%S)"
cp app.py "$BACKUP_NAME"
echo "✅ Backup chaosu: $BACKUP_NAME"

# 2. PRZYWRACAMY Z ORYGINALNEGO BACKUPU (sprzed naszej ingerencji)
if [ -f "app.py.backup" ]; then
    echo "Przywracam z app.py.backup..."
    cp app.py.backup app.py.fixed
    echo "✅ Przywrócono czysty app.py.backup"
else
    echo "Nie ma app.py.backup, używam app.py.debug_backup..."
    cp app.py.debug_backup app.py.fixed
    echo "✅ Przywrócono z app.py.debug_backup"
fi

# 3. DODAJEMY POPRAWNE FUNKCJE DO CZYSTEGO PLIKU
echo "Dodaję poprawne funkcje..."

# Znajdujemy miejsce po funkcji get_service_status w nowym pliku
SERVICE_LINE=$(grep -n "def get_service_status" app.py.fixed | head -1 | cut -d: -f1)

if [ -z "$SERVICE_LINE" ]; then
    echo "❌ Nie znaleziono get_service_status w czystym pliku"
    exit 1
fi

# Szukamy końca funkcji get_service_status
TOTAL_LINES=$(wc -l < app.py.fixed)
SERVICE_END=0
for (( i=SERVICE_LINE+1; i<=TOTAL_LINES; i++ )); do
    LINE_CONTENT=$(sed -n "${i}p" app.py.fixed)
    if [[ "$LINE_CONTENT" =~ ^[[:space:]]*$ ]] || [[ "$LINE_CONTENT" =~ ^[^[:space:]] ]]; then
        if [[ ! "$LINE_CONTENT" =~ ^[[:space:]] ]]; then
            SERVICE_END=$((i-1))
            break
        fi
    fi
done

if [ $SERVICE_END -eq 0 ]; then
    SERVICE_END=$((SERVICE_LINE + 10))
fi

# Tworzymy poprawne funkcje do wstawienia
cat > /tmp/correct_functions.py << 'EOF'

def get_system_status():
    """Zwraca status systemu dla szablonów HTML"""
    try:
        # Sprawdzamy kluczowe usługi
        nginx_ok = get_service_status('nginx')
        supervisor_ok = get_service_status('supervisor')
        
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

# Wstawiamy poprawne funkcje
head -n $SERVICE_END app.py.fixed > /tmp/app_part1.py
cat /tmp/correct_functions.py >> /tmp/app_part1.py
tail -n +$((SERVICE_END+1)) app.py.fixed >> /tmp/app_part1.py

# 4. DODAJEMY NOWY ROUTING (zastępując stary)
echo "Dodaję nowy routing..."

# Znajdujemy starą route '/'
OLD_ROUTE_LINE=$(grep -n "@app.route('/')" /tmp/app_part1.py | head -1 | cut -d: -f1)

if [ -n "$OLD_ROUTE_LINE" ]; then
    # Tworzymy nowy routing
    cat > /tmp/new_routing.py << 'EOF'
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
    
    # Zastępujemy starą route nowym routingiem
    head -n $((OLD_ROUTE_LINE-1)) /tmp/app_part1.py > /tmp/app_final.py
    cat /tmp/new_routing.py >> /tmp/app_final.py
    
    # Pomijamy starą funkcję index
    SKIP_LINES=20
    tail -n +$((OLD_ROUTE_LINE+SKIP_LINES)) /tmp/app_part1.py >> /tmp/app_final.py
else
    echo "⚠️ Nie znaleziono starej route '/', dodaję na końcu"
    cp /tmp/app_part1.py /tmp/app_final.py
    cat /tmp/new_routing.py >> /tmp/app_final.py
fi

# 5. ZASTĘPUJEMY ORYGINALNY PLIK
mv /tmp/app_final.py app.py
echo "✅ Utworzono naprawiony app.py"

# 6. WERYFIKACJA
echo ""
echo "=== WERYFIKACJA ==="
python3 -m py_compile app.py && echo "✅ Składnia OK" || echo "❌ Błąd składni"

echo ""
echo "=== STRUKTURA ==="
grep -n "^def \|^@app\." app.py | head -20

echo ""
echo "=== TEST IMPORTU ==="
python3 -c "
import sys
sys.path.insert(0, '.')
try:
    import app
    print('✅ Import udany')
    
    # Test funkcji
    status = app.get_system_status()
    print(f'Status: {status[\"status\"]}')
    print(f'Message: {status[\"message\"]}')
    
    # Sprawdź usługi
    print(f'Nginx: {app.get_service_status(\"nginx\")}')
    print(f'Supervisor: {app.get_service_status(\"supervisor\")}')
    
except Exception as e:
    print('❌ Błąd:', e)
    import traceback
    traceback.print_exc()
"

echo ""
echo "=== NASTĘPNE KROKI ==="
echo "1. Jeśli testy OK: sudo systemctl restart ai-firma-dashboard"
echo "2. Stworzyć szablony: index.html i archive.html"
echo "3. Przetestować w przeglądarce"

echo ""
echo "✅ NAPRAWA ZAKOŃCZONA"
