# Escenario AP: red social con Cassandra

En este escenario se simula una red social. La idea es demostrar que Cassandra puede priorizar la **disponibilidad** durante una particion de red.

En una red social es aceptable que una publicacion tarde en sincronizarse, pero se espera que la aplicacion siga funcionando.

---

## 1. Simular particion de red

```powershell
docker network disconnect actividad-viernes_default cassandra2
```

**Que hace:** desconecta `cassandra2` de la red Docker.

**Que decir:** estoy simulando una particion de red. Un nodo queda aislado, pero `cassandra1` sigue disponible para aceptar operaciones.

---

## 2. Insertar un post durante la particion

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; INSERT INTO redsocial.posts (id, usuario, contenido) VALUES ('2','ana','post durante particion');"
```

**Que hace:** inserta una publicacion mientras `cassandra2` esta aislado.

**Que decir:** Cassandra acepta la escritura porque usamos `CONSISTENCY ONE`. Solo necesita que un nodo responda, por eso el sistema sigue disponible.

---

## 3. Consultar desde cassandra1

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

**Que hace:** consulta los posts visibles desde `cassandra1`.

**Que decir:** muestro que la red social sigue funcionando durante la particion.

---

## 4. Recuperar el nodo aislado

```powershell
docker network connect actividad-viernes_default cassandra2
```

**Que hace:** vuelve a conectar `cassandra2` a la red Docker.

**Que decir:** aqui termina la particion y el nodo vuelve a comunicarse con el cluster.

```powershell
Start-Sleep -Seconds 30
```

**Que hace:** espera 30 segundos.

**Que decir:** doy tiempo para que Cassandra detecte nuevamente el nodo reconectado.

---

## 5. Sincronizar los datos

```powershell
docker exec -it cassandra1 nodetool repair redsocial
```

**Que hace:** ejecuta una reparacion del keyspace `redsocial`.

**Que decir:** `repair` ayuda a sincronizar las replicas despues de recuperar la red.

---

## 6. Verificar datos en ambos nodos

```powershell
docker exec -it cassandra1 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

**Que hace:** consulta los posts desde `cassandra1`.

**Que decir:** verifico que el nodo que recibio la escritura conserva los datos.

```powershell
docker exec -it cassandra2 cqlsh -e "CONSISTENCY ONE; SELECT * FROM redsocial.posts;"
```

**Que hace:** consulta los posts desde `cassandra2`.

**Que decir:** verifico que despues de la recuperacion el otro nodo tambien puede leer la informacion.

---

## Conclusion del escenario AP

Cassandra demuestra el comportamiento **AP** porque durante una particion sigue aceptando escrituras con `CONSISTENCY ONE`. Luego, cuando la red se recupera, los nodos pueden sincronizarse.
