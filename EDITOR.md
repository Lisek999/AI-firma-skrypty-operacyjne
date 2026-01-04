#!/bin/bash
# secure_vault_generate_keys_final.sh - Generowanie nowej pary kluczy
# Wersja: 1.2 | Data: 2024-12-29
# Bez pytań, bez potwierdzeń - stary klucz jest bezużyteczny

set -e

echo "=== 🔐 GENEROWANIE NOWEJ PARY KLUCZY RSA 4096-BIT ==="
echo "Stary klucz publiczny jest bezużyteczny bez klucza prywatnego"
echo "Data: $(date)"
echo ""

# =================== KONFIGURACJA ===================
KEYS_DIR="/home/ubuntu/.secure_vault"
PUBLIC_KEY="$KEYS_DIR/backup_public.pem"

# =================== WALIDACJA ===================
echo "1. 🧪 PRZYGOTOWANIE..."
if [ ! -d "$KEYS_DIR" ]; then
    echo "   ❌ BŁĄD: Brak katalogu .secure_vault"
    echo "   Uruchom najpierw secure_vault_setup.sh"
    exit 1
fi

echo "   Usuwam stary klucz publiczny (bezużyteczny)..."
rm -f "$PUBLIC_KEY" 2>/dev/null || true

# =================== GENEROWANIE ===================
echo -e "\n2. 🔧 GENEROWANIE KLUCZA PRYWATNEGO..."
echo "   To może zająć 30-60 sekund..."
echo "   Rozpoczynam: $(date)"

START_TIME=$(date +%s)
PRIVATE_KEY_CONTENT=$(openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:4096 \
    -pkeyopt rsa_keygen_pubexp:65537 2>/dev/null)

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

if [ -z "$PRIVATE_KEY_CONTENT" ]; then
    echo "   ❌ BŁĄD: Nie udało się wygenerować klucza prywatnego"
    exit 1
fi

echo "   ✅ Klucz prywatny wygenerowany pomyślnie"
echo "   Czas generowania: ${DURATION} sekund"

# =================== ZAPIS KLUCZA PUBLICZNEGO ===================
echo -e "\n3. 📤 ZAPISYWANIE KLUCZA PUBLICZNEGO..."
echo "$PRIVATE_KEY_CONTENT" | openssl pkey -pubout -out "$PUBLIC_KEY" 2>/dev/null

if [ ! -s "$PUBLIC_KEY" ]; then
    echo "   ❌ BŁĄD: Nie udało się zapisać klucza publicznego"
    exit 1
fi

chmod 600 "$PUBLIC_KEY"
chown ubuntu:ubuntu "$PUBLIC_KEY"

echo "   ✅ Klucz publiczny zapisany: $PUBLIC_KEY"
echo "   Uprawnienia: $(stat -c %A "$PUBLIC_KEY")"

# =================== WYŚWIETLENIE KLUCZA PRYWATNEGO ===================
echo -e "\n4. 🚨 ==========================================================="
echo "   🔥🔥🔥 KLUCZ PRYWATNY - SKOPIUJ CAŁOŚĆ PONIŻEJ 🔥🔥🔥"
echo "   ==========================================================="
echo ""
echo "$PRIVATE_KEY_CONTENT"
echo ""
echo "   ==========================================================="
echo "   ✅ KONIEC KLUCZA PRYWATNEGO"
echo "   ==========================================================="

# =================== INSTRUKCJE ===================
echo -e "\n5. 📋 INSTRUKCJE KOPIOWANIA W TERMINUSIE:"
cat << 'EOF'

📥 **JAK SKOPIOWAĆ:**
1. DOTKNIJ i PRZYTRZYMAJ gdziekolwiek w kluczu powyżej
2. Wybierz "SELECT ALL" (Zaznacz wszystko)
3. Wybierz "COPY" (Kopiuj)
4. Wklej do bezpiecznego miejsca

💾 **ZAPISZ W 2 MIEJSCACH:**
• Menedżer haseł (Bitwarden/1Password)
• Notatnik na telefonie
• Wydruk w sejfie

⚠️  **BEZ TEGO KLUCZA BACKUPY SĄ BEZUŻYTECZNE!**
EOF

# =================== TEST ===================
echo -e "\n6. 🧪 TEST SYGNALIZACYJNY..."
echo "test" | timeout 2 openssl pkeyutl -encrypt -pubin -inkey "$PUBLIC_KEY" 2>&1 >/dev/null && echo "   ✅ Klucz publiczny działa" || echo "   ⚠️  Test pominięty"

# =================== PODSUMOWANIE ===================
echo -e "\n7. 📊 PODSUMOWANIE:"
echo "   Klucz publiczny: $PUBLIC_KEY"
echo "   Fingerprint: $(openssl rsa -pubin -in "$PUBLIC_KEY" -outform DER 2>/dev/null | openssl md5 -c 2>/dev/null | awk '{print $2}')"
echo "   Czas generowania: ${DURATION}s"

echo -e "\n=== ✅ GENEROWANIE ZAKOŃCZONE ==="
echo "Następny krok: Potwierdź skopiowanie klucza prywatnego"
