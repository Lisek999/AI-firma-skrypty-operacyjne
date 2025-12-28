#!/bin/bash
# SKRYPT: git_weekly_full_backup.sh
# CEL: Tygodniowy pełny backup do archiwum .tar.gz
# DATA: 2024-12-29
# AUTOR: Wojtek (AI Programista)

echo "=== TYGODNIOWY PEŁNY BACKUP (ARCHIWUM .tar.gz) ==="
echo "Repozytorium: ai-firma-vps (folder: weekly_archives)"
echo ""

# 1. DIAGNOZA
echo "1. DIAGNOZA stanu wyjściowego..."
BACKUP_DIRS=(
    "/opt/ai_firma_dashboard"
    "/opt/ai_firma_skrypty"
    "/home/ubuntu/ai_firma_dokumenty"
)
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
REPO_DIR="/tmp/weekly_backup_$TIMESTAMP"
REMOTE_URL="git@github.com:Lisek999/ai-firma-vps.git"

echo "✅ Katalogi do backupu:"
for dir in "${BACKUP_DIRS[@]}"; do
    if [ -d "$dir" ]; then
        echo "   - $dir (istnieje)"
    else
        echo "   - $dir (NIE istnieje - pomijam)"
    fi
done

# 2. ANALIZA
echo ""
echo "2. ANALIZA planowanej zmiany..."
echo "   - Stworzenie tymczasowego katalogu: $REPO_DIR"
echo "   - Utworzenie archiwum .tar.gz dla każdego katalogu"
echo "   - Dodanie archiwów do folderu weekly_archives/"
echo "   - Commit i push"

# 3. ZMIANA - wykonanie
echo ""
echo "3. ZMIANA - wykonanie..."
read -p "Czy wykonać tygodniowy pełny backup? (TAK/n): " -n 1 -r
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

# Stwórz folder weekly_archives
mkdir -p weekly_archives

# Dla każdego katalogu utwórz archiwum
for source_dir in "${BACKUP_DIRS[@]}"; do
    if [ ! -d "$source_dir" ]; then
        echo "   ⚠️  Pomijam: $source_dir (nie istnieje)"
        continue
    fi
    
    dir_name=$(basename "$source_dir")
    archive_name="backup_${dir_name}_${TIMESTAMP}.tar.gz"
    archive_path="weekly_archives/$archive_name"
    
    echo "   c) Tworzenie archiwum dla $dir_name..."
    
    # Utwórz archiwum z wykluczeniami
    tar -czf "$archive_path" \
        -C "$(dirname "$source_dir")" \
        --exclude=".git" \
        --exclude="__pycache__" \
        --exclude="*.pyc" \
        --exclude="*.log" \
        --exclude="*.tmp" \
        "$(basename "$source_dir")"
    
    if [ $? -eq 0 ]; then
        archive_size=$(du -h "$archive_path" | cut -f1)
        echo "   ✅ Utworzono: $archive_name ($archive_size)"
    else
        echo "   ❌ Błąd tworzenia archiwum dla $dir_name"
    fi
done

# Sprawdź czy utworzono jakieś archiwa
if [ -z "$(ls -A weekly_archives 2>/dev/null)" ]; then
    echo "   ❌ Nie utworzono żadnych archiwów!"
    rm -rf "$REPO_DIR"
    exit 1
fi

# Commit
echo "   d) Commit archiwów..."
git add weekly_archives/
git commit -m "Tygodniowy pełny backup - $TIMESTAMP

- Archiwa: $(ls weekly_archives/ | xargs)
- Rozmiar: $(du -sh weekly_archives/ | cut -f1)
- Typ: pełny backup (archiwum .tar.gz)
- Retencja: tygodniowe snapshots"

# Push
echo "   e) Push do GitHub..."
git push origin main

# 4. WERYFIKACJA
echo ""
echo "4. WERYFIKACJA..."
if [ $? -eq 0 ]; then
    echo "   ✅ Tygodniowy backup udany!"
    echo "   📊 Utworzone archiwa:"
    ls -lh weekly_archives/
    echo ""
    echo "   🔗 Dostępne na GitHub:"
    echo "   https://github.com/Lisek999/ai-firma-vps/tree/main/weekly_archives"
else
    echo "   ❌ Błąd push!"
fi

# Czyszczenie
echo ""
echo "🧹 Czyszczenie katalogu tymczasowego..."
rm -rf "$REPO_DIR"

echo ""
echo "=== TYGODNIOWY BACKUP ZAKOŃCZONY ==="
echo "Archiwa .tar.gz zapisane w weekly_archives/"
echo "Następny krok: Konfiguracja cron dla backupów dziennych i tygodniowych."
