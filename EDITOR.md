#!/bin/bash
# ============================================================================
# SKRYPT: naprawa_app_py.sh
# CEL: Naprawa błędów wcięć w app.py po nieudanej aktualizacji
# ============================================================================

echo "=== NAPRAWA APP.PY ==="

cd /opt/ai_firma_dashboard || exit 1

# 1. Przywracamy ORYGINALNY stan funkcji get_service_status
echo "1. Przywracam funkcję get_service_status..."

# Tworzymy poprawną wersję funkcji
cat > /tmp/correct_get_service_status.py << 'EOF'
def get_service_status(service_name):
    try:
        result = subprocess.run(['systemctl', 'is-active', service_name],
                                capture_output=True, text=True, timeout=2)
        return result.stdout.strip() == 'active'
    except:
        return False
EOF

# Znajdujemy i zastępujemy uszkodzoną funkcję
# Usuwamy linie od def get_service_status do przed następną funkcją
START_LINE=$(grep -n "def get_service_status" app.py | head -1 | cut -d: -f1)
if [ -n "$START_LINE" ]; then
    # Szukamy gdzie kończy się funkcja (następna funkcja lub pusta linia)
    TOTAL_LINES=$(wc -l < app.py)
    for (( i=START_LINE+1; i<=TOTAL_LINES; i++ )); do
        LINE_CONTENT=$(sed -n "${i}p" app.py)
        if [[ "$LINE_CONTENT" =~ ^def\  ]] || [[ "$LINE_CONTENT" =~ ^@app\.route ]]; then
            END_LINE=$((i-1))
            break
        fi
        if [ $i -eq $TOTAL_LINES ]; then
            END_LINE=$TOTAL_LINES
        fi
    done
    
    # Tworzymy naprawiony plik
    head -n $((START_LINE-1)) app.py > /tmp/app_fixed.py
    cat /tmp/correct_get_service_status.py >> /tmp/app_fixed.py
    tail -n +$((END_LINE+1)) app.py >> /tmp/app_fixed.py
    
    mv /tmp/app_fixed.py app.py
    echo "✅ Przywrócono funkcję get_service_status"
else
    echo "❌ Nie znaleziono funkcji get_service_status"
    exit 1
fi

# 2. Sprawdzamy czy kontekstowy procesor jest w dobrym miejscu
echo "2. Sprawdzam kontekstowy procesor..."
CONTEXT_LINE=$(grep -n "@app.context_processor" app.py)
if [ -z "$CONTEXT_LINE" ]; then
    echo "Dodaję kontekstowy procesor w odpowiednim miejscu..."
    
    # Szukamy miejsca po definicji app
    APP_LINE=$(grep -n "app = Flask" app.py | head -1 | cut -d: -f1)
    INSERT_LINE=$((APP_LINE + 2))
    
    # Poprawny kontekstowy procesor
    cat > /tmp/correct_context_processor.py << 'EOF'
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
    
    sed -i "${INSERT_LINE}r /tmp/correct_context_processor.py" app.py
    echo "✅ Dodano kontekstowy procesor"
fi

# 3. Sprawdzamy czy funkcja get_system_status istnieje i jest poprawna
echo "3. Sprawdzam funkcję get_system_status..."
if ! grep -q "def get_system_status" app.py; then
    echo "Dodaję funkcję get_system_status..."
    
    # Szukamy gdzie wstawić (po get_service_status)
    SERVICE_LINE=$(grep -n "def get_service_status" app.py | head -1 | cut -d: -f1)
    SERVICE_END=0
    TOTAL_LINES=$(wc -l < app.py)
    
    # Szukamy końca funkcji get_service_status
    for (( i=SERVICE_LINE+1; i<=TOTAL_LINES; i++ )); do
        LINE_CONTENT=$(sed -n "${i}p" app.py)
        if [[ "$LINE_CONTENT" =~ ^[[:space:]]*$ ]]; then
            SERVICE_END=$i
            break
        fi
    done
    
    if [ $SERVICE_END -eq 0 ]; then
        SERVICE_END=$((SERVICE_LINE + 10))
    fi
    
    # Poprawna funkcja get_system_status
    cat > /tmp/correct_get_system_status.py << 'EOF'
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
EOF
    
    # Wstawiamy po funkcji get_service_status
    sed -i "${SERVICE_END}r /tmp/correct_get_system_status.py" app.py
    echo "✅ Dodano funkcję get_system_status"
fi

# 4. Weryfikacja
echo ""
echo "=== WERYFIKACJA NAPRAWY ==="
python3 -m py_compile app.py && echo "✅ Składnia Python OK" || echo "❌ Błąd składni"

echo ""
echo "=== STRUKTURA APP.PY ==="
grep -n "^def \|^@app\." app.py

echo ""
echo "=== TEST IMPORTU ==="
python3 -c "
import sys
sys.path.insert(0, '.')
try:
    import app
    print('✅ Import app.py udany')
    print('Funkcje:')
    print('  - get_service_status:', hasattr(app, 'get_service_status'))
    print('  - get_system_status:', hasattr(app, 'get_system_status'))
    print('  - inject_system_status:', hasattr(app, 'inject_system_status'))
except Exception as e:
    print('❌ Błąd importu:', e)
"

echo ""
echo "✅ NAPRAWA ZAKOŃCZONA"
