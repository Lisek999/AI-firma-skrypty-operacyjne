#!/bin/bash
# Skrypt: Dodanie endpointu Gold Image do app.py
# Użycie: Skopiuj całą zawartość tego bloku i wklej do EDITOR.md na GitHubie

cat > /tmp/add_gold_image_endpoint.py << 'EOF'
"""
Skrypt modyfikacji app.py - dodanie endpointu /api/gold_image/create
Uruchom: python3 /tmp/add_gold_image_endpoint.py
"""

import re
import sys
import os

def find_api_section(content):
    """Znajduje sekcję z endpointami API w app.py"""
    lines = content.split('\n')
    
    # Szukamy ostatniego endpointu przed blokiem if __name__ == '__main__'
    for i, line in enumerate(lines):
        if line.strip().startswith('@app.route("/api/'):
            api_start = i
        if line.strip() == "if __name__ == '__main__':":
            return api_start, i
    
    return None, None

def insert_endpoint(content):
    """Wstawia nowy endpoint do app.py"""
    
    # Szukamy gdzie wstawić nowy endpoint
    api_start, api_end = find_api_section(content)
    
    if api_start is None:
        print("ERROR: Nie znaleziono sekcji API w pliku app.py")
        return None
    
    # Nowy endpoint do wstawienia
    new_endpoint = '''
@app.route('/api/gold_image/create', methods=['POST'])
def create_gold_image_endpoint():
    """Endpoint do tworzenia Gold Image poprzez interfejs dashboardu"""
    # Weryfikacja nagłówka API Key
    api_key = request.headers.get('X-API-Key')
    if not api_key or api_key != os.environ.get('API_KEY'):
        return jsonify({
            "success": False,
            "message": "Brak autoryzacji. Nieprawidłowy lub brakujący token API.",
            "tag": None,
            "output": ""
        }), 401
    
    # Pobranie opisu z formularza (tylko do logowania)
    data = request.get_json()
    description = data.get('description', '') if data else ''
    
    # Logowanie żądania
    print(f"[GOLD IMAGE] Żądanie utworzenia backupu. Opis: {description[:100]}")
    
    # Ścieżka do skryptu (BEZWZGLĘDNA)
    script_path = "/home/ubuntu/ai_firma_dokumenty/skrypty_operacyjne/create_gold_image.sh"
    
    # Sprawdzenie czy skrypt istnieje
    if not os.path.exists(script_path):
        return jsonify({
            "success": False,
            "message": "Skrypt create_gold_image.sh nie został znaleziony.",
            "tag": None,
            "output": f"Ścieżka: {script_path}"
        }), 500
    
    try:
        # Bezpieczne wywołanie skryptu - NIE przekazujemy danych użytkownika jako argumentów
        import subprocess
        import shlex
        
        # Użycie Popen z przechwytywaniem outputu
        process = subprocess.Popen(
            ['bash', script_path],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True,
            cwd=os.path.dirname(script_path)  # Uruchom w katalogu skryptu
        )
        
        stdout, stderr = process.communicate(timeout=120)  # Timeout 2 minuty
        return_code = process.returncode
        
        # Łączenie outputu
        full_output = stdout + "\n" + stderr if stderr else stdout
        
        # Parsowanie tagu z outputu (szukamy wzorca tagu Git)
        import re
        tag_match = re.search(r'tag:\s*(v\d+\.\d+[\w\.-]*)', full_output, re.IGNORECASE)
        tag = tag_match.group(1) if tag_match else None
        
        if return_code == 0:
            return jsonify({
                "success": True,
                "message": f"Gold Image utworzony pomyślnie{' (tag: ' + tag + ')' if tag else ''}",
                "tag": tag,
                "output": full_output[-2000:]  # Ostatnie 2000 znaków dla bezpieczeństwa
            }), 200
        else:
            return jsonify({
                "success": False,
                "message": "Błąd podczas wykonywania skryptu tworzenia Gold Image.",
                "tag": tag,
                "output": full_output[-2000:]  # Ostatnie 2000 znaków
            }), 500
            
    except subprocess.TimeoutExpired:
        return jsonify({
            "success": False,
            "message": "Timeout: Skrypt wykonuje się zbyt długo (ponad 2 minuty).",
            "tag": None,
            "output": "Proces został zabity z powodu przekroczenia czasu."
        }), 500
    except Exception as e:
        return jsonify({
            "success": False,
            "message": f"Nieoczekiwany błąd: {str(e)}",
            "tag": None,
            "output": ""
        }), 500
'''
    
    # Wstawiamy nowy endpoint przed ostatnim znalezionym endpointem API
    lines = content.split('\n')
    
    # Szukamy ostatniego @app.route przed if __name__
    insert_position = api_end
    for i in range(api_end-1, api_start-1, -1):
        if '@app.route' in lines[i]:
            insert_position = i + 1  # Wstawiamy po tym endpointie
            break
    
    # Wstawiamy nowy endpoint
    lines.insert(insert_position, new_endpoint)
    
    return '\n'.join(lines)

def main():
    # Ścieżka do app.py (dostosuj jeśli potrzebne)
    app_py_path = "/home/ubuntu/ai_firma_dashboard/app.py"
    
    if not os.path.exists(app_py_path):
        print(f"ERROR: Plik {app_py_path} nie istnieje.")
        print("Podaj poprawną ścieżkę do app.py:")
        app_py_path = input().strip()
    
    # Wczytanie aktualnej zawartości
    with open(app_py_path, 'r') as f:
        content = f.read()
    
    # Wstawienie nowego endpointu
    new_content = insert_endpoint(content)
    
    if new_content:
        # Backup oryginalnego pliku
        backup_path = app_py_path + '.backup_' + datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
        import shutil
        shutil.copy2(app_py_path, backup_path)
        print(f"Utworzono backup: {backup_path}")
        
        # Zapisanie zmodyfikowanego pliku
        with open(app_py_path, 'w') as f:
            f.write(new_content)
        
        print("✓ Endpoint /api/gold_image/create został dodany do app.py")
        print("✓ Restart serwisu Flask wymagany: sudo supervisorctl restart ai-firma-dashboard")
    else:
        print("✗ Nie udało się dodać endpointu")

if __name__ == '__main__':
    import datetime
    main()
EOF

echo "Skrypt został zapisany jako /tmp/add_gold_image_endpoint.py"
echo ""
echo "KROKI DO WYKONANIA:"
echo "1. Uruchom skrypt: python3 /tmp/add_gold_image_endpoint.py"
echo "2. Jeśli pojawi się błąd 'Nie znaleziono sekcji API', podręczna ścieżka do app.py"
echo "3. Po sukcesie: sudo supervisorctl restart ai-firma-dashboard"
echo "4. Sprawdź logi: sudo supervisorctl tail -f ai-firma-dashboard"
echo ""
echo "Czy przechodzimy do wykonania tych kroków?"
