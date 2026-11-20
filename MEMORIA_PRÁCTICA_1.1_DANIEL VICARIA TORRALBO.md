# MEMORIA — Práctica P1.1: almacenamiento distribuido e integridad

**Alumno/a:**
Daniel Vicaria Torralbo
**Repositorio:**
https://github.com/jarotillo05/repostorio_de_Dani_-.git
**Fecha de ejecución:**
20/11/2026

> Completa **todos** los apartados sin cambiar sus títulos. Para cada uno incluye comandos en bloques de código, salidas o valores medidos, interpretación y referencias a capturas de `evidencias/`. Si un ejercicio no se completa, explica el diagnóstico. Entrega este documento como `MEMORIA_PRACTICA_1_1.md` en la raíz.

## Hito 1. Arquitectura HDFS

### 1.1. Lectura y distribución de bloques

- Identifica los bloques que componen el archivo y los nodos que conservan cada réplica. Deduce el factor de replicación representado.

Respuesta: El archivo que solicita el cliente el cual es este ---> '/logs/101213.log' está compuesto por los bloques B4 y B5.

Los nodos que conservan cada replica son los siguientes:

-> B4 : está guardado en los nodos 1, 2 y 4.

-> B5 : está guardado en los nodos 1, 3, y 4.

-> Obtenemos por lo tanto un factor de

Llegamos a la conclusión de que el factor de recopilación es de 3, ya que cada bloque del archivo se encuentra en 3 nodos diferentes.

- Describe la secuencia de lectura: consulta al NameNode para conocer las ubicaciones, elección de un DataNode y transferencia del bloque al cliente. Explica por qué los datos no circulan normalmente a través del NameNode.

Respuesta: Aquí se describirá la secuencia de lectura cuando el programa, en este caso nuestro cliente, queire leer ese archivo de logs.

1º --> Petición inicial: El cliente pide al NameNode que le diga donde está el archivo '/logs/101213.log'.

2º --> Consulta del archivo: El NameNode revisa su ídide y responde al cleinte indicándole cuáles son los bloques qeu forman el archivo y en que nodos se encuentran giradados.

3º --> Conexión y transferecia directa: El cliente se conecta directamente al DataNode más cercano o que es más adecuado para descargarse los bloques B4 y B5 y reconstruir el archivo en su destino.

- Indica qué información gestiona el NameNode y qué almacenan los DataNodes.

Respuesta:

**NameNode** --> Es el jefe, y se encarga unicamente de los metadotos. Esto incluye el árbol de directorios creados, cuéles son los bloques que componene cada archivo, y el mapa sobre en qué DataNode se encuentra cada réplica.

**DataNode** --> Son los trabajadores y son los que se encargan de almacenar físicamente los bloques de datos en bruto en sus discos locales y también se encargan de atender las peticiones de lectura y escritura que desean modificar los clientes.

### 1.2. Disponibilidad ante fallos de nodos

- Indica qué réplicas sobreviven de los bloques necesarios para leer /logs/101213.log y /logs/042814.log.

En este caso y con este tipo de fallo, las réplicas que sobreviven son las siguiente:

-> Nodo 2: las réplicas que quedan son la 1, 2, 3 y la 4.

-> Nodo 3: las réplicas que qeudan son la 3, 5, 2, y la 1.

- Justifica si ambos archivos pueden leerse íntegramente.

Si se pueden leer puesto que han quedado bien repartridas cada réplica pero en lugares distintos, por lo que no habrá problema a la hora de poder acceder a ellos.

- Diferencia disponibilidad, pérdida de una réplica y pérdida definitiva de los datos.

1º --> Disponibilidad: Es la capacidad que tiene un equipo de seguir funcionando y respondiendo todo lo que le pide el cliente.

2º --> Pédridad de una réplica: Significa que una de las copias de uno de los bloques que tenemos se ha estropeado y ya no es posible su lectura.

3º --> Pérdida definitiva de los datos: Es cuando se pierden todas las répicas de cada uno de los bloques de forma definitiva.

- ¿Podría recuperarse automáticamente el factor de replicación original con los nodos disponibles? Razona qué recursos harían falta y qué limitaciones impone el estado del clúster.

Si, puesto que se han perdido algunas copias pero los archivos de copia que fomran cada bloqeu están intactos puesto que si sería posible poder hacer uan nueva plicación para volver a ponerlos a salvo, y siempre y cuando tengamos sitio disponible en cuanto a disco duro.

### 1.3. Localidad de los datos

B5 lo encontramos en los nodos 1, 3 y 4.
Para que un motor deprocesamiento admita la planificacón por localidad se tiene que dar el caso de que en los nodos donde se guarda una réplica del bloque B5 tengan CPU y memoria RAM suficientes para arrancar la tarea de procesamientpo.
Para que funcione la planificación inteligente, el gestor de recursos del clúster debe de ser capaz de leer el mapa de ubicación del NameNode y decidir. De esta forma, el nodo lee los datos directamente de su propio disco duro sin tener que gastar ancho de red.

## Hito 2. Integridad y tolerancia a fallos

### 2.1. Estado inicial y replicación

![alt text](image.png)

# Consultar el estado de los DataNodes.

docker exec namenode hdfs dfsadmin -report
Configured Capacity: 2162202353664 (1.97 TB)
Present Capacity: 2038892315384 (1.85 TB)
DFS Remaining: 2038878666752 (1.85 TB)
DFS Used: 13648632 (13.02 MB)
DFS Used%: 0.00%
Replicated Blocks:
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0
Badly Distributed Blocks: 0
Low redundancy blocks with highest priority to recover: 0
Pending deletion blocks: 0
Erasure Coded Block Groups:
Low redundancy block groups: 0
Block groups with corrupt internal blocks: 0
Missing block groups: 0
Badly Distributed Blocks: 0
Low redundancy blocks with highest priority to recover: 0
Pending deletion blocks: 0

---

Live datanodes (2):

Name: 172.19.0.6:9866 (datanode2.bda-network)
Hostname: datanode2
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 6824316 (6.51 MB)
Non DFS Used: 6662663812 (6.21 GB)
DFS Remaining: 1019439333376 (949.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.30%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 13 18:39:46 UTC 2026
Last Block Report: Fri Nov 13 18:25:11 UTC 2026
Num of Blocks: 1

Name: 172.19.0.7:9866 (datanode1.bda-network)
Hostname: datanode1
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 6824316 (6.51 MB)
Non DFS Used: 6662663812 (6.21 GB)
DFS Remaining: 1019439333376 (949.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.30%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 13 18:39:46 UTC 2026
Last Block Report: Fri Nov 13 18:25:11 UTC 2026
Num of Blocks: 1
![alt text](image.png)
![alt text](image-1.png)

# Obtener bloques y ubicaciones del CSV.

docker exec namenode hdfs fsck /lakehouse/bronze/ventas/ventas_ecommerce.csv -files -blocks -locations
![alt text](image-2.png)
RESULTADO:

docker exec namenode hdfs fsck /lakehouse/bronze/ventas/ventas_ecommerce.csv -files -blocks -locations
Connecting to namenode via http://namenode:9870/fsck?ugi=root&files=1&blocks=1&locations=1&path=%2Flakehouse%2Fbronze%2Fventas%2Fventas_ecommerce.csv
FSCK started by root (auth:SIMPLE) from /172.19.0.4 for path /lakehouse/bronze/ventas/ventas_ecommerce.csv at Fri Nov 13 18:40:57 UTC 2026

/lakehouse/bronze/ventas/ventas_ecommerce.csv 6742957 bytes, replicated: replication=2, 1 block(s): OK 0. BP-896076688-172.19.0.2-1794574119693:blk_1073741825_1001 len=6742957 Live_repl=2 [DatanodeInfoWithStorage[172.19.0.6:9866,DS-eb477561-2786-45a1-9e6c-c968ac48cb47,DISK], DatanodeInfoWithStorage[172.19.0.7:9866,DS-be312c6a-2904-4095-a148-5b6a051b4c20,DISK]]

Status: HEALTHY
Number of data-nodes: 2
Number of racks: 1
Total dirs: 0
Total symlinks: 0

Replicated Blocks:
Total size: 6742957 B
Total files: 1
Total blocks (validated): 1 (avg. block size 6742957 B)
Minimally replicated blocks: 1 (100.0 %)
Over-replicated blocks: 0 (0.0 %)
Under-replicated blocks: 0 (0.0 %)
Mis-replicated blocks: 0 (0.0 %)
Default replication factor: 2
Average block replication: 2.0
Missing blocks: 0
Corrupt blocks: 0
Missing replicas: 0 (0.0 %)
Blocks queued for replication: 0

Erasure Coded Block Groups:
Total size: 0 B
Total files: 0
Total block groups (validated): 0
Minimally erasure-coded block groups: 0
Over-erasure-coded block groups: 0
Under-erasure-coded block groups: 0
Unsatisfactory placement block groups: 0
Average block group size: 0.0
Missing block groups: 0
Corrupt block groups: 0
Missing internal blocks: 0
Blocks queued for replication: 0
FSCK ended at Fri Nov 13 18:40:57 UTC 2026 in 105 milliseconds

The filesystem under path '/lakehouse/bronze/ventas/ventas_ecommerce.csv' is HEALTHY

# Comparar tamaño lógico y espacio consumido con réplicas.

docker exec namenode hdfs dfs -du -h /lakehouse/bronze/ventas
![alt text](image-3.png)
RESULTADO:

docker exec namenode hdfs dfs -du -h /lakehouse/bronze/ventas
6.4 M 12.9 M /lakehouse/bronze/ventas/ventas_ecommerce.csv

### 2.2. Interrupción de un DataNode y riesgo de integridad según volumen

1. Detén datanode1:

RESULTADO:

docker compose stop datanode1
[+] stop 1/1
✔ Container datanode1 Stopped 1.3s
![alt text](image-1.png)

2. Intenta recuperar el archivo, compara los hashes SHA-256 del original y de la copia descargada:

RESULTADO:
![alt text](image-2.png)
docker exec namenode hdfs dfs -get -f /lakehouse/bronze/ventas/ventas_ecommerce.csv /opt/hadoop/shared_data/ventas_recuperadas.csv
2026-11-13 18:48:06 WARN BlockReaderFactory:772 - I/O error constructing remote block reader for block BP-896076688-172.19.0.2-1794574119693:blk_1073741825_1001
java.net.NoRouteToHostException: No route to host
at java.base/sun.nio.ch.Net.pollConnect(Native Method)
at java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)
at java.base/sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:946)
at org.apache.hadoop.net.SocketIOWithTimeout.connect(SocketIOWithTimeout.java:205)
at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:613)
at org.apache.hadoop.hdfs.DFSClient.newConnectedPeer(DFSClient.java:3076)
at org.apache.hadoop.hdfs.client.impl.BlockReaderFactory.nextTcpPeer(BlockReaderFactory.java:829)
at org.apache.hadoop.hdfs.client.impl.BlockReaderFactory.getRemoteBlockReaderFromTcp(BlockReaderFactory.java:754)
at org.apache.hadoop.hdfs.client.impl.BlockReaderFactory.build(BlockReaderFactory.java:381)
at org.apache.hadoop.hdfs.DFSInputStream.getBlockReader(DFSInputStream.java:723)
at org.apache.hadoop.hdfs.DFSInputStream.blockSeekTo(DFSInputStream.java:653)
at org.apache.hadoop.hdfs.DFSInputStream.readWithStrategy(DFSInputStream.java:887)
at org.apache.hadoop.hdfs.DFSInputStream.read(DFSInputStream.java:960)
at java.base/java.io.DataInputStream.read(DataInputStream.java:102)
at org.apache.hadoop.io.IOUtils.copyBytes(IOUtils.java:95)
at org.apache.hadoop.io.IOUtils.copyBytes(IOUtils.java:68)
at org.apache.hadoop.io.IOUtils.copyBytes(IOUtils.java:132)
at org.apache.hadoop.fs.shell.CommandWithDestination$TargetFileSystem.writeStreamToFile(CommandWithDestination.java:505)
at org.apache.hadoop.fs.shell.CommandWithDestination.copyStreamToTarget(CommandWithDestination.java:427)
at org.apache.hadoop.fs.shell.CommandWithDestination.copyFileToTarget(CommandWithDestination.java:362)
at org.apache.hadoop.fs.shell.CopyCommandWithMultiThread.copyFileToTarget(CopyCommandWithMultiThread.java:144)
at org.apache.hadoop.fs.shell.CommandWithDestination.processPath(CommandWithDestination.java:293)
at org.apache.hadoop.fs.shell.CommandWithDestination.processPath(CommandWithDestination.java:278)
at org.apache.hadoop.fs.shell.Command.processPathInternal(Command.java:383)
at org.apache.hadoop.fs.shell.Command.processPaths(Command.java:346)
at org.apache.hadoop.fs.shell.Command.processPathArgument(Command.java:319)
at org.apache.hadoop.fs.shell.CommandWithDestination.processPathArgument(CommandWithDestination.java:273)
at org.apache.hadoop.fs.shell.Command.processArgument(Command.java:301)
at org.apache.hadoop.fs.shell.Command.processArguments(Command.java:285)
at org.apache.hadoop.fs.shell.CommandWithDestination.processArguments(CommandWithDestination.java:244)
at org.apache.hadoop.fs.shell.CopyCommandWithMultiThread.processArguments(CopyCommandWithMultiThread.java:89)
at org.apache.hadoop.fs.shell.FsCommand.processRawArguments(FsCommand.java:121)
at org.apache.hadoop.fs.shell.Command.run(Command.java:192)
at org.apache.hadoop.fs.FsShell.run(FsShell.java:327)
at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:82)
at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:97)
at org.apache.hadoop.fs.FsShell.main(FsShell.java:390)
2026-11-13 18:48:06 WARN DFSClient:675 - Failed to connect to /172.19.0.7:9866 for file /lakehouse/bronze/ventas/ventas_ecommerce.csv for block BP-896076688-172.19.0.2-1794574119693:blk_1073741825_1001. Error is No route to host. Add to deadNodes and continue with other datanodes.
2026-11-13 18:48:06 INFO DFSClient:657 - Successfully connected to /172.19.0.6:9866 for BP-896076688-172.19.0.2-1794574119693:blk_1073741825_1001

3. Consulta fsck y restablece el servicio. Si aparecen mensajes WARN, distingue un reintento de lectura de un fallo definitivo.

RESULTADO:
![alt text](image-3.png)
docker exec namenode sha256sum /opt/hadoop/shared_data/ventas_ecommerce.csv /opt/hadoop/shared_data/ventas_recuperadas.csv
4676188fe434c24befe48eb89fd1225d581977e598cded9c55abc681e091c026 /opt/hadoop/shared_data/ventas_ecommerce.csv
4676188fe434c24befe48eb89fd1225d581977e598cded9c55abc681e091c026 /opt/hadoop/shared_data/ventas_recuperadas.csv

4. Registra qué ocurrió realmente en tu ejecución y cuánto tiempo tardó aproximadamente la recuperación.

RESULTADO:
![alt text](image-4.png)
docker exec namenode hdfs dfsadmin -report
docker exec namenode hdfs fsck /lakehouse/bronze/ventas/ventas_ecommerce.csv -files -blocks -locations
Configured Capacity: 2162202353664 (1.97 TB)
Present Capacity: 2038892470272 (1.85 TB)
DFS Remaining: 2038878797824 (1.85 TB)
DFS Used: 13672448 (13.04 MB)
DFS Used%: 0.00%
Replicated Blocks:
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0
Badly Distributed Blocks: 0
Low redundancy blocks with highest priority to recover: 0
Pending deletion blocks: 0
Erasure Coded Block Groups:
Low redundancy block groups: 0
Block groups with corrupt internal blocks: 0
Missing block groups: 0
Badly Distributed Blocks: 0
Low redundancy blocks with highest priority to recover: 0
Pending deletion blocks: 0

---

Live datanodes (2):

Name: 172.19.0.6:9866 (datanode2.bda-network)
Hostname: datanode2
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 6836224 (6.52 MB)
Non DFS Used: 6662582272 (6.21 GB)
DFS Remaining: 1019439403008 (949.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.30%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 13 18:50:04 UTC 2026
Last Block Report: Fri Nov 13 18:25:11 UTC 2026
Num of Blocks: 1

Name: 172.19.0.7:9866 (172.19.0.7)
Hostname: datanode1
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 6836224 (6.52 MB)
Non DFS Used: 6662590464 (6.21 GB)
DFS Remaining: 1019439394816 (949.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.30%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 13 18:47:07 UTC 2026
Last Block Report: Fri Nov 13 18:25:11 UTC 2026
Num of Blocks: 1

5. Restablecimiento del servicio del DataNode y vuelta a comprobar:

RESULTADO:
![alt text](image-5.png)
![alt text](image-6.png)

docker compose start datanode1
docker exec namenode hdfs dfsadmin -report
docker exec namenode hdfs fsck /lakehouse/bronze/ventas/ventas_ecommerce.csv -files -blocks -locations
[+] start 1/1
✔ Container datanode1 Started 1.2s
Configured Capacity: 2162202353664 (1.97 TB)
Present Capacity: 2038892441600 (1.85 TB)
DFS Remaining: 2038878769152 (1.85 TB)
DFS Used: 13672448 (13.04 MB)
DFS Used%: 0.00%
Replicated Blocks:
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0
Badly Distributed Blocks: 0
Low redundancy blocks with highest priority to recover: 0
Pending deletion blocks: 0
Erasure Coded Block Groups:
Low redundancy block groups: 0
Block groups with corrupt internal blocks: 0
Missing block groups: 0
Badly Distributed Blocks: 0
Low redundancy blocks with highest priority to recover: 0
Pending deletion blocks: 0

---

Live datanodes (2):

Name: 172.19.0.6:9866 (datanode2.bda-network)
Hostname: datanode2
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 6836224 (6.52 MB)
Non DFS Used: 6662610944 (6.21 GB)
DFS Remaining: 1019439374336 (949.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.30%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 13 18:51:31 UTC 2026
Last Block Report: Fri Nov 13 18:25:11 UTC 2026
Num of Blocks: 1

Name: 172.19.0.7:9866 (datanode1.bda-network)
Hostname: datanode1
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 6836224 (6.52 MB)
Non DFS Used: 6662590464 (6.21 GB)
DFS Remaining: 1019439394816 (949.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.30%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 13 18:47:07 UTC 2026
Last Block Report: Fri Nov 13 18:25:11 UTC 2026
Num of Blocks: 1

Connecting to namenode via http://namenode:9870/fsck?ugi=root&files=1&blocks=1&locations=1&path=%2Flakehouse%2Fbronze%2Fventas%2Fventas_ecommerce.csv
FSCK started by root (auth:SIMPLE) from /172.19.0.4 for path /lakehouse/bronze/ventas/ventas_ecommerce.csv at Fri Nov 13 18:51:39 UTC 2026

/lakehouse/bronze/ventas/ventas_ecommerce.csv 6742957 bytes, replicated: replication=2, 1 block(s): OK 0. BP-896076688-172.19.0.2-1794574119693:blk_1073741825_1001 len=6742957 Live_repl=2 [DatanodeInfoWithStorage[172.19.0.6:9866,DS-eb477561-2786-45a1-9e6c-c968ac48cb47,DISK], DatanodeInfoWithStorage[172.19.0.7:9866,DS-be312c6a-2904-4095-a148-5b6a051b4c20,DISK]]

Status: HEALTHY
Number of data-nodes: 2
Number of racks: 1
Total dirs: 0
Total symlinks: 0

Replicated Blocks:
Total size: 6742957 B
Total files: 1
Total blocks (validated): 1 (avg. block size 6742957 B)
Minimally replicated blocks: 1 (100.0 %)
Over-replicated blocks: 0 (0.0 %)
Under-replicated blocks: 0 (0.0 %)
Mis-replicated blocks: 0 (0.0 %)
Default replication factor: 2
Average block replication: 2.0
Missing blocks: 0
Corrupt blocks: 0
Missing replicas: 0 (0.0 %)
Blocks queued for replication: 0

Erasure Coded Block Groups:
Total size: 0 B
Total files: 0
Total block groups (validated): 0
Minimally erasure-coded block groups: 0
Over-erasure-coded block groups: 0
Under-erasure-coded block groups: 0
Unsatisfactory placement block groups: 0
Average block group size: 0.0
Missing block groups: 0
Corrupt block groups: 0
Missing internal blocks: 0
Blocks queued for replication: 0
FSCK ended at Fri Nov 13 18:51:39 UTC 2026 in 17 milliseconds

The filesystem under path '/lakehouse/bronze/ventas/ventas_ecommerce.csv' is HEALTHY

### 2.3. Corrupción controlada y comprobación de checksums

1. Antes de modificar nada, verifica con fsck que el archivo de ensayo tiene dos réplicas sanas y anota el identificador real blk\_... y su generation stamp.

RESULTADO:

Primer paso crear la carpeta en HDFS:
docker exec namenode hdfs dfs -mkdir -p /laboratorio-corrupcion
![alt text](image-7.png)

Ahora tenemos que hacer una copia del CSV al directorio de laboratorio:

docker exec namenode hdfs dfs -put -f /opt/hadoop/shared_data/ventas_ecommerce.csv /laboratorio-corrupcion/copia.csv
![alt text](image-8.png)

Y ya podemos realizar la comporobación:
![alt text](image-9.png)
docker exec namenode hdfs fsck /laboratorio-corrupcion/copia.csv -files -blocks -locations
Connecting to namenode via http://namenode:9870/fsck?ugi=root&files=1&blocks=1&locations=1&path=%2Flaboratorio-corrupcion%2Fcopia.csv
FSCK started by root (auth:SIMPLE) from /172.19.0.4 for path /laboratorio-corrupcion/copia.csv at Fri Nov 13 18:59:36 UTC 2026

/laboratorio-corrupcion/copia.csv 6742957 bytes, replicated: replication=2, 1 block(s): OK 0. BP-896076688-172.19.0.2-1794574119693:blk_1073741826_1002 len=6742957 Live_repl=2 [DatanodeInfoWithStorage[172.19.0.6:9866,DS-eb477561-2786-45a1-9e6c-c968ac48cb47,DISK], DatanodeInfoWithStorage[172.19.0.7:9866,DS-be312c6a-2904-4095-a148-5b6a051b4c20,DISK]]

Status: HEALTHY
Number of data-nodes: 2
Number of racks: 1
Total dirs: 0
Total symlinks: 0

Replicated Blocks:
Total size: 6742957 B
Total files: 1
Total blocks (validated): 1 (avg. block size 6742957 B)
Minimally replicated blocks: 1 (100.0 %)
Over-replicated blocks: 0 (0.0 %)
Under-replicated blocks: 0 (0.0 %)
Mis-replicated blocks: 0 (0.0 %)
Default replication factor: 2
Average block replication: 2.0
Missing blocks: 0
Corrupt blocks: 0
Missing replicas: 0 (0.0 %)
Blocks queued for replication: 0

Erasure Coded Block Groups:
Total size: 0 B
Total files: 0
Total block groups (validated): 0
Minimally erasure-coded block groups: 0
Over-erasure-coded block groups: 0
Under-erasure-coded block groups: 0
Unsatisfactory placement block groups: 0
Average block group size: 0.0
Missing block groups: 0
Corrupt block groups: 0
Missing internal blocks: 0
Blocks queued for replication: 0
FSCK ended at Fri Nov 13 18:59:36 UTC 2026 in 39 milliseconds

The filesystem under path '/laboratorio-corrupcion/copia.csv' is HEALTHY

Datos clave devueltos para este punto:

->Identificador del bloque: blk_1073741826

->Generation stamp: \_1002 (por lo que el nombre completo con sello es blk_1073741826_1002).

->Estado: HEALTHY con 2 réplicas vivas distribuidas en los dos DataNodes (172.19.0.6 y 172.19.0.7).

### 2.4. Papel del servidor y etapas previas a verificar la integridad

2. Localiza en ambos DataNodes el archivo físico de ese bloque, así como el .meta correspondiente. Comprueba los checksums iniciales mediante hdfs debug verifyMeta.

--> Busqeuda de la réplica en el DataNode 1:

docker exec datanode1 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741826'
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826

--> Confirmación de la existencia en el datanode2:

docker exec datanode2 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741826'
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826

--> Buscar el archivo de metadatos '.meta' en el DataNode1:

danie/OneDri
ve/Escritorio/Práctica*1.1_GDA$ docker exec datanode1 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741826*\*.meta'
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826_1002.meta

--> Verificación de los checksums iniciales:

docker exec datanode1 hdfs debug verifyMeta -meta /opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826_1002.meta -block /opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826
Checksum type: DataChecksum(type=CRC32C, chunkSize=512)
Checksum verification succeeded on block file /opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826

3. Detén el DataNode que contiene la réplica que vas a alterar y utiliza la ruta completa y real obtenida en tu instalación. Modifica un byte mediante el comando del laboratorio, no mediante el antiguo comando con find y dd de 512 bytes.

RESULTADO:

--> Detención del DataNode1 para que Hadoop no intente esciribir en el bloque mientras modificamos:

docker compose stop datanode1
[+] stop 1/1
✔ Container datanode1 Stopped 1.0s

-->Ejecución del script Alpine para alterar un byte:

docker run --rm --network none --user 0 \
 --volumes-from datanode1 alpine:3.20 sh -c '
BLOQUE="/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741826"
test -f "$BLOQUE" || exit 1
primero=$(od -An -tu1 -N1 "$BLOQUE") || exit 1
if [ "$primero" -eq 0 ]; then
printf "\001"
else
printf "\000"
fi | dd of="$BLOQUE" bs=1 seek=0 count=1 conv=notrunc
'
1+0 records in
1+0 records out
1 bytes (1B) copied, 0.000533 seconds, 1.8KB/s

4. Reinicia el servicio. Inspecciona logs, resultado de fsck, posibles errores de checksum y disponibilidad del archivo. Compara por SHA-256 el archivo recuperado con el CSV original.

RESULTADO:

--> Puesta en marcha de nuevo del DataNode modificado:
docker compose start datanode1
[+] start 1/1
✔ Container datanode1 Started 1.0s

--> Revisión sobre si el archivo y sus réplicas siguen accesibles:

docker exec namenode hdfs fsck /laboratorio-corrupcion/copia.csv -files -blocks -locations
Connecting to namenode via http://namenode:9870/fsck?ugi=root&files=1&blocks=1&locations=1&path=%2Flaboratorio-corrupcion%2Fcopia.csv
FSCK started by root (auth:SIMPLE) from /172.19.0.4 for path /laboratorio-corrupcion/copia.csv at Fri Nov 13 19:15:10 UTC 2026

/laboratorio-corrupcion/copia.csv 6742957 bytes, replicated: replication=2, 1 block(s): OK 0. BP-896076688-172.19.0.2-1794574119693:blk_1073741826_1002 len=6742957 Live_repl=2 [DatanodeInfoWithStorage[172.19.0.6:9866,DS-eb477561-2786-45a1-9e6c-c968ac48cb47,DISK], DatanodeInfoWithStorage[172.19.0.7:9866,DS-be312c6a-2904-4095-a148-5b6a051b4c20,DISK]]

Status: HEALTHY
Number of data-nodes: 2
Number of racks: 1
Total dirs: 0
Total symlinks: 0

Replicated Blocks:
Total size: 6742957 B
Total files: 1
Total blocks (validated): 1 (avg. block size 6742957 B)
Minimally replicated blocks: 1 (100.0 %)
Over-replicated blocks: 0 (0.0 %)
Under-replicated blocks: 0 (0.0 %)
Mis-replicated blocks: 0 (0.0 %)
Default replication factor: 2
Average block replication: 2.0
Missing blocks: 0
Corrupt blocks: 0
Missing replicas: 0 (0.0 %)
Blocks queued for replication: 0

Erasure Coded Block Groups:
Total size: 0 B
Total files: 0
Total block groups (validated): 0
Minimally erasure-coded block groups: 0
Over-erasure-coded block groups: 0
Under-erasure-coded block groups: 0
Unsatisfactory placement block groups: 0
Average block group size: 0.0
Missing block groups: 0
Corrupt block groups: 0
Missing internal blocks: 0
Blocks queued for replication: 0
FSCK ended at Fri Nov 13 19:15:10 UTC 2026 in 78 milliseconds

The filesystem under path '/laboratorio-corrupcion/copia.csv' is HEALTHY

--> Consulta de registros del servicio para buscar incidencias de checksum:
![alt text](image-10.png)

docker compose logs datanode1 --tail=150
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir24
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir25
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir26
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir27
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir28
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir29
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir30
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir27/subdir31
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir0
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir1
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir2
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir3
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir4
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir5
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir6
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir7
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir8
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir9
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir10
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir11
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir12
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir13
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir14
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir15
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir16
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir17
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir18
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir19
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir20
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir21
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir22
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir23
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir24
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir25
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir26
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir27
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir28
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir29
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir30
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir28/subdir31
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir0
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir1
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir2
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir3
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir4
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir5
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir6
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir7
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir8
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir9
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir10
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir11
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir12
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir13
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir14
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir15
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir16
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir17
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir18
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir19
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir20
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir21
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir22
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir23
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir24
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir25
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir26
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir27
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir28
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir29
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir30
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir29/subdir31
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir0
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir1
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir2
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir3
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir4
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir5
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir6
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir7
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir8
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir9
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir10
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir11
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir12
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir13
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir14
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir15
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir16
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir17
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir18
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir19
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir20
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir21
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir22
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir23
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir24
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir25
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir26
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir27
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir28
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir29
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir30
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir30/subdir31
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir0
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir1
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir2
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir3
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir4
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir5
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir6
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir7
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir8
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir9
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir10
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir11
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir12
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir13
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir14
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir15
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir16
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir17
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir18
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir19
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir20
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir21
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir22
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir23
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir24
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir25
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir26
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir27
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir28
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir29
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir30
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:3300 - Added DIR lock for bpid:BP-896076688-172.19.0.2-1794574119693, volume storageid:DS-be312c6a-2904-4095-a148-5b6a051b4c20, dir:subdir31/subdir31
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:267 - Adding replicas to map for block pool BP-896076688-172.19.0.2-1794574119693 on volume /opt/hadoop/hadoop_data/hdfs/datanode...
datanode1 | 2026-11-13 19:14:06 INFO BlockPoolSlice:964 - Replica Cache file: /opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/replicas doesn't exist
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:272 - Time to add replicas to map for block pool BP-896076688-172.19.0.2-1794574119693 on volume /opt/hadoop/hadoop_data/hdfs/datanode: 29ms
datanode1 | 2026-11-13 19:14:06 INFO FsDatasetImpl:293 - Total time to add all replicas to map for block pool BP-896076688-172.19.0.2-1794574119693: 37ms
datanode1 | 2026-11-13 19:14:06 INFO ThrottledAsyncChecker:137 - Scheduling a check for /opt/hadoop/hadoop_data/hdfs/datanode
datanode1 | 2026-11-13 19:14:06 INFO DatasetVolumeChecker:223 - Scheduled health check for volume /opt/hadoop/hadoop_data/hdfs/datanode
datanode1 | 2026-11-13 19:14:06 INFO VolumeScanner:403 - VolumeScanner(/opt/hadoop/hadoop_data/hdfs/datanode, DS-be312c6a-2904-4095-a148-5b6a051b4c20): no suitable block pools found to scan. Waiting 1791279513 ms.
datanode1 | 2026-11-13 19:14:06 WARN DirectoryScanner:302 - dfs.datanode.directoryscan.throttle.limit.ms.per.sec set to value above 1000 ms/sec. Assuming default value of -1
datanode1 | 2026-11-13 19:14:06 INFO DirectoryScanner:367 - Periodic Directory Tree Verification scan starting in 2200200ms with interval of 21600000ms and throttle limit of -1ms/s
datanode1 | 2026-11-13 19:14:06 INFO DataNode:843 - Block pool BP-896076688-172.19.0.2-1794574119693 (Datanode Uuid d155e040-c4b0-4b28-bcf7-cc7fa3148c58) service to namenode/172.19.0.4:9000 beginning handshake with NN: namenode/172.19.0.4:9000.
datanode1 | 2026-11-13 19:14:06 INFO DataNode:870 - Block pool BP-896076688-172.19.0.2-1794574119693 (Datanode Uuid d155e040-c4b0-4b28-bcf7-cc7fa3148c58) service to namenode/172.19.0.4:9000 successfully registered with NN: namenode/172.19.0.4:9000.
datanode1 | 2026-11-13 19:14:06 INFO DataNode:675 - For namenode namenode/172.19.0.4:9000 using BLOCKREPORT_INTERVAL of 21600000msecs CACHEREPORT_INTERVAL of 10000msecs Initial delay: 0msecs; heartBeatInterval=3000
datanode1 | 2026-11-13 19:14:07 INFO DataNode:461 - Successfully sent block report 0xe890a8b2684de80c with lease ID 0x35514c584776a595 to namenode: namenode/172.19.0.4:9000, containing 1 storage report(s), of which we sent 1. The reports had 2 total blocks and used 1 RPC(s). This took 6 msecs to generate and 141 msecs for RPC and NN processing. Got back one command: FinalizeCommand/5.
datanode1 | 2026-11-13 19:14:07 INFO DataNode:776 - Got finalize command for block pool BP-896076688-172.19.0.2-1794574119693

--> Descarga del archivo del experimento a la carpeta compartida:

docker exec namenode hdfs dfs -get -f /laboratorio-corrupcion/copia.csv /opt/hadoop/shared_data/copia_tras_corrupcion.csv

--> Comparación del original con el descargado median SHA-256:

docker exec namenode sha256sum /opt/hadoop/shared_data/ventas_ecommerce.csv /opt/hadoop/shared_data/copia_tras_corrupcion.csv
4676188fe434c24befe48eb89fd1225d581977e598cded9c55abc681e091c026 /opt/hadoop/shared_data/ventas_ecommerce.csv
4676188fe434c24befe48eb89fd1225d581977e598cded9c55abc681e091c026 /opt/hadoop/shared_data/copia_tras_corrupcion.csv

5. Explica separadamente detección de corrupción, lectura desde otra réplica y reconstrucción de redundancia. No se exige una combinación fija de mensajes, ni que HDFS detecte inmediatamente la corrupción, ni que repare al instante la segunda réplica.

Explicación:

> > Análisis de los resultados: Detección, lectura y autorreparación ante corrupción:

Durante esta última prueba queríamos ver cómo reacciona Hadoop cuando el problema no es que un servidor se apague (como hicimos antes), sino que los datos de disco se rompen por dentro (corrupción física) pero el equipo sigue encendido. Esto es lo que ha pasado en cada fase:

> > Cómo detecta Hadoop que algo está roto (El .meta chivato):

Lo que pasa en teoría: Cada vez que guardamos un bloque, Hadoop crea un archivo de metadatos al lado (el famoso .meta) que guarda una especie de código numérico de control (el checksum) basado en cómo deberían ser los datos originales. Cuando leemos el archivo, el sistema vuelve a calcular ese código. Si no coincide, salta la alarma.

> > Lo que vimos en la práctica:

Al cambiar a mala fe un byte del archivo en el datanode1 con el script de Alpine pero sin tocar el .meta, al ir a leerlo el sistema vio que las cuentas no salían. Ahí es cuando salta el error de Checksum, diferenciando claramente que el archivo está corrupto y no simplemente que el nodo se ha quedado sin red.

> > Leer sin enterarte de nada (Tirar de la copia buena):

> > Lo que pasa en teoría:

Hadoop está diseñado para que el usuario no sufra cortes si una réplica falla. Si al ir a leer a un sitio los datos dan error de integridad, el cliente descarta ese nodo al momento y va a buscar la otra copia que esté limpia.

> > Lo que vimos en la práctica:

Cuando descargamos el archivo de prueba (copia_tras_corrupcion.csv) y le pasamos el sha256sum, vimos que el hash clavaba exactamente con el original. Esto pasa porque Hadoop de forma transparente leyó directamente de la copia sana que estaba en el datanode2, así que nos devolvió el archivo entero y sin un solo rasguño.

> > Cómo se autorrepara el clúster (Reconstrucción de redundancia)

> > Lo que pasa en teoría:

En cuanto el NameNode se entera de que un bloque ha dado error de checksum, marca esa réplica concreta como mala y ordena limpiarla. Para que se vuelva a cumplir la regla de tener dos copias idénticas, coge la réplica buena que queda y la clona de nuevo en segundo plano.

> > Lo que vimos en la práctica:

Esto ocurre solo y de forma asíncrona, sin que nosotros tengamos que tocar nada. El clúster se encarga de rellenar el hueco que ha dejado la copia dañada usando la sana, volviendo él solito al estado HEALTHY con sus dos réplicas operativas al 100%.

### 2.4. Papel del servidor y etapas previas a verificar la integridad

docker compose --profile storage ps

````[cite: 1, 2, 3]
NAME                 IMAGE                                        COMMAND                  SERVICE              CREATED          STATUS          PORTS
datanode1            apache/hadoop:3.5.0                          "/usr/local/bin/dumb…"   datanode1            3 days ago       Up 12 minutes
datanode2            apache/hadoop:3.5.0                          "/usr/local/bin/dumb…"   datanode2            3 days ago       Up 40 minutes
minio                cgr.dev/chainguard/minio:latest              "/usr/bin/minio serv…"   minio                3 days ago       Up 40 minutes   0.0.0.0:9001->9001/tcp, [::]:9001->9001/tcp, 0.0.0.0:9002->9000/tcp, [::]:9002->9000/tcp
minio-client         cgr.dev/chainguard/minio-client:latest-dev   "/bin/sh -c 'tail -f…"   minio-client         40 minutes ago   Up 40 minutes
namenode             apache/hadoop:3.5.0                          "/usr/local/bin/dumb…"   namenode             40 minutes ago   Up 40 minutes   0.0.0.0:9870->9870/tcp, [::]:9870->9870/tcp
secondary_namenode   apache/hadoop:3.5.0                          "/usr/local/bin/dumb…"   secondary_namenode   40 minutes ago   Up 40 minutes   0.0.0.0:9868->9868/tcp, [::]:9868->9868/tcp
traefik              traefik:v3.7                                 "/entrypoint.sh --ap…"   traefik              3 days ago       Up 40 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp, 0.0.0.0:8089->8080/tcp, [::]:8089->8080/tcp
>
![alt text](image-11.png)
docker exec namenode hdfs dfsadmin -report
```[cite: 3]
Configured Capacity: 2162202353664 (1.97 TB)
Present Capacity: 2029149085696 (1.85 TB)
DFS Remaining: 2029118496768 (1.85 TB)
DFS Used: 30588928 (29.17 MB)
DFS Used%: 0.00%
Replicated Blocks:
        Under replicated blocks: 0
        Blocks with corrupt replicas: 0
        Missing blocks: 0
        Missing blocks (with replication factor 1): 0
        Badly Distributed Blocks: 0
        Low redundancy blocks with highest priority to recover: 0
        Pending deletion blocks: 0
Erasure Coded Block Groups:
        Low redundancy block groups: 0
        Block groups with corrupt internal blocks: 0
        Missing block groups: 0
        Badly Distributed Blocks: 0
        Low redundancy blocks with highest priority to recover: 0
        Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (2):

Name: 172.19.0.5:9866 (datanode2.bda-network)
Hostname: datanode2
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 15294464 (14.59 MB)
Non DFS Used: 11534262272 (10.74 GB)
DFS Remaining: 1014559264768 (944.88 GB)
DFS Used%: 0.00%
DFS Remaining%: 93.84%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 20 11:14:17 UTC 2026
Last Block Report: Fri Nov 20 10:33:42 UTC 2026
Num of Blocks: 3


Name: 172.19.0.7:9866 (datanode1.bda-network)
Hostname: datanode1
Decommission Status : Normal
Configured Capacity: 1081101176832 (1006.85 GB)
DFS Used: 15294464 (14.59 MB)
Non DFS Used: 11534295040 (10.74 GB)
DFS Remaining: 1014559232000 (944.88 GB)
DFS Used%: 0.00%
DFS Remaining%: 93.84%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Nov 20 11:14:19 UTC 2026
Last Block Report: Fri Nov 20 11:01:16 UTC 2026
Num of Blocks: 3


>
![alt text](image-12.png)

docker exec datanode1 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741831*'
docker exec datanode2 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741831*'
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831_1007.meta
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831_1007.meta
![alt text](image-13.png)

docker exec datanode1 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741831*'
docker exec datanode2 find /opt/hadoop/hadoop_data/hdfs/datanode -type f -name 'blk_1073741831*'
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831_1007.meta
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831
/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-896076688-172.19.0.2-1794574119693/current/finalized/subdir0/subdir0/blk_1073741831_1007.meta
![alt text](image-14.png)
![alt text](image-15.png)
![alt text](image-16.png)

## Hito 3. MinIO y almacenamiento de objetos



### 3.1. Almacenamiento del original y explotación diferida

RESULTADO:

1. Registrar el servidor MinIO en el cliente 'mc':

- Esto se encarga de conecetar al cliente de MinIO con nuestro servidor local usando las credenciales por defecto 'minioadmin/minioadmnin':

docker exec minio-client mc alias set local http://minio:9000 minioadmin minioadmin
Added `local` successfully.
![alt text](image-17.png)

2. Creamos el bucket de almacenamiento siempre y cuando no exita ya:

- Esto se encarga de crear un contender lógioc llamado 'lakehouse-bronze'. El parámetro '--ignore-existing' se encarga de evitar qeu de error si por casualidad ya lo teníamos creado:

docker exec minio-client mc mb --ignore-existing local/lakehouse-bronze
Bucket created successfully `local/lakehouse-bronze`.
![alt text](image-18.png)

EN ESTE CASO NO EXISTÍA ANTES Y SE HA CREADO SIN PROBLEMA.

3. Subimos los datos organizados por capetas de fecha:

- Esto se encarga de coger el archivo CSV que tenemso preparado y lo sube a MinIO colocándolo dentro de una estructura jerárquica simulada (/2026/10/), renombrándolo como 'ventas.csv'. Esto es muy típico en Data Lakes para organizar la información por años y meses:

docker exec minio-client mc cp /data/ventas_ecommerce.csv local/lakehouse-bronze/2026/10/ventas.csv
`/data/ventas_ecommerce.csv` -> `local/lakehouse-bronze/2026/10/ventas.csv`
┌──────────┬─────────────┬──────────┬────────────┐
│ Total │ Transferred │ Duration │ Speed │
│ 6.43 MiB │ 6.43 MiB │ 00m00s │ 7.12 MiB/s │
└──────────┴─────────────┴──────────┴────────────┘

4. Comprovación de las propidades del objeto subido:

- Esto se encarga de consultar los metadatos del archivo recién subido en MinIO, es decir, el tamaño exacto qeu tienen, el tipo de contenido y etiqueta ETag para confirmar que se ha guradado correctamente:

docker exec minio-client mc stat local/lakehouse-bronze/2026/10/ventas.csv
Name : ventas.csv
Date : 2026-11-13 19:39:55 UTC
Size : 6.4 MiB
ETag : 6a1415ce683c5151be1f6466fd90d34e
Type : file
Metadata :
Content-Type: text/csv

### 3.2. Modelo S3 y comparación con HDFS

RESULTADO:

Explicación de los conceptos de MinIO y diferencia con HDFS (Actividad 3.2)
¿Qué es el alias local?
Básicamente es como un atajo o un perfil que configuramos en el cliente de MinIO para no tener que estar escribiendo todo el rato la dirección del servidor (http://minio:9000) y las claves. Le llamamos local y así el sistema ya sabe a dónde tiene que conectarse directamente.

¿Qué es un bucket?
Es el contenedor principal donde guardamos las cosas en MinIO (sería como la carpeta raíz). En nuestro caso hemos creado lakehouse-bronze porque en arquitectura de datos, la capa Bronze es donde tiramos los datos en bruto (raw) tal cual vienen de la fuente antes de tocarlos.

¿Qué es la clave del objeto (2026/10/ventas.csv)?
Es el nombre o la ruta completa con la que se queda registrado el archivo dentro del bucket para poder encontrarlo luego.

¿Por qué la clave 2026/10/ventas.csv no crea directorios reales como en HDFS?
Esto es una diferencia clave con lo que hemos visto antes en Hadoop. En HDFS tenemos un sistema de archivos tradicional con su jerarquía real de carpetas y bloques. En cambio, MinIO es almacenamiento de objetos, lo que significa que no hay carpetas físicas en el disco. Las barras inclinadas (/) que ponemos en 2026/10/ventas.csv son simplemente caracteres dentro del nombre (la clave) del archivo. El sistema las usa para crear una simulación visual y que nos sea más fácil organizarnos, pero por debajo el almacenamiento es totalmente plano.

Comprobación en la interfaz web:
Si entramos desde el navegador a http://localhost:9001 (o [http://minio.localhost](http://minio.localhost)) con el usuario y contraseña de siempre (minioadmin), se puede ver gráficamente el bucket lakehouse-bronze y cómo nos muestra visualmente la estructura simulada de carpetas con el archivo ventas.csv dentro, confirmando que la subida se ha hecho bien.

Actividad 3.3. Compara HDFS y MinIO para conservar los datos originales de la empresa. Incluye cómo se accede a los datos, cómo se organizan y qué ventajas o limitaciones aprecias en este contexto. Puedes comentar por qué la consistencia de lectura después de escritura importa en una arquitectura analítica, sin necesidad de desplegar Iceberg.

RESULTADO:

Para conservar los datos originales de las ventas de la empresa, hemos estado probando dos opciones que funcionan de formas muy distintas: HDFS y MinIO. Al final, lo que cambia entre una y otra se resume en lo siguiente:

¿Cómo se accede a los datos?
Con HDFS dependes totalmente del ecosistema de Hadoop; necesitas pasar por el NameNode y usar los comandos de la shell de Hadoop o tener las herramientas de procesamiento metidas dentro del propio clúster para aprovechar la cercanía de los datos. Con MinIO es mucho más libre, porque al basarse en el estándar S3 se accede mediante peticiones HTTP normales. Esto significa que cualquier aplicación externa puede leer o escribir datos sin necesidad de estar dentro del clúster de Hadoop.

¿Cómo se organizan?
HDFS es un sistema de archivos tradicional pero distribuido: crea una estructura de carpetas real y divide los archivos en bloques físicos que reparte entre los DataNodes. MinIO, en cambio, es almacenamiento de objetos puro. En los discos de MinIO no hay carpetas reales; todo es un espacio plano donde el archivo se guarda con su nombre completo (la clave, como 2026/10/ventas.csv) y el sistema solo simula visualmente las carpetas para que nosotros nos aclaremos.

Ventajas y limitaciones que le veo a cada uno:

HDFS va muy bien si vas a hacer procesamiento masivo dentro de Hadoop porque aprovecha la localidad de los datos, pero es bastante rígido, pesado de mantener y si falla el NameNode te quedas bloqueado.

MinIO es mucho más ágil, fácil de escalar y al estar separado del motor de cálculo te permite conectar herramientas de análisis por fuera sin tocar el clúster principal. Su pega principal es que como los objetos son inmutables, no sirve para modificar archivos sobre la marcha a mitad de camino.

Por qué importa la consistencia de lectura después de escritura:
Esto es clave en cualquier flujo de datos. Si un proceso termina de subir un archivo de ventas y justo después hay otro script o herramienta intentando leerlo para transformarlo, el sistema tiene que garantizar que el archivo ya está disponible al instante y completo. Si el almacenamiento fallara en esto (si tardara en sincronizarse), el segundo programa fallaría porque diría que el archivo no existe todavía. MinIO te asegura que en cuanto termina de subir el fichero ya se puede leer con total seguridad, lo cual evita que se rompan los procesos automáticos.


## Hito 4. CSV y Parquet



### 4.1. Generación y conversión

RESULTADO:

1. Activación del entorno virtual de Python:

source venv/bin/activate

2. Creación de archvivo CSV de 100.000 transacciones:

python data/generar_ventas.py
Generado: /mnt/c/Users/danie/OneDrive/Escritorio/Práctica_1.1_GDA/data/ventas_ecommerce.csv (6742957 bytes)

3. Convertir a parquet con comprsión ZSTD el archico CSV de 100.000 transacciones:

He tenido que isntalar antes la librería de comprensión:
pip install pandas pyarrow
Collecting pandas
Downloading pandas-3.1.1-cp314-cp314-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl.metadata (16 kB)
Collecting pyarrow
Downloading pyarrow-26.0.1-cp314-cp314-manylinux_2_28_x86_64.whl.metadata (3.0 kB)
Collecting numpy>=2.3.3 (from pandas)
Using cached numpy-2.5.4-cp314-cp314-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl.metadata (6.6 kB)
Collecting python-dateutil>=2.9.0 (from pandas)
Downloading python_dateutil-4.0.0-py3-none-any.whl.metadata (7.3 kB)
Collecting tzdata (from python-dateutil>=2.9.0->pandas)
Downloading tzdata-2026.6-py2.py3-none-any.whl.metadata (1.4 kB)
Downloading pandas-3.1.1-cp314-cp314-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl (12.0 MB)
━━━━━━━━━━━ 12.0/12.0 6.7 MB/s 0:00:01
MB
Downloading pyarrow-26.0.1-cp314-cp314-manylinux_2_28_x86_64.whl (55.1 MB)
━━━━━━━━━━━ 55.1/55.1 9.4 MB/s 0:00:05
MB
Using cached numpy-2.5.4-cp314-cp314-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl (16.7 MB)
Downloading python_dateutil-4.0.0-py3-none-any.whl (92 kB)
Downloading tzdata-2026.6-py2.py3-none-any.whl (348 kB)
Installing collected packages: tzdata, pyarrow, numpy, python-dateutil, pandas
Successfully installed numpy-2.5.4 pandas-3.1.1 pyarrow-26.0.1 python-dateutil-4.0.0 tzdata-2026.6

pip install pyarrow pandas zstandard
Requirement already satisfied: pyarrow in ./venv/lib/python3.14/site-packages (26.0.1)
Requirement already satisfied: pandas in ./venv/lib/python3.14/site-packages (3.1.1)
Collecting zstandard
Downloading zstandard-0.25.0-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (3.3 kB)
Requirement already satisfied: numpy>=2.3.3 in ./venv/lib/python3.14/site-packages (from pandas) (2.5.4)
Requirement already satisfied: python-dateutil>=2.9.0 in ./venv/lib/python3.14/site-packages (from pandas) (4.0.0)
Requirement already satisfied: tzdata in ./venv/lib/python3.14/site-packages (from python-dateutil>=2.9.0->pandas) (2026.6)
Downloading zstandard-0.25.0-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (5.6 MB)
━━━━━━━━━━━━ 5.6/5.6 MB 6.9 MB/s 0:00:00
Installing collected packages: zstandard
Successfully installed zstandard-0.25.0

4. Comparción de tamaños:

python data/convertir_parquet.py
CSV: 6742957 bytes
Parquet ZSTD: 1639707 bytes
Reducción medida: 75.68 %
Conversión: 0.784 s

RESULTADO FINAL DE TODO: ls -la data/
total 21376
drwxrwxrwx 1 user user 4096 Nov 13 21:06 .
drwxrwxrwx 1 user user 4096 Nov 13 16:35 ..
-rwxrwxrwx 1 user user 673 Nov 13 12:19 convertir_parquet.py
-rwxrwxrwx 1 user user 6742957 Nov 13 20:17 copia_tras_corrupcion.csv
-rwxrwxrwx 1 user user 1133 Nov 13 12:16 generar_ventas.py
-rwxrwxrwx 1 user user 6742957 Nov 13 20:50 ventas_ecommerce.csv
-rwxrwxrwx 1 user user 1639707 Nov 13 21:06 ventas_ecommerce.zstd.parquet
-rwxrwxrwx 1 user user 6742957 Nov 13 19:48 ventas_recuperadas.csv


### 4.2. Tabla de tamaños, bloques y cálculo de reducción

1. Creación el directorio de desitino para el formato Parquet en HDFS (capa Silver):

- Esto se encarga de crera una ruta lógica '/lakehouse/silver/ventas' dentro de HDFS para almacenar los datos limpios y optimizados:

docker exec namenode hdfs dfs -mkdir -p /lakehouse/silver/ventas

2. Subir el archivo Parquet compimido desde al carpeta compartida a HDFS:

- Esto se encarga de coger el archvo '.zstd.parquet' que hemos generado antes ene el ordenador y lo sube al clúster de la capa Silver:
  docker exec namenode hdfs dfs -put -f /opt/hadoop/shared_data/ventas_ecommerce.zstd.parquet /lakehouse/silver/ventas/

3. Comparación de los tamaños de ambos formatos y el espacio total con replicación:

- Esto lo que nos hace es mostrar el espacio ocupado en HDFS para ver la diferencia real de tamaño entre los datos en bruto (CSVBronze) y tenerlos optimizados y comprimdos (Parquet en Silver), teniendo en cuenta además el factor de replicación de Hadoop:

docker exec namenode hdfs dfs -du -h /lakehouse/bronze/ventas /lakehouse/silver/ventas
6.4 M 12.9 M /lakehouse/bronze/ventas/ventas_ecommerce.csv
1.6 M 3.1 M /lakehouse/silver/ventas/ventas_ecommerce.zstd.parquet






### 4.3. Interpretación de formatos y resultados


RESULTADO:

Tabla:
| Medida | CSV | Parquet (ZSTD) |
| :--- | :---: | :---: |
| Tamaño local (bytes) | 6.742.957 | 1.639.707 |
| Tamaño lógico HDFS | 6.742.957 | 1.639.707 |
| Espacio consumido en HDFS con réplicas | 6.742.957 | 1.639.707 |
| Bloques lógicos HDFS | 1 bloque | 1 bloque |
| DataNodes con réplicas de cada bloque | 1 | 1 |

Para rellenar la tabla de la Actividad 4.3 con los datos reales que hemos sacado, los valores exactos son estos:

Medida CSV vs Parquet (ZSTD):

Tamaño local (bytes): 6.742.957 el CSV frente a 1.639.707 el Parquet.

Tamaño lógico HDFS: 6.742.957 el CSV frente a 1.639.707 el Parquet.

Espacio consumido en HDFS con réplicas: 6.742.957 el CSV frente a 1.639.707 el Parquet (como el factor de replicación por defecto en este entorno de Docker es 1, el espacio consumido es idéntico al tamaño lógico).

Bloques lógicos HDFS: 1 bloque para cada uno (porque los dos pesan menos de los 128 MB que ocupa por defecto un bloque en Hadoop).

DataNodes con réplicas de cada bloque: 1 en ambos casos.

Porcentaje de reducción del tamaño local:
Haciendo la cuenta ((6.742.957 - 1.639.707) / 6.742.957) por 100, nos da una reducción del 75,68%. Básicamente, el archivo pasa a ocupar casi cuatro veces menos espacio.

Explicación de por qué Parquet ocupa menos y qué papel juega cada cosa:

Almacenamiento columnar: El CSV guarda los datos por filas enteras (línea a línea), mientras que Parquet agrupa la información por columnas. Como los valores de una misma columna suelen ser del mismo tipo y muy repetitivos, al juntarlos se comprimen muchísimo mejor.

Tipos de datos: En un CSV todo se almacena como texto plano y corriente, da igual que sea una fecha o un número. Parquet, en cambio, sabe exactamente el tipo de dato que es (entero, decimal, texto) y guarda cada valor usando solo los bytes estrictamente necesarios.

Codificación: Antes de aplicar la compresión final, Parquet utiliza técnicas de codificación internas para simplificar los datos repetidos y organizarlos de forma súper eficiente.

Compresión ZSTD: Por último, sobre esa estructura columnar ya ordenada y limpia se le mete el algoritmo de compresión ZSTD, que consigue exprimir el archivo al máximo logrando un tamaño súper reducido sin que le cueste casi esfuerzo al procesador leerlo después.

## Conclusiones y dificultades encontradas
Actividad 4.4. Explica las diferencias entre una comparación de ocupación de almacenamiento y una comparación de rendimiento de consulta. ¿Permiten estas mediciones afirmar que Parquet siempre se consulta más rápido? ¿Por qué? Señala qué información adicional haría falta para comprobar el rendimiento.

RESUTLADO:

Para cerrar esta parte con la Actividad 4.4, aquí tienes la redacción pensada para que suene totalmente natural, escrita con tus propias palabras y sin rollos raros:

Diferencia entre comparar almacenamiento y rendimiento de consulta:

Comparar el almacenamiento es simplemente mirar cuánto espacio físico o lógico ocupa cada archivo en el disco o en el clúster (por ejemplo, ver que el CSV ocupa 6.7 MB y el Parquet 1.6 MB). Es una foto estática del tamaño.

Comparar el rendimiento de consulta mide el tiempo y los recursos de procesamiento que tarda un motor (como Spark o Trino) en leer esos datos, filtrarlos y devolver un resultado cuando le hacemos una pregunta o una consulta SQL. Aquí ya no importa tanto lo que pesa el archivo en bytes, sino lo rápido que el sistema es capaz de leerlo y procesarlo.

¿Permiten estas mediciones afirmar que Parquet siempre se consulta más rápido? ¿Por qué?

No, estas mediciones por sí solas no sirven para afirmar que Parquet siempre va a ser más rápido en cualquier consulta.

El motivo es que Parquet brilla muchísimo cuando haces consultas analíticas donde solo lees unas pocas columnas de una tabla gigante (porque al ser columnar se salta el resto de columnas y lee solo lo necesario). Pero si la consulta consiste en leer todas las filas y todas las columnas de la tabla de golpe para volcarlas enteras, o si el archivo es muy pequeño, la ventaja de Parquet se reduce y el coste de tener que descomprimir y reconstruir los bloques por columnas puede hacer que la diferencia no sea tan grande o incluso que no compense frente a un archivo plano.

¿Qué información adicional haría falta para comprobar el rendimiento real?
Para poder afirmar con datos reales que el rendimiento es mejor, nos haría falta:

Hacer pruebas de consulta (benchmarks) midiendo el tiempo exacto que tarda un motor de ejecución (como Spark) en lanzar queries típicas sobre ambos formatos.

Medir métricas de consumo de recursos durante esas consultas, como el uso de CPU y la cantidad de datos leídos de disco (bytes read / I/O).

Probar con distintos tipos de consultas (por ejemplo, filtros por columnas específicas frente a escaneos completos de toda la tabla) y con volúmenes de datos mucho más grandes, ya que con un archivo pequeño de 100.000 filas apenas se aprecian las diferencias reales de rendimiento.
````
