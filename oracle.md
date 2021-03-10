# JDBC
jdbc:oracle:thin:@localhost:1521:database

# Gestión
- Usuarios en mayusculas
- Proceso de creacion de una PDB
-- Se pueden crear desde el asistente de creación de BBDD directamente
-- Se pueden desde la linea de comandos de sql
````´sql
--create PCB
CREATE PLUGGABLE DATABASE pruebasbd ADMIN USER pruebasuseradmin IDENTIFIED BY pruebasuseradmin TABLESPACE pruebastablespace; 
alter pluggable database pruebasbd open;
alter pluggable database pruebasbd save state;

--Crear  tablespace
CREATE TABLESPACE USERS DATAFILE 'usersMine.dbf'   SIZE 5m  AUTOEXTEND ON MAXSIZE UNLIMITED;
alter user USER_OPOS quota unlimited on USERS;
GRANT UNLIMITED TABLESPACE TO USER_OPOS;
````
-- Desde el Oracle Sql Developer
--- Hay que conectarse al SID por defecto XE con el usuario SYS como SYSADM
--- Sacar el navegador de DBAs
---- Crear una PDB 
---- poner como administrador un usuario con un paasword
---- Crear un disparador para el PDB (Para que la arranque)
---- Cambiarla de estado
--- Crear uan nueva conexion con SYS como SYSADM y al servicio del nombre de la PCB
---- Crear un nuevo usuairo par autilizar de schema
---- Cambiarle los permisos para que pueda crear cosas
````sql
GRANT CREATE SESSION, SYSOPER,PDB_DBA,DBA TO "USUARIO1";
GRANT ALL PRIVILEGES TO USUARIO1 WITH ADMIN OPTION;
GRANT "CONNECT" TO USUARIO1 WITH ADMIN OPTION;
GRANT RESOURCE TO USUARIO1 WITH ADMIN OPTION;
````
--- Crear uan nueva conexion con ese nuevo usuario (con permisos por defecto) y al servicio del nombre de la PCB
---- Eso creará cosas en el esquema



# Ejemplo PCB completa
````sql
--------------------------------------------------------
-- Archivo creado  - miércoles-marzo-10-2021   
--------------------------------------------------------
--------------------------------------------------------
--  DDL for Sequence BANCO_SEQ
--------------------------------------------------------

   CREATE SEQUENCE  "USER_OPOS"."BANCO_SEQ"  MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 21 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE  GLOBAL ;
--------------------------------------------------------
--  DDL for Sequence CLIENTE_SEQ
--------------------------------------------------------

   CREATE SEQUENCE  "USER_OPOS"."CLIENTE_SEQ"  MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 21 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE  GLOBAL ;
--------------------------------------------------------
--  DDL for Sequence CUENTA_SEQ
--------------------------------------------------------

   CREATE SEQUENCE  "USER_OPOS"."CUENTA_SEQ"  MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 1 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE  GLOBAL ;
--------------------------------------------------------
--  DDL for Sequence CUENTA_SEQ1
--------------------------------------------------------

   CREATE SEQUENCE  "USER_OPOS"."CUENTA_SEQ1"  MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 21 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE  GLOBAL ;
--------------------------------------------------------
--  DDL for Sequence HISTORICO_OPERACIONES_SEQ
--------------------------------------------------------

   CREATE SEQUENCE  "USER_OPOS"."HISTORICO_OPERACIONES_SEQ"  MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 21 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE  GLOBAL ;
--------------------------------------------------------
--  DDL for Table BANCO
--------------------------------------------------------

  CREATE TABLE "USER_OPOS"."BANCO" 
   (	"ID" NUMBER, 
	"BIC" VARCHAR2(20), 
	"DIRECCION" VARCHAR2(20), 
	"CIF" VARCHAR2(20)
   ) SEGMENT CREATION IMMEDIATE 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Table CLIENTE
--------------------------------------------------------

  CREATE TABLE "USER_OPOS"."CLIENTE" 
   (	"NOMBRE" VARCHAR2(20), 
	"DOCUMENTO" VARCHAR2(20), 
	"APELLIDOS" VARCHAR2(20), 
	"ID" NUMBER, 
	"DIRECCION" VARCHAR2(20)
   ) SEGMENT CREATION IMMEDIATE 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Table CUENTA
--------------------------------------------------------

  CREATE TABLE "USER_OPOS"."CUENTA" 
   (	"ID" NUMBER, 
	"ID_BANCO" NUMBER, 
	"ID_CLIENTE" NUMBER, 
	"SALDO" NUMBER DEFAULT 0, 
	"IBAN" VARCHAR2(20)
   ) SEGMENT CREATION IMMEDIATE 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Table HISTORICO_OPERACIONES
--------------------------------------------------------

  CREATE TABLE "USER_OPOS"."HISTORICO_OPERACIONES" 
   (	"ID" NUMBER, 
	"SALDO_POST" NUMBER, 
	"FECHA" DATE, 
	"SALDO_ANT" NUMBER, 
	"ID_CUENTA" NUMBER
   ) SEGMENT CREATION IMMEDIATE 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
REM INSERTING into USER_OPOS.BANCO
SET DEFINE OFF;
Insert into USER_OPOS.BANCO (ID,BIC,DIRECCION,CIF) values ('1','1','direccion banco 1','1');
Insert into USER_OPOS.BANCO (ID,BIC,DIRECCION,CIF) values ('2','2','direccion banco 2','2');
Insert into USER_OPOS.BANCO (ID,BIC,DIRECCION,CIF) values ('3','3','direccion banco 3','3');
REM INSERTING into USER_OPOS.CLIENTE
SET DEFINE OFF;
Insert into USER_OPOS.CLIENTE (NOMBRE,DOCUMENTO,APELLIDOS,ID,DIRECCION) values ('nombre cliente1','32880382H','Apellido 1','1','direccion 1');
Insert into USER_OPOS.CLIENTE (NOMBRE,DOCUMENTO,APELLIDOS,ID,DIRECCION) values ('nombre2','32880381H','Apellido 2','2','Direccion 2');
REM INSERTING into USER_OPOS.CUENTA
SET DEFINE OFF;
Insert into USER_OPOS.CUENTA (ID,ID_BANCO,ID_CLIENTE,SALDO,IBAN) values ('1','1','1','100','1111');
Insert into USER_OPOS.CUENTA (ID,ID_BANCO,ID_CLIENTE,SALDO,IBAN) values ('2','1','2','160','2222');
REM INSERTING into USER_OPOS.HISTORICO_OPERACIONES
SET DEFINE OFF;
Insert into USER_OPOS.HISTORICO_OPERACIONES (ID,SALDO_POST,FECHA,SALDO_ANT,ID_CUENTA) values ('1','180',to_date('13/02/21','DD/MM/RR'),'200','2');
Insert into USER_OPOS.HISTORICO_OPERACIONES (ID,SALDO_POST,FECHA,SALDO_ANT,ID_CUENTA) values ('2','160',to_date('13/02/21','DD/MM/RR'),'180','2');
--------------------------------------------------------
--  DDL for Index BANCO_PK
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."BANCO_PK" ON "USER_OPOS"."BANCO" ("ID") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index CLIENTE_PK
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."CLIENTE_PK" ON "USER_OPOS"."CLIENTE" ("ID") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index CUENTA_PK
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."CUENTA_PK" ON "USER_OPOS"."CUENTA" ("ID") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index INDEX_BIC
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."INDEX_BIC" ON "USER_OPOS"."BANCO" ("BIC") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index INDEX_DOCUMENTO
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."INDEX_DOCUMENTO" ON "USER_OPOS"."CLIENTE" ("DOCUMENTO") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index INDEX_IBAN
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."INDEX_IBAN" ON "USER_OPOS"."CUENTA" ("IBAN") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index BANCO_PK
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."BANCO_PK" ON "USER_OPOS"."BANCO" ("ID") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index INDEX_BIC
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."INDEX_BIC" ON "USER_OPOS"."BANCO" ("BIC") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index CLIENTE_PK
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."CLIENTE_PK" ON "USER_OPOS"."CLIENTE" ("ID") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index INDEX_DOCUMENTO
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."INDEX_DOCUMENTO" ON "USER_OPOS"."CLIENTE" ("DOCUMENTO") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index CUENTA_PK
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."CUENTA_PK" ON "USER_OPOS"."CUENTA" ("ID") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Index INDEX_IBAN
--------------------------------------------------------

  CREATE UNIQUE INDEX "USER_OPOS"."INDEX_IBAN" ON "USER_OPOS"."CUENTA" ("IBAN") 
  PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS" ;
--------------------------------------------------------
--  DDL for Trigger AUDITORIA
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."AUDITORIA" 
BEFORE UPDATE OF SALDO ON CUENTA 
REFERENCING OLD AS ANTERIOR NEW AS NUEVO 
FOR EACH ROW
BEGIN
   INSERT INTO HISTORICO_OPERACIONES
        (SALDO_POST,FECHA,SALDO_ANT,ID_CUENTA) 
        VALUES
        (:NUEVO.SALDO, SYSDATE, :ANTERIOR.SALDO, :ANTERIOR.ID)
        ;  
END;
/
ALTER TRIGGER "USER_OPOS"."AUDITORIA" ENABLE;
--------------------------------------------------------
--  DDL for Trigger BANCO_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."BANCO_TRG" 
BEFORE INSERT ON BANCO 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT BANCO_SEQ.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."BANCO_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger CLIENTE_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."CLIENTE_TRG" 
BEFORE INSERT ON CLIENTE 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT CLIENTE_SEQ.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."CLIENTE_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger CUENTA_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."CUENTA_TRG" 
BEFORE INSERT ON CUENTA 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT CUENTA_SEQ1.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."CUENTA_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger HISTORICO_OPERACIONES_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."HISTORICO_OPERACIONES_TRG" 
BEFORE INSERT ON HISTORICO_OPERACIONES 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT HISTORICO_OPERACIONES_SEQ.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."HISTORICO_OPERACIONES_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger BANCO_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."BANCO_TRG" 
BEFORE INSERT ON BANCO 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT BANCO_SEQ.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."BANCO_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger CLIENTE_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."CLIENTE_TRG" 
BEFORE INSERT ON CLIENTE 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT CLIENTE_SEQ.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."CLIENTE_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger AUDITORIA
--------------------------------------------------------

CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."AUDITORIA" 
BEFORE UPDATE OF SALDO ON CUENTA 
REFERENCING OLD AS ANTERIOR NEW AS NUEVO 
FOR EACH ROW
BEGIN
   INSERT INTO HISTORICO_OPERACIONES
        (SALDO_POST,FECHA,SALDO_ANT,ID_CUENTA) 
        VALUES
        (:NUEVO.SALDO, SYSDATE, :ANTERIOR.SALDO, :ANTERIOR.ID)
        ;  
END;
/
ALTER TRIGGER "USER_OPOS"."AUDITORIA" ENABLE;
--------------------------------------------------------
--  DDL for Trigger CUENTA_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."CUENTA_TRG" 
BEFORE INSERT ON CUENTA 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT CUENTA_SEQ1.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."CUENTA_TRG" ENABLE;
--------------------------------------------------------
--  DDL for Trigger HISTORICO_OPERACIONES_TRG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "USER_OPOS"."HISTORICO_OPERACIONES_TRG" 
BEFORE INSERT ON HISTORICO_OPERACIONES 
FOR EACH ROW 
BEGIN
  <<COLUMN_SEQUENCES>>
  BEGIN
    IF INSERTING AND :NEW.ID IS NULL THEN
      SELECT HISTORICO_OPERACIONES_SEQ.NEXTVAL INTO :NEW.ID FROM SYS.DUAL;
    END IF;
  END COLUMN_SEQUENCES;
END;
/
ALTER TRIGGER "USER_OPOS"."HISTORICO_OPERACIONES_TRG" ENABLE;
--------------------------------------------------------
--  Constraints for Table BANCO
--------------------------------------------------------

  ALTER TABLE "USER_OPOS"."BANCO" MODIFY ("ID" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."BANCO" ADD CONSTRAINT "BANCO_PK" PRIMARY KEY ("ID")
  USING INDEX PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS"  ENABLE;
--------------------------------------------------------
--  Constraints for Table CLIENTE
--------------------------------------------------------

  ALTER TABLE "USER_OPOS"."CLIENTE" MODIFY ("ID" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."CLIENTE" ADD CONSTRAINT "CLIENTE_PK" PRIMARY KEY ("ID")
  USING INDEX PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS"  ENABLE;
--------------------------------------------------------
--  Constraints for Table CUENTA
--------------------------------------------------------

  ALTER TABLE "USER_OPOS"."CUENTA" MODIFY ("ID" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."CUENTA" MODIFY ("ID_BANCO" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."CUENTA" MODIFY ("ID_CLIENTE" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."CUENTA" MODIFY ("SALDO" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."CUENTA" MODIFY ("IBAN" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."CUENTA" ADD CONSTRAINT "CUENTA_PK" PRIMARY KEY ("ID")
  USING INDEX PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS 
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "USERS"  ENABLE;
--------------------------------------------------------
--  Constraints for Table HISTORICO_OPERACIONES
--------------------------------------------------------

  ALTER TABLE "USER_OPOS"."HISTORICO_OPERACIONES" MODIFY ("ID" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."HISTORICO_OPERACIONES" MODIFY ("SALDO_POST" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."HISTORICO_OPERACIONES" MODIFY ("FECHA" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."HISTORICO_OPERACIONES" MODIFY ("SALDO_ANT" NOT NULL ENABLE);
  ALTER TABLE "USER_OPOS"."HISTORICO_OPERACIONES" MODIFY ("ID_CUENTA" NOT NULL ENABLE);
--------------------------------------------------------
--  Ref Constraints for Table CUENTA
--------------------------------------------------------

  ALTER TABLE "USER_OPOS"."CUENTA" ADD CONSTRAINT "CUENTA_BANCO_FK" FOREIGN KEY ("ID_BANCO")
	  REFERENCES "USER_OPOS"."BANCO" ("ID") ON DELETE CASCADE ENABLE;
  ALTER TABLE "USER_OPOS"."CUENTA" ADD CONSTRAINT "CUENTA_CLIENTE_FK" FOREIGN KEY ("ID_CLIENTE")
	  REFERENCES "USER_OPOS"."CLIENTE" ("ID") ON DELETE CASCADE ENABLE;
--------------------------------------------------------
--  Ref Constraints for Table HISTORICO_OPERACIONES
--------------------------------------------------------

  ALTER TABLE "USER_OPOS"."HISTORICO_OPERACIONES" ADD CONSTRAINT "HISTORICO_OPERACIONES_CUENTA_FK" FOREIGN KEY ("ID_CUENTA")
	  REFERENCES "USER_OPOS"."CUENTA" ("ID") ON DELETE CASCADE ENABLE;

````




