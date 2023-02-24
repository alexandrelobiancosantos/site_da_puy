###############################################################################
# Replace
# landingpage to the name of the gunicorn file you want
# ubuntu to your user name
# app to the folder name of your project
# lp to the folder name where you find a file called wsgi.py
#
###############################################################################
# Criando o arquivo landingpage.socket
sudo nano /etc/systemd/system/landingpage.socket

###############################################################################
# Conteúdo do arquivo
[Unit]
Description=gunicorn blog socket

[Socket]
ListenStream=/run/landingpage.socket

[Install]
WantedBy=sockets.target

###############################################################################
# Criando o arquivo landingpage.service
sudo nano /etc/systemd/system/landingpage.service

###############################################################################
# Conteúdo do arquivo
[Unit]
Description=Gunicorn daemon (You can change if you want)
Requires=landingpage.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
Restart=on-failure
EnvironmentFile=/home/ubuntu/app/.env
WorkingDirectory=/home/ubuntu/app
# --error-logfile --enable-stdio-inheritance --log-level and --capture-output
# are all for debugging purposes.
ExecStart=/home/ubuntu/app/venv/bin/gunicorn \
          --error-logfile /home/ubuntu/app/gunicorn-error-log \
          --enable-stdio-inheritance \
          --log-level "debug" \
          --capture-output \
          --access-logfile - \
          --workers 6 \
          --bind unix:/run/landingpage.socket \
          lp.wsgi:application

[Install]
WantedBy=multi-user.target

###############################################################################
# Ativando
sudo systemctl start landingpage.socket
sudo systemctl enable landingpage.socket

# Checando
sudo systemctl status landingpage.socket
curl --unix-socket /run/landingpage.socket localhost
sudo systemctl status landingpage

# Restarting
sudo systemctl restart landingpage.service
sudo systemctl restart landingpage.socket
sudo systemctl restart landingpage

# After changing something
sudo systemctl daemon-reload

# Debugging
sudo journalctl -u landingpage.service
sudo journalctl -u landingpage.socket