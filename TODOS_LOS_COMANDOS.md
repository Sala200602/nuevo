# Todos los comandos utilizados

Este archivo contiene la secuencia completa de comandos para ejecutar la demo desde cero.

> Ejecutar los comandos en PowerShell desde la carpeta raiz del proyecto.

---

## 1. Entrar a la carpeta del proyecto

```powershell
cd "C:\Users\Usuario\OneDrive\Documentos\4to Semestre\base de datos distribuida\Actividad-viernes\01_infraestructura_docker"
```

---

## 2. Levantar contenedores

```powershell
docker compose up -d
docker ps
```

---

## 3. Inicializar MongoDB Replica Set

```powershell
docker exec -it mongo1 mongosh --eval "rs.initiate({_id:'rs0',members:[{_id:0,host:'mongo1:27017',priority:2},{_id:1,host:'mongo2:27017'},{_id:2,host:'mongo3:27017'}]})"
Start-Sleep -Seconds 10
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

Si sale `already initialized`, continuar con el siguiente paso.

---

## 4. Insertar operacion bancaria inicial

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:1000,operacion:'deposito inicial'},{writeConcern:{w:'majority',wtimeout:5000}})"
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.find().toArray()"
```

Si `mongo1` no es `PRIMARY`, ejecutar el insert en el nodo que aparezca como `PRIMARY`.

---

## 5. Simular particion en MongoDB

```powershell
docker network disconnect actividad-viernes_default mongo2
docker network disconnect actividad-viernes_default mongo3
Start-Sleep -Seconds 15
```

---

## 6. Intentar escritura bancaria durante la particion

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:500,operacion:'transferencia durante particion'},{writeConcern:{w:'majority',wtimeout:5000}})"
```

Resultado esperado: la escritura falla, se bloquea o muestra `not primary`.

---

## 7. Recuperar MongoDB

```powershell
docker network connect actividad-viernes_default mongo2
docker network connect actividad-viernes_default mongo3
Start-Sleep -Seconds 20
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

---

## 8. Insertar despues de recuperar MongoDB

Ejecutar solo en el nodo que aparezca como `PRIMARY`.

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:1500,operacion:'deposito despues de recuperacion'},{writeConcern:{w:'majority',wtimeout:5000}})"
```

```powershell
docker exec -it mongo2 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:1500,operacion:'deposito despues de recuperacion'},{writeConcern:{w:'majority',wtimeout:5000}})"
```

```powershell
docker exec -it mongo3 mongosh --eval "db.getSiblingDB('banco').cuentas.insertOne({cuenta:'A001',saldo:1500,operacion:'deposito despues de recuperacion'},{writeConcern:{w:'majority',wtimeout:5000}})"
```

---

## 9. Preparar Cassandra

```powershell
docker exec -it cassandra1 nodetool status
docker exec -it cassandra1 cqlsh -e "CREATE KEYSPACE IF NOT EXISTS redsocial WITH replication = {'class':'SimpleStrategy','replication_factor':2};"
docker exec -it cassandra1 cqlsh -e "CREATE TABLE IF NOT EXISTS redsocial.posts (id text PRIMARY KEY, usuario text, contenido text);"
```

---

## 10. Insertar post inicial

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; INSERT INTO redsocial.posts (id, usuario, contenido) VALUES ('1','ana','post antes de particion');"
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

---

## 11. Simular particion en Cassandra

```powershell
docker network disconnect actividad-viernes_default cassandra2
```

---

## 12. Insertar post durante la particion

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; INSERT INTO redsocial.posts (id, usuario, contenido) VALUES ('2','ana','post durante particion');"
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

---

## 13. Recuperar Cassandra

```powershell
docker network connect actividad-viernes_default cassandra2
Start-Sleep -Seconds 30
docker exec -it cassandra1 nodetool repair redsocial
```

---

## 14. Verificar datos en ambos nodos

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
docker exec -it cassandra2 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

---

## 15. Limpieza de MongoDB

Primero verificar el `PRIMARY`:

```powershell
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
```

Luego ejecutar solo en el nodo `PRIMARY`:

```powershell
docker exec -it mongo1 mongosh --eval "db.getSiblingDB('banco').cuentas.deleteMany({})"
docker exec -it mongo2 mongosh --eval "db.getSiblingDB('banco').cuentas.deleteMany({})"
docker exec -it mongo3 mongosh --eval "db.getSiblingDB('banco').cuentas.deleteMany({})"
```

---

## 16. Limpieza de Cassandra

```powershell
docker exec -it cassandra1 cqlsh -e "TRUNCATE redsocial.posts;"
```

---

## 17. Apagar la maqueta

```powershell
docker compose down
```

---

## 18. Apagar y borrar todo

```powershell
docker compose down -v
```

