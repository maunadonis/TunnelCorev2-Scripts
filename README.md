#!/bin/bash

timedatectl set-timezone Asia/Riyadh
install_require () {
clear
echo 'Installing dependencies.'
{
export DEBIAN_FRONTEND=noninteractive
apt update
apt install -y curl wget cron python-minimal libpython-stdlib
apt install -y iptables
apt install -y screen squid stunnel4 dropbear gnutls-bin python
apt install -y dos2unix nano unzip jq virt-what net-tools default-mysql-client
apt install -y mlocate dh-make libaudit-dev build-essential fail2ban
clear
}&>/dev/null
clear
}

install_squid(){
clear
echo 'Installing proxy.'
{
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
echo "deb http://ftp.debian.org/debian/ jessie main contrib non-free
    deb-src http://ftp.debian.org/debian/ jessie main contrib non-free
    deb http://security.debian.org/ jessie/updates main contrib
    deb-src http://security.debian.org/ jessie/updates main contrib
    deb http://ftp.debian.org/debian/ jessie-updates main contrib non-free
    deb-src http://ftp.debian.org/debian/ jessie-updates main contrib non-free" >> /etc/apt/sources.list
    apt update
    apt install -y gcc-4.9 g++-4.9
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 10
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.9 10
    update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30
    update-alternatives --set cc /usr/bin/gcc
    update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 30
    update-alternatives --set c++ /usr/bin/g++
    cd /usr/src
    wget http://www.squid-cache.org/Versions/v3/3.1/squid-3.1.23.tar.gz
    tar zxvf squid-3.1.23.tar.gz
    cd squid-3.1.23
    ./configure --prefix=/usr \
      --localstatedir=/var/squid \
      --libexecdir=/usr/lib/squid \
      --srcdir=. \
      --datadir=/usr/share/squid \
      --sysconfdir=/etc/squid \
      --with-default-user=proxy \
      --with-logdir=/var/log/squid \
      --with-pidfile=/var/run/squid.pid
    make -j$(nproc)
    make install
    wget --no-check-certificate -O /etc/init.d/squid http://firenetvpn.net/files/slowdns/squid.sh
    chmod +x /etc/init.d/squid
    update-rc.d squid defaults
    chown -cR proxy /var/log/squid
    squid -z
    cd /etc/squid/
    rm squid.conf
    echo "acl Firenet dst `curl -s https://api.ipify.org`" >> squid.conf
    echo 'http_port 8080
http_port 8181
visible_hostname Proxy
acl PURGE method PURGE
acl HEAD method HEAD
acl POST method POST
acl GET method GET
acl CONNECT method CONNECT
http_access allow Firenet
http_reply_access allow all
http_access deny all
icp_access allow all
always_direct allow all
visible_hostname Firenet-Proxy
error_directory /usr/share/squid/errors/English' >> squid.conf
    cd /usr/share/squid/errors/English
    rm ERR_INVALID_URL
    echo '<!--FirenetDev--><!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><title>SECURE PROXY</title><meta name="viewport" content="width=device-width, initial-scale=1"><meta http-equiv="X-UA-Compatible" content="IE=edge"/><link rel="stylesheet" href="https://bootswatch.com/4/slate/bootstrap.min.css" media="screen"><link href="https://fonts.googleapis.com/css?family=Press+Start+2P" rel="stylesheet"><style>body{font-family: "Press Start 2P", cursive;}.fn-color{color: #ffff; background-image: -webkit-linear-gradient(92deg, #f35626, #feab3a); -webkit-background-clip: text; -webkit-text-fill-color: transparent; -webkit-animation: hue 5s infinite linear;}@-webkit-keyframes hue{from{-webkit-filter: hue-rotate(0deg);}to{-webkit-filter: hue-rotate(-360deg);}}</style></head><body><div class="container" style="padding-top: 50px"><div class="jumbotron"><h1 class="display-3 text-center fn-color">SECURE PROXY</h1><h4 class="text-center text-danger">SERVER</h4><p class="text-center">😍 %w 😍</p></div></div></body></html>' >> ERR_INVALID_URL
    chmod 755 *
    /etc/init.d/squid start
cd /etc || exit
wget 'https://pastebin.com/raw/xtPc5t1k' -O /etc/socks.py
dos2unix /etc/socks.py
chmod +x /etc/socks.py
rm /etc/apt/sources.list
sudo cp /etc/apt/sources.list_backup /etc/apt/sources.list
} &>/dev/null
}

install_firewall_kvm () {
clear
echo "Installing iptables."
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
{
iptables -t filter -A INPUT -p udp -m udp --dport 20100:20900 -m state --state NEW -m recent --update --seconds 30 --hitcount 10 --name DEFAULT --mask 255.255.255.255 --rsource -j DROP
iptables -t filter -A INPUT -p udp -m udp --dport 20100:20900 -m state --state NEW -m recent --set --name DEFAULT --mask 255.255.255.255 --rsource
iptables-save > /etc/iptables_rules.v4
ip6tables-save > /etc/iptables_rules.v6
}&>/dev/null
}

install_stunnel() {
  {
cd /etc/stunnel/ || exit

echo "-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQClmgCdm7RB2VWK
wfH8HO/T9bxEddWDsB3fJKpM/tiVMt4s/WMdGJtFdRlxzUb03u+HT6t00sLlZ78g
ngjxLpJGFpHAGdVf9vACBtrxv5qcrG5gd8k7MJ+FtMTcjeQm8kVRyIW7cOWxlpGY
6jringYZ6NcRTrh/OlxIHKdsLI9ddcekbYGyZVTm1wd22HVG+07PH/AeyY78O2+Z
tbjxGTFRSYt3jUaFeUmWNtxqWnR4MPmC+6iKvUKisV27P89g8v8CiZynAAWRJ0+A
qp+PWxwHi/iJ501WdLspeo8VkXIb3PivyIKC356m+yuuibD2uqwLZ2//afup84Qu
pRtgW/PbAgMBAAECggEAVo/efIQUQEtrlIF2jRNPJZuQ0rRJbHGV27tdrauU6MBT
NG8q7N2c5DymlT75NSyHRlKVzBYTPDjzxgf1oqR2X16Sxzh5uZTpthWBQtal6fmU
JKbYsDDlYc2xDZy5wsXnCC3qAaWs2xxadPUS3Lw/cjGsoeZlOFP4QtV/imLseaws
7r4KZE7SVO8dF8Xtcy304Bd7UsKClnbCrGsABUF/rqA8g34o7yrpo9XqcwbF5ihQ
TbnB0Ns8Bz30pjgGjJZTdTL3eskP9qMJWo/JM76kSaJWReoXTws4DlQHxO29z3eK
zKdxieXaBGMwFnv23JvXKJ5eAnxzqsL6a+SuNPPN4QKBgQDQhisSDdjUJWy0DLnJ
/HjtsnQyfl0efOqAlUEir8r5IdzDTtAEcW6GwPj1rIOm79ZeyysT1pGN6eulzS1i
6lz6/c5uHA9Z+7LT48ZaQjmKF06ItdfHI9ytoXaaQPMqW7NnyOFxCcTHBabmwQ+E
QZDFkM6vVXL37Sz4JyxuIwCNMQKBgQDLThgKi+L3ps7y1dWayj+Z0tutK2JGDww7
6Ze6lD5gmRAURd0crIF8IEQMpvKlxQwkhqR4vEsdkiFFJQAaD+qZ9XQOkWSGXvKP
A/yzk0Xu3qL29ZqX+3CYVjkDbtVOLQC9TBG60IFZW79K/Zp6PhHkO8w6l+CBR+yR
X4+8x1ReywKBgQCfSg52wSski94pABugh4OdGBgZRlw94PCF/v390En92/c3Hupa
qofi2mCT0w/Sox2f1hV3Fw6jWNDRHBYSnLMgbGeXx0mW1GX75OBtrG8l5L3yQu6t
SeDWpiPim8DlV52Jp3NHlU3DNrcTSOFgh3Fe6kpot56Wc5BJlCsliwlt0QKBgEol
u0LtbePgpI2QS41ewf96FcB8mCTxDAc11K6prm5QpLqgGFqC197LbcYnhUvMJ/eS
W53lHog0aYnsSrM2pttr194QTNds/Y4HaDyeM91AubLUNIPFonUMzVJhM86FP0XK
3pSBwwsyGPxirdpzlNbmsD+WcLz13GPQtH2nPTAtAoGAVloDEEjfj5gnZzEWTK5k
4oYWGlwySfcfbt8EnkY+B77UVeZxWnxpVC9PhsPNI1MTNET+CRqxNZzxWo3jVuz1
HtKSizJpaYQ6iarP4EvUdFxHBzjHX6WLahTgUq90YNaxQbXz51ARpid8sFbz1f37
jgjgxgxbitApzno0E2Pq/Kg=
-----END PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIIDRTCCAi2gAwIBAgIUOvs3vdjcBtCLww52CggSlAKafDkwDQYJKoZIhvcNAQEL
BQAwMjEQMA4GA1UEAwwHS29ielZQTjERMA8GA1UECgwIS29iZUtvYnoxCzAJBgNV
BAYTAlBIMB4XDTIxMDcwNzA1MzQwN1oXDTMxMDcwNTA1MzQwN1owMjEQMA4GA1UE
AwwHS29ielZQTjERMA8GA1UECgwIS29iZUtvYnoxCzAJBgNVBAYTAlBIMIIBIjAN
BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApZoAnZu0QdlVisHx/Bzv0/W8RHXV
g7Ad3ySqTP7YlTLeLP1jHRibRXUZcc1G9N7vh0+rdNLC5We/IJ4I8S6SRhaRwBnV
X/bwAgba8b+anKxuYHfJOzCfhbTE3I3kJvJFUciFu3DlsZaRmOo64p4GGejXEU64
fzpcSBynbCyPXXXHpG2BsmVU5tcHdth1RvtOzx/wHsmO/DtvmbW48RkxUUmLd41G
hXlJljbcalp0eDD5gvuoir1CorFduz/PYPL/AomcpwAFkSdPgKqfj1scB4v4iedN
VnS7KXqPFZFyG9z4r8iCgt+epvsrromw9rqsC2dv/2n7qfOELqUbYFvz2wIDAQAB
o1MwUTAdBgNVHQ4EFgQUcKFL6tckon2uS3xGrpe1Zpa68VEwHwYDVR0jBBgwFoAU
cKFL6tckon2uS3xGrpe1Zpa68VEwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0B
AQsFAAOCAQEAYQP0S67eoJWpAMavayS7NjK+6KMJtlmL8eot/3RKPLleOjEuCdLY
QvrP0Tl3M5gGt+I6WO7r+HKT2PuCN8BshIob8OGAEkuQ/YKEg9QyvmSm2XbPVBaG
RRFjvxFyeL4gtDlqb9hea62tep7+gCkeiccyp8+lmnS32rRtFa7PovmK5pUjkDOr
dpvCQlKoCRjZ/+OfUaanzYQSDrxdTSN8RtJhCZtd45QbxEXzHTEaICXLuXL6cmv7
tMuhgUoefS17gv1jqj/C9+6ogMVa+U7QqOvL5A7hbevHdF/k/TMn+qx4UdhrbL5Q
enL3UGT+BhRAPiA1I5CcG29RqjCzQoaCNg==
-----END CERTIFICATE-----" >> stunnel.pem

echo "debug = 0
output = /tmp/stunnel.log
cert = /etc/stunnel/stunnel.pem
[ssh]
connect = 22  
accept = 443
" >> stunnel.conf

sed -i "s|PORT_TCP|$PORT_TCP|g" /etc/stunnel/stunnel.conf
sed -i "s|PORT_UDP|$PORT_UDP|g" /etc/stunnel/stunnel.conf
cd /etc/default && rm stunnel4

echo 'ENABLED=1
FILES="/etc/stunnel/*.conf"
OPTIONS=""
PPP_RESTART=0
RLIMITS=""' >> stunnel4 

chmod 755 stunnel4

sed -i 's/NO_START=1/NO_START=0/g' /etc/default/dropbear
sed -i "s/DROPBEAR_PORT=22/DROPBEAR_PORT=442/g" /etc/default/dropbear
echo "/bin/false" >> /etc/shells

sudo service stunnel4 restart
sudo service dropbear restart
  } &>/dev/null
}

install_sudo(){
  {
    /usr/sbin/useradd -p $(openssl passwd -1 tunnelcorev2) -s /bin/false -M tunnelcorev2 &> /dev/null;
  }&>/dev/null
}

install_rclocal(){
  {
    sudo systemctl restart stunnel4
    
    echo "[Unit]
Description=firenet service
Documentation=http://firenetvpn.com

[Service]
Type=oneshot
ExecStart=/bin/bash /etc/rc.local
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/firenet.service
    echo '#!/bin/sh -e
iptables-restore < /etc/iptables_rules.v4
ip6tables-restore < /etc/iptables_rules.v6
sysctl -p
service stunnel4 restart
screen -dmS socks python /etc/socks.py
exit 0' >> /etc/rc.local
    sudo chmod +x /etc/rc.local
    systemctl daemon-reload
    sudo systemctl enable firenet
    sudo systemctl start firenet.service
  }&>/dev/null
}

start_service () {

echo 'Starting..'
{

sudo crontab -l | { echo "* * * * * pgrep -x stunnel4 >/dev/null && echo 'GOOD' || /etc/init.d/stunnel4 restart"; } | crontab -
sudo systemctl restart cron
} &>/dev/null

echo '++++++++++++++++++++++++++++++++++'
echo '*       SSH  is ready!    *'
echo '+++++++++++************+++++++++++'
echo -e "[IP] : $server_ip\n[OpenSSH Port] : 22\n[Ssl Port] : 443\n[Proxy Socks ] : 80\n[Proxy Squid 1] : 8080\n[Proxy Squid 2] : 3128\n"
history -c;
rm ~/.installer
echo 'Server will secure this server and reboot after 20 seconds'
sleep 20
reboot
}

server_ip=$(curl -s https://api.ipify.org)
server_interface=$(ip route get 8.8.8.8 | awk '/dev/ {f=NR} f&&NR-1==f' RS=" ")

install_require
install_sudo  
install_squid
install_firewall_kvm
install_stunnel
install_rclocal
start_service
