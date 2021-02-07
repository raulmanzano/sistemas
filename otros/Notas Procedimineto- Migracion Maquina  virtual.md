#Post importacion Mieres Oviedo

<!-- MarkdownTOC -->

- Eliminar interfaces de red existentes
- Cambio del nombre de la maquina en XenCenter
- Restitucion de discos eliminados en origen
- Arrancar
- Problermas con el montaje de discos
- Parar servicios
- Deshabilitar servicios en el arranque
- NFS
      - Cliente
      - Servidor
- Particionar
- Cambiar nombre de la maquina
- Eliminar referencias DNS
- cambiar el servidor NTP
- Cambiar prompt
- Cambiar password
- Cambiar password otros usuarios
- Cambiar password de usuairos de la BBDD
      - actividad y contenido
      - Revisar si hay Mysql
- Cambiar certificado de acceso
- Xentools
- Configurar IPs y DSR
      - Configuracion de IP basicas
      - Configuracion para que funciona el DSR
      - Configuracion de ips VIP para DSR donde sea necesario
- Apagar
- Asignar interfaces de red XenCenter
- Arrancar
- Certificados SSL
- Postgre - Autorizacion de conexiones desde el nuevo rango
- autorizacion de nuevos sercvidores a los que accede mediante postgres
- Sincronizar discos externos y/o recursos
      - Sincronizar codigo o recursos estaticos \(no datos\)
      - CRON
            - para saber que hay
      - BBDD
            - hay que autorizar el acceso antes en el pg_hba.conf
            - Hay que cambiar la password en /root/.pgpass para que funicone la importación
            - Importar
- actividad y contenido
            - En caso de migracion parcial - Recordar deshabilitar\(Cambiar de nombre\) las BBDD de Mieres para que no se creen dos origenes operativos de datos
      - NAS
      - Indices locales o SOLR
- Limpieza de logs y basura
- Limpiar directorio de administracion \(Manual\)
- Cambiar IPs en todos los ficheros de texto
      - localizar referencias a innova
      - Localizar IPs a cambiar y ver si lso cambios son los esperados
      - Cambiar las ips a las ips externas
      - Cambiar las ips de los nuevos servidores
      - Buscar referencias dentro de jars
      - BUscar referencias a equipos locales de Mieres
- Revisar enlaces de conexiones cluster
- Habilitar servicios en el arranque
- Apagar
- Arrancar y revisar logs
- Netscaler
- Cambio de certificados para https
- Revision funcional
- POR COMPLETAR - Copias de seguridad
- POR COMPLETAR - Monitorización nagios
      - Instalar zabbix agent

<!-- /MarkdownTOC -->



#Eliminar interfaces de red existentes

#Cambio del nombre de la maquina en XenCenter
	-Incluir IP con el mismo nombre

#Restitucion de discos eliminados en origen
- Recreación de nuevos discos virtuales en sustitucion de los que hayan podido ser eliminados (o simplemente reconectar)
	- Deben estar y tener los mismos nombres que en el entorno de origen
- Si se eliminan hay que eliminar el automontaje!!!

#Arrancar

# Problermas con el montaje de discos
      El shel despues de los errores de disco no permite realizar todas la soperaciones. Reboot
      O seguir con CTL-D

      
#Parar servicios
````sh
service tomcat6 stop
service tomcat7 stop
service apache2 stop
service postgresql stop
service cron stop
````



#Deshabilitar servicios en el arranque
````sh
insserv --showall
insserv --remove apache2
insserv --remove tomcat6
insserv --remove tomcat7
insserv --remove postgresql
insserv --remove cron
````

# NFS
## Cliente
- Eliminar lo no necesario
````sh
nano /etc/fstab
````


## Servidor
- Eliminar lo no necesario
````sh
nano /etc/exports
## exportfs -ra para reload
````

# Particionar
````sh
sudo parted -l | grep Error   
##fdisk -l tambien para ver los discos
##-> indica sin particionar sí
cfdisk /dev/xvdXXXX
mkfs.ext4 /dev/xvdXXXX1
tune2fs -U random -L "XXnombreXX" /dev/xvdXXXX1 
#->(esto pone el nombre)
mkdir -p /srv/nfs/XXdondeXX
chown root:root /srv/nfs/XXdondeXX
echo "/dev/xvdc1      /srv/nfs/XXdondeXX       ext4    defaults        0       2" >> /etc/fstab
## hace un reload del /etc/fstab
##mount -a 
sed -i -e "s/\(^command\[check_all_disks\]=.*\)/\1 -p \"\/srv\/nfs\/XXdondeXX\"/" /etc/nagios/nrpe.d/10_system.nrpe.cfg; invoke-rc.d nagios-nrpe-server force-reload
echo "/srv/nfs/XXdondeXX               156.35.233.0/24(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)"  >> /etc/exports; exportfs -rv
````
# Cambiar nombre de la maquina
````sh
nano /etc/hostname
cat /etc/hostname | figlet > /etc/motd.tail

##cambia el nombre en el fichero 
sed -i "s/\(SiteDomain=\).*\$/\1\"XXXXXXXX\"/" /etc/awstats/awstats.conf.local

nano /etc/mailname
##esta configuracion para eUniovi quizas sea erronea

nano /etc/exim4/update-exim4.conf.conf
##esta configuracion para eUniovi quizas sea erronea
>>>>>>>>>>>>>>>>>>cambiar el relay //////////hacer un inliner
````

#Eliminar referencias DNS
````sh
nano /etc/hosts
## habría que dejar el localhost y el nombre de la propia maquina, a poder ser eliminar todas las referencias a innova
##dejamos el nombre simple al 127 hasta saber qcomo sería mejor resolver
````
- no esta claro que esto lo haga bien, hab´riq ue revisar en cada caso

````sh
cat /etc/resolvconf/resolv.conf.d/original
cat > /etc/resolvconf/resolv.conf.d/original <<EOF
nameserver 156.35.14.2
EOF

cat /etc/resolv.conf
````
#cambiar el servidor NTP

Al ejecutar desde la consola del xenserver quizas introduzca una / de mas
````sh
cat /etc/ntp.conf
sed -i 's/server ntp.innova.local/server hora.rediris.es\\\nserver hora.roa.es/g' /etc/ntp.conf
sed -i 's/server 156.35.33.1/server hora.rediris.es\\\nserver hora.roa.es/g' /etc/ntp.conf
sed -i 's/server 156.35.33.188/server hora.rediris.es\\\nserver hora.roa.es/g' /etc/ntp.conf
sed -i 's/server 192.168.25.188/server hora.rediris.es\\\nserver hora.roa.es/g' /etc/ntp.conf
sed -i 's/server 192.168.25.1/server hora.rediris.es\\\nserver hora.roa.es/g' /etc/ntp.conf
nano /etc/ntp.conf

````
# Cambiar prompt
````sh
cat >> /root/.bashrc <<EOF
## 2020-01-29
## Cambios PARA SIC 
##export PS1="\e[0;31mSIC-\e[m\u@\h:\w# "
export PS1="\[\e[0;31m\]SIC-\[\e[m\]\u@\h:\w# "
EOF
````

# Cambiar password
-Recordar incluirla en el KeePASS
````sh
passwd
````


# Cambiar password otros usuarios
 - Parece que hay variso usuarios y que solo el usuairo innova debe´ria ser interactivos, los otros es por tema de owneship, etc
````sh
grep bash /etc/passwd -> verificar que son los que hay interactivos
passwd postgres
passwd tomcat
passwd tomcat6
passwd innova
````

# Cambiar password de usuairos de la BBDD
- utilizar la misma que el root
````sh
service postgresql start
sudo -u postgres psql --command '\password postgres'
sudo -u postgres psql --command '\password dba'
service postgresql stop
````

## actividad y contenido
````sh
sudo -u postgres psql -d postgres -c 'SELECT * FROM pg_stat_activity;'
sudo -u postgres psql -c '\list'
sudo -u postgres psql -c '\du'
````
## Revisar si hay Mysql
````sh
mysql -u root -p -e 'show databases;
````

# Cambiar certificado de acceso
- Se eliminan lso existentes y se incluye el nuevo
- Si esta configurado para respaldo hayque descomentar el certificado de respaldo.

````sh
cat > /root/.ssh/authorized_keys <<EOF
##innova@innova.uniovi.es
##ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5LDSt51Xd1CbZcVAjtkRr06w3UvZpaujMyNt+rMRNrIW4s66PHGA2IVVCtMW3yVHp6MWsEEtJXZoaeG98GyWnpNJHzRF839Lq3AAEWQrGvV0x1QJjyHg0z0dxIDy5dxOo73rGJgRP/yMLeSqqxKwYXO6zDoxv+nyCgSNydRnLNydWX3X2RFaOCYDwP/57PouZSbxjAxejKyNVuBqS4MNVp8ENryMGPrzj7LnNUj9PXM6kjAxL2b0ZDLAS9kEyy85Z/1AhS7EoRl+v+TQecw/OVF9PWb4Pu3/BjsVZ0TbEy0pgOzFTCLpYs/bSJv980JoIX7hXgZA1mVOCIdftwORx innova@innova.uniovi.es

##respaldo@innova.uniovi.es
##no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEArcxBmvF9WXr3bth80vibEYV4phJDyoqms0nYKOitOtqzT9YTFtyoIb0LkujHFmwY567P7uou6vqlJcI3u8h9DTGu65PugQbmWoM2e0uMrigceE5EuEgeSMqPbBn0a1qSr5wEgA2y/J/RxoXZQV2hqaBHtXHNqNNdr/iVwcaR6IWiDVZ4aWLos6TXBPbCD0lYGAOrBsyD7vCjrb9+z9n7G+1On0KVaUpAbzHhGW+D8yKZTzE3z3SGAfNG63GMtABYkZxdfv6Mqxq+s8XsmMIHY7N3O92DPfCFRht/lvKgY1yDSefts8m3FG6I5mX8LZi0aNNSqCDr9o7rS5mEbYHQwQ== respaldo@innova.uniovi.es

##eadmon@sic.uniovi.es
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDuh34R17muFb/omfRbMx6u3rGLIieo05Lp8qmt/5fxjKdR1OEpM2QlquiTuWHCBbb3mbe5ybgPYybUhPHF5JA+wieV1mCJInqCX57fHG0NhpoWNFJp1QDBmNCs3vpaIaSYy2T4wuOh9JG0N1LxIIO7BBMHJKTgYlGlWBlO0nQNJMUhxYTBJzrjd8+tPg+eY4d0IH2z02wWYmIvBH9oyGCcvxFMCmGbuBxcFncAFNgVDyKw3EhPj+M6PggNam8M4YPEOzBfyfZqL9DeRnv91lN9ngB68w5/qCTVEiSAfIKZJS5zBYtIu495ZqyEt9xmJXWx27BZABscVXcCewxMZNS3 eadmon@sic.uniovi.es
EOF
````

#Xentools
````sh
##Meter el CD en la parte superior del XenTools 
dpkg --purge xe-guest-utilities
mkdir /mnt/xentools
mount -o ro,exec /dev/disk/by-label/XenServer\\x20Tools /mnt/xentools
##blkid -t LABEL="XenServer Tools"
/mnt/xentools/Linux/install.sh
umount /mnt/xentools
````


#Configurar IPs y DSR
- Se ejecuta esta despues de realizar el cambio para poder insertar la subred privada (que no se usa). Si no se producen incoherencias

## Configuracion de IP basicas
- Este cambio sobreescribe
````sh
cat > /etc/network/interfaces <<EOF
### This file describes the network interfaces available on your system
### and how to activate them. For more information, see interfaces(5).

### The loopback network interface
auto lo
iface lo inet loopback

###
### publica
###
auto eth0
iface eth0 inet static
      address 156.35.233.131
      netmask 255.255.255.0
      gateway 156.35.233.205
      ### dns-* options are implemented by the resolvconf package, if installed
      dns-nameservers 156.35.14.2

###
### privada
###
auto eth1 
iface eth1 inet static
      address 192.168.25.131
      netmask 255.255.255.0
      #gateway 192.168.25.205
      ### dns-* options are implemented by the resolvconf package, if installed
      dns-nameservers 156.35.14.2
EOF
````


## Configuracion para que funciona el DSR
- Este cambio concatena
````sh
cat >> /etc/sysctl.conf <<EOF
net.ipv4.conf.all.arp_ignore=1
net.ipv4.conf.eth0.arp_ignore=1
net.ipv4.conf.eth1.arp_ignore=1
net.ipv4.conf.all.arp_announce=2
net.ipv4.conf.eth0.arp_announce=2
net.ipv4.conf.eth1.arp_announce=2
EOF

````

## Configuracion de ips VIP para DSR donde sea necesario
- este cambio concatena
````sh
cat >> /etc/network/interfaces <<EOF
###
### DSR uniovi
###
auto lo:0
iface lo:0 inet static
      address 156.35.233.143
      netmask 255.255.255.255

###
### DSR intranet
###
auto lo:1
iface lo:1 inet static
      address 156.35.233.100
      netmask 255.255.255.255

EOF
````
````sh

cat >> /etc/network/interfaces <<EOF
###
### DSR Portales
###
auto lo:0
iface lo:0 inet static
      address 156.35.233.143
      netmask 255.255.255.255
EOF
````

#Apagar

#Asignar interfaces de red XenCenter
- Hay qu eponerlos por orden, primero publico y luego privado. Si existiese management, el ultimo

#Arrancar

# Certificados SSL
- Verificar los certtificados necesarios en ruta /etc/ssl/private
- es almacen de certificados de aplicacion en ocasiones.
>>>>> hay que revisar en todos lso servidores para que son estos certificados (si son para acceder a otros equipos hab´ria que eliminarlos y/o poner lso nuevos)

# Postgre - Autorizacion de conexiones desde el nuevo rango
- hay que meter servisdres y equipos de usuarios
````sh
nano /etc/postgresql/9.1/main/pg_hba.conf

### SIC-eAdmom LAN IPv4 connections:
host    all             all             192.168.25.0/24         md5
host    all             all             156.35.233.0/24         md5
host    all             all             156.35.210.0/24         md5
````


# autorizacion de nuevos sercvidores a los que accede mediante postgres
````sh
nano /root/.pgpass
````

#Sincronizar discos externos y/o recursos

## Sincronizar codigo o recursos estaticos (no datos)

````sh
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.145:/var/lib/tomcat6/ /var/lib/tomcat6/

````

##CRON
Revisar si hay que cambiar en /usr/local

     Actualizar IPs del CRON
>>>>OJO QUE LOS JARS NO SE ACTUALIZAN con el cambio de IPS

### para saber que hay 
      >>>>>>Existe un script para saber todo lo que se está ejecutando
      nano /etc/cron.d/XXXXX


## BBDD
### hay que autorizar el acceso antes en el pg_hba.conf
````sh
nano /etc/postgresql/9.1/main/pg_hba.conf
host    all             all             156.35.233.0/24         md5
````
### Hay que cambiar la password en /root/.pgpass para que funicone la importación
````sh
nano /root/.pgpass
````

### Importar

#actividad y contenido
````sh
sudo -u postgres psql -d postgres -c 'SELECT * FROM pg_stat_activity;'
sudo -u postgres psql -c '\list'
sudo -u postgres psql -c '\du'
````
En destino
````sh
#zcat 192.168.25.164_intranetfuo_2019-04-14_03h32m03s.pg_dump.sql.gz | psql -h 192.168.25.165 -U dba postgres

pg_dump -C -h 156.35.33.164 -U dba alfrescorepo_20190129 > 156.35.33.164_alfrescorepo_20190129.pg_dump.sql

dropdb -h localhost -p 5432 -U dba alfrescorepo_20190129
dropuser -h localhost -p 5432 -U dba alfrescorepo
createuser -h localhost -p 5432 -U dba --pwprompt
cat 156.35.33.164_alfrescorepo_20190129.pg_dump.sql | psql -h localhost -U dba postgres

                     
````
###En caso de migracion parcial - Recordar deshabilitar(Cambiar de nombre) las BBDD de Mieres para que no se creen dos origenes operativos de datos

Se peuden añadir bloqueos para bases de datos concretas tanto en destino como en Origen
````sh
nano /etc/postgresql/9.1/main/pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    liferay6             liferay6             156.35.33.0/24         reject
````

##NAS
````sh
## el -n es el dry-run
date
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.194:/srv/nfs/personales/  /srv/nfs/personales/
date

luIx511-tik4lly

````
##Indices locales o SOLR
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.184:/var/local/solr/data/liferay_portales/ /var/local/solr/data/liferay_portales/
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.109:/var/local/alfresco/lucene/ /var/local/alfresco/lucene/

rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.186:/var/local/liferay/data/lucene/ /var/local/liferay/data/lucene/



NOTA: pueden estar compartidos en algunas maquinas en el NFS


#Limpieza de logs y basura
````sh
rm -f /etc/network/interfaces~
rm -f /etc/hosts~
rm -f /etc/exports~
rm -f /etc/fstab~

##Comando original      
##find /var/log -type f -exec rm -v {} + ; find /var/lib/awstats/ -type f -exec rm -v {} + ; find /var/spool/mail/mail -type f -exec rm -v {} + ; find /var/mail/mail -type f -exec rm -v {} + ; find /var/cache/awstats -type f -exec rm -v {} +
find /var/log /var/lib/awstats /var/spool/mail/mail /var/mail/mail /var/cache/awstats -type f -exec rm -v {} + 


rm -rf /etc/tomcat6/server.xml~
rm -rf /var/lib/tomcat6/webapps/solr-web/WEB-INF/classes/META-INF/solr-spring.xml~
rm -rf /var/lib/tomcat6/webapps/ROOT/WEB-INF/classes/system-ext.properties~
rm -rf /var/lib/tomcat6/webapps/ROOT/WEB-INF/classes/portal-ext.properties~
rm -rf /var/lib/tomcat6/webapps/ROOT/WEB-INF/classes/#system-ext.properties#
rm -rf /var/lib/tomcat6/webapps/ROOT/WEB-INF/web.xml~
rm -rvf /etc/tomcat6/Catalina/localhost/*
rm -rvf /var/lib/tomcat6/work/*
rm -rf /etc/default/tomcat7~
rm -rf /etc/tomcat7/server.xml~
rm -rvf /etc/tomcat7/Catalina/localhost/*
rm -rvf /var/lib/tomcat7/work/*


//////////////find /var/lib/tomcat6/webapps -type d  -path "*.svn" -exec rm -rfv {} +
      

````
#Limpiar directorio de administracion (Manual)

#Cambiar IPs en todos los ficheros de texto

## localizar referencias a innova
adminst@innova.uniovi.es
c1nn@innova.uniovi.es
me@innova.uniovi.es

````sh
grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts,*.class,*.jar} --exclude-dir={work,log,logs} -e 'innova.uniovi.es' --color
grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts,*.class,*.jar} --exclude-dir={work,log,logs} -e 'innova.local' --color
````

>>>>>>> Habrñiq eu cambiar esto para hacerlo con fing y poder utilizar ZGREP para los comprimidos (jars)
##Localizar IPs a cambiar y ver si lso cambios son los esperados
````sh
grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts} -e '156.35.' --color
grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts} -e '192.168.' --color

````
##Cambiar las ips a las ips externas 
- Cambia las ips internas de origen por las externas en origen
````sh
grep -rl {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts,*.class} -e '192.168.' --color | xargs sed -i 's/192.168.25/156.35.33/g'
grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts} -e '192.168.' --color
````

##Cambiar las ips de los nuevos servidores
- Cambia las ips externas en origen por las ips en destino de lso servicios que ya existen.
      - Hay que actualizar la lista de los servicios existentes
````sh
for servidor in 1 35 100 102 105 141 142 143 145 146 155 156 162 164 169 184 185 184 190 192 194 199; do grep -rl {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts,*.class} -e '156.35.33' --color | xargs sed -i "s/156.35.33.$servidor/156.35.233.$servidor/g" ; done;

>>>>>>IMPORTANTE: añadir las maquinas de todo el cluster segun se vayan poniendo en marcha

grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} -e '156.35.233' --color
````

##Buscar referencias dentro de jars
````sh
#find /usr/local /etc /var/lib/tomcat* -type f  -name "*.jar" -print0 | xargs -0 zgrep '192.168' --color=always
#find /usr/local /etc /var/lib/tomcat* -type f  -name "*.jar" -print0 | xargs -0 zgrep '156.35' --color=always

find /usr/local /etc /var/lib/tomcat* -name "*.jar" | xargs -I{} sh -c 'echo buscando "{}"; zipgrep  "192.168" {}'
find /usr/local /etc /var/lib/tomcat* -name "*.jar" | xargs -I{} sh -c 'echo buscando "{}"; zipgrep  "156.35" {}'

````

##BUscar referencias a equipos locales de Mieres
````sh
grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts} -e '192.168.24' --color

grep -rl {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts,*.class} -e '192.168.24' --color | xargs sed -i 's/192.168.24/156.35.210/g'

grep -rnis {'/usr/local','/etc',/var/lib/tomcat*} --exclude={interfaces,hosts} -e '156.35.210' --color
````


# Revisar enlaces de conexiones cluster
-NAS
      cat /etc/fstab | grep nfs
-BBDD
      cat /var/lib/tomcat6/webapps/ROOT/WEB-INF/classes/portal-node-ext.properties  | grep  jdbc.default.url
-SOLR
      cat /var/lib/tomcat6/webapps/solr-web/WEB-INF/classes/META-INF/solr-spring.xml | grep solr_liferay

-SERVIDOR SOLR
      cat /etc/local/solr/solr.xml (define los cores)
      cat /etc/local/solr/CORE/conf/solrconfig.xml (configuracion de cada core)
      cat /var/local/solr/data/liferay_CORE (datos)
      cat /etc/tomcat6/Catalina/localhost/solr_liferay.xml (Valvula de acceso)      

#Habilitar servicios en el arranque
````sh
insserv --showall
insserv apache2
insserv tomcat6
insserv tomcat7
insserv postgresql
insserv cron
      >>>>>>deshabilitar ejecutables no deseados o con ips erróneas
            nano /etc/cron.d/researchsites-synchronizer
            nano /etc/cron.d/cambiocrupotemporales
      >>>>>>Existe un script para saber todo lo que se está ejecutando

#insserv --add apache2
#insserv --add tomcat6
#insserv --add postgres
````
#Apagar

#Arrancar y revisar logs
>>>Revisar que el cron (Si es necesario o no)
>>>Revisar que no existen las conexiones indeseadas a BBDD cruzadas
      netstat -auntp | grep 156.35.33
>>>Revisar logs buscando errores de conexion
>>>Buscar actividad en la BBDD de destino con el PgAdmin
>>>Buscar Dentro de los jars con el MC

>>>>>>>>
Revisar que esta igual y no había configuracion a fuego 
/etc/tomcat6/Catalina/localhost/


#Netscaler
Habilitar nuevo direccionamiento si Cluster

#Cambio de certificados para https
      Suele ser habitual tener que pasarlos del Netscaler al apache si hay que configurar DSR
#Revision funcional



#POR COMPLETAR - Copias de seguridad 
#POR COMPLETAR - Monitorización nagios
##Instalar zabbix agent
````sh
lsb_release -a
      wget http://repo.zabbix.com/zabbix/2.2/debian/pool/main/z/zabbix-release/zabbix-release_2.2-2%2Bsqueeze_all.deb      
      wget http://repo.zabbix.com/zabbix/3.5/debian/pool/main/z/zabbix-release/zabbix-release_3.5-1%2Bwheezy_all.deb
      wget http://repo.zabbix.com/zabbix/3.4/debian/pool/main/z/zabbix-release/zabbix-release_3.4-1%2Bwheezy_all.deb
dpkg -i ./zabbix-
rm  ./zabbix-
apt-get update
##Si problemas con proxy 
      nano /etc/apt/apt.conf
apt-get install zabbix-agent
insserv zabbix-agent
## luego hay que meterlo a mano ocon el autodiscover en el zabbix, peor hayque asignarle plantillas
````

____________________


/usr/local/sbin/sincronizar_respaldo_nas.sh -f "/var/log/sincronizar_respaldo_nas/$(/bin/date +\%Y/\%m/\%d)/sincronizar_respaldo_NAS-09_$(/bin/date +\%Y-\%m-\%d_\%Hh\%Mm\%Ss).log" "root@156.35.233.199:/srv/nfs/*" "/srv/nfs/replicacion/NAS-09/srv/nfs/."
_____________________________________________________________________________________________________
__________________________________________________________________________________________________________________________






