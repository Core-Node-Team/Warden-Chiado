## Oracle İndirelim
```
cd $HOME
rm -rf slinky
git clone https://github.com/skip-mev/slinky.git
cd slinky
git checkout v1.0.12
make build
mv build/slinky /usr/local/bin/
```
### Servis oluşturalım
```
echo 'export WARDEN_PORT="19"' >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
sudo tee /etc/systemd/system/slinkyd.service > /dev/null <<EOF
[Unit]
Description=Warden Slinky Oracle
After=network-online.target

[Service]
User=$USER
ExecStart=$(which slinky) --market-map-endpoint 127.0.0.1:${WARDEN_PORT}090 --port ${WARDEN_PORT}080
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Başlatalım
```
sudo systemctl daemon-reload
sudo systemctl enable slinkyd
sudo systemctl start slinkyd
```
### Warden Oracle Ayarları
```
nano /root/.warden/config/app.toml
```
CTRL+W ile Oracle aratalım. Aşağıdaki gibi bir kısım olacak gördüğünüz şekilde düzenleyelim. Sonra CTRL+X Y ENTER.
```
###############################################################################
###                                  Oracle                                 ###
###############################################################################
[oracle]
# Enabled indicates whether the oracle is enabled.
enabled = "true"

# Oracle Address is the URL of the out of process oracle sidecar. This is used to
# connect to the oracle sidecar when the application boots up. Note that the address
# can be modified at any point, but will only take effect after the application is
# restarted. This can be the address of an oracle container running on the same
# machine or a remote machine.
oracle_address = "127.0.0.1:19080"

# Client Timeout is the time that the client is willing to wait for responses from 
# the oracle before timing out.
client_timeout = "2s"

# MetricsEnabled determines whether oracle metrics are enabled. Specifically
# this enables instrumentation of the oracle client and the interaction between
# the oracle and the app.
metrics_enabled = "true"
```
### yenilden başlatalım
```
sudo systemctl daemon-reload && sudo systemctl restart wardend && sudo systemctl restart slinkyd
```
### warden loglarını kontrol edelim
```
sudo journalctl -u wardend -f -o cat
```
### oracle loglarını kontrol edelim
```
journalctl -fu slinkyd --no-hostname
```
### Price çıktısı gelmesi lazım
```
curl localhost:19080/slinky/oracle/v1/prices | jq
```
### Tamamen silme
```
cd $HOME
sudo systemctl stop slinkyd
sudo systemctl disable slinkyd
sudo rm -rf /etc/systemd/system/slinkyd.service
sudo systemctl daemon-reload
sudo rm -f /usr/local/bin/slinkyd
sudo rm -f $(which slinky)
sudo rm -rf $HOME/.slinky $HOME/slinky
```
