
<!-- MarkdownTOC -->

- Mieres - Maquinas servidoras : Permisos nuevo rango de maquinas de Oviedo
- Mieres - Netscaler parar cabezas de servicio
- Mieres - Maquina  a Migrar : Parar servicios.
- Mieres - Maquinas clientes : Parar
- Oviedo - Parar servicios Maquina Migrada
- Oviedo - Sincronizacion de datos entre maquina/s migrada
        - ejmplo
                - NAS
        - Indices locales
                - SOLR
                - BBDD
- Oviedo - Maquina Migrada como servidor: Revisar Permisos viejo rango, añadir rango maquinas consumidoras Mieres
- Oviedo - Maquina migrada como cliente:  Revisar ips, Consumidores Oviedo Revisar la IP vieja si sigue vigente o ya existe una en Oviedo.
- Mieres - Maquinas clientes : Configurar nueva IP de maquina servidora oviedo
- Mieres - Maquina migrada apagar e indicar en el nombre en el XenServer que está migrada
- Oviedo - Maquina migrada habilitar servicios
- Oviedo - Comprobar funcionamiento maquinas
- Oviedo - Cambio de DNS
- Mieres - Maquinas clientes : Revisar logs por errores
- Mieres - Maquinas Oviedo : Revisar logs por errores
- Mieres - Deshabilitar backup Maquina migrada
- Mieres - Deshabilitar Alertas en NAgios Maquina migrada
- Oviedo - Habilitar backup
- Oviedo - Habilitar Alertas en Nagios y/o Zabbix

<!-- /MarkdownTOC -->

# Mieres - Maquinas servidoras : Permisos nuevo rango de maquinas de Oviedo	
# Mieres - Netscaler parar cabezas de servicio
# Mieres - Maquina  a Migrar : Parar servicios.	
# Mieres - Maquinas clientes : Parar
# Oviedo - Parar servicios Maquina Migrada
# Oviedo - Sincronizacion de datos entre maquina/s migrada

## ejmplo

###NAS
luIx511-tik4lly
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.194:/srv/nfs/portales/var  /srv/nfs/portales/var

##Indices locales
NOTA: pueden estar compartidos en algunas maquinas en el NFS por lo que no sería necesario sincronizarlo
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.184:/var/local/solr/data/liferay_webuniovi/ /var/local/solr/data/liferay_webuniovi/

###SOLR
pr0perteX-.MinCalculaToR
rsync -n -avz --delete --itemize-changes --numeric-ids root@156.35.33.184:/var/local/solr/data/liferay_portales/ /var/local/solr/data/liferay_portales/


### BBDD
-hay que autorizar el acceso antes en el pg_hba.conf
````sh
nano /etc/postgresql/9.1/main/pg_hba.conf
host    all             all             156.35.233.0/24         md5
````
-Hay que insertar la password del origen y destino en /root/.pgpass para que funicone la importación (o que no la pregunte)
````sh
nano /root/.pgpass
````

-En destino
````sh
#zcat 192.168.25.164_intranetfuo_2019-04-14_03h32m03s.pg_dump.sql.gz | psql -h 192.168.25.165 -U dba postgres


pg_dump -C -h 156.35.33.164 -U dba liferayportales > 156.35.33.164_liferayportales.pg_dump.sql
pg_dump -C -h 156.35.33.164 -U dba liferay523addons > 156.35.33.164_liferay523addons.pg_dump.sql
pg_dump -C -h 156.35.33.164 -U dba liferayroles > 156.35.33.164_liferayroles.pg_dump.sql
pg_dump -C -h 156.35.33.164 -U dba gestionpfe > 156.35.33.164_gestionpfe.pg_dump.sql
pg_dump -C -h 156.35.33.164 -U dba viewsflash > 156.35.33.164_viewsflash.pg_dump.sql
pg_dump -C -h 156.35.33.164 -U dba filtroencuestas > 156.35.33.164_filtroencuestas.pg_dump.sql
pg_dump -C -h 156.35.33.164 -U dba censoescuelainformatica > 156.35.33.164_censoescuelainformatica.pg_dump.sql

dropdb -h localhost -p 5432 -U dba liferayportales
dropdb -h localhost -p 5432 -U dba liferay523addons
dropdb -h localhost -p 5432 -U dba liferayroles
dropdb -h localhost -p 5432 -U dba gestionpfe
dropdb -h localhost -p 5432 -U dba viewsflash
dropdb -h localhost -p 5432 -U dba filtroencuestas
dropdb -h localhost -p 5432 -U dba censoescuelainformatica

cat 156.35.33.164_liferayportales.pg_dump.sql | psql -h localhost -U dba postgres
cat 156.35.33.164_liferay523addons.pg_dump.sql | psql -h localhost -U dba postgres
cat 156.35.33.164_liferayroles.pg_dump.sql | psql -h localhost -U dba postgres
cat 156.35.33.164_gestionpfe.pg_dump.sql | psql -h localhost -U dba postgres
cat 156.35.33.164_viewsflash.pg_dump.sql | psql -h localhost -U dba postgres
cat 156.35.33.164_filtroencuestas.pg_dump.sql | psql -h localhost -U dba postgres
cat 156.35.33.164_censoescuelainformatica.pg_dump.sql | psql -h localhost -U dba postgres

````

>>>>recordar eliminar las pass de 
````sh
nano /root/.pgpass
````

-En caso de migracion parcial - Recordar deshabilitar(Cambiar de nombre) las BBDD de Mieres para que no se creen dos origenes operativos de datos

--Se peuden añadir bloqueos para bases de datos concretas tanto en destino como en Origen
````sh
nano /etc/postgresql/9.1/main/pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    liferay6             liferay6             156.35.33.0/24         reject
````

# Oviedo - Maquina Migrada como servidor: Revisar Permisos viejo rango, añadir rango maquinas consumidoras Mieres
# Oviedo - Maquina migrada como cliente:  Revisar ips, Consumidores Oviedo Revisar la IP vieja si sigue vigente o ya existe una en Oviedo.

````sh
##script para buscar ips de equipos a poner en marcha en posibles servidores clientes
for servidorremoto in 141 142 145 146 155 156 160 162 164 184 185 184 190 192 194;
do
        echo buscando en 156.35.233.$servidorremoto;
        for servidor in 141 142 145 146 155 156 160 162 164 184 185 184 190 192 194;
        do
        ssh root@156.35.233.$servidorremoto grep -rnis {'/usr/local','/etc','/var/lib/tomcat6','/var/lib/tomcat7'} --exclude-dir={log,logs} -e 156.35.33.$servidor --color;        
        done;
done;
````


# Mieres - Maquinas clientes : Configurar nueva IP de maquina servidora oviedo

--Quizás es mejor simplemente esperar a que pete
````sh
##script para buscar ips de equipos a poner en marcha en posibles servidores clientes
for servidorremoto in 145 146 155 156 186 131 132 133;
do
        echo buscando en 156.35.33.$servidorremoto;
        for servidor in 141 142 185 162 184 192 105 100 190;
        do
        ssh root@156.35.33.$servidorremoto grep -rnis {'/usr/local','/etc','/var/lib/tomcat6','/var/lib/tomcat7'} --exclude-dir={log,logs} -e 156.35.33.$servidor --color;
        ssh root@156.35.33.$servidorremoto grep -rnis {'/usr/local','/etc','/var/lib/tomcat6','/var/lib/tomcat7'} --exclude-dir={log,logs} -e 192.168.25.$servidor --color;
        done;
done;
````

# Mieres - Maquina migrada apagar e indicar en el nombre en el XenServer que está migrada
# Oviedo - Maquina migrada habilitar servicios
-revisar podibles deshabilitaciones del cron 
````sh
nano /etc/cron.d/XXXXX
````

# Oviedo - Comprobar funcionamiento maquinas
# Oviedo - Cambio de DNS

# Mieres - Maquinas clientes : Revisar logs por errores
# Mieres - Maquinas Oviedo : Revisar logs por errores
# Mieres - Deshabilitar backup Maquina migrada
# Mieres - Deshabilitar Alertas en NAgios Maquina migrada
# Oviedo - Habilitar backup
# Oviedo - Habilitar Alertas en Nagios y/o Zabbix


