<!-- MarkdownTOC -->

- DSR - Direct Server Response
	- Comandos de instalacion de DSR para las VM Centos de pruebas
	- Comandos de instalacion de DSR para las VM Centos de pruebas
- XenServer
	- Cambio nombre iSCSI
	- configurar disco iSCSI
		- Proceso
	- XenServer ISOTOOLS
		- xen-tools mediante apt -> no se puede por el tipo de instalacion
	- XenSever Creacion, format y  montaje de un disco
	- [Motar disco en Host0 XenServer](#Peligroso!!!!!!!)
	- XenServer - Creacion de snapShot, exportacion y eliminado
- WINDOWS NFS en 156.35.201.41
- Windows Fordward
- JAVA command line
	- proceoss java en el SO y propiedades del ssistema
	- CACERTS
- Linux general
	- Recuperacion de archivos
	- Postgresql
	- mysql
	- access.log
	- Networking debian
	- Creacion, format y  montaje de un disco
		- Particionamiento
		- resize de discos
	- LVM
		- Conceptos
		- Comandos para informacion
		- Anañir disco nuevo a una LV existente y cambiar el tamaño
	- Crear un nuevo VG
	- Crear un nuevo LV
		- Ejemplo completo añadir un disco a un LV Existente
	- NFS Linux
		- Servidor
		- Cliente
		- Instalar en equipos C1NN qu eparece que no tienen
	- Cadenas de texto
		- Sustituir en java properties
- cheatsheet migracion de maquinas virtuales con NFS de pivote
- Migracion  Mieres Oviedo EJ
	- Xen147 - NAS-00 \(mieres\)
	- Dasboard \(Oviedo\)
	- eadmon03 - Dasboard
	- Desde pool viejo
	- instalacion mauqinas virtuales XEN con preseed
- Dirty ansible
- SVN
	- Instalacion SVN
	- creacion de nuevo repositorio
	- creacion de usuairos
	- Exportar/immportar
- Copias con dd
- Desde pool viejo
- Desde el pool nuevo
- Rsync y enlaces duros
- Mysql
- HITACHI

<!-- /MarkdownTOC -->




# DSR - Direct Server Response
## Comandos de instalacion de DSR para las VM Centos de pruebas
````sh
yum infom nano
yum install nano
nano /etc/sysconfig/network-scripts/ifcfg-lo:0
	DEVICE=lo:0	
	IPADDR=156.35.233.250
	NETMASK=255.255.255.255	
	ONBOOT=yes
	NAME=DRS
  
service NetworkManager restart
service network restart

nano /etc/sysctl.conf
	net.ipv4.conf.all.rp_filter=0
	net.ipv4.conf.all.arp_ignore=1
	net.ipv4.conf.eth0.arp_ignore=1
	net.ipv4.conf.eth1.arp_ignore=1
	net.ipv4.conf.all.arp_announce=2
	net.ipv4.conf.eth0.arp_announce=2
	net.ipv4.conf.eth1.arp_announce=2

sysctl -p

Apache
	<VirtualHost 156.35.233.250 156.35.233.10>

ifconfig lo:0

route add default gw 156.35.233.205

ifconfig lo:0 -> ip route
````

## Comandos de instalacion de DSR para las VM Centos de pruebas
````sh
cat >> /etc/sysctl.conf <<EOF
net.ipv4.conf.all.arp_ignore=1
net.ipv4.conf.eth0.arp_ignore=1
net.ipv4.conf.eth1.arp_ignore=1
net.ipv4.conf.dsr.arp_ignore=1
net.ipv4.conf.all.arp_announce=2
net.ipv4.conf.eth0.arp_announce=2
net.ipv4.conf.eth1.arp_announce=2
net.ipv4.conf.dsr.arp_announce=2
EOF
````
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
      address 156.35.233.141
      netmask 255.255.255.0
      gateway 156.35.233.205
      ### dns-* options are implemented by the resolvconf package, if installed
      dns-nameservers 156.35.14.2

###
### privada
###
auto eth1 
iface eth1 inet static
      address 192.168.25.141
      netmask 255.255.255.0
      #gateway 192.168.25.205
      ### dns-* options are implemented by the resolvconf package, if installed
      dns-nameservers 156.35.14.2

###
### DSR
###
auto dsr
iface dsr inet static 
      address 156.35.233.105 
      netmask 255.255.255.255
EOF
````


# XenServer 

##Cambio nombre iSCSI
````sh
#Parar maquinas virtuales
#Desde la consola del servidor  buecar el hsot UUID
xe host-list
#cambiar el valor con el comando
xe host-param-set uuid=<host UUD> other-config:iscsi_iqn=<New IQN>
#xe host-param-set uuid=a7654ae2-e530-4cae-8785-1f11007dd0d3 other-config:iscsi_iqn=iqn.2019-07.es.uniovi:7f6e6fc7
#Verificar en el XenCenter que ha cambiado
#Hacerlo permanente en 
 	nano /etc/iscsi/initiatorname.iscsi
 		InitiatorName=iqn.2019-07.es.uniovi:fc6		
	service open-iscsi restart	
#Reboot!!!
````

##configurar disco iSCSI
- No funciona correctamente desde el xenCenter
- el sr-prove no funciona tampoco correctamente y o muestra informacio 
xe sr-probe type=lvmoiscsi device-config:target=X.X.X.X device-config:targetIQN=XXXXXX device-config:chapuser="tecmint" device-config:chappassword="tecmint_chap"

###Proceso

````sh
#Reinciar por si acaso las herramientas iscsi
systemctl restart iscsid
#muestra lo disponible
iscsiadm -m discovery -t sendtargets -p 192.168.0.206
#utilizar el commando sr-create va mostrando informacion
#la siguiente no conecta porque solo esta pinchado al nodo 3, así que hay que indicar que no es compartida y donde esta pinchada
xe sr-create name-label=ISCSI3 type=lvmoiscsi device-config:target=192.168.0.206 device-config:targetIQN=iqn.1994-04.jp.co.hitachi:rsd.h8s.t.81102.1a004
##esta rula y muestra mas info (host-uuid=a7654ae2-e530-4cae-8785-1f11007dd0d3 es el del nodo xen3)
xe sr-create shared=false host-uuid=a7654ae2-e530-4cae-8785-1f11007dd0d3 name-label=ISCSI3 type=lvmoiscsi device-config:target=192.168.0.206 device-config:targetIQN=iqn.1994-04.jp.co.hitachi:rsd.h8s.t.81102.1a004 
# estas son la sválidas
xe sr-create shared=false host-uuid=a7654ae2-e530-4cae-8785-1f11007dd0d3 name-label=ISCSI1 type=lvmoiscsi device-config:target=192.168.0.206 device-config:targetIQN=iqn.1994-04.jp.co.hitachi:rsd.h8s.t.81102.1a004 device-config:SCSIid=360060e80223cce0050413cce00000400
xe sr-create shared=false host-uuid=a7654ae2-e530-4cae-8785-1f11007dd0d3 name-label=ISCSI2 type=lvmoiscsi device-config:target=192.168.0.206 device-config:targetIQN=iqn.1994-04.jp.co.hitachi:rsd.h8s.t.81102.1a004 device-config:SCSIid=360060e80223cce0050413cce00000401
````


     


## XenServer ISOTOOLS


````sh
#Meter el CD en la parte superior del XenTools 
dpkg --purge xe-guest-utili
mkdir /mnt/xentools
mount -o ro,exec /dev/disk/by-label/XenServer\\x20Tools /mnt/xentools
	blkid -t LABEL="XenServer Tools"
/mnt/xentools/Linux/install.sh
umount /mnt/xentools
````
###xen-tools mediante apt -> no se puede por el tipo de instalacion
````sh
apt-cache search xe-guest-utilities
apt-cache show xe-guest-utilities
apt-cache policy xe-guest-utilities
sed '1 {s/^/#/}' /etc/apt/apt.conf > /etc/apt/apt.conf
apt-get update 
sudo apt install xe-guest-utilities=XXXXXX
apt-get purge --autoremove 
````
##XenSever Creacion, format y  montaje de un disco
````sh
xe vm-disk-add device=/dev/xvdd disk-size=1Gb
xe vbd-param-set userdevice=3 vdi-name-label="disco_nas-01_3_particion_srv_nfs"
````
- Hab´riq eu seguir con el particionamiento habitual de linux

##Motar disco en Host0 XenServer [Peligroso!!!!!!!]
````sh
XenCenter - En SR local Disk 
	Add.. y crear VDi en el espacio que se quiera. Guardar el nombre

xe vdi-list name-label=Disco_dom0_XenServer145 
	uuid ( RO)                : 7ac28282-e536-4af5-90af-536eed3ccd9d
    name-label ( RW): Disco_dom0_XenServer145
    name-description ( RW):
    sr-uuid ( RO): 859c4187-95c9-130b-ad32-3886a49a3475
    virtual-size ( RO): 96636764160
    sharable ( RO): false
    read-only ( RO): false

Get control domain
xe vm-list | grep -B 1 -e Control
	uuid ( RO)           : 3c4c2996-34fd-420a-b6d2-9b524fb6f7a6
     name-label ( RW): Control domain on host: xenserver-145

Create VBD
	xe vbd-create device=0 vm-uuid=3c4c2996-34fd-420a-b6d2-9b524fb6f7a6 vdi-uuid=7ac28282-e536-4af5-90af-536eed3ccd9d bootable=false mode=RW type=Disk
		0bdccf86-a4c7-9482-31a6-51bee019806e

Plug the VBD
	xe vbd-plug uuid=0bdccf86-a4c7-9482-31a6-51bee019806e	


xe vbd-list vm-name-label="Control domain on host: eadmon03"
/dev/sm/backend/405317ce-d45f-1751-4cc1-e9597dd3966d/a476a9ce-68e5-4799-acb5-74d28505a001

xe vbd-list vm-name-label="Control domain on host: xenserver-145"
xe vbd-list vdi-uuid=7ac28282-e536-4af5-90af-536eed3ccd9d params=all
xe vbd-list vm-name-label={VM name}{code}
xe vbd-unplug uuid=0bdccf86-a4c7-9482-31a6-51bee019806e
````
## XenServer - Creacion de snapShot, exportacion y eliminado
````sh
snapshot=`xe vm-snapshot vm=$uuid new-name-label=backup_$date`
vm_log[${#vm_log[@]}]="Snapshot: $snapshot"

# Set as VM not template

snapshot_template=`xe template-param-set is-a-template=false uuid=$snapshot`
vm_log[${#vm_log[@]}]="Set as VM"

# Export

snapshot_export=`xe vm-export vm=$snapshot filename="$backup_dir$label-$date$backup_ext"`
vm_log[${#vm_log[@]}]="Export: $snapshot_export"

# Delete snapshot

snapshot_delete=`xe vm-uninstall uuid=$snapshot force=true`
vm_log[${#vm_log[@]}]="Delete Snapshot: $snapshot_delete"
````


# WINDOWS NFS en 156.35.201.41
````sh
##https://docs.microsoft.com/en-us/powershell/module/nfs/new-nfsshare?view=win10-ps
New-NfsShare -name nfs1 -Path C:\NFS-Shared -EnableAnonymousAccess
mount -t nfs -o vers=3 192.168.24.35:/compartida /mnt/compartida -vvv
````


#Windows Fordward
````sh
 netsh interface portproxy add v4tov4 listenaddress=localaddress listenport=localport connectaddress=destaddress connectport=destport
 Todas la peticiones entrantes
 netsh interface portproxy add v4tov4 listenport=3389 listenaddress=0.0.0.0 connectport=3389 connectaddress=192.168.100.101
 HAcer que un servicio remoto parezca local
 netsh interface portproxy add v4tov4 listenport=5555 connectport=80 connectaddress= 157.166.226.25 protocol=tcp
Para wl firewall
  netsh advfirewall firewall add rule name=”forwarded_RDPport_3340” protocol=TCP dir=in localip=10.1.1.110  localport=3340 action=allow
Para ver que hay
	netsh interface portproxy show all
Para elilminar
	netsh interface portproxy delete v4tov4 listenport=3340 listenaddress=10.1.1.110
	O eliminar todos netsh interface portproxy reset

Port 111 (TCP and UDP) and 2049 (TCP and UDP) for the NFS server.
netsh interface portproxy add v4tov4 listenport=111 connectport=111 connectaddress=156.35.210.41 protocol=tcp
netsh interface portproxy add v4tov4 listenport=2049 connectport=2049 connectaddress=156.35.210.41 protocol=tcp
netsh advfirewall firewall add rule name=”forwarded_NFS_111” protocol=TCP dir=in localip=192.168.24.35  localport=111 action=allow
netsh advfirewall firewall add rule name=”forwarded_NFS_2049” protocol=TCP dir=in localip=192.168.24.35  localport=2049 action=allow

ssh
netsh interface portproxy add v4tov4 listenport=22 connectport=22 connectaddress=156.35.210.41 protocol=tcp
netsh advfirewall firewall add rule name=”forwarded_ssh_22” protocol=TCP dir=in localip=192.168.25.35  localport=22 action=allow

netsh interface portproxy add v4tov4 listenport=22 connectport=22 connectaddress=192.168.233.213 protocol=tcp
netsh advfirewall firewall add rule name=”forwarded_ssh_22” protocol=TCP dir=in localip=156.35.210.41  localport=22 action=allow
````


#JAVA command line

## proceoss java en el SO y propiedades del ssistema
jps -lvm
8246 com.ieci.integration.Consolidation
10861 sun.tools.jps.Jps -lvm -Dapplication.home=/usr/lib/jvm/jdk-7-oracle-x64 -Xms8m
30306 org.apache.catalina.startup.Bootstrap start -Djava.util.logging.config.file=/var/lib/tomcat7/conf/logging.properties -Djava.awt.headless=true -XX:+UseConcMarkSweepGC -XX:+CMSPermGenSweepingEnabled -XX:+CMSClassUnloadingEnabled -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/log/tomcat7/dump.hprof -Xms5632M -Xmx5632M -XX:PermSize=1536M -XX:MaxPermSize=1536M -Djava.rmi.server.hostname=156.35.233.131 -Dcom.sun.management.jmxremote.port=8006 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dsun.net.client.defaultConnectTimeout=10000 -Dsun.net.client.defaultReadTimeout=20000 -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=/usr/share/tomcat7/endorsed -Dcatalina.base=/var/lib/tomcat7 -Dcatalina.home=/usr/share/tomcat7 -Djava.io.tmpdir=/tmp/tomcat7-tomcat7-tmp

jinfo -sysprops <vmid>
jinfo -flags <vmid> 

##CACERTS
cp $JAVA_HOME/jre/lib/security/cacerts $JAVA_HOME/jre/lib/security/cacerts_20201130
keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts | grep redsara
keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts | grep profesional

keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -file /root/Administracion/Certificados/redsara.es_20201130/CA_ROOT_FIRMAPROFESIONAL.cer -alias "CA_ROOT_FIRMAPROFESIONAL"
keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -file /root/Administracion/Certificados/redsara.es_20201130/SUBCA_FIRMAPROFESIONAL_SECURE_WEB_2020.cer -alias "SUBCA_FIRMAPROFESIONAL_SECURE_WEB_2020"
keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -file /root/Administracion/Certificados/redsara.es_20201130/SSL_REDSARA_ES.cer -alias "SSL_REDSARA_ES"

keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -file /root/Administracion/Certificados/20201130_redsara.es/CA_ROOT_FIRMAPROFESIONAL.cer -alias "CA_ROOT_FIRMAPROFESIONAL"
keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -file /root/Administracion/Certificados/20201130_redsara.es/SUBCA_FIRMAPROFESIONAL_SECURE_WEB_2020.cer -alias "SUBCA_FIRMAPROFESIONAL_SECURE_WEB_2020"
keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -file /root/Administracion/Certificados/20201130_redsara.es/SSL_REDSARA_ES.cer -alias "SSL_REDSARA_ES"


# Linux general 


## Recuperacion de archivos
extundelete /dev/xvde1 --restore-file /log/tomcat6/* -o /mnt/recuperacion/portales01
--restore-directory /log/tomcat6
--restore-files /log/tomcat6
Dan diferentes resultados

ext4magic /dev/sdXY -f path/to/lost/file -a $(date -d -5days +%s) -r -d path/to/dest/dir.
ext4magic /dev/vdxg1 -f /log/tomcat6 -a $(date -d -1years +%s) -r -d /mnt/r


mejor utilizar testdisk
Puede entrar en bucle , mejor pillar archivos sueltos poco a poco

## Postgresql
sudo -u postgres psql -d postgres -c 'SELECT * FROM pg_stat_activity;'
sudo -u postgres psql -c '\list'
sudo -u postgres psql -c '\du'

sudo -u postgres psql -d postgres -c "SELECT * FROM pg_stat_activity WHERE current_query IN ('<IDLE> in transaction', 'active');" > salida.txt

sudo -u postgres psql -d postgres -c 'SELECT * FROM pg_stat_activity;' > salida.txt

SELECT
  pid,
  now() - pg_stat_activity.xact_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '4 minutes';
if there is indeed a query that has been alive longer than it should be, you can delete like so:

SELECT pg_cancel_backend(<pid>);

You can match a specific Postgres backend ID to a system process ID using the pg_stat_activity system table.

SELECT pid, datname, usename, query FROM pg_stat_activity; can be a good starting point.
Once you know what queries are running you can investigate further (EXPLAIN/EXPLAIN ANALYZE; check locks, etc.)

psql -h 156.35.233.164 -U dba -d postgres -P pager=off -c '\list'

_____________________
recargar la configuracion postgre

Option 1: From the command-line shell
su - postgres
/usr/bin/pg_ctl reload

Option 2: Using SQL
SELECT pg_reload_conf();

incluir esto de como hacer consultas de modificacion con mayusculas
psql -h 156.35.33.164 -U dba -d postgres -c 'ALTER DATABASE uniovimovil_eliminar RENAME TO "uniovimovil_ELIMINAR";'
psql -h 156.35.33.164 -U dba -d postgres -P pager=off -c '\list' | grep uniovimovil


psql -h 156.35.233.164 -U dba -d postgres -P pager=off -c '\list'



psql -h 127.0.0.1 -d liferayweb03Integracion -U liferayweb03Integracion < /root/liferayweb03.sql


CREATE DATABASE lportal52 WITH OWNER = postgres ENCODING = 'UTF8' TABLESPACE = pg_default;
GRANT TEMPORARY ON DATABASE lportal52 TO public;
GRANT ALL ON DATABASE lportal52 TO postgres;
GRANT ALL ON DATABASE lportal52 TO liferay52;

CREATE ROLE liferay LOGIN
  NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE;


## mysql
Cambiar password DBA en mysql
renombrar BBDD origen mysql (sno se puede en mysql)

SET PASSWORD FOR 'user-name-here'@'hostname' = PASSWORD('new-password');






## access.log
````sh
cat access.log | awk '{print $1}' | sort -n | uniq -c | sort -nr | head -20
````

## Networking debian
/etc/network/interfaces


##Creacion, format y  montaje de un disco
- Desde xenserver se puede crear si utilizar el xenCenter
````sh
cfdisk /dev/xvdXXXX
mkfs.ext4 /dev/xvdXXXX1
tune2fs -U random -L "/srv/nfs/nombre" /dev/xvdXXXX1 ->(esto pone el nombre)
mkdir -p /srv/nfs/ 
chown root:root /srv/nfs/
echo "/dev/xvdc1      /srv/nfs/       ext4    defaults        0       2" >> /etc/fstab
mount /srv/nfs/
sed -i -e "s/\(^command\[check_all_disks\]=.*\)/\1 -p \"\/srv\/nfs\"/" /etc/nagios/nrpe.d/10_system.nrpe.cfg; invoke-rc.d nagios-nrpe-server force-reload
echo "/srv/nfs/               156.35.233.0/24(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)"  >> /etc/exports; exportfs -rv
````

### Particionamiento
````sh
sudo parted -l | grep Error   -> indica sin particionar
lsblk -> mas info
parted /dev/sda mklabel gpt -> formato gpt
parted -a opt /dev/sda mkpart primary ext4 0% 100%
lsblk
	aparece una particion
mkfs.ext4 -L datapartition /dev/sda1

mkdir -p /mnt/local
mount -o defaults /dev/sda1 /mnt/data
	hay que declararlo en el /etc/fstab para que sea definitivo

````

###resize de discos
````sh
umount /dev/xvdd1
fdisk /dev/xvdd
	p - saca informacion, sobr eloq ue hay (tomar nota por si acaso)
	d - Emilminar al particion que existe
	w - hayquye escribir el cambio
	n - crear una nueva con las opciones por defecto estira el tamaño (se pued eespecificar a mano tambien)
	w - Escribir 
>>>>si no deja escoger el mismo sector inicial 
	fdisk -c=dos /dev/xvdd
e2fsck -f /dev/xvdd1
resize2fs /dev/xvdd1
mount /dev/xvdd1 xxx
````

Disk /dev/xvdc: 2147.5 GB, 2147483648000 bytes
255 heads, 63 sectors/track, 261083 cylinders, 4194304000 sectores en total
Units = sectores of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Identificador del disco: 0x00000000

 Disposit. Inicio    Comienzo      Fin      Bloques  Id  Sistema
/dev/xvdc1              63  3145727999  1572863968+  83  Linux
## LVM
### Conceptos 
- Physical Volumes (PV) 
- Volume Groups (VG)
- Logical Volumes (LV)

### Comandos para informacion
````sh 
pvs
pvdisplay
vgs
vgdisplay
lvs
lvdisplay
````
### Anañir disco nuevo a una LV existente y cambiar el tamaño

Inicalmente hay que particionar el disco (primaria como siempre)
````sh
parted --align optimal -s /dev/xvdg "mklabel gpt unit mib mkpart primary 0 100% set 1 lvm on print"
````
Los LV salen conel df Hay que revisar si en el VG hay espacion para aumentar
````sh
df -h 
vgdisplay nombreVG
````
Se busca el disco a añadir a mano o se pued eescanear lo que se puede utilizar para LVM
````sh
fdisk -l
lvmdiskscan
````
Se crea un PV nuevo para el disco y verificamos que existe (con el display o con el lvmdisk)
pvcreate sobre la particion
````sh
pvcreate /dev/xvdg1; pvdisplay
lvmdiskscan -l
````
Se añade el PV al VG existente y se cambia el tamaño del LV y se hace un resize
````sh
 vgextend vg-existente /dev/vdb
 lvextend -l +100%FREE /dev/mapper/vg_bbdd_2020-lv_bbdd_2020
 #lvextend -L +2G /dev/mapper/vg_cloud-LogVol00
 resize2fs -p /dev/mapper/vg_bbdd_2020-lv_bbdd_2020
````
 
##Crear un nuevo VG
````sh
vgcreate vg_bbdd_2020 /dev/xvdg1
vgdisplay
vgs
````

##Crear un nuevo LV
````sh
lvcreate -l +100%FREE vg_bbdd_2020 -n lv_bbdd_2020
lvs
##lvcreate -n nombreLV --size 12G nombreVG
mkfs.ext4 /dev/mapper/vg_bbdd_2020-lv_bbdd_2020
tune2fs -U random -L "BBDD-2020" /dev/mapper/vg_bbdd_2020-lv_bbdd_2020
##mkdir -v -p /var/tmp/lv_bbdd_2020
 ##mount /dev/mapper/vg_bbdd_2020-lv_bbdd_2020 /var/tmp/lv_bbdd_2020/
````
 
### Ejemplo completo añadir un disco a un LV Existente
````sh
vgs
vgdisplay vg_bbdd_2020
lvmdiskscan
parted --align optimal -s /dev/xvdh "mklabel gpt unit mib mkpart primary 0 100% set 1 lvm on print"
pvcreate /dev/xvdh1; pvdisplay
vgextend vg_bbdd_2020 /dev/xvdh1
vgdisplay vg_bbdd_2020
lvextend -l +100%FREE /dev/mapper/vg_bbdd_2020-lv_bbdd_2020
resize2fs -p /dev/mapper/vg_bbdd_2020-lv_bbdd_2020
````


## NFS Linux
###Servidor 
````sh
	showmount -e 
		Muestra lo que hay exportado en el servidor	(local)
	exportfs -v 
		Muestra lo que hay exportado en el servidor (local)
	cat /etc/exports
		Muestra lo que hay indicado para exportar
 			/srv/nfs/                192.168.24.0/23(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)
			/srv/nfs/respaldo/migracion/        192.168.25.221(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)			
			##parece ser que en ocasiones es mejor indicar la maquina que va a acceder, como por ejemplo en las maquinas XenServers

	service nfs restart
		Reinicia el servicio
````
###Cliente
````sh
	showmount -e 192.168.25.190 
		Muestra lo que hay exportado en el servidor	remoto

	## ver que puertos hay (deben salir un monton)
	rpcinfo -p 192.168.11.190

 	mount -t nfs4 192.168.25.190:/ /mnt/migracion -vvv 		
 	mount -t nfs -o vers=3 192.168.24.35:/compartida /mnt/compartida -vvv
 	mount -t nfs4 192.168.25.190:/ /mnt/migracion -vvv
 		Mejor indicar el tipo para evitar problemas como en los servidores Xenserver
 		Origen -> destino El origen empieza la ruta desde los indicado en el /etc/exports del servidor
 		Puede ser que no se localice la maquina cliente a si mismo por el nombre . Hay que incluir en el /etc/hosts

### delde el 147 al NAS-00
	- Monta bien la ruta aunque se ponga solo la barra
	mount -t nfs  192.168.11.190:/ /mnt/migracion -vvv


 	umount -f -l PATH
 		Para desmontar

 	cat /etc/fstab
 		Dispositivos montados durante el arranque

	## reload del /etc/exports
		exportfs -ra

		## arrancar servicio nfs (debian)
	service nfs-kernel-server start


````

### Instalar en equipos C1NN qu eparece que no tienen
````sh
 grep NFSD /boot/config-`uname -r`
 apt install nfs-kernel-server portmap
 perl -pi -e 's/^OPTIONS/#OPTIONS/' /etc/default/portmap
 echo "portmap: 192.168.1." >> /etc/hosts.allow
 /etc/init.d/portmap restart

  echo "/example 192.168.1.0/255.255.255.0(rw,no_root_squash,subtree_check)" >> /etc/exports
  exportfs -a
 /etc/init.d/nfs-kernel-server reload
````


## Cadenas de texto
### Sustituir en java properties
````sh
sed -i "s/\(example\.java\.property=\).*\$/\1VALOR/" /ruta/java.properties
````

#cheatsheet migracion de maquinas virtuales con NFS de pivote
````sh
cat /etc/exports
	/srv/nfs/respaldo/migracion/        192.168.25.221(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)
	/srv/nfs/respaldo/migracion/        192.168.10.147(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)

 mkdir /mnt/migracion
 mount -t nfs4 192.168.25.190:/ /mnt/migracion -vvv  	  

xe vm-export preserve-power-state=false vm="BBDD-02_WebUniovi [192.168.25.162]" filename=/mnt/migracion/BBDD-02_WebUniovi.XVA
xe vm-import filename=/mnt/migracion/BBDD-02_WebUniovi.XVA force=true sr-uuid=67cc10b2-5d87-3aa2-99fe-f665c8442878 preserve=true
````


_______________________________________________________________________

#Migracion  Mieres Oviedo EJ

##Xen147 - NAS-00 (mieres)
````sh
xe template-param-set is-a-template=false uuid=2846ec12-6fde-0f5b-e343-60f6921f13c9 
nohup xe vm-export preserve-power-state=false vm=2846ec12-6fde-0f5b-e343-60f6921f13c9 filename=/mnt/migracion/eUniovi01.XVA

````
>>>> hay que elimina los snapshots despues de exportarlos
##Dasboard (Oviedo)
````sh
	scp root@156.35.33.190:/srv/nfs/replicacion/migracion/*  .
````
##eadmon03 - Dasboard
````sh
nohup xe vm-import force=true preserve=true filename=/mnt/intercambio/RepoUniovi-01.XVA sr-uuid=5e0020ac-a437-c3d6-a521-65193da121d1

nohup xe vm-import force=true filename=/mnt/intercambio/RepoUniovi-01.XVA sr-uuid=dde4f3a6-6b39-7e43-ad78-d560e2610e57

````
>>>>>>hay que eliminar los XVA de donde esten en mieres y en oviedo para que no se hagan copias de seguridad
````sh
root@nas-00:/srv/nfs# find . -name "*.XVA"
````
##Desde pool viejo
Desde el Xen 192.168.25.221
````sh

mount -t nfs4 192.168.25.190:/ /mnt/migracion/ -vvv

xe template-param-set is-a-template=false uuid=34891e7d-5dfe-8457-ae27-d41f0649554a
nohup xe vm-export  vm=34891e7d-5dfe-8457-ae27-d41f0649554a filename=/mnt/migracion/datio.XVA &


#(a lo mejor no esta en esta ruta)
scp root@156.35.33.190:/srv/nfs/replicacion/migracion/*  .

root@nas-00:/srv/nfs# find . -name "*.XVA"


mover maquinas
192.168.233.213 nodo xenserver que se conecta al 192.1168.233.1
mount -t nfs4 192.168.233.1:/ /mnt/intercambio/ -vvv

No usar (preserve=true , esto mantiene la MAC)
xe vm-import filename=./datio.XVA force=true sr-uuid=bdf7c451-2ef0-9982-8ce4-8c31497f3270
````
______________________________________

## instalacion mauqinas virtuales XEN con preseed
Iso imagenes en :
	192.168.25.68/srv/cifs/imagenesISO/	(en el Pool viejo de mieres)
	192.168.11.197:/iso-imagenes	(en el Pool nuevo de mieres)
Preseed en 
	http://192.168.25.125/d-i/stretch/c1nn-debian-stretch-amd64.seed


>>> este es el proceso habitual, pero lo suyo sería crear una plantilla de Xen ya que evita tener que hacer muchos de los pasos	
Hay que crear la maquina virtual con el Xen con las características fisicas deseadas (recordar poner las tarjetas de red en orden)
Como no hay plantiya del SO deseado se puede selecionar el netboot PXB
Uan vez creada la maquina hay que indicarle la imagen ISO a selecionar en le NFS del xenserver.(parece que no se puede indicar imagenes exteriores con la netboot)
Durante el porceso de arranque de la ISO hay que indicar el preseed para evitar hacer la instalación a mano
	>>>> se puede poner el preseed tambien con la ISO (mas practicco) pero mas practico aún sería tener una plantilla Xen creada con todo.
	se presiona ESC cuando aparece el menu de boot del instalador
	auto url=http://192.168.25.125/d-i/stretch/c1nn-debian-stretch-amd64.seed

Sale el interfz grafico.No hay que salir de el, solo hay que editar la linea de las avanzadas [ESC + F6 +ESC]
install auto=true priority=critical url=http://192.168.25.125/d-i/xenial/preseed.cfg


# Dirty ansible
ansible activos -i hosts -m command -a 'ip addr' 


ansible <grupo> -i <fichero de hosts (ini) desde ruta actual> -m <modulo> -a <comando a ejecutar>
ansible-playbook -i hosts executeScriptSalidaLocal.yml --limit activos  

  ansible-playbook -i ../inventory/pruebas/pruebas.ini site-base.yml --limit pruebas
- local_action: copy content={{ output.stdout }} dest=/root/manzano/salida.txt

ansible manzano -i hosts.ini -m command -a 'cat /root/.ssh/innova@innova.uniovi.es.id_rsa' 


# SVN
## Instalacion SVN

````sh
apt install subversion libapache2-mod-svn libsvn-dev
service apache2 restart
## mejor no tocar este nano /etc/apache2/mods-enabled/dav_svn.conf

cat >> /etc/apache2/mods-enabled/dav_svn.conf <<EOF

LDAPTrustedGlobalCert CA_BASE64 /etc/ssl/certs/ca-direcorio.uniovi.es
<Location /svn >
       ##define el acceso al SVN en general
        DAV svn
        SVNParentPath /srv/svn
        SVNListParentPath On

       ##tipo de autenticacion
        AuthType Basic
        AuthName "Universidad de Oviedo"
        AuthBasicProvider file ldap

       ##Tipo fichero
        AuthUserFile /etc/apache2/dav_svn.passwd

       ##Autenticacion con LDAP de Innova que es un active directory
        #AuthBasicProvider ldap
        #AuthzLDAPAuthoritative off
        #AuthLDAPURL ldap://innova.local:389/OU=C1NN,DC=innova,DC=local?sAMAccountName?sub?(objectClass=user)
        #AuthLDAPBindDN "CN=subversion,OU=Roles,OU=C1NN,DC=innova,DC=local"
        #AuthLDAPBindPassword "subversion.subversion"

       ##Autenticacion con LDAP uniovi
        #AuthzLDAPAuthoritative on #deprecated
        #AuthBasicProvider ldap
        AuthLDAPURL ldaps://directorio.uniovi.es:636/dc=uniovi,dc=es?uid?sub
        AuthLDAPBindDN "uid=ti21102004,dc=uniovi,dc=es"
        AuthLDAPBindPassword "pkh_62.zx"

       ##politica de acceso
        AuthzSVNAccessFile /etc/apache2/dav_svn.authz

       ##se exige solo usuairos validados
        Satisfy any
        Require valid-user

        </Location>

EOF
mkdir -p /srv/svn
````
## creacion de nuevo repositorio
````sh
svnadmin create /srv/svn/ejemplo
chown -R www-data:www-data /srv/svn/ejemplo
chmod -R 775 /srv/svn/ejemplo
````

## creacion de usuairos
````sh
## el -c Lo que hace es crear el fichero inicialmente, no usar despues que sobreescribe
htpasswd -cm /etc/apache2/dav_svn.passwd admin
htpasswd -m /etc/apache2/dav_svn.passwd user1
htpasswd -m /etc/apache2/dav_svn.passwd user2

htpasswd -bm /etc/apache2/dav_svn.passwd user2 password


http://example.com/svn/myrepo/
````

## Exportar/immportar
````sh
svnadmin dump -q /srv/svn/padrinoEmprendedor/ > /mnt/temp/padrinoEmprendedor.dump
svnadmin dump -q /srv/svn/web/ > /mnt/temp/web.dump
svnadmin dump -q /srv/svn/desarrollo/ > /mnt/temp/desarrollo.dump

svnadmin create /srv/svn/web
chown -R www-data:www-data /srv/svn/web
chmod -R 775 /srv/svn/web
svnadmin create /srv/svn/desarrollo
chown -R www-data:www-data /srv/svn/desarrollo
chmod -R 775 /srv/svn/desarrollo
svnadmin create /srv/svn/padrinoEmprendedor
chown -R www-data:www-data /srv/svn/padrinoEmprendedor
chmod -R 775 /srv/svn/padrinoEmprendedor

svnadmin load -q /srv/svn/padrinoEmprendedor < /srv/svn/padrinoEmprendedor.dump
svnadmin load -q /srv/svn/web < /srv/svn/web.dump
svnadmin load --force-uuid --memory-cache-size 64 --quiet /srv/svn/desarrollo < /srv/svn/desarrollo.dump
````


El svnrdump puede dar errores de checlksum entre versione sy quizas no haga lo mismo.

rm -rf "/srv/svn/desarrollo" 
svnrdump dump -q http://156.35.33.111/svn/desarrollo/ > /srv/svn/desarrollo.dump
svnadmin create /srv/svn/desarrollo
chown -R www-data:www-data /srv/svn/desarrollo
chmod -R 775 /srv/svn/desarrollo
svnadmin load --force-uuid --memory-cache-size 64 --quiet /srv/svn/desarrollo < /srv/svn/desarrollo.dump




# Copias con dd

en otras versiones hay salida del progreso , en las antiguas no , pero se puede preguntar al proceso con un kill

````sh
dd if=/dev/xvdd1 of=/dev/xvde1 bs=4096 conv=notrunc,noerror,sync
````
````sh
pgrep -l 'dd$'
kill -USR1 26368
````

svnadmin create /srv/svn/prueba
chown -R www-data:www-data /srv/svn/prueba
chmod -R 775 /srv/svn/prueba






#Desde pool viejo
Desde el Xen 192.168.25.221
````sh
mount -t nfs4 192.168.25.190:/ /mnt/migracion/ -vvv
##xe template-param-set is-a-template=false uuid=34891e7d-5dfe-8457-ae27-d41f0649554a
nohup xe vm-export  vm=c22b6261-4daf-35f6-25f7-9084349684d4 filename=/mnt/migracion/datio.XVA &

#(a lo mejor no esta en esta ruta)

scp root@156.35.33.190:/srv/nfs/respaldo/migracion/*  .

root@nas-00:/srv/nfs# find . -name "*.XVA"

mover maquinas
192.168.233.213 nodo xenserver que se conecta al 192.1168.233.1
mount -t nfs4 192.168.233.1:/ /mnt/intercambio/ -vvv


192.168.233.213
No usar (preserve=true , esto mantiene la MAC)
xe vm-import filename=./datio.XVA force=true sr-uuid=bdf7c451-2ef0-9982-8ce4-8c31497f3270
````
#Desde el pool nuevo
Hay que utilizar el 147 y el NAS00
Hay qu earrancar el servicio en el NAS00 y descomentar la ultima linea del /etc/exports
````sh
nano cat /etc/exports
	#/srv/nfs/replicacion/migracion/        192.168.10.0/23(rw,sync,fsid=root,crossmnt,no_subtree_check,no_root_squash)
service nfs-kernel-server restart
````

Desd eel 147
````sh
mount -t nfs  192.168.11.190:/ /mnt/migracion -vvv
xe template-param-set is-a-template=false uuid=7bb8526d-777a-3b15-8a32-e5572d14083f
nohup xe vm-export  vm=7bb8526d-777a-3b15-8a32-e5572d14083f filename=/mnt/migracion/campusVirtualOviedo.XVA &
````

Para ver el progreso en el NAS00
````sh
cd /srv/nfs/replicacion/migracion
watch ls -la
````


# Rsync y enlaces duros
No se pueden hacer parciales a subdirectorios
https://www.cyberciti.biz/faq/linux-unix-apple-osx-bsd-rsync-copy-hard-links/
rsync -az -H --delete --numeric-ids /srv/nfs/respaldo/ /srv/nfs/respaldo2/
rsync -n -avz --delete --itemize-changes --numeric-ids /srv/nfs/respaldo/ /srv/nfs/respaldo2/


#Mysql
````sh
mysql -u root -p -e 'show databases;'
##ALTER USER 'root'@'localhost' IDENTIFIED BY 'XXXX';
##SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxx');

rm *.sql
mysqldump --host=156.35.33.160 -u root -p helpdesk > 156.35.33.160_helpdesk.mysql_dump.sql
#mysqldump --host=156.35.33.160 -u root -p osticket > 156.35.33.160_osticket.mysql_dump.sql
#mysqldump --host=156.35.33.160 -u root -p helpdesk > 156.35.33.160_vauth.mysql_dump.sql
mysqladmin -u root -p drop helpdesk
mysqladmin -u root -p create helpdesk
mysql -u root -p helpdesk < 156.35.33.160_helpdesk.mysql_dump.sql
````


# HITACHI
Hitrack Monitor v.8.6
SStorage Navigator

port 1099

URLs for HDS Applicatons (Command Suite)
Device Manager (DvM): http://servername:23015/DeviceManager/login.do
Tuning Manager (TnM): http://servername:23015/TuningManager/
Tiered Storage Manager (TSM): http://servername:23015/TieredStorageManager/
Storage Navigator Modular 2 (SNM2): http://servername:23015/StorageNavigatorModular/Login
Hi-Track Modular: http://servername:6696/CGI-LOGON


https://localhost:4430/Account/Login?ReturnUrl=%2f
administrator/monitor with password hds.


http://127.0.0.1:1099/StorageNavigatorModular/Login
http://localhost:1099/StorageNavigatorModular/Login
http://156.35.210.41:1099/StorageNavigatorModular/Login
http://156.35.210.41:23015/StorageNavigatorModular/Login
http://156.35.210.41:4430/Account/Login
http://156.35.210.41:4430/StorageNavigatorModular/Login
http://156.35.210.41:1099/Account/Login



Instalar hitachi vantara y el monitor en windows
http://156.35.210.41:22015/StorageNavigatorModular/jsp/Title.jsp
System/manager

Montar monitorizacion HUS110
	Cabina HUS
		https://localhost:4430/Account/Login?ReturnUrl=%2f
		administrator/monitor with password hds.

http://localhost:22015/StorageNavigatorModular/jsp/Title.jsp


___________________________________________________________________


