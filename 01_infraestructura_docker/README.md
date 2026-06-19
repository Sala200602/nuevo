# Levantar e inicializar MongoDB Replica Set

Esta parte prepara el escenario **CP** de la demo. MongoDB se usara como una aplicacion bancaria, donde la consistencia es mas importante que aceptar operaciones durante una falla.

---

## 1. Levantar los contenedores

```powershell
docker compose up -d
```

**Que hace:** levanta en segundo plano todos los servicios definidos en `docker-compose.yml`.

**Que decir:** este comando inicia la infraestructura de la demo: tres nodos MongoDB y dos nodos Cassandra.

```powershell
docker ps
```

**Que hace:** muestra los contenedores que estan encendidos.

**Que decir:** verifico que los nodos esten activos antes de hacer operaciones.

---

## 2. Inicializar MongoDB Replica Set

Este paso es obligatorio en una computadora limpia. Si no se ejecuta, puede salir el error:

```text
MongoServerError: no replset config has been received
```

```powershell
docker exec -it mongo1 mongosh --eval "rs.initiate({_id:'rs0',members:[{_id:0,host:'mongo1:27017',priority:2},{_id:1,host:'mongo2:27017'},{_id:2,host:'mongo3:27017'}]})"
```

**Que hace:** entra al contenedor `mongo1` y crea el Replica Set `rs0` con tres nodos: `mongo1`, `mongo2` y `mongo3`.

**Que decir:** este comando convierte tres MongoDB independientes en un cluster. Asi MongoDB puede elegir un nodo `PRIMARY` y replicar datos en los `SECONDARY`.

> Si sale `already initialized`, no es un problema. Significa que el Replica Set ya estaba creado.

---

## 3. Esperar la eleccion del PRIMARY

```powershell
Start-Sleep -Seconds 10
```

**Que hace:** espera 10 segundos.

**Que decir:** despues de inicializar el Replica Set, doy tiempo para que MongoDB elija el nodo primario.

---

## 4. Verificar el estado del Replica Set

```powershell
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

**Que hace:** muestra si cada nodo esta como `PRIMARY` o `SECONDARY`.

**Que decir:** solo el nodo `PRIMARY` acepta escrituras. Por eso reviso este estado antes de insertar datos.

---

## Resultado esperado

Deberias ver algo parecido a:

```text
mongo1: PRIMARY
mongo2: SECONDARY
mongo3: SECONDARY
```

Si el `PRIMARY` es otro nodo, no pasa nada. Para escribir debes usar el nodo que aparezca como `PRIMARY`.
