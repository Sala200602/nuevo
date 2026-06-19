# Escenario CP: aplicacion bancaria con MongoDB

En este escenario se simula una aplicacion bancaria. La idea es demostrar que MongoDB, configurado como Replica Set, prioriza la **consistencia**.

En una aplicacion bancaria es preferible bloquear una operacion antes que guardar un saldo incorrecto.

---

## 1. Verificar el nodo PRIMARY

```powershell
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

**Que hace:** consulta el estado del Replica Set y muestra que nodo es `PRIMARY`.

**Que decir:** antes de insertar operaciones bancarias debo saber cual nodo acepta escrituras. En MongoDB Replica Set, solo escribe el `PRIMARY`.

---

## 2. Insertar una operacion bancaria inicial

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:1000,operacion:'deposito inicial'},{writeConcern:{w:'majority',wtimeout:5000}})"
```

**Que hace:** inserta un documento en la base `banco`, coleccion `cuentas`.

**Que decir:** uso `writeConcern: majority` porque una aplicacion bancaria necesita que la escritura sea confirmada por la mayoria de nodos antes de aceptarse.

> Si `mongo1` no es `PRIMARY`, ejecuta el mismo comando en el nodo que aparezca como `PRIMARY`.

---

## 3. Consultar las operaciones guardadas

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.find().toArray()"
```

**Que hace:** muestra los documentos guardados en la coleccion `cuentas`.

**Que decir:** con esta consulta confirmo que la operacion bancaria inicial fue aceptada.

---

## 4. Simular particion de red

```powershell
docker network disconnect actividad-viernes_default mongo2
docker network disconnect actividad-viernes_default mongo3
```

**Que hace:** desconecta `mongo2` y `mongo3` de la red Docker.

**Que decir:** estoy simulando una particion de red. `mongo1` queda separado de la mayoria del Replica Set.

```powershell
Start-Sleep -Seconds 15
```

**Que hace:** espera 15 segundos.

**Que decir:** doy tiempo a MongoDB para detectar que ya no tiene comunicacion con la mayoria.

---

## 5. Intentar escribir durante la particion

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:500,operacion:'transferencia durante particion'},{writeConcern:{w:'majority',wtimeout:5000}})"
```

**Que hace:** intenta registrar una transferencia cuando no hay mayoria disponible.

**Que decir:** esta operacion debe fallar, bloquearse o mostrar un error como `not primary`. Eso demuestra CP: MongoDB prefiere no aceptar la operacion antes que guardar datos inconsistentes.

---

## 6. Recuperar MongoDB

```powershell
docker network connect actividad-viernes_default mongo2
docker network connect actividad-viernes_default mongo3
```

**Que hace:** vuelve a conectar los nodos a la red Docker.

**Que decir:** aqui termina la particion y el Replica Set puede volver a formar mayoria.

```powershell
Start-Sleep -Seconds 20
```

**Que hace:** espera a que MongoDB estabilice el Replica Set.

**Que decir:** despues de recuperar la red, MongoDB puede elegir nuevamente un `PRIMARY`.

```powershell
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

**Que hace:** revisa que nodo quedo como `PRIMARY`.

**Que decir:** verifico el lider actual antes de volver a escribir.

---

## Conclusion del escenario CP

MongoDB demuestra el comportamiento **CP** porque, ante una particion de red, prefiere mantener la consistencia y bloquear escrituras si no puede confirmar mayoria.
