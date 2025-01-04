# Custom-systemd-timer
Automatiserad säkerhetskopiering med Restic och Systemd

Översikt
Det här projektet automatiserar säkerhetskopiering av filer med hjälp av Restic, SSH och systemd. Säkerhetskopieringsprocessen utlöses automatiskt via en systemd-timer, vilket säkerställer att dina filer regelbundet sparas på en fjärrserver.


Förutsättningar

Två Linux-baserade system (Ubuntu som källan och Fedora som destinationen).
Restic installerat på båda systemen.
SSH konfigurerat för lösenordslös inloggning från källan till destinationen.



Steg

Installera nödvändiga verktyg



Ubuntu:

sudo apt-get update && sudo apt-get install -y restic openssh-client




Fedora:
sudo dnf install -y restic openssh-server


Konfigurera SSH



Generera en SSH-nyckel på Ubuntu:
ssh-keygen -t rsa -b 4096 -C "backup@fedora"


Kopiera SSH-nyckel till Fedora:
ssh-copy-id minerva@192.168.56.3



Initiera Restic-repositoryt på Fedora


Skapa säkerhetskopieringskatalogen och initiera repositoryt:
mkdir -p /home/minerva/restic-backup
restic init --repo /home/minerva/restic-backup


Konfigurera Firewall
4.1 I Ubuntu (källmaskin)


Tillåt utgående SSH-anslutningar:
sudo apt-get install -y ufw
sudo ufw allow out 22/tcp
sudo ufw enable
4.2 I Fedora (destinationsmaskin)
Tillåt inkommande SSH-anslutningar.
sudo dnf install -y firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
sudo systemctl enable firewalld
sudo systemctl start firewalld


Konfigurera säkerhetskopieringsskriptet
På Ubuntu, skapa skriptet vim /home/minerva/backup.sh:
#!/bin/bash


Variabler
RESTIC_REPO="sftp:minerva@192.168.56.3:/home/minerva/restic-backup"
RESTIC_PASSWORD="Pisanpu1"
BACKUP_SOURCE="/home/minerva/data-to-backup"

Exportera miljövariabler
export RESTIC_REPOSITORY=RESTICREPOexportRESTICPASSWORD=RESTIC_REPO export RESTIC_PASSWORD=RESTICR​EPOexportRESTICP​ASSWORD=RESTIC_PASSWORD

Kör säkerhetskopieringen
restic backup $BACKUP_SOURCE


Gör skriptet körbart:
chmod +x /home/minerva/backup.sh


Testa skriptet manuellt.
./backup.sh



Konfigurera Systemd-tjänsten och timern


Skapa tjänstfilen sudo vim /etc/systemd/system/backup.service

[Unit]
Description=Kör Restic-säkerhetskopiering
After=network.target
[Service]
Type=oneshot
ExecStart=/home/minerva/backup.sh
Environment="RESTIC_PASSWORD=Pisanpu1"
Environment="RESTIC_REPOSITORY=sftp:minerva@192.168.56.3:/home/minerva/restic-backup"
User=minerva
[Install]
WantedBy=multi-user.target


Ladda om tjänsten systemd:
sudo systemctl daemon-reload


starta om tjänsten systemd
sudo systemctl restart backup.service


Kontrollera tjänstens status:
sudo systemctl status backup.service


Skapa timerfilen /etc/systemd/system/backup.timer:
[Unit]
Description=Timer för Restic-säkerhetskopiering


[Timer]
OnBootSec=5min
Unit=backup.service
[Install]
WantedBy=timers.target


Ladda om systemkonfiguration:
sudo systemctl daemon-reload


Aktivera automatisk start av timern:
sudo systemctl enable backup.timer


Starta timern:
sudo systemctl start backup.timer


Hur man testar
1- För att manuellt utlösa säkerhetskopieringen:
sudo systemctl start backup.service
2- För att kontrollera tjänstens status:
sudo systemctl status backup.service
3- För att verifiera säkerhetskopiorna:
restic snapshots
Felsökning
1- Kontrollera systemd-loggar för fel:
sudo journalctl -xeu backup.service
2- Verifiera SSH-anslutningen:
ssh minerva@192.168.56.3
3- Testa Restic manuellt:
restic snapshots
Författare
Minerva González Brana
