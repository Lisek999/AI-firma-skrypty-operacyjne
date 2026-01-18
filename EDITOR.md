#!/bin/bash
# ============================================================================
# SKRYPT: sprzatanie_starego_dashboardu.sh
# CEL: Bezpieczne sprzątanie starych plików dashboardu (z backupem)
# ZASADA: "Najpierw backup, potem usuwanie"
# ============================================================================

echo "=== BEZPIECZNE SPRZĄTANIE STAREGO DASHBOARDU ==="

cd /opt/ai_firma_dashboard || exit 1

# 1. TWORZENIE ARCHIWUM BACKUPOWEGO
BACKUP_DIR="/home/ubuntu/old_dashboard_backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
echo "✅ Utworzono katalog backupowy: $BACKUP_DIR"

# 2. KOPIOWANIE STARYCH PLIKÓW DO BACKUPU (zamiast od razu usuwać)
echo "Kopiuję stare pliki do backupu..."

# Stare wersje index.html z static/
if ls static/index.html.* 1>/dev/null 2>&1; then
    echo "📁 Kopiuję stare index.html.* do backupu..."
    cp -v static/index.html.* "$BACKUP_DIR/" 2>/dev/null
fi

# Stary backup_management.html
if [ -f "static/backup_management.html.old" ]; then
    echo "📁 Kopiuję backup_management.html.old..."
    cp -v static/backup_management.html.old "$BACKUP_DIR/"
fi

# Stary folder dashboard_backup (jeśli istnieje)
if [ -d "dashboard_backup" ]; then
    echo "📁 Kopiuję cały katalog dashboard_backup..."
    cp -rv dashboard_backup "$BACKUP_DIR/" 2>/dev/null
fi

# Stare backup app.py (zachowujemy tylko 2 najnowsze)
echo "🗃️ Porządkuję backupowe app.py..."
BACKUP_FILES=$(ls -t app.py.backup* 2>/dev/null | wc -l)
if [ "$BACKUP_FILES" -gt 2 ]; then
    echo "Znaleziono $BACKUP_FILES backupów app.py, zachowuję 2 najnowsze..."
    
    # Lista do usunięcia (wszystkie poza 2 najnowszymi)
    FILES_TO_REMOVE=$(ls -t app.py.backup* 2>/dev/null | tail -n +3)
    
    for file in $FILES_TO_REMOVE; do
        if [ -f "$file" ]; then
            echo "📦 Kopiuję do backupu przed usunięciem: $file"
            cp -v "$file" "$BACKUP_DIR/"
        fi
    done
fi

# 3. USUWANIE STARYCH PLIKÓW (PO BACKUPIE)
echo ""
echo "=== USUWANIE ZBĘDNYCH PLIKÓW ==="

# Stare index.html.* z static/ (ale NIE index.html.old.backup - już przeniesiony wcześniej)
echo "🧹 Czyszczę static/index.html.* (oprócz .old.backup)..."
find static/ -name "index.html.*" ! -name "*.old.backup" -type f -delete 2>/dev/null && echo "✅ Usunięto"

# Inne stare pliki HTML w static/
echo "🧹 Czyszczę inne stare HTML w static/..."
find static/ -name "*.html.backup*" -type f -delete 2>/dev/null && echo "✅ Usunięto"
find static/ -name "*.html.before_*" -type f -delete 2>/dev/null && echo "✅ Usunięto"
find static/ -name "*.html.pre_*" -type f -delete 2>/dev/null && echo "✅ Usunięto"

# Stare backup app.py (zachowujemy tylko 2 najnowsze)
if [ -n "$FILES_TO_REMOVE" ]; then
    echo "🧹 Usuwam stare backup app.py (zachowano 2 najnowsze)..."
    for file in $FILES_TO_REMOVE; do
        if [ -f "$file" ]; then
            rm -v "$file"
        fi
    done
fi

# 4. SPRZĄTANIE KATALOGU dashboard_backup (jeśli istnieje)
if [ -d "dashboard_backup" ]; then
    echo "🧹 Czyścię katalog dashboard_backup (zachowano kopię w $BACKUP_DIR)..."
    rm -rf dashboard_backup && echo "✅ Usunięto cały katalog"
fi

# 5. WERYFIKACJA
echo ""
echo "=== WERYFIKACJA PO SPRZĄTANIU ==="
echo "📊 Zawartość static/:"
ls -la static/ | grep -E "\.html$|razem"
echo ""
echo "📊 Backupowe app.py (powinny być max 2):"
ls -la app.py.backup* 2>/dev/null || echo "Brak backupowych app.py"
echo ""
echo "📊 Rozmiar backupu:"
du -sh "$BACKUP_DIR" 2>/dev/null || echo "Brak katalogu backupu"

# 6. INFORMACJE O BEZPIECZEŃSTWIE
echo ""
echo "=== INFORMACJE BEZPIECZEŃSTWA ==="
echo "✅ Wszystkie usuwane pliki są w backupie: $BACKUP_DIR"
echo "✅ Zachowano:"
echo "   - 2 najnowsze backup app.py"
echo "   - index.html.old.backup (jako zabezpieczenie)"
echo "   - backup_management.html.old (w katalogu backup)"
echo ""
echo "⚠️  Jeśli coś poszło nie tak, przywróć z:"
echo "    cp $BACKUP_DIR/* ."
echo ""
echo "🎯 Dashboard jest teraz CZYSTY i PRZEJRZYSTY"
echo "📁 Tylko potrzebne pliki:"
echo "    templates/ (8 szablonów)"
echo "    app.py (główny plik)"
echo "    static/ (tylko aktualne pliki)"
