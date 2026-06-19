# Preparar Cassandra para el escenario AP

En esta parte se prepara Cassandra para simular una red social. Cassandra permite trabajar con consistencia flexible, por eso se usa para demostrar el escenario **AP**.

---

## 1. Verificar que Cassandra este activo

```powershell
docker exec -it cassandra1 nodetool status
```

**Que hace:** muestra el estado de los nodos Cassandra.

**Que decir:** si aparece `UN`, significa `Up/Normal`. Eso indica que el nodo esta encendido y listo.

> Si el contenedor no esta corriendo, revisa con `docker ps -a`. Si aparece `Exited (137)`, normalmente fue por falta de memoria.

---

## 2. Crear el keyspace de la red social

```powershell
docker exec -it cassandra1 cqlsh -e "CREATE KEYSPACE IF NOT EXISTS redsocial WITH replication = {'class':'SimpleStrategy','replication_factor':2};"
```

**Que hace:** crea el keyspace `redsocial` con factor de replicacion 2.

**Que decir:** el keyspace es como la base de datos de la red social. El `replication_factor:2` indica que Cassandra debe guardar replicas en dos nodos.

---

## 3. Crear la tabla de publicaciones

```powershell
docker exec -it cassandra1 cqlsh -e "CREATE TABLE IF NOT EXISTS redsocial.posts (id text PRIMARY KEY, usuario text, contenido text);"
```

**Que hace:** crea la tabla `posts` con tres campos: `id`, `usuario` y `contenido`.

**Que decir:** esta tabla representa publicaciones de usuarios en una red social.

---

## 4. Insertar un post inicial

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; INSERT INTO redsocial.posts (id, usuario, contenido) VALUES ('1','ana','post antes de particion');"
```

**Que hace:** inserta una publicacion usando `CONSISTENCY ONE`.

**Que decir:** con `CONSISTENCY ONE`, Cassandra acepta la escritura con que un solo nodo responda. Esto favorece disponibilidad.

---

## 5. Consultar los posts

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

**Que hace:** muestra las publicaciones guardadas.

**Que decir:** confirmo que Cassandra acepto la publicacion inicial.

---

## Nota importante

En Cassandra la consistencia se puede ajustar por consulta. Para esta demo usamos `CONSISTENCY ONE` porque queremos demostrar disponibilidad: el sistema sigue respondiendo aunque no todos los nodos esten disponibles.
