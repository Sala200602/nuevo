# Material de exposicion

Esta carpeta contiene los documentos de apoyo para explicar la demo.

---

## Archivos incluidos

| Archivo | Descripcion |
|---|---|
| `Guia_demo_CP_AP_MongoDB_Cassandra.docx` | Guia inicial con comandos para la demo |
| `Guia_demo_CP_AP_MongoDB_Cassandra_explicada.docx` | Guia ampliada con explicacion de cada comando y frases para exponer |

---

## Que se utilizo en la demo

- **Docker Desktop** para ejecutar contenedores.
- **Docker Compose** para levantar todos los servicios desde un solo archivo `docker-compose.yml`.
- **MongoDB 8.0** con tres nodos para simular una aplicacion bancaria CP.
- **Cassandra 4.1** con dos nodos para simular una red social AP.
- **PowerShell** para ejecutar los comandos.
- **Particiones de red con Docker** usando `docker network disconnect` y `docker network connect`.

---

## Comparacion usada en la exposicion

| Escenario | Base de datos | Tipo | Comportamiento |
|---|---|---|---|
| Aplicacion bancaria | MongoDB Replica Set | CP | Bloquea escrituras si no hay mayoria |
| Red social | Cassandra | AP | Acepta escrituras con `CONSISTENCY ONE` aunque un nodo este aislado |

---

## Frase para cerrar

MongoDB prioriza consistencia, por eso es adecuado para un banco. Cassandra prioriza disponibilidad, por eso es adecuado para una red social donde se puede sincronizar despues.

