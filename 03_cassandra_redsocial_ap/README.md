# Cassandra AP: red social

Esta carpeta contiene la preparacion y la demo del escenario **AP** usando Cassandra.

---

## Archivos

| Archivo | Contenido |
|---|---|
| [01_preparar_cassandra.md](01_preparar_cassandra.md) | Crea el keyspace, la tabla y el primer post |
| [02_demo_red_social_ap.md](02_demo_red_social_ap.md) | Simula particion de red, inserta durante la falla y recupera el sistema |

---

## Idea del escenario

Cassandra representa una red social porque puede priorizar disponibilidad. Con `CONSISTENCY ONE`, una escritura puede aceptarse aunque solo un nodo responda.

Esto permite que la aplicacion siga funcionando durante una particion de red, aunque luego necesite sincronizar los nodos.

