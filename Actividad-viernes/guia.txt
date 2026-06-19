#3. Inicializar MongoDB Replica Set
##Este paso es obligatorio en una computadora limpia. Evita el error no replset config has been received.
docker exec -it mongo1 mongosh --eval "rs.initiate({_id:'rs0',members:[{_id:0,host:'mongo1:27017',priority:2},{_id:1,host:'mongo2:27017'},{_id:2,host:'mongo3:27017'}]})"
##Entra a mongo1 y crea el Replica Set llamado rs0 con tres miembros: mongo1, mongo2 y mongo3.
##Este comando convierte los tres MongoDB independientes en un cluster con replica set. MongoDB necesita esto para elegir un PRIMARY y replicar datos.
Start-Sleep -Seconds 10
##Espera a que MongoDB termine la eleccion interna del PRIMARY.
##Despues de iniciar el Replica Set, doy unos segundos para que los nodos acuerden quien sera el lider.
##Guia explicada - Demo CP vs AP
docker exec -it mongo1 mongosh --eval "rs.status().members.map(m => ({name:m.name,state:m.stateStr}))"
##Consulta el estado de cada miembro del Replica Set y muestra si es PRIMARY o SECONDARY.
##Solo el PRIMARY acepta escrituras. Este comando me dice en que nodo debo insertar datos.