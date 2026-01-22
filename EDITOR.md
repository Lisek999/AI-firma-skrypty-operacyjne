#!/bin/bash
# Skrypt aktualizacji funkcji backup_status()

cat > /tmp/update_backup_endpoint.py << 'EOF'
import os

# Odczytaj obecny plik app.py
with open('/opt/ai_firma_dashboard/app.py', 'r') as f:
    lines = f.readlines()

# Znajdź funkcję backup_status()
in_function = False
new_lines = []
for line in lines:
    if line.strip().startswith('def backup_status():'):
        in_function = True
    
    if in_function and line.strip() == '# Pobierz rzeczywiste daty modyfikacji skryptów':
        # Zastąp komentarz bardziej opisowym
        new_lines.append('    # Pobierz daty ostatnich wykonan dla każdego backupu\n')
        continue
    
    if in_function and 'mtime = os.path.getmtime(path)' in line:
        # Zastąp tę linię nową logiką
        new_lines.append('            # Dla daily sprawdzamy log, dla innych - datę skryptu\n')
        new_lines.append('            if name == "daily":\n')
        new_lines.append('                log_path = "/var/log/dashboard_backup.log"\n')
        new_lines.append('                if os.path.exists(log_path):\n')
        new_lines.append('                    mtime = os.path.getmtime(log_path)\n')
        new_lines.append('                else:\n')
        new_lines.append('                    mtime = os.path.getmtime(path)  # Fallback do skryptu\n')
        new_lines.append('            else:\n')
        new_lines.append('                mtime = os.path.getmtime(path)  # Weekly i Gold Image\n')
        continue
    
    new_lines.append(line)

# Zapisz zmodyfikowany plik
with open('/opt/ai_firma_dashboard/app.py', 'w') as f:
    f.writelines(new_lines)

print("✅ Zaktualizowano funkcję backup_status()")
print("🔁 Restartuję usługę Dashboard...")

# Restart Dashboard
os.system("sudo systemctl restart ai_firma_dashboard")
EOF

python3 /tmp/update_backup_endpoint.py
