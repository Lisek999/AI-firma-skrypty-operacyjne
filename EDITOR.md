#!/bin/bash

# ============================================================================
# create_gold_image.sh - POPRAWIONA WERSJA
# Gold Image Creator - Faza 1 Stabilna
# 
# POPRAWKA: Ignoruje katalog gold_image_v1.0/ w walidacji Git
# ============================================================================

set -e
set -u

# --- KONFIGURACJA ---
REPO_ROOT="/home/ubuntu/ai_firma_dokumenty"
BACKUP_DIR="${REPO_ROOT}/gold_image_v1.0"
REPORT_FILE="${REPO_ROOT}/GOLD_IMAGE_v1.0.md"
TAG_NAME="v1.0-stable"
COMMIT_MSG="Gold Image - Faza 1 Stabilna"
UPDATE_MSG="Update report with final commit hash"

# --- PRZYKŁADOWA TABLICA PLIKÓW (DO ZASTĄPIENIA LISTĄ Z BRIEFU) ---
declare -a FILES_TO_BACKUP=(
    "/etc/nginx/nginx.conf"
    "/home/ubuntu/.bashrc"
    "/opt/moja_aplikacja/config.yaml"
    "/var/www/html/index.php"
)

# --- FUNKCJE POMOCNICZE ---
log_info() {
    echo "[INFO] $(date '+%Y-%m-%d %H:%M:%S') - $1"
}

log_warning() {
    echo "[WARNING] $(date '+%Y-%m-%d %H:%M:%S') - $1" >&2
}

log_error() {
    echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') - $1" >&2
    exit 1
}

# --- WALIDACJA WCZEŚNIEJSZA (pkt 3.3) - POPRAWIONA ---
validate_git_status() {
    log_info "Sprawdzanie stanu repozytorium Git..."
    
    # Sprawdzenie czy jesteśmy w repozytorium Git
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        log_error "Brak repozytorium Git w bieżącym katalogu!"
    fi
    
    # Sprawdzenie czy repozytorium jest "czyste" - IGNORUJEMY gold_image_v1.0/
    local git_changes=$(git status --porcelain | grep -v '^?? gold_image_v1.0/')
    
    if [[ -n "${git_changes}" ]]; then
        echo "================================================"
        echo "BŁĄD: Repozytorium ma niezcommitowane zmiany!"
        echo "Zmiany wykryte:"
        echo "${git_changes}"
        echo ""
        echo "Proszę wykonaj:"
        echo "  git status               # zobacz zmiany"
        echo "  git add .                # dodaj wszystko"
        echo "  git commit -m 'message'  # zcommituj"
        echo "lub:"
        echo "  git stash                # schowaj zmiany tymczasowo"
        echo "================================================"
        exit 1
    fi
    
    log_info "Repozytorium Git jest czyste - można kontynuować."
}

# --- PRZYGOTOWANIE KATALOGU (pkt 3.4) ---
prepare_backup_directory() {
    log_info "Przygotowywanie katalogu backup: ${BACKUP_DIR}"
    
    # Usunięcie poprzedniego katalogu (jeśli istnieje)
    if [[ -d "${BACKUP_DIR}" ]]; then
        rm -rf "${BACKUP_DIR}"
        log_info "Usunięto stary katalog backup."
    fi
    
    # Utworzenie nowego katalogu
    mkdir -p "${BACKUP_DIR}"
    log_info "Utworzono nowy katalog backup."
}

# --- KOPIOWANIE PLIKÓW (pkt 3.4) - POPRAWIONA OBSŁUGA BŁĘDÓW ---
copy_files_to_backup() {
    local copied_count=0
    local missing_count=0
    local error_count=0
    
    log_info "Rozpoczynanie kopiowania plików..."
    
    for source_file in "${FILES_TO_BACKUP[@]}"; do
        # Sprawdzenie czy plik istnieje
        if [[ ! -f "${source_file}" ]] && [[ ! -d "${source_file}" ]]; then
            log_warning "Plik nie istnieje, pomijam: ${source_file}"
            ((missing_count++))
            continue
        fi
        
        # Określenie docelowej ścieżki
        local target_file="${BACKUP_DIR}${source_file}"
        local target_dir=$(dirname "${target_file}")
        
        # Utworzenie katalogu docelowego
        mkdir -p "${target_dir}"
        
        # Kopiowanie pliku/katalogu z lepszym handlingiem błędów
        if cp -r "${source_file}" "${target_file}" 2>&1; then
            log_info "Skopiowano: ${source_file}"
            ((copied_count++))
        else
            local cp_error=$?
            log_warning "Błąd kopiowania (kod: ${cp_error}): ${source_file}"
            ((error_count++))
            # Kontynuuj mimo błędu
        fi
    done
    
    echo "================================================"
    echo "PODSUMOWANIE KOPIOWANIA:"
    echo "  Skopiowano plików: ${copied_count}"
    echo "  Pominięto plików:  ${missing_count}"
    echo "  Błędy kopiowania:  ${error_count}"
    echo "================================================"
    
    if [[ ${copied_count} -eq 0 ]]; then
        log_warning "Nie skopiowano żadnego pliku! Sprawdź listę FILES_TO_BACKUP."
        # NIE przerywamy - może to być celowe (pusta lista)
    fi
}

# --- TWORZENIE RAPORTU (pkt 3.5) ---
create_initial_report() {
    log_info "Tworzenie wstępnego raportu..."
    
    cat > "${REPORT_FILE}" << EOF
# GOLD IMAGE - v1.0-stable
## Raport wykonania zrzutu systemowego

**Data utworzenia:** $(date '+%Y-%m-%d %H:%M:%S')
**Tag:** ${TAG_NAME}
**Commit hash:** [PENDING - zostanie uzupełniony po commicie]

---

## Lista skopiowanych plików:

EOF
    
    # Dodanie listy plików do raportu
    for source_file in "${FILES_TO_BACKUP[@]}"; do
        local target_file="${BACKUP_DIR}${source_file}"
        if [[ -e "${target_file}" ]]; then
            echo "- ${source_file}" >> "${REPORT_FILE}"
        fi
    done
    
    cat >> "${REPORT_FILE}" << EOF

---

## Instrukcje przywracania:

Aby przywrócić pojedynczy plik:
\`\`\`bash
git checkout ${TAG_NAME} -- gold_image_v1.0/ścieżka/do/pliku
\`\`\`

Przykład dla nginx.conf:
\`\`\`bash
git checkout ${TAG_NAME} -- gold_image_v1.0/etc/nginx/nginx.conf
\`\`\`

---

**Uwaga:** Ten raport zostanie zaktualizowany o finalny hash commita po tagowaniu.
EOF
    
    log_info "Utworzono wstępny raport: ${REPORT_FILE}"
}

# --- OPERACJE GIT (pkt 3.6) ---
perform_git_operations() {
    log_info "Rozpoczynanie operacji Git..."
    
    # Przejście do katalogu repozytorium
    cd "${REPO_ROOT}"
    
    # Dodanie wszystkich nowych plików
    git add .
    
    # Commit z wiadomością
    git commit -m "${COMMIT_MSG}"
    
    # Pobranie hasha commita
    local commit_hash=$(git rev-parse HEAD)
    log_info "Utworzono commit: ${commit_hash}"
    
    # Utworzenie tagu lokalnego
    git tag "${TAG_NAME}"
    log_info "Utworzono lokalny tag: ${TAG_NAME}"
    
    # Wypchnięcie tylko taga do zdalnego repozytorium
    git push origin "${TAG_NAME}"
    log_info "Wypchnięto tag do zdalnego repozytorium."
    
    # Zwrócenie hasha commita
    echo "${commit_hash}"
}

# --- AKTUALIZACJA RAPORTU (pkt 3.8) ---
update_report_with_final_hash() {
    local final_hash="$1"
    
    log_info "Aktualizowanie raportu o finalny hash..."
    
    # Aktualizacja hasha w raporcie
    sed -i "s/Commit hash: \[PENDING.*\]/Commit hash: ${final_hash}/" "${REPORT_FILE}"
    
    # Dodanie sekcji z linkiem do GitHub
    cat >> "${REPORT_FILE}" << EOF

---

## Link do repozytorium:

\`\`\`
https://github.com/[twoja_nazwa_użytkownika]/ai_firma_dokumenty/releases/tag/${TAG_NAME}
\`\`\`

**Finalny hash commita:** \`${final_hash}\`
EOF
    
    # Commit zaktualizowanego raportu
    git add "${REPORT_FILE}"
    git commit -m "${UPDATE_MSG}"
    
    # Wypchnięcie zmian (bez tagowania)
    git push origin main
    
    log_info "Zaktualizowano raport i wypchnięto zmiany."
}

# --- PODSUMOWANIE (pkt 3.9) ---
print_summary() {
    local final_hash="$1"
    local copied_count=0
    
    # Liczenie skopiowanych plików
    for source_file in "${FILES_TO_BACKUP[@]}"; do
        local target_file="${BACKUP_DIR}${source_file}"
        if [[ -e "${target_file}" ]]; then
            ((copied_count++))
        fi
    done
    
    echo ""
    echo "================================================"
    echo "🎉 GOLD IMAGE v1.0-stable UTWORZONY POMYŚLNIE!"
    echo "================================================"
    echo ""
    echo "📊 PODSUMOWANIE:"
    echo "   • Skopiowanych plików: ${copied_count}"
    echo "   • Tag: ${TAG_NAME}"
    echo "   • Hash commita: ${final_hash}"
    echo "   • Katalog backup: ${BACKUP_DIR}"
    echo "   • Raport: ${REPORT_FILE}"
    echo ""
    echo "🔗 LINK DO ZDALNEGO REPOZYTORIUM:"
    echo "   https://github.com/[twoja_nazwa_użytkownika]/ai_firma_dokumenty/releases/tag/${TAG_NAME}"
    echo ""
    echo "✅ WERYFIKACJA:"
    echo "   1. git tag -l | grep v1.0-stable"
    echo "   2. Sprawdź tag na GitHubie"
    echo "   3. cat GOLD_IMAGE_v1.0.md"
    echo "================================================"
}

# --- GŁÓWNA FUNKCJA WYKONAWCZA ---
main() {
    log_info "Rozpoczynanie tworzenia Gold Image v1.0-stable..."
    log_info "Repozytorium: ${REPO_ROOT}"
    
    # Kolejność wykonywania zgodna z Planem Ataku
    validate_git_status          # pkt 3.3
    prepare_backup_directory     # pkt 3.4 (przygotowanie)
    copy_files_to_backup         # pkt 3.4 (kopiowanie)
    create_initial_report        # pkt 3.5
    final_hash=$(perform_git_operations)  # pkt 3.6 + 3.7
    update_report_with_final_hash "${final_hash}"  # pkt 3.8
    print_summary "${final_hash}"          # pkt 3.9
    
    log_info "Gold Image creation completed successfully!"
}

# --- URUCHOMIENIE SKRYPTU ---
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main
fi
