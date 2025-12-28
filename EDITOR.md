#!/bin/bash
# SKRYPT: init_git_dashboard.sh
# CEL: Inicjalizacja Git w katalogu dashboard - JEDYNA zmiana w tym cyklu
# DATA: 2024-12-29
# AUTOR: Wojtek (AI Programista)

echo "=== ETAP 3: ZMIANA - Inicjalizacja Git dla Dashboard ==="
echo "Cel: Zainicjalizować Git w /var/www/ai_firma_dashboard i połączyć z GitHub"
echo ""

# 1. DIAGNOZA - sprawdzenie stanu wyjściowego
echo "1. DIAGNOZA stanu wyjściowego..."
DASHBOARD_DIR="/var/www/ai_firma_dashboard"
REMOTE_URL="https://github.com/Lisek999/ai-firma-dashboard.git"

if [ ! -d "$DASHBOARD_DIR" ]; then
    echo "❌ BŁĄD: Katalog $DASHBOARD_DIR nie istnieje!"
    echo "Stan: Katalog nie znaleziony"
    exit 1
fi

echo "✅ Katalog istnieje: $DASHBOARD_DIR"

cd "$DASHBOARD_DIR"
echo "   Praca w katalogu: $(pwd)"

# Sprawdź czy Git jest już zainicjalizowany
if [ -d ".git" ]; then
    echo "ℹ️  Git jest już zainicjalizowany w tym katalogu"
    echo "   Remote:"
    git remote -v
    echo "   Branch: $(git branch --show-current 2>/dev/null || echo 'brak')"
    exit 0
else
    echo "✅ Git NIE jest zainicjalizowany - kontynuujemy"
fi

# 2. ANALIZA - co zrobimy
echo ""
echo "2. ANALIZA planowanej zmiany..."
echo "   - Inicjalizacja: git init"
echo "   - Konfiguracja: git config user"
echo "   - Dodanie plików: git add ."
echo "   - Pierwszy commit: git commit -m 'Initial commit'"
echo "   - Dodanie remote: git remote add origin $REMOTE_URL"

# 3. ZMIANA - wykonanie (tylko z potwierdzeniem)
echo ""
echo "3. ZMIANA - wykonanie..."
read -p "Czy kontynuować inicjalizację Git? (TAK/n): " -n 1 -r
echo ""
if [[ ! $REPLY =~ ^[Tt]$ ]] && [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Anulowano - brak potwierdzenia"
    exit 0
fi

# Inicjalizacja Git
echo "   a) Inicjalizacja repozytorium..."
git init

# Konfiguracja użytkownika
echo "   b) Konfiguracja użytkownika Git..."
git config user.email "ci@ai-firma.com"
git config user.name "AI Firma CI"

# Dodaj wszystkie pliki
echo "   c) Dodawanie plików do stage..."
git add .

# Pierwszy commit
echo "   d) Tworzenie pierwszego commita..."
git commit -m "Initial commit - dashboard aplikacji AI Firma
- Data: $(date '+%Y-%m-%d %H:%M:%S')
- Autor: AI Firma CI
- Cel: Backup dzienny do GitHub"

# Dodaj remote
echo "   e) Dodawanie zdalnego repozytorium..."
git remote add origin "$REMOTE_URL"

# 4. WERYFIKACJA - stan po zmianie
echo ""
echo "4. WERYFIKACJA - stan po zmianie..."
echo "   ✅ Git zainicjalizowany: $(git rev-parse --git-dir 2>/dev/null && echo 'TAK' || echo 'NIE')"
echo "   ✅ Remote ustawiony:"
git remote -v
echo ""
echo "   ✅ Status plików:"
git status --short

echo ""
echo "=== INICJALIZACJA ZAKOŃCZONA ==="
echo ""
echo "NASTĘPNY KROK (ręczny):"
echo "Aby wypchnąć kod na GitHub, wykonaj:"
echo "  cd /var/www/ai_firma_dashboard"
echo "  git push -u origin main"
echo ""
echo "Uwaga: Może być wymagane uwierzytelnienie (token GitHub)."
echo "Po pushu przejdziemy do kolejnego cyklu DA-CZ-W."
