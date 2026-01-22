#!/bin/bash
# =============================================================================
# SKRYPT: dodaj_bezpieczenstwo_api.sh
# CEL: Dodanie mechanizmu autoryzacji X-API-Key do wszystkich endpointów backup
# PRIORYTET: KRYTYCZNY - bez tego system jest niezabezpieczony
# =============================================================================

set -e

echo "🔒 ROZPOCZĘCIE: Dodawanie zabezpieczeń API..."

# ----------------------------------------------------------------------------
# KROK 1: Sprawdź obecne endpointy backup
# ----------------------------------------------------------------------------
echo "📋 Analizuję istniejące endpointy backup..."
ENDPOINTS=$(grep -n "@app.route.*backup" /opt/ai_firma_dashboard/app.py)
echo "Znalezione endpointy:"
echo "$ENDPOINTS"

# ----------------------------------------------------------------------------
# KROK 2: Utwórz dekorator require_api_key
# ----------------------------------------------------------------------------
echo "🛡️ Tworzę dekorator require_api_key..."

# Sprawdź gdzie dodać dekorator (najlepiej po importach)
HEAD_LINES=$(head -40 /opt/ai_firma_dashboard/app.py | grep -n "app = Flask" | head -1 | cut -d: -f1)

sudo python3 -c "
import sys

# Odczytaj plik
with open('/opt/ai_firma_dashboard/app.py', 'r') as f:
    lines = f.readlines()

# Znajdź linię z 'app = Flask'
for i, line in enumerate(lines):
    if 'app = Flask(__name__)' in line:
        insert_pos = i + 1
        break
else:
    insert_pos = 20  # Fallback

# Dodaj dekorator po deklaracji app
decorator = '''
# =============================================================================
# BEZPIECZEŃSTWO API
# =============================================================================
API_KEYS = {
    'dashboard': 'AI_FIRMA_SECURE_KEY_2024'  # TODO: Zmień na silny klucz!
}

def require_api_key(f):
    \"\"\"Dekorator wymagający nagłówka X-API-Key\"\"\"
    from flask import request, jsonify
    from functools import wraps
    
    @wraps(f)
    def decorated_function(*args, **kwargs):
        api_key = request.headers.get('X-API-Key')
        
        if not api_key:
            return jsonify({
                'status': 'error',
                'message': 'Brak nagłówka X-API-Key'
            }), 401
        
        if api_key not in API_KEYS.values():
            return jsonify({
                'status': 'error', 
                'message': 'Nieprawidłowy klucz API'
            }), 403
        
        return f(*args, **kwargs)
    
    return decorated_function
'''

lines.insert(insert_pos, decorator)

# Zapisz plik
with open('/opt/ai_firma_dashboard/app.py', 'w') as f:
    f.writelines(lines)

print('✅ Dodano dekorator require_api_key')
"

# ----------------------------------------------------------------------------
# KROK 3: Zabezpiecz istniejące endpointy backup
# ----------------------------------------------------------------------------
echo "🔐 Zabezpieczam endpointy backup..."

# Lista endpointów do zabezpieczenia
ENDPOINT_LINES=$(grep -n "@app.route.*backup" /opt/ai_firma_dashboard/app.py | cut -d: -f1)

for line_num in $ENDPOINT_LINES; do
    echo "Sprawdzam linię $line_num..."
    
    # Sprawdź czy już ma dekorator
    sudo python3 -c "
import sys
line_num = int($line_num)

with open('/opt/ai_firma_dashboard/app.py', 'r') as f:
    lines = f.readlines()

# Sprawdź 3 linie przed endpointem
has_decorator = False
for i in range(max(0, line_num-4), line_num):
    if '@require_api_key' in lines[i]:
        has_decorator = True
        break

if not has_decorator:
    # Dodaj dekorator przed @app.route
    lines.insert(line_num-1, '@require_api_key\\n')
    
    with open('/opt/ai_firma_dashboard/app.py', 'w') as f:
        f.writelines(lines)
    
    print(f'✅ Zabezpieczono endpoint w linii {line_num}')
else:
    print(f'⚠️  Endpoint w linii {line_num} już zabezpieczony')
"
done

# ----------------------------------------------------------------------------
# KROK 4: Sprawdź i utwórz klucz API
# ----------------------------------------------------------------------------
echo "🔑 Konfiguruję klucz API..."

# Sprawdź czy klucz istnieje w zmiennych środowiskowych
if ! grep -q "API_KEY" /opt/ai_firma_dashboard/.env 2>/dev/null; then
    echo "Generuję nowy klucz API..."
    NEW_KEY=$(openssl rand -hex 32 2>/dev/null || echo "AI_FIRMA_SECURE_KEY_$(date +%s)")
    
    # Zapisz do .env
    sudo tee -a /opt/ai_firma_dashboard/.env > /dev/null << EOF
# Klucz API dla Dashboard
API_KEY=$NEW_KEY
EOF
    
    echo "✅ Wygenerowano nowy klucz API"
    echo "🔑 Twój klucz API: $NEW_KEY"
    echo "⚠️  Zapisz ten klucz! Będzie potrzebny do konfiguracji frontendu."
else
    echo "✅ Klucz API już istnieje w .env"
fi

# ----------------------------------------------------------------------------
# KROK 5: Zaktualizuj klucz w kodzie
# ----------------------------------------------------------------------------
echo "🔄 Aktualizuję klucz w kodzie..."

sudo python3 -c "
import re

with open('/opt/ai_firma_dashboard/app.py', 'r') as f:
    content = f.read()

# Odczytaj klucz z .env
try:
    with open('/opt/ai_firma_dashboard/.env', 'r') as f:
        env_content = f.read()
    import re
    match = re.search(r'API_KEY=([^\\n]+)', env_content)
    if match:
        api_key = match.group(1).strip()
    else:
        api_key = 'AI_FIRMA_SECURE_KEY_2024'
except:
    api_key = 'AI_FIRMA_SECURE_KEY_2024'

# Zaktualizuj słownik API_KEYS
new_content = re.sub(
    r\"API_KEYS = \\{.*?\\}\",
    f\"API_KEYS = {{'dashboard': '{api_key}'}}\",
    content,
    flags=re.DOTALL
)

with open('/opt/ai_firma_dashboard/app.py', 'w') as f:
    f.write(new_content)

print(f'✅ Zaktualizowano API_KEYS z kluczem: {api_key[:10]}...')
"

# ----------------------------------------------------------------------------
# KROK 6: Restart Dashboard
# ----------------------------------------------------------------------------
echo "🔄 Restartuję Dashboard..."
sudo pkill -f "gunicorn.*dashboard" 2>/dev/null || true
sleep 2
cd /opt/ai_firma_dashboard && sudo /opt/ai_firma_dashboard/venv/bin/python3 /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 1 --bind 0.0.0.0:5000 app:app --daemon
sleep 3

# ----------------------------------------------------------------------------
# KROK 7: Test zabezpieczeń
# ----------------------------------------------------------------------------
echo "🧪 Testuję zabezpieczenia..."
echo "Test 1: Bez klucza API (powinien zwrócić 401):"
curl -s -o /dev/null -w "%{http_code}" -X POST http://57.128.247.215:5000/api/backup/restore
echo ""

echo "Test 2: Z nieprawidłowym kluczem (powinien zwrócić 403):"
curl -s -o /dev/null -w "%{http_code}" -X POST http://57.128.247.215:5000/api/backup/restore -H "X-API-Key: WRONG_KEY"
echo ""

echo "🎯 Zabezpieczenia API zostały dodane!"
echo "📋 Następny krok: dodać endpoint /api/backup/trigger_manual"
