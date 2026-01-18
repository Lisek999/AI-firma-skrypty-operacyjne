#!/bin/bash
# SKRYPT 2: Naprawa importu datetime w app.py
# Autor: Wojtek (AI)

cd /opt/ai_firma_dashboard || exit 1

# Backup
cp app.py app.py.backup_before_datetime_fix

# Dodanie 'from datetime import datetime' po pierwszym imporcie flask
if ! grep -q "from datetime import datetime" app.py; then
    sed -i '1 a from datetime import datetime' app.py
    echo "Dodano import: from datetime import datetime"
else
    echo "Import datetime już istnieje."
fi

# Restart gunicorn (ponowne zabicie i uruchomienie)
sudo pkill -f "gunicorn.*app:app"
sleep 2
cd /opt/ai_firma_dashboard
sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon
echo "Gunicorn zrestartowany z poprawionym importem."
