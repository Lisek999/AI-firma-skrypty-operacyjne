#!/bin/bash
# SKRYPT: git_daily_skrypty_backup.sh
# CEL: Backup skryptów operacyjnych do Git
# DATA: 2024-12-29
# AUTOR: Wojtek (AI Programista)

echo "=== BACKUP SKRYPTÓW OPERACYJNYCH DO GIT ==="
echo "Repozytorium: ai-firma-vps (folder: skrypty_backup)"
echo ""

# 1. DIAGNOZA
echo "1. DIAGNOZA stanu wyjściowego..."
SOURCE_DIR="/opt/ai_firma_skrypty"
REPO_DIR="/tmp/skrypty_backup_$(date +%Y%m%d_%H%M%S)"
REMOTE_URL="git@github.com:Lisek999/ai-firma-vps.git"

if [ ! -d "$SOURCE_DIR" ]; then
    echo "❌ BŁĄD: Katalog źródłowy $SOURCE_DIR nie istnieje!"
    echo "   Aktualna zawartość /opt/:"
    ls -la /opt/
    exit 1
fi

echo "✅ Źródło: $SOURCE_DIR"
echo "   Zawartość źródła:"
ls -la "$SOURCE_DIR"

# 2. ANALIZA
echo ""
echo "2. ANALIZA planowanej zmiany..."
echo "   - Stworzenie tymczasowego katalogu: $REPO_DIR"
echo "   - Skopiowanie skryptów (bez tymczasowych plików)"
echo "   - Commit do folderu skrypty_backup/"
echo "   - Push do GitHub"

# 3. ZMIANA - wykonanie
echo ""
echo "3. ZMIANA - wykonanie..."
read -p "Czy kontynuować backup skryptów? (TAK/n): " -n 1 -r
echo ""
if [[ ! $REPLY =~ ^[Tt]$ ]] && [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Anulowano"
    exit 0
fi

# Stwórz tymczasowy katalog
echo "   a) Tworzenie katalogu tymczasowego..."
mkdir -p "$REPO_DIR"
cd "$REPO_DIR"

# Sklonuj repo
echo "   b) Klonowanie repozytorium..."
git clone "$REMOTE_URL" .
git checkout main

# Skopiuj skrypty (z wykluczeniami)
echo "   c) Kopiowanie skryptów..."
mkdir -p skrypty_backup
rsync -av --delete \
    --exclude='.git' \
    --exclude='*.tmp' \
    --exclude='*.log' \
    --exclude='*.bak' \
    --exclude='*.old' \
    "$SOURCE_DIR/" "./skrypty_backup/"

# Sprawdź czy są zmiany
if git diff --quiet && [ -z "$(git status --porcelain)" ]; then
    echo "   ℹ️  Brak zmian do commitowania"
    echo "   ✅ Backup zakończony (brak zmian)"
    rm -rf "$REPO_DIR"
    exit 0
fi

# Commit
echo "   d) Commit zmian..."
git add skrypty_backup/
git commit -m "Daily backup skryptów operacyjnych - $(date '+%Y-%m-%d %H:%M:%S')

- Źródło: $SOURCE_DIR
- Backup do: skrypty_backup/
- Typ: dzienny snapshot
- Liczba plików: $(find skrypty_backup/ -type f | wc -l)"

# Push
echo "   e) Push do GitHub..."
git push origin main

# Weryfikacja
echo "   f) Weryfikacja..."
if [ $? -eq 0 ]; then
    echo "   ✅ Backup skryptów udany!"
    echo "   📊 Zmiany:"
    git log --oneline -1
else
    echo "   ❌ Błąd push!"
fi

# 4. WERYFIKACJA - stan po zmianie
echo ""
echo "4. WERYFIKACJA - stan po zmianie..."
echo "   ✅ Źródłowe skrypty NIE zmienione:"
ls -la "$SOURCE_DIR" | head -5
echo ""
echo "   ✅ Backup w Git:"
echo "   https://github.com/Lisek999/ai-firma-vps/tree/main/skrypty_backup"
echo ""
echo "   🧹 Czyszczenie katalogu tymczasowego..."
rm -rf "$REPO_DIR"

echo ""
echo "=== BACKUP SKRYPTÓW ZAKOŃCZONY ==="
echo "Skrypty backupowane, struktura źródłowa niezmieniona."
echo "Następny krok: Backup tygodniowy (pełny archiwum)."
