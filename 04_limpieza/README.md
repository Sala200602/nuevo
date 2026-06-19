# Limpieza de datos y cierre de la demo

Estos comandos sirven para repetir la demo sin datos acumulados o para apagar toda la maqueta.

---

## 1. Verificar que nodo MongoDB es PRIMARY

```powershell
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

**Que hace:** muestra que nodo MongoDB esta como `PRIMARY`.

**Que decir:** antes de borrar datos en MongoDB verifico el `PRIMARY`, porque solo ese nodo acepta escrituras y eliminaciones.

---

## 2. Borrar datos de la aplicacion bancaria

Usa solo el comando del nodo que aparezca como `PRIMARY`.

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.deleteMany({})"
```

**Que hace:** borra todos los documentos de la coleccion `cuentas` si `mongo1` es el `PRIMARY`.

**Que decir:** limpio las operaciones bancarias para poder repetir la demo sin borrar el cluster.

```powershell
docker exec -it mongo2 mongosh --eval "db.getSiblingDB('banco').cuentas.deleteMany({})"
```

**Que hace:** borra los documentos si `mongo2` es el `PRIMARY`.

**Que decir:** tengo esta alternativa porque despues de una particion el `PRIMARY` puede cambiar.

```powershell
docker exec -it mongo3 mongosh --eval "db.getSiblingDB('banco').cuentas.deleteMany({})"
```

**Que hace:** borra los documentos si `mongo3` es el `PRIMARY`.

**Que decir:** si un nodo responde `not primary`, significa que debo ejecutar el borrado en el lider correcto.

---

## 3. Borrar datos de la red social

```powershell
docker exec -it cassandra1 cqlsh -e "TRUNCATE redsocial.posts;"
```

**Que hace:** elimina todos los registros de la tabla `redsocial.posts`.

**Que decir:** `TRUNCATE` limpia la tabla de publicaciones para repetir el escenario AP.

---

## 4. Apagar la maqueta

```powershell
docker compose down
```

**Que hace:** apaga y elimina los contenedores y la red creados por Compose.

**Que decir:** uso este comando cuando ya termine la demo y quiero detener los servicios.

---

## 5. Apagar y borrar todo desde cero

```powershell
docker compose down -v
```

**Que hace:** apaga la maqueta y borra tambien los volumenes de datos.

**Que decir:** este comando deja el entorno limpio, como si fuera una computadora nueva. Si luego vuelvo a levantar la demo, debo inicializar MongoDB otra vez con `rs.initiate`.
