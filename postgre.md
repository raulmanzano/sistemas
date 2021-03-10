# JDBC
jdbc:postgresql://192.168.0.204:5432/basededatos

# Ejemplo bbdd

````sql
--
-- PostgreSQL database dump
--

-- Dumped from database version 10.15
-- Dumped by pg_dump version 13.1

-- Started on 2021-03-10 19:02:25

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;




-- Role: "USER_OPOS"
-- DROP ROLE "USER_OPOS";

CREATE ROLE "USER_OPOS" WITH
  LOGIN
  SUPERUSER
  INHERIT
  CREATEDB
  CREATEROLE
  NOREPLICATION
  ENCRYPTED PASSWORD 'md5eb4f522e31b9f0f3e54f6cb43a0e312b';

ALTER ROLE "USER_OPOS" IN DATABASE basededatos SET search_path TO user_opos;



--
-- TOC entry 2856 (class 1262 OID 16393)
-- Name: basededatos; Type: DATABASE; Schema: -; Owner: -
--

CREATE DATABASE "basededatos" WITH TEMPLATE = template0 ENCODING = 'UTF8' LOCALE = 'Spanish_Spain.1252';


\connect "basededatos"

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- TOC entry 2857 (class 0 OID 0)
-- Name: basededatos; Type: DATABASE PROPERTIES; Schema: -; Owner: -
--

ALTER ROLE "USER_OPOS" IN DATABASE "basededatos" SET "search_path" TO 'user_opos';


\connect "basededatos"

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- TOC entry 8 (class 2615 OID 24586)
-- Name: USER_OPOS; Type: SCHEMA; Schema: -; Owner: -
--

CREATE SCHEMA "USER_OPOS";


--
-- TOC entry 206 (class 1255 OID 24665)
-- Name: cuenta_trigger_function(); Type: FUNCTION; Schema: USER_OPOS; Owner: -
--

CREATE FUNCTION "USER_OPOS"."cuenta_trigger_function"() RETURNS "trigger"
    LANGUAGE "plpgsql"
    AS $$
BEGIN
INSERT
	INTO "USER_OPOS".historico_operaciones(
	saldo_anterior, saldo_posterior, fecha, id_cuenta)
	VALUES (old.saldo,new.saldo, CURRENT_TIMESTAMP ,old.id);
	return new;
END;
$$;


--
-- TOC entry 199 (class 1259 OID 24589)
-- Name: banco; Type: TABLE; Schema: USER_OPOS; Owner: -
--

CREATE TABLE "USER_OPOS"."banco" (
    "id" integer NOT NULL,
    "BIC" numeric NOT NULL,
    "nombre" "text" NOT NULL,
    "CIF" numeric NOT NULL,
    "Direccion" "text" NOT NULL
);


--
-- TOC entry 198 (class 1259 OID 24587)
-- Name: banco_id_seq; Type: SEQUENCE; Schema: USER_OPOS; Owner: -
--

CREATE SEQUENCE "USER_OPOS"."banco_id_seq"
    AS integer
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


--
-- TOC entry 2858 (class 0 OID 0)
-- Dependencies: 198
-- Name: banco_id_seq; Type: SEQUENCE OWNED BY; Schema: USER_OPOS; Owner: -
--

ALTER SEQUENCE "USER_OPOS"."banco_id_seq" OWNED BY "USER_OPOS"."banco"."id";


--
-- TOC entry 201 (class 1259 OID 24603)
-- Name: cliente; Type: TABLE; Schema: USER_OPOS; Owner: -
--

CREATE TABLE "USER_OPOS"."cliente" (
    "id" integer NOT NULL,
    "nombre" "text" NOT NULL,
    "apellidos" "text" NOT NULL,
    "documento" "text" NOT NULL,
    "direccion" "text" NOT NULL
);


--
-- TOC entry 200 (class 1259 OID 24601)
-- Name: cliente_id_seq; Type: SEQUENCE; Schema: USER_OPOS; Owner: -
--

CREATE SEQUENCE "USER_OPOS"."cliente_id_seq"
    AS integer
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


--
-- TOC entry 2859 (class 0 OID 0)
-- Dependencies: 200
-- Name: cliente_id_seq; Type: SEQUENCE OWNED BY; Schema: USER_OPOS; Owner: -
--

ALTER SEQUENCE "USER_OPOS"."cliente_id_seq" OWNED BY "USER_OPOS"."cliente"."id";


--
-- TOC entry 203 (class 1259 OID 24617)
-- Name: cuenta; Type: TABLE; Schema: USER_OPOS; Owner: -
--

CREATE TABLE "USER_OPOS"."cuenta" (
    "id" integer NOT NULL,
    "iban" numeric NOT NULL,
    "saldo" numeric NOT NULL,
    "id_cliente" integer NOT NULL,
    "id_banco" integer NOT NULL
);


--
-- TOC entry 202 (class 1259 OID 24615)
-- Name: cuenta_id_seq; Type: SEQUENCE; Schema: USER_OPOS; Owner: -
--

CREATE SEQUENCE "USER_OPOS"."cuenta_id_seq"
    AS integer
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


--
-- TOC entry 2860 (class 0 OID 0)
-- Dependencies: 202
-- Name: cuenta_id_seq; Type: SEQUENCE OWNED BY; Schema: USER_OPOS; Owner: -
--

ALTER SEQUENCE "USER_OPOS"."cuenta_id_seq" OWNED BY "USER_OPOS"."cuenta"."id";


--
-- TOC entry 205 (class 1259 OID 24652)
-- Name: historico_operaciones; Type: TABLE; Schema: USER_OPOS; Owner: -
--

CREATE TABLE "USER_OPOS"."historico_operaciones" (
    "id" integer NOT NULL,
    "saldo_anterior" integer NOT NULL,
    "saldo_posterior" integer NOT NULL,
    "fecha" "date" NOT NULL,
    "id_cuenta" integer NOT NULL
);


--
-- TOC entry 204 (class 1259 OID 24650)
-- Name: historico_operaciones_id_seq; Type: SEQUENCE; Schema: USER_OPOS; Owner: -
--

CREATE SEQUENCE "USER_OPOS"."historico_operaciones_id_seq"
    AS integer
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


--
-- TOC entry 2861 (class 0 OID 0)
-- Dependencies: 204
-- Name: historico_operaciones_id_seq; Type: SEQUENCE OWNED BY; Schema: USER_OPOS; Owner: -
--

ALTER SEQUENCE "USER_OPOS"."historico_operaciones_id_seq" OWNED BY "USER_OPOS"."historico_operaciones"."id";


--
-- TOC entry 197 (class 1259 OID 16394)
-- Name: tablamaestro; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE "public"."tablamaestro" (
    "campo1" "text",
    "campo2" "text",
    "id" integer NOT NULL
);


--
-- TOC entry 2698 (class 2604 OID 24592)
-- Name: banco id; Type: DEFAULT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."banco" ALTER COLUMN "id" SET DEFAULT "nextval"('"USER_OPOS"."banco_id_seq"'::"regclass");


--
-- TOC entry 2699 (class 2604 OID 24606)
-- Name: cliente id; Type: DEFAULT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cliente" ALTER COLUMN "id" SET DEFAULT "nextval"('"USER_OPOS"."cliente_id_seq"'::"regclass");


--
-- TOC entry 2700 (class 2604 OID 24620)
-- Name: cuenta id; Type: DEFAULT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cuenta" ALTER COLUMN "id" SET DEFAULT "nextval"('"USER_OPOS"."cuenta_id_seq"'::"regclass");


--
-- TOC entry 2701 (class 2604 OID 24655)
-- Name: historico_operaciones id; Type: DEFAULT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."historico_operaciones" ALTER COLUMN "id" SET DEFAULT "nextval"('"USER_OPOS"."historico_operaciones_id_seq"'::"regclass");


--
-- TOC entry 2844 (class 0 OID 24589)
-- Dependencies: 199
-- Data for Name: banco; Type: TABLE DATA; Schema: USER_OPOS; Owner: -
--

INSERT INTO "USER_OPOS"."banco" VALUES (1, 1, 'banco1', 1, 'direccion1');
INSERT INTO "USER_OPOS"."banco" VALUES (2, 2, 'banco2
', 2, 'dieccion2');


--
-- TOC entry 2846 (class 0 OID 24603)
-- Dependencies: 201
-- Data for Name: cliente; Type: TABLE DATA; Schema: USER_OPOS; Owner: -
--

INSERT INTO "USER_OPOS"."cliente" VALUES (1, 'cliente1', 'apellidos 1', '1', 'difrecciion cliente');
INSERT INTO "USER_OPOS"."cliente" VALUES (2, 'cliente2
', 'aplellidos2', '2', 'direccion 2');


--
-- TOC entry 2848 (class 0 OID 24617)
-- Dependencies: 203
-- Data for Name: cuenta; Type: TABLE DATA; Schema: USER_OPOS; Owner: -
--

INSERT INTO "USER_OPOS"."cuenta" VALUES (1, 1, 100, 1, 1);
INSERT INTO "USER_OPOS"."cuenta" VALUES (2, 2, 180, 2, 1);


--
-- TOC entry 2850 (class 0 OID 24652)
-- Dependencies: 205
-- Data for Name: historico_operaciones; Type: TABLE DATA; Schema: USER_OPOS; Owner: -
--

INSERT INTO "USER_OPOS"."historico_operaciones" VALUES (3, 200, 180, '2021-02-14', 2);


--
-- TOC entry 2842 (class 0 OID 16394)
-- Dependencies: 197
-- Data for Name: tablamaestro; Type: TABLE DATA; Schema: public; Owner: -
--



--
-- TOC entry 2862 (class 0 OID 0)
-- Dependencies: 198
-- Name: banco_id_seq; Type: SEQUENCE SET; Schema: USER_OPOS; Owner: -
--

SELECT pg_catalog.setval('"USER_OPOS"."banco_id_seq"', 2, true);


--
-- TOC entry 2863 (class 0 OID 0)
-- Dependencies: 200
-- Name: cliente_id_seq; Type: SEQUENCE SET; Schema: USER_OPOS; Owner: -
--

SELECT pg_catalog.setval('"USER_OPOS"."cliente_id_seq"', 2, true);


--
-- TOC entry 2864 (class 0 OID 0)
-- Dependencies: 202
-- Name: cuenta_id_seq; Type: SEQUENCE SET; Schema: USER_OPOS; Owner: -
--

SELECT pg_catalog.setval('"USER_OPOS"."cuenta_id_seq"', 2, true);


--
-- TOC entry 2865 (class 0 OID 0)
-- Dependencies: 204
-- Name: historico_operaciones_id_seq; Type: SEQUENCE SET; Schema: USER_OPOS; Owner: -
--

SELECT pg_catalog.setval('"USER_OPOS"."historico_operaciones_id_seq"', 3, true);


--
-- TOC entry 2705 (class 2606 OID 24599)
-- Name: banco BIC_INDEX; Type: CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."banco"
    ADD CONSTRAINT "BIC_INDEX" UNIQUE ("BIC");


--
-- TOC entry 2708 (class 2606 OID 24597)
-- Name: banco banco_PK; Type: CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."banco"
    ADD CONSTRAINT "banco_PK" PRIMARY KEY ("id");


--
-- TOC entry 2710 (class 2606 OID 24611)
-- Name: cliente cliente_PK; Type: CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cliente"
    ADD CONSTRAINT "cliente_PK" PRIMARY KEY ("id");


--
-- TOC entry 2715 (class 2606 OID 24625)
-- Name: cuenta cuenta_PK; Type: CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cuenta"
    ADD CONSTRAINT "cuenta_PK" PRIMARY KEY ("id");


--
-- TOC entry 2713 (class 2606 OID 24613)
-- Name: cliente dni; Type: CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cliente"
    ADD CONSTRAINT "dni" UNIQUE ("documento");


--
-- TOC entry 2717 (class 2606 OID 24657)
-- Name: historico_operaciones historico_operaciones_pkey; Type: CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."historico_operaciones"
    ADD CONSTRAINT "historico_operaciones_pkey" PRIMARY KEY ("id");


--
-- TOC entry 2703 (class 2606 OID 16401)
-- Name: tablamaestro PK; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY "public"."tablamaestro"
    ADD CONSTRAINT "PK" PRIMARY KEY ("id");


--
-- TOC entry 2706 (class 1259 OID 24600)
-- Name: BI_ index; Type: INDEX; Schema: USER_OPOS; Owner: -
--

CREATE INDEX "BI_ index" ON "USER_OPOS"."banco" USING "btree" ("BIC");


--
-- TOC entry 2711 (class 1259 OID 24614)
-- Name: cliente_documento_index; Type: INDEX; Schema: USER_OPOS; Owner: -
--

CREATE INDEX "cliente_documento_index" ON "USER_OPOS"."cliente" USING "btree" ("documento");


--
-- TOC entry 2720 (class 2620 OID 24668)
-- Name: cuenta cuenta_trigger; Type: TRIGGER; Schema: USER_OPOS; Owner: -
--

CREATE TRIGGER "cuenta_trigger" BEFORE UPDATE OF "saldo" ON "USER_OPOS"."cuenta" FOR EACH ROW EXECUTE PROCEDURE "USER_OPOS"."cuenta_trigger_function"();


--
-- TOC entry 2719 (class 2606 OID 24645)
-- Name: cuenta banco_fk; Type: FK CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cuenta"
    ADD CONSTRAINT "banco_fk" FOREIGN KEY ("id_banco") REFERENCES "USER_OPOS"."banco"("id") ON DELETE CASCADE NOT VALID;


--
-- TOC entry 2718 (class 2606 OID 24640)
-- Name: cuenta cliente_fk; Type: FK CONSTRAINT; Schema: USER_OPOS; Owner: -
--

ALTER TABLE ONLY "USER_OPOS"."cuenta"
    ADD CONSTRAINT "cliente_fk" FOREIGN KEY ("id_cliente") REFERENCES "USER_OPOS"."cliente"("id") ON DELETE CASCADE NOT VALID;


-- Completed on 2021-03-10 19:02:28

--
-- PostgreSQL database dump complete
--


````