# Usuario root
````sh
# cambia al usuario root 
sudo –i
#habilitar
sudo –i passwd root
#deshabilitar
sudo passwd –dl root

````


# Configuracion de red
## Detectar interfaz de red
````sh
networkctl
networkctl status
ip a //hayque buscar lo que hay configurado
sudo lshw -class network //logical name
````
## Retplan
### DHCP
````sh
nano /etc/netplan/99_config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    <logical name interface>:
      dhcp4: true

sudo netplan try
sudo netplan apply
networkctl
networkctl status
````

### Stratic IP
````sh
network:
version: 2
renderer: NetworkManager
ethernets:
 ens33:
  dhcp4: no
  addresses:
  - 192.168.72.140/24
  gateway4: 192.168.72.2
  nameservers:
   addresses: [8.8.8.8, 8.8.4.4]

sudo netplan try
sudo netplan apply
networkctl
networkctl status
````
### nameservers
/etc/systemd/resolved.conf (nuevo /etc/resolve.conf)



## hostnames
````sh
nano /etc/hosts
127.0.0.1   localhost
127.0.1.1   ubuntu-server
10.0.0.11   server1 server1.example.com vpn
````


## Establecer ip temporales
````sh
sudo ip addr add <IP>/<CIDR> dev <logical name interface>
ip link set dev <logical name interface> up  //ifup and ifdown no rulan
ip link set dev <logical name interface> down
sudo ip route add default via <gateway>
````
### dns
````sh
nano /etc/systemd/resolved.conf (nuevo /etc/resolve.conf)
nameserver 8.8.8.8
nameserver 8.8.4.4

ip addr flush <logical name interface>
````

## old way /etc/network/interfaces

### DHCP
````sh
nano /etc/network/interfaces
iface eth0 inet dhcp
sudo /etc/init.d/networking restart
````
### static ip
````sh
nano /etc/network/interfaces
iface eth0 inet static
address 192.168.1.100
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.254
sudo /etc/init.d/networking restart
````

### DNS
````sh
sudo nano /etc/resolv.conf
search myisp.com
nameserver 192.168.1.254
nameserver 202.54.1.20
nameserver 202.54.1.30
````

# Servidor SSH

## utilidades de red 
````sh
sudo apt install net-tools
````

## Instalacion de servidor open ssh 
- Ojo hay un  /etc/ssh/ssh_config
````sh
sudo apt install openssh-server
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
sudo systemctl restart sshd
````

## modificacion de paramteros
````sh
sudo nano /etc/ssh/sshd_config
  #cambio de puertos
  Port 2200
  #cambio de disclaimer al conectarse
  Banner /etc/issue.net
  #permitir root remoto
  PermitRootLogin no
  # usuarios a denegar
  DenyUsers root
  # usuariso a permitir
  AllowUsers rob admin
  #tipos de autenticacion requeridos
  AuthenticationMethods
````

## No se arranque solo
 - el servicio sshd es un alias del ssh , pero los nombres d ela sconfiguraciones se refieren a cliente y servidor
````sh
sudo systemctl disable sshd
sudo systemctl enable ssh
````

## gestion de certificados
````sh
# se almacenan en ~/.ssh/id_rsa.pub (publica) y  ~/.ssh/id_rsa (privada)
# hayq ue copiar la publica en el servidor remoto en ~/.ssh/authorized_keys
#generar la clave
ssh-keygen -t rsa
# copia remota del certificado
ssh-copy-id username@remotehost
````

# Iptables
````sh
#borra todo
sudo iptables -F
#muestra las reglas definidas
sudo iptables -L 
#nueva cadena para loguear
#chains: input, forward, and output.
sudo iptables -N LOGGING
#añade reglas para la cadena , para que no haga nada
sudo iptables -A LOGGING -d 127.0.0.1 -j RETURN
#añade para que no deje pasar
sudo iptables -A INPUT -p tcp --dport 22  -s 10.10.10.10 -j DROP
#añade para que deje pasar
sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
# denegar todo al final (si se acepta todo en la politica)- OJO al orden!!!
sudo iptables -A INPUT -j DROP

#politicas por defecto 
sudo iptables --policy INPUT ACCEPT
#iptables -P INPUT DROP
sudo iptables --policy INPUT DROP      

##PAra dejarlo permanente hay variso metodos, usar 
sudo apt install -y iptables-save
sudo apt install -y iptables-persistent
#/etc/iptables/rules.v4
#para guardar las nuevas
iptables-save > /etc/iptables/rules.v4
````

````sh
## ejemplo loggig
iptables -N LOGGING
iptables -A LOGGING -d 127.0.0.1 -j RETURN
iptables -A LOGGING -d 192.168.25.185 -j RETURN
iptables -A LOGGING -d 156.35.33.185 -j RETURN
iptables -A LOGGING -p tcp -m state --state NEW -m limit --limit 1/minute --limit-burst 5 -j LOG --log-prefix "Nueva conexion OUTGOING TCP: "
iptables -A LOGGING -p udp -m state --state NEW -m limit --limit 1/minute --limit-burst 5 -j LOG --log-prefix "Nueva conexion OUTGOING UDP: "
iptables -A OUTPUT -j LOGGING

````



# Firewall ufw
````sh
sudo ufw enable/disable
#regals
sudo ufw allow/deny 22
#reglas numeradas
sudo ufw insert 1 allow 80
#elimina reglas
sudo ufw delete allow 80
#simtaxis apmpliada
sudo ufw allow proto tcp from 192.168.0.2/24 to any port 22

# muestra listado de aplicacine spredefinidas
sudo ufw app list
sudo ufw allow Samba
#simtaxis apmpliada
ufw allow from 192.168.0.0/24 to any app Samba


````





# Script CSV
````sh
#!/bin/bash
# Purpose: Read Comma Separated CSV File
INPUT=data.cvs
OLDIFS=$IFS
IFS=','
[ ! -f $INPUT ] && { echo "$INPUT file not found"; exit 99; }
while read flname dob ssn tel status
do
  echo "Name : $flname"
  echo "DOB : $dob"
  echo "SSN : $ssn"
  echo "Telephone : $tel"
  echo "Status : $status"
done < $INPUT
IFS=$OLDIFS
````


# OPENSSL
````sh
#generar clave
openssl genrsa -out yourdomain.key 2048
#crear csr
openssl req -newkey rsa:2048 -keyout PRIVATEKEY.key -out MYCSR.csr
#generar certificado
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
#ver contenido
openssl x509 -text -in yourdomain.crt -noout
````

# Apache
## Instalacion
````sh
sudo apt-get install apache2 apache2-doc apache2-utils
#habilitar mod_ssl ; a2dismod para deshabilitar
sudo a2enmod ssl  
#habilitar sitios ; a2dissite para deshabilitar
sudo a2ensite 000-default.conf
sudo a2ensite default-ssl.conf
sudo systemctl reload apache2

#verificar sintaxis
sudo httpd –t
sudo apachectl configtest

````

## Virtual hosts
````sh
#directiva para compatibilidad con DNS
#NameVirtualHost *:* 
NameVirtualHost *:80 
<VirtualHost 192.168.0.108:80>
  #correo de servicio
  ServerAdmin webmaster@example1.com
  #Ruta base de lo que se va a servir
  DocumentRoot /var/www/html/example1.com      
  #DNS al virtual hosts
  ServerName www.example1.com
  #Alias alternativos
  ServerAlias example.com www.example
  #Redireccion permanente con condiciones dentro de lso mismso server alias
  <If "%{HTTP_HOST} == 'www.example.com'">
    Redirect permanent / https://example.com/
  </If>
  <Directory />
        Options FollowSymLinks
        AllowOverride None
    </Directory>
    <Directory /XXXXXX>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride None
        Order allow,deny
        #Deny from all
        allow from all
    </Directory>
</VirtualHost>

<VirtualHost 192.168.0.108:80>
  ServerAdmin admin@example2.com
  DocumentRoot /var/www/html/example2.com
  ServerName www.example2.com
</VirtualHost>


<VirtualHost _default_:443>
 

  ServerName www.yourdomain.com
  DocumentRoot /usr/local/apache2/htdocs
  SSLEngine On
  SSLCertificateFile /path/to/your_domain_name.crt
  SSLCertificateKeyFile /path/to/your_private.key
  SSLCertificateChainFile /path/to/DigiCertCA.crt
  # etc...
</VirtualHost>
````
## Forzado HTTPS 
### Con rewrite
````sh
sudo a2enmod rewrite 

nano .htaccess (o donde correesponda)
RewriteEngine On 
RewriteCond %{HTTPS}  !=on 
RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L] 
````

### Con redirect en virtual host
````sh
Redirect permanent / https://www.yourdomain.com
````

## Rewrite
- Ejecuta en cascada a no ser que se indique la L

### ejemplo de redireccion
````sh
RewriteEngine On #Obligatorio siempre
RewriteRule ^my-old-url.html$ /my-new-url.html [R=301,L]
#RewriteRule <patron> <sustitucion> <parametros>
#^ indica cualquiercosa desde el inicio
#$ indica el final
#[R=301,L] indica Response 301 y L que no aplique mas
````
### eejmplo de formateo de direcciones
````sh
RewriteEngine On 
RewriteRule ^articles/([^/]+)/?$ display_article.php?articleId=$1 [L]
#([^/]+) es un subpatron que indica cualquier caracter menos uan barra repetido + 
#/? indica barra opcional
#$ indica el final
#[L]L que no aplique mas
#$1 indica el subpatron de entre parentesis (backreference)
````
### prohibir el acceos a recursos en funcion del origen
````sh
RewriteEngine On 
RewriteCond %{HTTP_REFERER} !^$ # si es cierta se sigue aplicando la siguiente rewrite rule
#! negador de condicion
#^$ si existe la cabecera %{HTTP_REFERER} 
RewriteCond %{HTTP_REFERER} !^http://(www.)?example.com/.*$ [NC]
#(www.)? www opcional
#.*$ cualquier cosa hasta el final
# sin HTTP_REFERER no empieza por example.com
# [NC] - No case sensitive
RewriteRule .+.(gif|jpg|png)$ - [F]
# .+. almenos un caracter seguido de punto y extension de imagen
# - significa no hacer sustitucion 
# [F] envía 403 Forbidden
````

### otros flags (van separados por comas)
- N - Empieza otra vez por el princio
- R - Redirect con el codigo suministrado

## Autenticacion
````sh
##ejecutable en  /usr/local/apache2/bin/htpasswd
htpasswd -c /usr/local/apache/passwd/passwords rbowen
````
````sh
<Directory "/usr/local/apache/htdocs/secret">
AuthType Basic
AuthName "Restricted Files"
# (Following line optional)
AuthBasicProvider file
AuthUserFile "/usr/local/apache/passwd/passwords"
Require user rbowen
#mejor que indicar un usuario
#Require valid-user
</Directory>

<Directory "/usr/local/apache/htdocs/secret">
AuthType Basic
AuthName "By Invitation Only"
# Optional line:
AuthBasicProvider file
AuthUserFile "/usr/local/apache/passwd/passwords"
AuthGroupFile "/usr/local/apache/passwd/groups" 
#formato ficheros grupos
#GroupName: rbowen dpitts sungo rshersey
Require group GroupName
</Directory>

````

## Autorizacion
````sh
<RequireAll>
    Require all granted
    Require not ip 192.168.205
    Require not host phishers.example.com moreidiots.example
</RequireAll>
````
````sh
<If "%{HTTP_USER_AGENT} == 'BadBot'">
    Require all denied
</If>
````

## Paginas de servicio personalizadas
````sh
ErrorDocument 500 /cgi-bin/crash-recover
ErrorDocument 500 "Sorry, our script crashed. Oh dear"
ErrorDocument 500 http://xxx/
ErrorDocument 404 /Lame_excuses/not_found.html
````

# Tomcat 9 
Se pueden instalar el modulo del manager y el modulo de los examples y del doc.

## Validacion de usuarios

````sh
sudo nano web.xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
        id="WebApp_ID" version="4.0">
        <session-config>
                <session-timeout>30</session-timeout>
        </session-config>
        <security-constraint>
                <display-name>AdminUserConstraint</display-name>
                <web-resource-collection>
                        <web-resource-name>Administration</web-resource-name>
                        <description />
                        <!-- <url-pattern>/faces/admin/*</url-pattern> -->
                        <!-- <url-pattern>/admin/*</url-pattern> -->
                        <url-pattern>/*</url-pattern>
                        <http-method>GET</http-method>
                        <http-method>POST</http-method>
                </web-resource-collection>
                <auth-constraint>
                        <role-name>tomcatseguro</role-name>
                </auth-constraint>
        </security-constraint>

        <login-config>
                <auth-method>BASIC</auth-method>
           <!-- <auth-method>FORM</auth-method> -->
           	   	<!-- <form-login-config>
                	 	<form-login-page>/login.html</form-login-page>
                        <form-error-page>/login.html</form-error-page>
                     </form-login-config> -->
        </login-config>
        <security-role>
                <description />
                <role-name>tomcatseguro</role-name>
        </security-role>
</web-app>
````

_____________________________________________
## Conexion AJP:
````sh
#instalar ajp en apache2
sudo a2enmod proxy_ajp

#configurar virtual host con 
sudo nano /etc/apache2/sites-enabled/000-default.conf
<Proxy *>
        Order deny,allow
        Allow from all
</Proxy>
ProxyRequests           Off
ProxyPass               /       ajp://127.0.0.1:8009/
ProxyPassReverse        /       ajp://127.0.0.1:8009/

#habilitar conector tomcat (OJO AL address y al secret)
   <Connector protocol="AJP/1.3"
               address="127.0.0.1"
               port="8009"
               redirectPort="8443"
               secretRequired="false"
               />
````





# Gestion de usuarios
- https://ubuntu.com/server/docs/security-users
````sh
sudo adduser/userdel usuario
sudo passwd usuario

#Visualizar usuairos del sistema
cat /etc/passwd

#cambia el home de un usuario
usermod -d -m /nuevohome/usuario usuario

sudo groupadd/groupdel grupo
sudo adduser/deluser usuario grupo
#Visualizar grupos del sistema
cat /etc/group
````

## propietarios
````sh
#cambiar propietario
sudo chown -R usuario ruta-directorio
#cambiar grupo
sudo chown -R :grupo ruta-directorio
````
## permisos
````sh
sudo chmod XYZ ruta-archivo
#usuario-grupo-restousuarios
#0 – Ningún permiso
#1 – Ejecución
#2 – Escritura
#3 – Escritura y ejecución
#4 – Lectura
#5 – Lectura y ejecución
#6 – Lectura y escritura
#7 – Lectura, escritura y ejecución
sudo chmod [u,g,o][=,+,-][r,w,x] ruta-archivo
sudo chmod u=rw,g=rw,o=r script.py
````
- Aparte de lso permisos normale sexisten unso permisos especiales.

## usuario sudoers
````sh
sudo nano /etc/sudoers
#<host> : [(<user list)] <command list | command alias> 
#<desde que host> : (<en nombre de que usuarios>) <que comandos>
#%grupo son lo grupos

#Sepuedne hace alias, con cosas separada por comas y ! para excluir
User_Alias MYSQL_USERS = andy, marce, juan, %mysql, !jose

````


# CRONTAB
- minuto hora dia(delmes) mes dia(semana)
- todos los dias a las 8
  0 8 * * *
- sabados y domingos a las 8
  0 8 * * 0,6
- Cada 3 horas
  0 */3 * * *

````sh 
#visualizar
crontab -l
#editar
crontab -e 
````

# Gestion delogs
-Existe uan aplicacion especifica logrotate, sino s epuede hacer a mano
````sh
sudo nano /etc/logrotate.d/apache2
logrotate -f /etc/logrotate.d/apache2
````

## con scrpt
````sh
#!/bin/bash
#comprime logs de mas de 7 dias
LOG_DIR="/var/logs/whatever_dir_you_want"
LOG_TAG="Log_20??-??-??.log"
(
cd "${LOG_DIR}" || exit
find . -name "${LOG_TAG}" -type f -mtime +7 | xargs gzip 
)
````

# DHCP

````sh
sudo apt-get install isc-dhcp-server -y
sudo nano /etc/dhcp/dhcpd.conf
  subnet 192.168.110.0 netmask 255.255.255.0 {
  range 192.168.110.5 192.168.1.10;
  option routers 192.168.110.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option domain-name "mydomain.example";
  default-lease-time 600;
  max-lease-time 7200;
  authoritative;
}
````
# FTP
Para el uso anonimo es el usuario ftp
````sh
sudo apt install vsftpd
nano /etc/vsftpd.conf
````

# samba
-El samaba es para mas cosas(impresoras, LDAP) , spero se utiliza para ficheros principalmente
````sh
sudo apt install samba
sudo nano /etc/samba/smb.conf
  [share]
      comment = Ubuntu File Server Share
      path = /srv/samba/share
      browsable = yes
      guest ok = yes
      read only = no
      create mask = 0755
sudo mkdir -p /srv/samba/share
sudo chown nobody:nogroup /srv/samba/share/
sudo systemctl restart smbd.service nmbd.service
````

# NFS
## server
````sh
sudo apt install nfs-kernel-server
sudo systemctl start nfs-kernel-server.service
nano /etc/exports
  /srv     *(ro,sync,subtree_check)
  /home    *.hostname.com(rw,sync,no_subtree_check)
  /scratch *(rw,async,no_subtree_check,no_root_squash,noexec)
sudo exportfs -a
````
## cliente
````sh
sudo apt install nfs-common
sudo mkdir /opt/example
sudo mount example.hostname.com:/srv /opt/example
nano /etc/fstab
````

# procesamiento de textos inliner
## cat 
## grep
## cut
- c columna
- f field
- d delimitador
````sh
# trocea por el caracter = y selecciona el segundo
cut -d'=' -f2
````
## sort
- f considera iguales las mayúsculas y minúsculas.
- n ordena por valor numérico.
- r invertirá el orden.

## uniq
- elimina duplicados 
- c añade contador de repeticiones
- i case insensitive
````sh
uniq -c file1
````

## wc
- l numero de lineas
- w numero de palabras

## awk
- procesa textos
- $0 - toda la linea
- $1 - primer campo
- -F= - Establece = como delimitador

````sh
# imprime columna 3 y 4 separados por tab
awk '{print $3 "\t" $4}' marks.txt
cat /var/log/kern.2018-12-05.log | grep Nueva | grep UDP | cut -d' ' -f12 | cut -d'=' -f2 > traficoUDP-185-temp.txt
````
    
# rsync
-n dry-run
-r recursivo
-z compresion
-a archive mode, preserva metadatos
````sh
rsync -n -avz --delete --itemize-changes --numeric-ids username@remote_host:/home/username/dir1 place_to_sync_on_local_machine
````



