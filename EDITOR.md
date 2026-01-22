#!/bin/bash
# SKRYPT: URUCHOMIENIE DASHBOARD V2 PRZEZ SUPERVISOR
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== URUCHAMIANIE DASHBOARD V2 PRZEZ SUPERVISOR ==="

# 1. BACKUP - LINIA OBOWIĄZKOWA (§2)
TIMESTAMP=$(date +%s)
echo "1. Tworzę backup konfiguracji Supervisor (jeśli istnieje)..."
sudo cp /etc/supervisor/conf.d/dashboard-v2.conf /etc/supervisor/conf.d/dashboard-v2.conf.backup_${TIMESTAMP} 2>/dev/null || echo "   Brak istniejącej konfiguracji v2"

# 2. Tworzymy konfigurację Supervisor dla dashboard-v2
echo "2. Tworzę konfigurację Supervisor dla dashboard-v2..."

cat > /tmp/dashboard-v2.conf << 'EOF'
[program:dashboard-v2]
command=/var/www/dashboard_v2/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5001 --access-logfile /var/log/dashboard-v2.access.log --error-logfile /var/log/dashboard-v2.error.log app:app
directory=/var/www/dashboard_v2
user=www-data
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stderr_logfile=/var/log/dashboard-v2.err.log
stdout_logfile=/var/log/dashboard-v2.out.log
environment=PYTHONUNBUFFERED="1"
EOF

# 3. Kopiujemy konfigurację
sudo cp /tmp/dashboard-v2.conf /etc/supervisor/conf.d/dashboard-v2.conf
echo "   Konfiguracja zapisana: /etc/supervisor/conf.d/dashboard-v2.conf"

# 4. Ustawiamy poprawne uprawnienia
echo "3. Ustawiam uprawnienia dla katalogu dashboard_v2..."
sudo chown -R www-data:www-data /var/www/dashboard_v2
sudo chmod -R 755 /var/www/dashboard_v2

# 5. Aktualizujemy i uruchamiamy Supervisor
echo "4. Uruchamiam dashboard-v2 przez Supervisor..."
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start dashboard-v2

# 6. Sprawdzamy status
echo "5. Sprawdzam status..."
sleep 2
sudo supervisorctl status dashboard-v2

# 7. Testujemy połączenie
echo "6. Testuję połączenie..."
curl -s http://localhost:5001/api/health && echo "   ✅ Dashboard v2 działa" || echo "   ❌ Dashboard v2 nie odpowiada"

echo ""
echo "=== PODSUMOWANIE ==="
echo "Dashboard v2 powinien być teraz dostępny pod:"
echo "http://57.128.247.215/  (przez Nginx)"
echo "http://57.128.247.215:5001/  (bezpośrednio)"
echo ""
echo "Jeśli nie działa, sprawdź logi:"
echo "sudo tail -f /var/log/dashboard-v2.err.log"
echo "=== ZAKOŃCZONO ==="
