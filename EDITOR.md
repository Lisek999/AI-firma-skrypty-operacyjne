#!/bin/bash

echo "=== NAPRAWA FUNKCJI getscript ==="
echo "1. Usuwam starą, uszkodzoną funkcję..."

# Znajdź i usuń starą funkcję getscript (od 'getscript() {' do następnego '}')
sed -i '/^getscript() {/,/^}/d' ~/.bashrc

echo "2. Dodaję nową, poprawną funkcję..."

# Dodaj nową funkcję na końcu pliku
cat >> ~/.bashrc << 'EOF'

getscript() {
    # 1. Pobierz adres źródłowy z pliku ustawień
    SETTINGS_URL="https://raw.githubusercontent.com/Lisek999/AI-firma-skrypty-operacyjne/main/SKRYPTY_OPERACYJNE.md"
    SOURCE_URL=$(curl -s "$SETTINGS_URL")

    if [ -z "$SOURCE_URL" ]; then
        echo "BŁĄD: Nie udało się pobrać adresu źródłowego z pliku ustawień."
        return 1
    fi

    # 2. Pobierz skrypt
    SCRIPT_CONTENT=$(curl -s "$SOURCE_URL")

    if [ -z "$SCRIPT_CONTENT" ]; then
        echo "BŁĄD: Nie udało się pobrać skryptu z: $SOURCE_URL"
        echo "Sprawdź, czy plik EDITOR.md nie jest pusty."
        return 1
    fi

    # 3. Przygotuj ścieżkę zapisu i nazwę pliku
    SCRIPT_DIR="$HOME/skrypty"
    mkdir -p "$SCRIPT_DIR"

    if [ -n "$1" ]; then
        FILENAME="$1.sh"
    else
        FILENAME="skrypt_$(date +%Y%m%d_%H%M%S).sh"
    fi

    FULL_PATH="$SCRIPT_DIR/$FILENAME"

    # 4. Zapisz skrypt do pliku
    echo "$SCRIPT_CONTENT" > "$FULL_PATH"
    chmod +x "$FULL_PATH"

    echo "========================================"
    echo "SKRYPT ZAPISANY JAKO: $FULL_PATH"
    echo "ŹRÓDŁO: $SOURCE_URL"
    echo "========================================"

    # 5. Wykonaj zapisany skrypt
    echo "URUCHAMIANIE..."
    echo "---"
    bash "$FULL_PATH"
}
EOF

echo "3. Przeładowuję ~/.bashrc..."
source ~/.bashrc 2>/dev/null || echo "Uwaga: source zwrócił błąd, ale plik jest zaktualizowany."

echo "4. Testuję..."
if type getscript &>/dev/null; then
    echo "✅ SUKCES: Funkcja getscript jest gotowa do użycia."
    echo "   Możesz teraz użyć: getscript test"
else
    echo "⚠️  Funkcja może nie być załadowana do bieżącej sesji."
    echo "   Uruchom: source ~/.bashrc"
    echo "   Lub otwórz nowy terminal."
fi

echo "=== NAPRAWA ZAKOŃCZONA ==="
