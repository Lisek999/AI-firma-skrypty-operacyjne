# SKRYPTY OPERACYJNE AI FIRMA
*Wygenerowane automatycznie. Nie edytuj ręcznie.*

## deploy_nginx_site.sh
```bash
#!/bin/bash
# deploy_nginx_site.sh - Testowy skrypt IaC Iteracja 1
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
```

## backup_configs.sh
```bash
#!/bin/bash
# backup_configs.sh - Tworzy backup konfiguracji
echo "[INFO] Rozpoczynam backup konfiguracji..."
BACKUP_DIR="$HOME/backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup konfiguracji Nginx
if [ -d "/etc/nginx" ]; then
    sudo cp -r /etc/nginx "$BACKUP_DIR/nginx"
    echo "[INFO] Backup Nginx wykonany: $BACKUP_DIR/nginx"
fi

# Backup konfiguracji systemowych
if [ -f "/etc/hosts" ]; then
    cp /etc/hosts "$BACKUP_DIR/"
fi

echo "[SUCCESS] Backup zakończony: $BACKUP_DIR"
```

## test_system.sh
```bash
#!/bin/bash
# test_system.sh - Testuje podstawowe funkcje systemu
echo "=== TEST SYSTEMU AI FIRMA ==="
echo "Data: $(date)"
echo "Host: $(hostname)"
echo "Użytkownik: $(whoami)"
echo "Katalog: $(pwd)"
echo ""
echo "✅ Test zakończony pomyślnie"
```
