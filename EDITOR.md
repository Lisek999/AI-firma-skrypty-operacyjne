#!/bin/bash

echo "=== AUTOMAT NAPRAWCZY FUNKCJI getscript ==="

# 1. Tworzymy poprawną definicję funkcji
NOWA_FUNKCJA='getscript() {
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
        echo "Sprawdź, czy plik EDITOR_SKRYPTÓW.md nie jest pusty."
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
}'

# 2. Znajdź i usuń starą definicję funkcji z ~/.bashrc
echo "Szukam starej definicji funkcji getscript..."
sed -i '/^getscript() {/,/^}/d' ~/.bashrc

# 3. Dodaj nową definicję na końcu pliku
echo "Dodaję nową, poprawną funkcję..."
echo "$NOWA_FUNKCJA" >> ~/.bashrc

# 4. Załaduj zaktualizowany plik
echo "Ładuję zaktualizowany ~/.bashrc..."
source ~/.bashrc

# 5. Testujemy naprawę
echo "Testowanie nowej funkcji..."
if type getscript &>/dev/null; then
    echo "✅ SUKCES: Funkcja getscript jest poprawnie zdefiniowana."
    echo "Możesz teraz użyć: getscript test"
else
    echo "❌ BŁĄD: Coś poszło nie tak."
fi

echo "=== NAPRAWA ZAKOŃCZONA ==="
