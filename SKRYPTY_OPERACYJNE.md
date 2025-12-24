# SKRYPTY OPERACYJNE AI FIRMA
*Wygenerowane automatycznie. Nie edytuj ręcznie.*

## deploy_nginx_site.sh
```bash
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
```

## backup_configs.sh
```bash
#!/bin/bash
echo "[INFO] Ten skrypt będzie tworzył backup konfiguracji"
echo "[TODO]: Zaimplementuj backup"
```

## nazwa_skryptu_3.sh
```bash
#!/bin/bash
echo "Przykład - wklej tu nowy skrypt"
```

## nazwa_skryptu_4.sh
```bash
#!/bin/bash
echo "Kolejny przykład"
```
