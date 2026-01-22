#!/bin/bash
# Naprawa błędnego dodania dekoratorów @require_api_key

echo "🔧 Naprawiam błędne dekoratory..."

sudo python3 -c "
with open('/opt/ai_firma_dashboard/app.py', 'r') as f:
    lines = f.readlines()

# Usuń błędne dekoratory w środku funkcji
new_lines = []
i = 0
while i < len(lines):
    line = lines[i]
    
    # Jeśli linia zawiera @require_api_key i następna linia to '}), 202'
    # to usuwamy @require_api_key (jest w złym miejscu)
    if '@require_api_key' in line and i+1 < len(lines) and '}), 202' in lines[i+1]:
        print(f'⚠️  Usuwam błędny dekorator w linii {i+1}')
        i += 1  # Pomijamy tę linię
        continue
    
    new_lines.append(line)
    i += 1

# Zapisz poprawiony plik
with open('/opt/ai_firma_dashboard/app.py', 'w') as f:
    f.writelines(new_lines)

print('✅ Usunięto błędne dekoratory')
"

# Teraz dodaj dekoratory we właściwych miejscach
echo "🔄 Dodaję dekoratory we właściwych miejscach..."

sudo python3 -c "
with open('/opt/ai_firma_dashboard/app.py', 'r') as f:
    lines = f.readlines()

# Lista endpointów do zabezpieczenia
endpoints = [
    ('def backup_status():', 'GET'),
    ('def backup_restore():', 'POST'), 
    ('def backup_configure():', 'POST')
]

new_lines = []
for i, line in enumerate(lines):
    new_lines.append(line)
    
    # Sprawdź czy to początek funkcji backup
    for func_def, method in endpoints:
        if func_def in line:
            print(f'Znaleziono {func_def} w linii {i+1}')
            
            # Sprawdź czy już ma dekorator (szukaj 3 linie wcześniej)
            has_decorator = False
            for j in range(max(0, i-3), i):
                if '@require_api_key' in lines[j]:
                    has_decorator = True
                    break
            
            if not has_decorator:
                # Dodaj dekorator przed definicją funkcji
                new_lines.pop()  # Usuń ostatnio dodaną linię
                new_lines.append('@require_api_key\\n')
                new_lines.append(line)
                print(f'✅ Dodano @require_api_key przed {func_def}')

with open('/opt/ai_firma_dashboard/app.py', 'w') as f:
    f.writelines(new_lines)
"

# Restart Dashboard
echo "🔄 Restartuję Dashboard..."
sudo pkill -f "gunicorn.*dashboard" 2>/dev/null || true
sleep 2
cd /opt/ai_firma_dashboard && sudo /opt/ai_firma_dashboard/venv/bin/python3 /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 1 --bind 0.0.0.0:5000 app:app --daemon
sleep 3

# Test
echo "🧪 Testuję poprawiony endpoint..."
curl -s -o /dev/null -w "Status: %{http_code}\n" -X POST http://57.128.247.215:5000/api/backup/restore -H "X-API-Key: WRONG_KEY"
echo "Z nieprawidłowym kluczem powinno być 403"
