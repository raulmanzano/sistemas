# JDBC
jdbc:informix-sqli://192.168.0.204:9088/oposiciones:INFORMIXSERVER=ol_informix1410;user=informix;password=XXXX


# Gestión
- Hay un interface de linea de comandos en los accesos directos.
-- Abre un command (mejor como administrador) y despues hay que ejecutar C:\Program Files\IBM Informix Software Bundle\bin\dbacess.exe
-- Existen ejemplos de sql en C:\Program Files\IBM Informix Software Bundle\demo\dbaccess\demo_ids


# Ejemplo BBDD completa

````sql
CREATE TABLE banco
       (
       id              serial not null,--no hace falta unique, es redundante
       bic                varchar(255),
       cif                varchar(255),       
       nombre                varchar(255),
       direccion                varchar(255),
       primary key (id)
       );

create index banco_bic_index on banco (bic);--no hay qu eponer el del pk        

insert into banco(bic,cif,nombre,direccion) values ("1111","111","banco1","direccion banco 1"); 
insert into banco(bic,cif,nombre,direccion) values ("2222","222","banco2","direccion banco 2"); 


create table cliente
       (
       id              serial not null,
       nombre                varchar(255),
       apellidos                varchar(255),
       direccion                varchar(255),       
       documento                varchar(255),       
       primary key (id)
       );


create index cliente_documento_index on cliente (documento);        


create table cuenta
       (
       id              serial not null,
       iban                varchar(255),
       saldo                integer,
       id_banco                integer,
       id_cliente                integer,
       primary key (id),
       foreign key (id_cliente) references cliente (id),
       foreign key (id_banco) references banco (id)
       );

create index cuenta_iban_index on cuenta (iban);        


create table historico_ope
       (
       id              serial not null,       
       id_cuenta                integer,
       saldo_anterior                integer,
       saldo_posterior                integer,
       fecha  datetime year to minute,
       primary key (id),
       foreign key (id_cuenta) references cuenta (id)
       );



create trigger cuenta_trigger
update of saldo on cuenta
referencing old as old
new as new
for each row(
insert into historico_ope(id_cuenta,saldo_anterior,saldo_posterior,fecha)
    values (old.id,old.saldo,new.saldo,current)
        );       


````



`````sql
-- oposiciones:informix.banco definition

-- Drop table

-- DROP TABLE oposiciones:informix.banco;

CREATE TABLE oposiciones:informix.banco (
	id serial NOT NULL,
	bic varchar,
	cif varchar,
	nombre varchar,
	direccion varchar,
	PRIMARY KEY (id) CONSTRAINT u108_22
);
CREATE UNIQUE INDEX 108_22 ON oposiciones:informix.banco (id);
CREATE INDEX banco_bic_index ON oposiciones:informix.banco (bic);


-- oposiciones:informix.cliente definition

-- Drop table

-- DROP TABLE oposiciones:informix.cliente;

CREATE TABLE oposiciones:informix.cliente (
	id serial NOT NULL,
	nombre varchar,
	apellidos varchar,
	direccion varchar,
	documento varchar,
	PRIMARY KEY (id) CONSTRAINT u109_24
);
CREATE UNIQUE INDEX 109_24 ON oposiciones:informix.cliente (id);
CREATE INDEX cliente_documento_index ON oposiciones:informix.cliente (documento);


-- oposiciones:informix.ejemplo definition

-- Drop table

-- DROP TABLE oposiciones:informix.ejemplo;

CREATE TABLE oposiciones:informix.ejemplo (
	col1 integer NOT NULL,
	col2 char(25),
	col3 varchar(25),
	col4 decimal(10,2) NOT NULL,
	col5 date,
	PRIMARY KEY (col1) CONSTRAINT u100_1
);
CREATE UNIQUE INDEX 100_1 ON oposiciones:informix.ejemplo (col1);
CREATE UNIQUE INDEX 100_2 ON oposiciones:informix.ejemplo (col3);


-- oposiciones:informix.ejemplo2 definition

-- Drop table

-- DROP TABLE oposiciones:informix.ejemplo2;

CREATE TABLE oposiciones:informix.ejemplo2 (
	id integer NOT NULL,
	PRIMARY KEY (id) CONSTRAINT ejemplo2_pk
);
CREATE UNIQUE INDEX 102_10 ON oposiciones:informix.ejemplo2 (id);


-- oposiciones:informix.ejemplo3 definition

-- Drop table

-- DROP TABLE oposiciones:informix.ejemplo3;

CREATE TABLE oposiciones:informix.ejemplo3 (
	id serial NOT NULL,
	PRIMARY KEY (id) CONSTRAINT ejemplo3_pk
);
CREATE UNIQUE INDEX 103_12 ON oposiciones:informix.ejemplo3 (id);


-- oposiciones:informix.cuenta definition

-- Drop table

-- DROP TABLE oposiciones:informix.cuenta;

CREATE TABLE oposiciones:informix.cuenta (
	id serial NOT NULL,
	iban varchar,
	saldo integer,
	id_banco integer,
	id_cliente integer,
	PRIMARY KEY (id) CONSTRAINT u110_26,
	FOREIGN KEY (id_cliente) REFERENCES oposiciones:informix.cliente(id) ON DELETE RESTRICT ON UPDATE RESTRICT CONSTRAINT r110_27,
	FOREIGN KEY (id_banco) REFERENCES oposiciones:informix.banco(id) ON DELETE RESTRICT ON UPDATE RESTRICT CONSTRAINT r110_28
);
CREATE UNIQUE INDEX 110_26 ON oposiciones:informix.cuenta (id);
CREATE INDEX 110_27 ON oposiciones:informix.cuenta (id_cliente);
CREATE INDEX 110_28 ON oposiciones:informix.cuenta (id_banco);
CREATE INDEX cuenta_iban_index ON oposiciones:informix.cuenta (iban);
create trigger "informix".cuenta_trigger update of saldo on "informix".cuenta referencing old as old new as new                                                                                                                                                 
    for each row
        (
        insert into "informix".historico_ope (id_cuenta,saldo_anterior,saldo_posterior,fecha)  values (old.id ,old.saldo ,new.saldo ,CURRENT year to fraction(3) ));                                                                 

create trigger "informix".cuenta_trigger_2 update of saldo on "informix".cuenta referencing old as old new as new                                                                                                                                               
    for each row
        (
        insert into "informix".historico_ope (id_cuenta,saldo_anterior,saldo_posterior,fecha)  values (old.id ,old.saldo ,new.saldo ,CURRENT year to fraction(3) ));


-- oposiciones:informix.historico_ope definition

-- Drop table

-- DROP TABLE oposiciones:informix.historico_ope;

CREATE TABLE oposiciones:informix.historico_ope (
	id serial NOT NULL,
	id_cuenta integer,
	saldo_anterior integer,
	saldo_posterior integer,
	fecha datetime year to minute,
	PRIMARY KEY (id) CONSTRAINT u111_30,
	FOREIGN KEY (id_cuenta) REFERENCES oposiciones:informix.cuenta(id) ON DELETE RESTRICT ON UPDATE RESTRICT CONSTRAINT r111_31
);
CREATE UNIQUE INDEX 111_30 ON oposiciones:informix.historico_ope (id);
CREATE INDEX 111_31 ON oposiciones:informix.historico_ope (id_cuenta);


create trigger "informix".cuenta_trigger update of saldo on "informix".cuenta referencing old as old new as new                                                                                                                                                 
    for each row
        (
        insert into "informix".historico_ope (id_cuenta,saldo_anterior,saldo_posterior,fecha)  values (old.id ,old.saldo ,new.saldo ,CURRENT year to fraction(3) ));

create trigger "informix".cuenta_trigger_2 update of saldo on "informix".cuenta referencing old as old new as new                                                                                                                                               
    for each row
        (
        insert into "informix".historico_ope (id_cuenta,saldo_anterior,saldo_posterior,fecha)  values (old.id ,old.saldo ,new.saldo ,CURRENT year to fraction(3) ));
````



````sql
{ DATABASE oposiciones  delimiter | }

grant dba to "manzano";

{ TABLE "informix".ejemplo row size = 65 number of columns = 5 index size = 40 }

{ unload file name = ejemp00100.unl number of rows = 0 }

create table "informix".ejemplo 
  (
    col1 integer not null ,
    col2 char(25),
    col3 varchar(25),
    col4 decimal(10,2) not null ,
    col5 date,
    unique (col3) ,
    primary key (col1) 
  );

revoke all on "informix".ejemplo from "public" as "informix";

{ TABLE "informix".ejemplo2 row size = 4 number of columns = 1 index size = 9 }

{ unload file name = ejemp00102.unl number of rows = 0 }

create table "informix".ejemplo2 
  (
    id integer not null ,
    primary key (id)  constraint "informix".ejemplo2_pk
  );

revoke all on "informix".ejemplo2 from "public" as "informix";

{ TABLE "informix".ejemplo3 row size = 4 number of columns = 1 index size = 9 }

{ unload file name = ejemp00103.unl number of rows = 0 }

create table "informix".ejemplo3 
  (
    id serial not null ,
    primary key (id)  constraint "informix".ejemplo3_pk
  );

revoke all on "informix".ejemplo3 from "public" as "informix";

{ TABLE "informix".banco row size = 1028 number of columns = 5 index size = 270 }

{ unload file name = banco00108.unl number of rows = 2 }

create table "informix".banco 
  (
    id serial not null ,
    bic varchar(255),
    cif varchar(255),
    nombre varchar(255),
    direccion varchar(255),
    primary key (id) 
  );

revoke all on "informix".banco from "public" as "informix";

{ TABLE "informix".cliente row size = 1028 number of columns = 5 index size = 270 }

{ unload file name = clien00109.unl number of rows = 0 }

create table "informix".cliente 
  (
    id serial not null ,
    nombre varchar(255),
    apellidos varchar(255),
    direccion varchar(255),
    documento varchar(255),
    primary key (id) 
  );

revoke all on "informix".cliente from "public" as "informix";

{ TABLE "informix".cuenta row size = 272 number of columns = 5 index size = 288 }

{ unload file name = cuent00110.unl number of rows = 0 }

create table "informix".cuenta 
  (
    id serial not null ,
    iban varchar(255),
    saldo integer,
    id_banco integer,
    id_cliente integer,
    primary key (id) 
  );

revoke all on "informix".cuenta from "public" as "informix";

{ TABLE "informix".historico_ope row size = 23 number of columns = 5 index size = 18 }

{ unload file name = histo00111.unl number of rows = 0 }

create table "informix".historico_ope 
  (
    id serial not null ,
    id_cuenta integer,
    saldo_anterior integer,
    saldo_posterior integer,
    fecha datetime year to minute,
    primary key (id) 
  );

revoke all on "informix".historico_ope from "public" as "informix";

grant select on "informix".ejemplo to "public" as "informix";
grant update on "informix".ejemplo to "public" as "informix";
grant insert on "informix".ejemplo to "public" as "informix";
grant delete on "informix".ejemplo to "public" as "informix";
grant index on "informix".ejemplo to "public" as "informix";
grant select on "informix".ejemplo2 to "public" as "informix";
grant update on "informix".ejemplo2 to "public" as "informix";
grant insert on "informix".ejemplo2 to "public" as "informix";
grant delete on "informix".ejemplo2 to "public" as "informix";
grant index on "informix".ejemplo2 to "public" as "informix";
grant select on "informix".ejemplo3 to "public" as "informix";
grant update on "informix".ejemplo3 to "public" as "informix";
grant insert on "informix".ejemplo3 to "public" as "informix";
grant delete on "informix".ejemplo3 to "public" as "informix";
grant index on "informix".ejemplo3 to "public" as "informix";
grant select on "informix".banco to "public" as "informix";
grant update on "informix".banco to "public" as "informix";
grant insert on "informix".banco to "public" as "informix";
grant delete on "informix".banco to "public" as "informix";
grant index on "informix".banco to "public" as "informix";
grant select on "informix".cliente to "public" as "informix";
grant update on "informix".cliente to "public" as "informix";
grant insert on "informix".cliente to "public" as "informix";
grant delete on "informix".cliente to "public" as "informix";
grant index on "informix".cliente to "public" as "informix";
grant select on "informix".cuenta to "public" as "informix";
grant update on "informix".cuenta to "public" as "informix";
grant insert on "informix".cuenta to "public" as "informix";
grant delete on "informix".cuenta to "public" as "informix";
grant index on "informix".cuenta to "public" as "informix";
grant select on "informix".historico_ope to "public" as "informix";
grant update on "informix".historico_ope to "public" as "informix";
grant insert on "informix".historico_ope to "public" as "informix";
grant delete on "informix".historico_ope to "public" as "informix";
grant index on "informix".historico_ope to "public" as "informix";
















revoke usage on language SPL from public ;

grant usage on language SPL to public ;





create index "informix".banco_bic_index on "informix".banco (bic) 
    using btree ;
create index "informix".cliente_documento_index on "informix".cliente 
    (documento) using btree ;
create index "informix".cuenta_iban_index on "informix".cuenta 
    (iban) using btree ;



alter table "informix".cuenta add constraint (foreign key (id_cliente) 
    references "informix".cliente );
alter table "informix".cuenta add constraint (foreign key (id_banco) 
    references "informix".banco );
alter table "informix".historico_ope add constraint (foreign 
    key (id_cuenta) references "informix".cuenta );



create trigger "informix".cuenta_trigger update of saldo on "informix"
    .cuenta referencing old as old new as new
    for each row
        (
        insert into "informix".historico_ope (id_cuenta,saldo_anterior,
    saldo_posterior,fecha)  values (old.id ,old.saldo ,new.saldo ,CURRENT 
    year to fraction(3) ));

create trigger "informix".cuenta_trigger_2 update of saldo on 
    "informix".cuenta referencing old as old new as new
    for each row
        (
        insert into "informix".historico_ope (id_cuenta,saldo_anterior,
    saldo_posterior,fecha)  values (old.id ,old.saldo ,new.saldo ,CURRENT 
    year to fraction(3) ));




update statistics medium for table "informix".sysaggregates (
     owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysams (
     am_owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysattrtypes (
     seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysblobs (
     colno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".syscasts (
     argument_xid, result_xid) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".syscolauth (
     colno, grantee, grantor) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".syscoldepend (
     colno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".syscolumns (
     colno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysconstraints (
     owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysdefaults (
     class, colno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysdistrib (
     colno, seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysfragments (
     evalpos, fragtype, indexname) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysindices (
     owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".syslangauth (
     grantee, grantor) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysobjstate (
     name, owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysopclasses (
     owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysprocauth (
     grantee, grantor) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysprocbody (
     datakey, seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysproccolumns (
     paramid) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysprocedures (
     isproc, numargs, owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysprocplan (
     datakey, planid, seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".systabauth (
     grantee, grantor) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".systables (
     owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".systrigbody (
     datakey, seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".systriggers (
     owner) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysviews (
     seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysxtddesc (
     seqno) 
     resolution   2.00000   0.95000 ;
update statistics medium for table "informix".sysxtdtypes (
     owner, source) 
     resolution   2.00000   0.95000 ;
````

