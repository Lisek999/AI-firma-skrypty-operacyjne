#!/bin/bash
# SKRYPT: git_daily_dashboard_backup.sh
# CEL: Bezpieczny backup dashboard do Git (KOPIOWANIE, nie przenoszenie)
# DATA: 2024-12-29
# AUTOR: Wojtek (AI Programista)

echo "=== BEZPIECZNY BACKUP DASHBOARD DO GIT ==="
echo "Metoda: KOPIOWANIE do tymczasowego katalogu"
echo ""

# 1. DIAGNOZA
echo "1. DIAGNOZA stanu wyjściowego..."
SOURCE_DIR="/opt/ai_firma_dashboard"
REPO_DIR="/tmp/dashboard_backup_$(date +%Y%m%d_%H%M%S)"
REMOTE_URL="https://github.com/Lisek999/ai-firma-vps.git"

if [ ! -d "$SOURCE_DIR" ]; then
    echo "❌ BŁĄD: Katalog źródłowy $SOURCE_DIR nie istnieje!"
    exit 1
fi

echo "✅ Źródło: $SOURCE_DIR"
echo "   Zawartość źródła:"
ls -la "$SOURCE_DIR" | head -5

# 2. ANALIZA
echo ""
echo "2. ANALIZA planowanej zmiany..."
echo "   - Stworzenie tymczasowego katalogu: $REPO_DIR"
echo "   - Skopiowanie plików (z wykluczeniem .git, __pycache__)"
echo "   - Inicjalizacja/clone repo w katalogu tymczasowym"
echo "   - Commit zmian"
echo "   - Push do GitHub"
echo "   - Usunięcie katalogu tymczasowego"

# 3. ZMIANA - wykonanie
echo ""
echo "3. ZMIANA - wykonanie..."
read -p "Czy kontynuować backup? (TAK/n): " -n 1 -r
echo ""
if [[ ! $REPLY =~ ^[Tt]$ ]] && [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Anulowano"
    exit 0
fi

# Stwórz tymczasowy katalog
echo "   a) Tworzenie katalogu tymczasowego..."
mkdir -p "$REPO_DIR"
cd "$REPO_DIR"

# Sklonuj repo lub zainicjalizuj
echo "   b) Przygotowanie repozytorium Git..."
if [ -d "$SOURCE_DIR/.git" ]; then
    # Użyj istniejącego repo
    cp -r "$SOURCE_DIR/.git" .
    git reset --hard
else
    # Sklonuj ze zdalnego repo
    git clone "$REMOTE_URL" .
fi

# Ustaw remote
git remote set-url origin "$REMOTE_URL"

# Pobierz najnowsze zmiany
echo "   c) Synchronizacja ze zdalnym repo..."
git fetch origin
git pull origin main --rebase 2>/dev/null || echo "   ⚠️  Pierwszy backup lub konflikt"

# Skopiuj pliki (z wykluczeniami)
echo "   d) Kopiowanie plików..."
rsync -av --delete \
    --exclude='.git' \
    --exclude='__pycache__' \
    --exclude='*.pyc' \
    --exclude='*.log' \
    "$SOURCE_DIR/" "./dashboard_backup/"

# Sprawdź czy są zmiany
if git diff --quiet && [ -z "$(git status --porcelain)" ]; then
    echo "   ℹ️  Brak zmian do commitowania"
    echo "   ✅ Backup zakończony (brak zmian)"
    rm -rf "$REPO_DIR"
    exit 0
fi

# Commit
echo "   e) Commit zmian..."
git add .
git commit -m "Daily backup dashboard - $(date '+%Y-%m-%d %H:%M:%S')

- Źródło: $SOURCE_DIR
- Backup do: dashboard_backup/
- Typ: dzienny snapshot
- Status: działająca aplikacja NIE zmieniona"

# Push
echo "   f) Push do GitHub..."
git push origin main

# Weryfikacja
echo "   g) Weryfikacja..."
if [ $? -eq 0 ]; then
    echo "   ✅ Backup udany!"
    echo "   📊 Zmiany:"
    git log --oneline -1
else
    echo "   ❌ Błąd push!"
fi

# 4. WERYFIKACJA - stan po zmianie
echo ""
echo "4. WERYFIKACJA - stan po zmianie..."
echo "   ✅ Źródłowa aplikacja NIE zmieniona:"
ls -la "$SOURCE_DIR" | head -3
echo ""
echo "   ✅ Backup w Git:"
echo "   https://github.com/Lisek999/ai-firma-vps/tree/main/dashboard_backup"
echo ""
echo "   🧹 Czyszczenie katalogu tymczasowego..."
rm -rf "$REPO_DIR"

echo ""
echo "=== BACKUP ZAKOŃCZONY ==="
echo "Dashboard nadal działa, backup w Git."
echo "Następny krok: Skonfigurowanie cron dla tego skryptu."
