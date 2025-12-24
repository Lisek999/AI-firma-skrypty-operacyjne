### DEPLOY_NGINX
#!/bin/bash
echo "[INFO] Rozpoczynam test deployu Nginx..."
CONF_SRC="$HOME/ai-firma-vps/konfiguracje/nginx/sites-available/default"
CONF_DST="/etc/nginx/sites-available/default"

if [ ! -f "$CONF_SRC" ]; then
    echo "[ERROR] Nie znaleziono pliku źródłowego: $CONF_SRC"
    exit 1
fi

echo "[INFO] Kopiuję konfigurację i testuję..."
sudo cp -v "$CONF_SRC" "$CONF_DST"
if sudo nginx -t; then
    echo "[SUCCESS] Konfiguracja poprawna. Przeładowuję Nginx..."
    sudo systemctl reload nginx
    echo "[DONE] Deploy zakończony pomyślnie."
else
    echo "[ERROR] Błąd konfiguracji Nginx. Zmiany nie zostały zastosowane."
    exit 1
fi

### BACKUP_CONFIGS
(Miejsce na następny skrypt)

### NAZWA_SKRYPTU_3
#!/bin/bash
echo "Przykład - wklej tu nowy skrypt"

### NAZWA_SKRYPTU_4
#!/bin/bash
echo "Kolejny przykład"
