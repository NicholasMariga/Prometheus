PROMETHEUS WITHOUT DOCKER

//Update your server first
apt update -y

//Create user prometheus
sudo useradd --no-create-home --shell /bin/false prometheus

//Download zipped file from official prometheus website
wget prometheus......*.tar.gz 


//Extract the file
tar -xvf prometheus......*.tar.gz


//Change directory to the extracted one
cd prometheus......*

//Create Necessary Directories
mkdir /etc/prometheus /var/lib/prometheus

//Copy Binaries 
cp prometheus promtool /usr/local/bin
cp -r prometheus.yml consoles console_libraries /etc/prometheus


//Change Ownership
chown -R prometheus.prometheus /etc/prometheus/
chown -R prometheus.prometheus /var/lib/prometheus

//Update the targets in prometheus.yml to the right server details
vi /etc/prometheus/prometheus.yml
global:
  scrape_interval: 5s
  evaluation_interval: 1m
scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 10s
    static_configs:
      - targets: ['localhost:9090']


//Create a deamon service, to start prometheus onboot
vi /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target


// Enable and Start Prometheus Service

systemctl daemon-reload
systemctl enable prometheus
systemctl start prometheus

//On your Browser
ip-address:9090