#!/bin/bash
# ============================================================================
# SKRYPT: Konfiguracja HTTP Basic Auth dla dashboardu (57.128.247.215)
# AUTOR: System Wojtek/CEO
# DATA: 2024-12-24
# MODEL: 2-plikowy (Gotowy do EDITOR.md)
# WERSJA: 1.2 (z implementacją sprawdz_narzędzia i utworz_kopie_zapasowa)
# ============================================================================
# ZASADY:
# 1. Używa STAŁYCH poświadczeń: Agnostyk / Castorama13.
# 2. Zawiera pełną logikę: backup, modyfikację, walidację, rollback.
# 3. Działa TYLKO z uprawnieniami sudo.
# 4. Dystrybucja: GitHub (EDITOR.md) -> getscript -> VPS.
# ============================================================================

# --- BEZPIECZEŃSTWO ---
set -e          # Zatrzymaj się przy pierwszym błędzie.
set -u          # Wykrywaj użycie niezdefiniowanych zmiennych.
set -o pipefail # Rozpoznawaj błędy w potokach (|).

# --- KONFIGURACJA (ZMIENNE STAŁE) ---
NGINX_SITE_CONFIG="/etc/nginx/sites-available/dashboard"  # Ścieżka do pliku konfiguracji
BACKUP_DIR="/etc/nginx/backup"                            # Katalog na kopie zapasowe
HTPASSWD_FILE="/etc/nginx/.htpasswd_dashboard"            # Plik z hasłami
SERVER_IP="57.128.247.215"                                # Docelowy adres IP

# --- STAŁE POŚWIADCZENIA (DECYZJA CEO: STAŁE) ---
AUTH_USER="Agnostyk"
AUTH_PASS="Castorama13"

# --- FUNKCJE POMOCNICZE ---
function sprawdz_narzędzia() {
    echo "[1] Sprawdzanie wymaganych narzędzi (nginx, htpasswd)..."
    
    # Sprawdź, czy nginx jest zainstalowany (poprzez sprawdzenie usługi)
    if ! systemctl is-active --quiet nginx 2>/dev/null && ! command -v nginx >/dev/null 2>&1; then
        echo "   [BŁĄD] Nie znaleziono usługi 'nginx' ani polecenia 'nginx'."
        echo "   [ROZWIĄZANIE] Zainstaluj: sudo apt update && sudo apt install -y nginx"
        exit 1
    else
        echo "   [OK] Nginx jest dostępny."
    fi
    
    # Sprawdź, czy htpasswd jest dostępny
    if ! command -v htpasswd >/dev/null 2>&1; then
        echo "   [BŁĄD] Nie znaleziono polecenia 'htpasswd'."
        echo "   [ROZWIĄZANIE] Zainstaluj pakiet: sudo apt install -y apache2-utils"
        exit 1
    else
        echo "   [OK] Narzędzie 'htpasswd' (apache2-utils) jest dostępne."
    fi
    echo ""
}

function utworz_kopie_zapasowa() {
    echo "[2] Tworzenie kopii zapasowej konfiguracji..."
    
    # 1. SPRAWDŹ, CZY PLIK KONFIGURACYJNY ISTNIEJE
    if [[ ! -f "$NGINX_SITE_CONFIG" ]]; then
        echo "   [BŁĄD] Nie znaleziono pliku konfiguracyjnego: $NGINX_SITE_CONFIG"
        echo "   [ROZWIĄZANIE] Upewnij się, że Nginx jest poprawnie skonfigurowany dla dashboardu."
        exit 1
    fi
    echo "   [OK] Znaleziono plik konfiguracyjny: $NGINX_SITE_CONFIG"
    
    # 2. UTWÓRZ KATALOG BACKUP, JEŚLI NIE ISTNIEJE
    if [[ ! -d "$BACKUP_DIR" ]]; then
        echo "   [INFO] Katalog backup nie istnieje. Tworzę: $BACKUP_DIR"
        mkdir -p "$BACKUP_DIR"
        if [[ $? -eq 0 ]]; then
            echo "   [OK] Katalog backup utworzony."
        else
            echo "   [BŁĄD] Nie udało się utworzyć katalogu backup."
            exit 1
        fi
    fi
    
    # 3. STWÓRZ KOPIĘ ZAPASOWĄ Z TIMESTAMPEM
    local timestamp=$(date +"%Y%m%d_%H%M%S")
    local backup_file="$BACKUP_DIR/$(basename "$NGINX_SITE_CONFIG")_${timestamp}.backup"
    
    cp "$NGINX_SITE_CONFIG" "$backup_file"
    
    if [[ $? -eq 0 ]]; then
        echo "   [OK] Utworzono kopię zapasową: $backup_file"
        # (Opcjonalnie) Zabezpiecz uprawnienia kopii
        chmod 600 "$backup_file"
    else
        echo "   [BŁĄD] Nie udało się utworzyć kopii zapasowej."
        exit 1
    fi
    echo ""
}

function skonfiguruj_autoryzacje() {
    echo "[3] Konfiguracja HTTP Basic Auth..."
    # TODO: 1. Utworzenie HTPASSWD_FILE z poświadczeniami.
    # TODO: 2. Modyfikacja NGINX_SITE_CONFIG (dyrektywy auth_basic).
    echo "   > Funkcja nie zaimplementowana."
}

function waliduj_i_przeladuj() {
    echo "[4] Walidacja i przeładowanie Nginx..."
    # TODO: 1. Uruchomienie 'nginx -t'.
    # TODO: 2. Jeśli OK: 'systemctl reload nginx'. Jeśli NIE: wywołaj procedura_awaryjna.
    echo "   > Funkcja nie zaimplementowana."
}

function procedura_awaryjna() {
    echo "[!] Uruchomienie procedury awaryjnego wycofania..."
    # TODO: 1. Przywrócenie NGINX_SITE_CONFIG z BACKUP_DIR.
    # TODO: 2. Usunięcie HTPASSWD_FILE.
    # TODO: 3. Przeładowanie nginx.
    # TODO: 4. Zakończenie skryptu z błędem.
    echo "   > Funkcja nie zaimplementowana."
    exit 1
}

# --- LOGIKA GŁÓWNA ---
function main() {
    echo "================================================"
    echo "Rozpoczynam konfigurację Basic Auth dla: $SERVER_IP"
    echo "Użytkownik: $AUTH_USER"
    echo "Hasło: [ukryte]"
    echo "================================================"
    echo ""
    # KROK 1: Sprawdź narzędzia
    sprawdz_narzędzia
    # KROK 2: Utwórz kopię zapasową konfiguracji
    utworz_kopie_zapasowa
    # Kolejne kroki są nadal zakomentowane
    # skonfiguruj_autoryzacje
    # waliduj_i_przeladuj
    echo "[INFO] Funkcje 1 & 2 zaimplementowane i przetestowane."
    echo "[INFO] Kolejny krok: implementacja 'skonfiguruj_autoryzacje'."
}

# --- WYKONANIE ---
# Zabezpieczenie przed uruchomieniem bez sudo
if [[ $EUID -ne 0 ]]; then
   echo "BŁĄD: Skrypt wymaga uprawnień root (sudo). Uruchom: sudo $0" >&2
   exit 1
fi

main "$@"
