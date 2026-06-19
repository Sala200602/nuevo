# Demo CP vs AP con Docker, MongoDB y Cassandra

Este repositorio contiene una demo practica para comparar dos escenarios de sistemas distribuidos:

- **Aplicacion bancaria CP** usando MongoDB Replica Set.
- **Red social AP** usando Cassandra.

La demo simula una **particion de red** para observar que operaciones se aceptan, cuales se bloquean y como se recupera el sistema.

---

## Estructura del repositorio

```text
Actividad-viernes/
├── 01_infraestructura_docker/
│   ├── README.md
│   └── docker-compose.yml
├── 02_mongodb_bancaria_cp/
│   └── README.md
├── 03_cassandra_redsocial_ap/
│   ├── 01_preparar_cassandra.md
│   └── 02_demo_red_social_ap.md
├── 04_limpieza/
│   └── README.md
├── 05_material_exposicion/
│   ├── README.md
│   ├── Guia_demo_CP_AP_MongoDB_Cassandra.docx
│   └── Guia_demo_CP_AP_MongoDB_Cassandra_explicada.docx
├── README.md
└── TODOS_LOS_COMANDOS.md
```

---

## Tecnologias utilizadas

| Tecnologia | Uso en la demo |
|---|---|
| Docker | Levantar los contenedores de MongoDB y Cassandra |
| Docker Compose | Definir toda la infraestructura en `docker-compose.yml` |
| MongoDB Replica Set | Simular el escenario CP de una aplicacion bancaria |
| Cassandra | Simular el escenario AP de una red social |
| PowerShell | Ejecutar los comandos de la demo |

---

## Orden recomendado para la demo

1. Abrir [01_infraestructura_docker](01_infraestructura_docker/README.md).
2. Ejecutar la demo bancaria en [02_mongodb_bancaria_cp](02_mongodb_bancaria_cp/README.md).
3. Preparar Cassandra con [03_cassandra_redsocial_ap/01_preparar_cassandra.md](03_cassandra_redsocial_ap/01_preparar_cassandra.md).
4. Ejecutar la demo AP en [03_cassandra_redsocial_ap/02_demo_red_social_ap.md](03_cassandra_redsocial_ap/02_demo_red_social_ap.md).
5. Limpiar el entorno con [04_limpieza](04_limpieza/README.md).

Tambien puedes ver la secuencia completa en [TODOS_LOS_COMANDOS.md](TODOS_LOS_COMANDOS.md).

---

## Idea principal

MongoDB representa **CP** porque durante una particion de red prefiere bloquear escrituras si no puede confirmar mayoria.

Cassandra representa **AP** porque con `CONSISTENCY ONE` puede seguir aceptando escrituras aunque un nodo este aislado.

