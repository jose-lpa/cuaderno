Title: Gestión de procesos en PostgreSQL
Date: 2021-06-01 11:02
Category: SQL
Tags: postgresql,backend,procesos,dba
Slug: postgresql-procesos
Authors: José L. Patiño-Andrés
Summary: Comprobación y gestión de procesos en base de datos PostgreSQL

## Obtener procesos en el backend de PostgreSQL

Los procesos del backend de una base de datos PostgreSQL se encuentran 
detallados en la tabla del sistema `pg_stat_activity`.

    :::postgresql
    postgresql=> \d pg_stat_activity
                          View "pg_catalog.pg_stat_activity"
          Column      |           Type           | Collation | Nullable | Default
    ------------------+--------------------------+-----------+----------+---------
     datid            | oid                      |           |          |
     datname          | name                     |           |          |
     pid              | integer                  |           |          |
     leader_pid       | integer                  |           |          |
     usesysid         | oid                      |           |          |
     usename          | name                     |           |          |
     application_name | text                     |           |          |
     client_addr      | inet                     |           |          |
     client_hostname  | text                     |           |          |
     client_port      | integer                  |           |          |
     backend_start    | timestamp with time zone |           |          |
     xact_start       | timestamp with time zone |           |          |
     query_start      | timestamp with time zone |           |          |
     state_change     | timestamp with time zone |           |          |
     wait_event_type  | text                     |           |          |
     wait_event       | text                     |           |          |
     state            | text                     |           |          |
     backend_xid      | xid                      |           |          |
     backend_xmin     | xid                      |           |          |
     query            | text                     |           |          |
     backend_type     | text                     |           |          |

Así por ejemplo si quisiéramos ver un detalle de los procesos activos en una
determinada base de datos podríamos hacer la siguiente consulta:

    :::postgresql
    postgresql=> SELECT pid, client_addr, client_hostname, query_start, state, query FROM pg_stat_activity;
     pid |  client_addr   | client_hostname |          query_start          | state  |                                                                                  query
    -----+----------------+-----------------+-------------------------------+--------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      15 |                |                 |                               |        |
      17 |                |                 |                               |        |
      24 | 127.0.0.1      |                 | 2021-06-01 09:59:12.755756+00 | idle   | ;
     715 | 127.0.0.1      |                 | 2021-06-01 09:59:04.219432+00 | idle   | SELECT archived_count, last_archived_time, failed_count, last_failed_time, stats_reset, current_timestamp FROM pg_catalog.pg_stat_archiver
     739 | 127.0.0.1      |                 | 2021-06-01 09:59:03.984473+00 | idle   | SELECT CAST(ROUND(EXTRACT(EPOCH FROM (current_timestamp - pg_catalog.pg_postmaster_start_time()))) AS BIGINT), pg_catalog.pg_postmaster_start_time(), current_timestamp
     698 | 127.0.0.1      |                 | 2021-06-01 09:58:31.759264+00 | idle   | select google_insights_get_query_string_length_distribution();
     681 | 78.147.238.189 |                 | 2021-06-01 09:54:19.02829+00  | active | ALTER TABLE product ALTER COLUMN payload_metadata TYPE JSONB
     737 | 78.147.238.189 |                 | 2021-06-01 09:59:13.632249+00 | active | SELECT pid, client_addr, client_hostname, query_start, state, query FROM pg_stat_activity;
      13 |                |                 |                               |        |
      12 |                |                 |                               |        |
      14 |                |                 |                               |        |
    (11 rows)

Imaginemos ahora que hemos lanzado una consulta que está bloqueando una tabla y
queremos cancelarla. Lo que haríamos sería ejecutar la consulta anterior para
ver en la tabla qué consultas están actualmente en el backend de PostgreSQL y
revisar cuál de ellas esté posiblemente afectando a nuestra tabla (bloqueándola,
en un estado `idle`, etcétera...).

Entonces, cuando tengamos claro qué consulta queremos cancelar, usamos el
comando `pg_terminate_backend` con el `pid` del proceso de dicha consulta:

    :::postgresql
    SELECT pg_terminate_backend(<PID_CONSULTA>)

Esto nos permite también hacer consultas más elaboradas. Por ejemplo, si
quisiéramos terminar todos los procesos que estuvieran en el estado
`idle in transaction`, podríamos hacer directamente una consulta como ésta:

    :::postgresql
    SELECT pg_terminate_backend(pg_stat_activity.pid) 
    FROM pg_stat_activity 
    WHERE pg_stat_activity.state = 'idle in transaction';

## Detalles

Los diferentes estados de un proceso o consulta en PostgreSQL, columna `state`
en la tabla [`pg_stat_activity`](https://www.postgresql.org/docs/13/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW)
son:

- `active`: El backend está ejecutando la consulta.
- `idle`: El backend está esperando a un nuevo comando del cliente.
- `idle in transaction`: El backend está en una transacción, pero no está
  ejecutando ninguna consulta actualmente.
- `idle in transaction (aborted)`: Este estado es similar al anterior, salvo en
  que una de las declaraciones de la consulta ha arrojado un ERROR.
- `fastpath function call`: El backend está ejecutando una función [fast-path](https://www.postgresql.org/docs/13/libpq-fastpath.html).
- `disabled`: Se reportará este estado si [`track_activities`](https://www.postgresql.org/docs/13/runtime-config-statistics.html#GUC-TRACK-ACTIVITIES)
  se encuentra deshabilitado en este backend.
