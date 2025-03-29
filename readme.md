# Network Security and design Project

![Image: "ARCHITECTURE"](./ARCHITECTURE.png)

## IOS IMAGES UTILISEES

**Routers** :

- CE11, CE21, CE12, CE22, PE1, PE2, P1 et P2: c7200-adventerprisek9-mz.152-4.M7.image

**Switches**:

- L3 Switchs: Federateur1, Federateur2: (EtherSwitch) c3745-adventerprisek9-mz123-26.image
- L2 Switchs: SW1, SW2, SW3, SW4: Nous avons utilisé QEMU vios_l2-adventerprisek9-m.03.2017.qcow2

**Computing assets**:

- PC0, PC1, PC2, PC3: VPCs GNS3

## CONFIGURATION ASA

**ACCESS LIST OUTSIDE-INSIDE** :

- access-list OUTSIDE-IN extended permit ip any any
- access-list OUTSIDE-IN extended permit ip any any echo-reply
- access-group OUTSIDE-IN in interface OUTSIDE

**ACCESS LIST DMZ-INSIDE** :

- access-list DMZ-IN extended permit icmp any any
- access-list DMZ-IN extended permit icmp any any echo-reply
- access-group DMZ-IN in interface DMZ

activation-key 0xb23bcf4a 0x1c713b4f 0x7d53bcbc 0xc4f8d09c 0x0e24c6b6

##   RADIUS CLIENT CONFIGURATION

- conf t
- aaa new-model
- aaa authentication login default group radius
- radius-server host 192.168.20.201 auth-port 1812 acct-port 1813 key tekup

**OU**:

- aaa new-model
- aaa group server radius ADM_RADIUS
- server-private IP_SRV_RADIUS key 0 votre_clé_radius_en_clair
- aaa authentication login default local group ADM_RADIUS
- aaa authorization exec default local group ADM_RADIUS

## SNMP CLIENT CONFIGURATION

- conf t
- snmp-server community private RW
- snmp-server host 192.168.20.201 version 2c private udp-port 161
- snmp-server enable traps entity-sensor threshold

**Test SNMP**:

- snmpwalk -v 2c -c private @IPhost

## CONFIGURING RADIUS SERVEUR WITHIN CentOS

### All of the Computing servers should be within the same Network

- yum update
- yum install freeradius freeradius-utils freeradius-mysql freeradius-perl -y

**Key generation** :

- cd  /etc/raddb/certs
- ./bootstrap

- systemctl status radiusd
- systemctl enable --now radiusd

**Firewall** :

- sudo firewall-cmd --add-service={http,https,radius} --permanent
- sudo firewall-cmd --reload

**Config check with debugging** :

- /usr/sbin/radiusd -C -lstdout -xxx

** Add users to clients.conf and their password** :

- cd /etc/raddb/
- vim clients.conf

** Configure clients.conf with the below configuration** :

	client username {
	    secret = valeur_du_secret
	    ipaddr = valeur_de_l'IPv4
	}

**In users file, add the pasword** :

- username Cleartext-Password := "Valeur_du_password"

systemctl restart radiusd

** Test in radius server** :

- radtest  \<user>  \<password>  <Radius_server_IP>  <NAS_port> \<Secret>

**Example** : radtest leo pass 127.0.0.1 0 secret

## CONFIGURING NAGIOS IN CentOS

**Packages Installation** :

- yum install -y httpd httpd-tools php gcc glibc glibc-common gd gd-devel make net-snmp

**User Creation and  groups for nagios** :

- useradd nagios
- groupadd nagcmd

**Modifying group permissions** :

- usermod -G nagcmd nagios
- usermod -G nagcmd apache

**Directory creation for NAGIOS installation** :

- mkdir /root/nagios
- cd /root/nagios

**Download archives of Nagios and its plugins** :

- wget <https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz>;
- wget <https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz>;

**Decompress** :

- tar -xvf nagios-4.4.6.tar.gz
- tar -xvf nagios-plugins-2.2.1.tar.gz

**Move them to the approapriate directory** :

- cd nagios-4.4.6/

**Start the configuration** :

- ./configure --with-command-group=nagcmd

**Compiling and installing** :

- make all
- make install
- make install-init
- make install-commandmode
- make install-config

**Install web interface** :

- make install-webconf

**Create a password for the web interface** :

- htpasswd -s -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

**Start httpd** :

- systemctl start httpd.service

**Configure Firewall** :

- firewall-cmd --zone=public --add-port=80/tcp --permanent
- firewall-cmd --reload

**Plugins Setup** :

- cd /root/nagios
- cd nagios-plugins-2.2.1/

**Installing** :

- ./configure --with-nagios-user=nagios --with-nagios-group=nagios

**Compilation and installation** :

- make all
- make install

**Verification** :

- /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

**SELINUX Adjustment** :

- getenforce
- setenforce 0

**Services adjustment** :

- systemctl enable nagios
- systemctl enable httpd

**Nagios configuration file** :

- /usr/local/nagios/etc/cgi.cfg

## Helpful resources

- MySql: <https://unixcop.com/how-to-install-mysql-on-centos-9-stream/>
- Zabbix: <https://www.zabbix.com/download?zabbix=6.4&os_distribution=centos&os_version=9&components=server_frontend_agent&db=mysql&ws=apache>
