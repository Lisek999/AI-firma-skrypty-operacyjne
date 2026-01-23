#!/bin/bash
echo "=== WERYFIKACJA STATUSU DASHBOARD - CEO Tomek ==="
echo "Data: $(date)"
echo ""

echo "1. SZCZEGÓŁOWY STATUS SUPERVISOR:"
sudo supervisorctl status
echo ""

echo "2. PROCESY GUNICORN/PYTHON:"
ps aux | grep -E "(gunicorn|python.*app)" | grep -v grep
echo ""

echo "3. PORT NASŁUCHUJĄCY:"
sudo netstat -tlnp | grep :5000 || sudo ss -tlnp | grep :5000
echo ""

echo "4. TEST ENDPOINTU HEALTH:"
curl -s http://localhost:5000/api/health || echo "CURL nie działa na localhost:5000"
echo ""

echo "5. SPRAWDZENIE LOGÓW SUPERVISOR:"
sudo tail -5 /var/log/supervisor/supervisord.log 2>/dev/null || echo "Brak logów supervisor"
echo ""

echo "6. WERYFIKACJA INSTALACJI FLASK:"
python3 -c "import flask; print(f'Flask wersja: {flask.__version__}')" 2>/dev/null || echo "Flask nie zaimportowany"
echo ""

echo "7. KONFIGURACJA SUPERVISOR DASHBOARD:"
sudo cat /etc/supervisor/conf.d/dashboard.conf 2>/dev/null | head -20 || echo "Brak pliku konfiguracyjnego"
echo ""

echo "=== KONIEC WERYFIKACJI ==="
