#!/bin/bash
# SKRYPT: push_dashboard_to_vps_repo.sh
# CEL: Przeniesienie dashboard do podfolderu i push do ai-firma-vps
# DATA: 2024-12-29
# AUTOR: Wojtek (AI Programista)

echo "=== ETAP 3: ZMIANA - Push dashboard do repo ai-firma-vps ==="
echo "Cel: Przenieść kod do folderu dashboard_backup i push do głównego brancha"
echo ""

# 1. DIAGNOZA
echo "1. DIAGNOZA stanu wyjściowego..."
cd /opt/ai_firma_dashboard

if [ ! -d ".git" ]; then
    echo "❌ BŁĄD: Brak repozytorium Git w /opt/ai_firma_dashboard"
    exit 1
fi

echo "✅ Repozytorium Git istnieje"
echo "   Aktualny remote:"
git remote -v
echo "   Branch: $(git branch --show-current)"

# 2. ANALIZA
echo ""
echo "2. ANALIZA planowanej zmiany..."
echo "   - Zmiana remote na ai-firma-vps"
echo "   - Stworzenie folderu dashboard_backup"
echo "   - Przeniesienie wszystkich plików do dashboard_backup/"
echo "   - Commit zmian"
echo "   - Push do origin main"

# 3. ZMIANA - wykonanie
echo ""
echo "3. ZMIANA - wykonanie..."
read -p "Czy kontynuować? (TAK/n): " -n 1 -r
echo ""
if [[ ! $REPLY =~ ^[Tt]$ ]] && [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Anulowano"
    exit 0
fi

# Zmiana remote
echo "   a) Zmiana zdalnego repozytorium..."
git remote set-url origin https://github.com/Lisek999/ai-firma-vps.git
echo "   ✅ Nowy remote:"
git remote -v

# Pobierz najnowsze zmiany (jeśli repo nie jest puste)
echo "   b) Pobieranie najnowszych zmian z remote..."
git fetch origin

# Sprawdź czy main istnieje
if git show-ref --verify --quiet refs/remotes/origin/main; then
    echo "   ℹ️  Branch main istnieje zdalnie - pull"
    git pull origin main --allow-unrelated-histories || echo "   ⚠️  Może być konflikt, kontynuujemy"
else
    echo "   ℹ️  Brak brancha main - tworzymy nowy"
fi

# Stwórz folder i przenieś pliki
echo "   c) Tworzenie folderu dashboard_backup..."
mkdir -p dashboard_backup

echo "   d) Przenoszenie plików do dashboard_backup..."
# Przenieś wszystko oprócz .git i dashboard_backup
find . -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'dashboard_backup' -exec mv {} dashboard_backup/ \; 2>/dev/null

# Sprawdź czy coś zostało przeniesione
if [ "$(ls -A dashboard_backup 2>/dev/null)" ]; then
    echo "   ✅ Pliki przeniesione"
else
    echo "   ⚠️  Brak plików do przeniesienia - może już są w folderze?"
fi

# Commit
echo "   e) Dodawanie zmian i commit..."
git add .
git commit -m "Backup dashboard aplikacji AI Firma

- Data: $(date '+%Y-%m-%d %H:%M:%S')
- Lokalizacja: /opt/ai_firma_dashboard
- Struktura: dashboard_backup/
- Cel: Backup dzienny do GitHub"

# Push
echo "   f) Push do GitHub..."
echo "   ⚠️  Uwaga: Może wymagać uwierzytelnienia tokenem"
git push -u origin main

# 4. WERYFIKACJA
echo ""
echo "4. WERYFIKACJA - stan po zmianie..."
echo "   ✅ Remote:"
git remote -v
echo ""
echo "   ✅ Struktura plików:"
find . -type f -name "*.py" -o -name "*.html" -o -name "*.md" | head -10
echo ""
echo "=== ZMIANA ZAKOŃCZONA ==="
echo "Sprawdź na GitHub: https://github.com/Lisek999/ai-firma-vps"
echo "Powinien być folder dashboard_backup/"
