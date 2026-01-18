#!/bin/bash
# ============================================================================
# SKRYPT: aktualizacja_endpoint_gold_image.sh
# CEL: Minimalna zmiana - podpięcie gold_image_manage() pod rzeczywisty skrypt
# DATA: $(date)
# ============================================================================

APP_PATH="/opt/ai_firma_dashboard/app.py"
BACKUP_PATH="/opt/ai_firma_dashboard/app.py.backup_gold_image_$(date +%Y%m%d_%H%M%S)"

echo "=== Rozpoczynam aktualizację endpointu Gold Image ==="

# 1. Tworzenie backupu
echo "[1] Tworzenie backupu: $BACKUP_PATH"
cp "$APP_PATH" "$BACKUP_PATH" || {
    echo "ERROR: Nie można utworzyć backupu!"
    exit 1
}

# 2. Sprawdzenie obecnej zawartości (diagnostyka)
echo "[2] Obecna funkcja gold_image_manage():"
grep -n -A 7 "def gold_image_manage()" "$APP_PATH"

# 3. Tworzenie tymczasowego pliku z nową funkcją
TEMP_FILE=$(mktemp)
cat > "$TEMP_FILE" << 'EOF'
def gold_image_manage():
    """Endpoint do zarządzania Gold Image (RZECZYWISTY)"""
    print(f"[GOLD_IMAGE] Rozpoczynanie tworzenia Gold Image o {datetime.now()}")
    
    # Ścieżka do skryptu
    script_path = "/home/ubuntu/skrypty/create_gold_image.sh"
    
    try:
        # Uruchomienie skryptu z timeoutem 300 sekund (5 minut)
        result = subprocess.run(
            ["bash", script_path],
            capture_output=True,
            text=True,
            timeout=300
        )
        
        return jsonify({
            "status": "success" if result.returncode == 0 else "error",
            "returncode": result.returncode,
            "stdout": result.stdout,
            "stderr": result.stderr,
            "timestamp": datetime.now().isoformat(),
            "message": "Gold Image creation executed"
        }), 200 if result.returncode == 0 else 500
        
    except subprocess.TimeoutExpired:
        return jsonify({
            "status": "error",
            "message": "Script execution timeout (300 seconds)",
            "timestamp": datetime.now().isoformat()
        }), 500
    except Exception as e:
        return jsonify({
            "status": "error",
            "message": f"Unexpected error: {str(e)}",
            "timestamp": datetime.now().isoformat()
        }), 500
EOF

# 4. Wykonanie zamiany w pliku app.py
echo "[3] Wykonywanie zamiany funkcji..."
python3 -c "
import re

with open('$APP_PATH', 'r') as f:
    content = f.read()

# Wczytaj nową funkcję
with open('$TEMP_FILE', 'r') as f:
    new_function = f.read()

# Zamień starą funkcję na nową
old_pattern = r'def gold_image_manage\(\):.*?\n@app.route'
new_content = re.sub(old_pattern, new_function + '\n\n@app.route', content, flags=re.DOTALL)

# Jeśli powyższe nie zadziałało (brak @app.route po funkcji), spróbuj innego wzorca
if 'def gold_image_manage():' in content and 'def gold_image_manage():' not in new_content:
    old_pattern2 = r'def gold_image_manage\(\):.*?^\s*def '
    new_content = re.sub(old_pattern2, new_function + '\n\ndef ', content, flags=re.DOTALL | re.MULTILINE)

with open('$APP_PATH', 'w') as f:
    f.write(new_content)

print('Zamiana wykonana pomyślnie')
"

# 5. Weryfikacja zmian
echo "[4] Weryfikacja zmian:"
grep -n -A 10 "def gold_image_manage()" "$APP_PATH"

# 6. Sprawdzenie importów
echo "[5] Sprawdzanie importów..."
if ! grep -q "import subprocess" "$APP_PATH"; then
    echo "  -> Dodaję import subprocess"
    sed -i '1s/^/import subprocess\n/' "$APP_PATH"
fi

if ! grep -q "import json" "$APP_PATH"; then
    echo "  -> Dodaję import json"
    sed -i '1s/^/import json\n/' "$APP_PATH"
fi

# 7. Czyszczenie
rm -f "$TEMP_FILE"

echo "[6] Gotowe! Plik $APP_PATH zaktualizowany."
echo "=== Koniec aktualizacji ==="
echo ""
echo "Aby przetestować:"
echo "1. Zrestartuj usługę: sudo systemctl restart dashboard"
echo "2. Przetestuj endpoint: curl -X POST http://localhost:5000/api/gold_image/manage"
