#!/bin/bash
# SKRYPT: INSTALACJA GUNICORN DLA DASHBOARD V2
# Data: 2026-01-23

echo "=== INSTALACJA GUNICORN W VENV DASHBOARD V2 ==="

cd /var/www/dashboard_v2

# 1. Aktywuj venv i zainstaluj gunicorn
source venv/bin/activate
pip install gunicorn

# 2. Sprawdź czy gunicorn jest dostępny
if [ -f "venv/bin/gunicorn" ]; then
    echo "✅ Gunicorn zainstalowany: $(venv/bin/gunicorn --version)"
else
    echo "❌ Błąd instalacji gunicorn"
    exit 1
fi

# 3. Zrestartuj dashboard-v2
echo "Restartuję dashboard-v2..."
sudo supervisorctl restart dashboard-v2

sleep 3

# 4. Sprawdź status
echo "Status dashboard-v2:"
sudo supervisorctl status dashboard-v2

# 5. Test
echo "Test połączenia:"
curl -s http://localhost:5001/api/health && echo " ✅ Działa" || echo " ❌ Nie działa"
