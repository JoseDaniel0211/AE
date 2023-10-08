# Creación de un Namenode y dos Datanodes

## Introducción
Para la realización de esta actividad se han necesitado dos máquinas virtuales con sistema operativo linux, una máquina será la maestra y la otra será la esclava, en la maestra se hallará el Namenode y un Datanode, mientras que en la máquina esclava habrá un Datanode. Esto no es lo más correcto, ya que si por algún casual se cae el Namenode en este caso también se caería el Datanode y luego seriía más dificil recuperar ambos, pero se ha realizado así la tarea para solo tener que disponer de dos máquinas virtuales en lugar de tres.

## Intalamo Hadoop y se realiza su configuración:
Creamos el usuario _hadoop_
```
sudo adduser hadoop
```
A continuación, añade el usuario hadoop al grupo sudo
```
sudo usermod -aG sudo hadoop
```
Inicia sesión con el usuario _hadoop_ y genera un par de claves:
```
su - hadoop
ssh-keygen -t rsa
```
Deberías obtener una salida del tipo

> Generating public/private rsa key pair.  
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa):  
Created directory '/home/hadoop/.ssh'.  
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in /home/hadoop/.ssh/id_rsa  
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub  
The key fingerprint is:  
SHA256:HG2K6x1aCGuJMqRKJb+GKIDRdKCd8LXnGsB7WSxApno hadoop@ubuntu2004  
The key's randomart image is:  
+---[RSA 3072]----+  
|..=..            |  
| O.+.o   .       |  
|oo*.o + . o      |  
|o .o * o +       |  
|o+E.= o S        |  
|=.+o * o         |  
|*.o.= o o        |  
|=+ o.. + .       |  
|o ..  o .        |
+----[SHA256]-----+  

Añade la clave al fichero de claves autorizadas:
```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 640 ~/.ssh/authorized_keys
```

Verifica que hayas realizado los pasos anteriores correctamente **(cuando hagas ssh no debe pedirte contraseña)**:
```
ssh localhost
```

## Instalar Hadoop

1. Accedemos a su [web](https://hadoop.apache.org) y descargamos la versión indicada por el profesor. 
2. A continuación descomprimimos el fichero
```
tar -xvzf hadoop-X.Y.Z.tar.gz
```
3. Instalamos la aplicación
```
sudo mv hadoop-X.Y.Z /usr/local/hadoop
```
4. Creamos el directorio de _logs_ y le asignamos como propietario hadoop
```
sudo mkdir /usr/local/hadoop/logs
sudo chown -R hadoop:hadoop /usr/local/hadoop
```
5. Añadimos las variables de entorno de Hadoop, para ello
```
nano ~/.bashrc
```
y añadimos las variables:

> export HADOOP_HOME=/usr/local/hadoop  
 export HADOOP_INSTALL=$HADOOP_HOME  
 export HADOOP_MAPRED_HOME=$HADOOP_HOME  
 export HADOOP_COMMON_HOME=$HADOOP_HOME  
 export HADOOP_HDFS_HOME=$HADOOP_HOME  
 export YARN_HOME=$HADOOP_HOME  
 export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native  
 export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin  
 export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"

6. Finalmente, recargamos el bashrc
```
source ~/.bashrc
```
## Configurar Hadoop

### Fichero hadoop-env.sh

Ahora es necesario configurar las variables de entorno de Java en el archivo **hadoop-env.sh** para ajustar la configuración de YARN, HDFS, MapReduce y otros parámetros relacionados con el proyecto de Hadoop.

Añadir al las rutas al JDK de Java, para ello, realizamos la siguiente búsqueda.
1. Buscamos donde está el binario de Java
```
which javac
```
Debería devolver una salida como:
> /usr/bin/javac

2. Con esa ruta, buscamos el directorio del jdk
```
readlink -f /usr/bin/javac
```
Debería devolver la ruta que nosotros necesitamos:
> /usr/lib/jvm/java-11-openjdk-amd64/bin/javac

A continuación, editamos el fichero **hadoop-env.sh**
```
sudo nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
y añadimos las siguientes líneas:
> export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64  
export HADOOP_CLASSPATH+=" $HADOOP_HOME/lib/*.jar"

Activamos Javax
```
cd /usr/local/hadoop/lib
sudo wget https://jcenter.bintray.com/javax/activation/javax.activation-api/1.2.0/javax.activation-api-1.2.0.jar
```

Finalmente, comprobamos que está instalado:
```
hadoop version
```
debería devolver algo como:
> Hadoop 3.2.1  
 Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r b3cbbb467e22ea829b3808f4b7b01d07e0bf3842  
 Compiled by rohithsharmaks on 2019-09-10T15:56Z  
 Compiled with protoc 2.5.0  
 From source with checksum 776eaf9eee9c0ffc370bcbc1888737  
 This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-3.2.1.jar


### Fichero core-site.sh
El archivo "core-site.xml" en Hadoop es un archivo de configuración importante que se utiliza para especificar propiedades fundamentales relacionadas con el sistema de archivos y la infraestructura central de Hadoop. Este archivo se encuentra en el directorio de configuración de Hadoop y se utiliza para definir diversas configuraciones clave. Algunos de los usos y configuraciones comunes del archivo "core-site.xml" incluyen:

1. **Configuración del sistema de archivos Hadoop:** En este archivo, se especifica la ubicación del sistema de archivos distribuido de Hadoop (HDFS). Esto incluye detalles como la dirección del NameNode, que es el componente central del sistema de archivos distribuido, y otros ajustes relacionados con el acceso y la administración de datos en HDFS.

2. **Configuración de ubicación de recursos:** El archivo "core-site.xml" se utiliza para definir la ubicación de recursos compartidos o compartidos, como el directorio raíz del sistema de archivos HDFS o cualquier otro sistema de archivos que se use para el almacenamiento de datos.

3. **Configuración de seguridad:** Puedes configurar propiedades relacionadas con la seguridad en "core-site.xml", como la especificación de rutas de directorios seguros o la configuración de autenticación y autorización.

4. **Configuración de propiedades generales:** Además de las configuraciones específicas de HDFS, también puedes definir propiedades generales de Hadoop en este archivo, como puertos de red y ubicaciones de archivos de registro.

Editamos el fichero core-site.xml
```
sudo nano $HADOOP_HOME/etc/hadoop/core-site.xml
```
y añadimos esto:
```xml
 <configuration>  
   <property>  
      <name>fs.default.name</name>  
      <value>hdfs://0.0.0.0:9000</value>  
      <description>The default file system URI</description>  
   </property>  
 </configuration>
```


### Fichero hdfs-site.sh
El archivo "hdfs-site.xml" en Hadoop es un archivo de configuración utilizado para establecer propiedades específicas del Sistema de Archivos Distribuido de Hadoop (HDFS). Este archivo es crucial para definir cómo se comporta HDFS y cómo se gestionan y almacenan los datos en un clúster de Hadoop. Algunos de los usos y configuraciones comunes del archivo "hdfs-site.xml" incluyen:

1. **Replicación de bloques:** Puedes configurar el número de réplicas que HDFS debe mantener para cada bloque de datos. Esto afecta la redundancia y la tolerancia a fallos en el sistema.

2. **Ubicación de directorios de almacenamiento de datos:** Puedes especificar la ubicación en el sistema de archivos local donde se almacenarán los bloques de datos de HDFS. Esto es útil para definir rutas personalizadas o configurar discos específicos para el almacenamiento de datos.

3. **Tiempos de vencimiento y caducidad:** Puedes definir políticas de tiempo de vencimiento y caducidad para los archivos en HDFS. Esto puede incluir configuraciones para eliminar automáticamente archivos antiguos o no utilizados después de un cierto período.

4. **Configuración de caché y rendimiento:** Puedes configurar la caché de lectura y escritura, así como otros ajustes de rendimiento para optimizar el acceso a los datos en HDFS.

5. **Configuración de seguridad:** Puedes establecer propiedades de seguridad relacionadas con HDFS, como la habilitación de cifrado de datos en reposo o la configuración de políticas de control de acceso.

6. **Configuración de nodos secundarios:** HDFS utiliza nodos secundarios para mejorar el rendimiento y la integridad de los sistemas de archivos. Puedes definir la ubicación de estos nodos y otras configuraciones relacionadas con ellos en "hdfs-site.xml".

Editamos el fichero hdfs-site.xml
```
sudo nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
y añadimos esto:
```xml
 <configuration>
    <property>
       <name>dfs.replication</name>
       <value>1</value>
    </property>
    <property>
       <name>dfs.name.dir</name>
       <value>file:///home/hadoop/hdfs/namenode</value>
    </property>
    <property>
       <name>dfs.data.dir</name>
       <value>file:///home/hadoop/hdfs/datanode</value>
    </property>
 </configuration>
```
**Observa** que hemos incluido dos nuevas rutas, por tanto, debemos crearlas en nuestro sistema de ficheros
```
sudo mkdir -p /home/hadoop/hdfs/{namenode,datanode}
sudo chown -R hadoop:hadoop /home/hadoop/hdfs
```

## Fichero mapred-site.xml

El archivo "mapred-site.xml" en Hadoop es un archivo de configuración utilizado para establecer propiedades específicas del procesamiento de datos y la programación de tareas en el marco de trabajo MapReduce de Hadoop. MapReduce es una parte fundamental de Hadoop y se utiliza para procesar grandes conjuntos de datos de manera paralela y distribuida. Algunas de las configuraciones comunes que se pueden establecer en "mapred-site.xml" incluyen:

1. **Configuración del framework de ejecución de tareas:** Puedes especificar el framework de ejecución de tareas que MapReduce utilizará para la ejecución de tareas. Puedes elegir entre varios, como el framework local, el framework clásico o el framework YARN (Yet Another Resource Negotiator), según tus necesidades.

2. **Configuración de recursos de clúster:** Puedes definir las cantidades de recursos que se asignarán a las tareas de MapReduce, como la cantidad de memoria, la cantidad de núcleos de CPU, etc.

3. **Programación de tareas y ajustes de rendimiento:** Puedes establecer configuraciones relacionadas con la programación de tareas de MapReduce, como la cantidad máxima de tareas concurrentes, el tiempo de espera antes de reintentar tareas, etc. También puedes ajustar la configuración para el rendimiento, como la compresión de datos, el uso de combinadores y otras optimizaciones.

4. **Configuración de seguridad:** Puedes definir propiedades de seguridad relacionadas con MapReduce, como la configuración de la autenticación y autorización de tareas, o el acceso a recursos seguros.

Editamos el fichero mapred-site.xml
```
sudo nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
y añadimos esto:
```xml
 <configuration>
    <property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
    </property>
 </configuration>
```

## Fichero yarn-site.xml

El archivo "yarn-site.xml" en Hadoop es un archivo de configuración utilizado para establecer propiedades específicas del administrador de recursos YARN (Yet Another Resource Negotiator). YARN es un componente fundamental de Hadoop que se encarga de la gestión y asignación eficiente de recursos en un clúster, lo que permite ejecutar aplicaciones de procesamiento de datos de manera escalable y paralela. Algunas de las configuraciones comunes que se pueden definir en "yarn-site.xml" incluyen:

1. **Recursos de clúster:** Puedes configurar la cantidad de recursos disponibles en el clúster, como la cantidad total de memoria y la cantidad de núcleos de CPU que se pueden asignar a las aplicaciones.

2. **Programación de tareas:** Puedes definir políticas de programación, como las colas de aplicaciones, prioridades de trabajos y límites de recursos para garantizar un uso justo y eficiente de los recursos del clúster.

3. **Configuración de alta disponibilidad:** Puedes especificar configuraciones relacionadas con la alta disponibilidad, como la ubicación de los recursos de recuperación y las políticas de conmutación por error para los componentes de YARN.

4. **Configuración de seguridad:** Puedes definir propiedades de seguridad, como configuraciones de autenticación, autorización y el cifrado de datos en YARN.

5. **Configuración de registros y diagnóstico:** Puedes establecer propiedades para la recopilación de registros y diagnóstico de aplicaciones en ejecución en el clúster.

6. **Configuración de contenedores y nodos de trabajo:** Puedes definir propiedades relacionadas con la administración de contenedores y nodos de trabajo, como los límites de memoria y CPU por contenedor, entre otros.

Editamos el fichero mapred-site.xml
```
sudo nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
y añadimos esto:
```xml
 <configuration>
    <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

**Hemos terminado de configurar Hadoop**

## Preparación del disco distribuido
Una vez instalado y configurado _hadoop_ debemos formatear el disco, para ello:
```
su - hadoop
hdfs namenode -format
```
Debería dar una salida como:
> 2020-06-07 11:35:57,691 INFO util.GSet: VM type       = 64-bit  
2020-06-07 11:35:57,692 INFO util.GSet: 0.25% max memory 1.9 GB = 5.0 MB  
2020-06-07 11:35:57,692 INFO util.GSet: capacity      = 2^19 = 524288 entries  
2020-06-07 11:35:57,706 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10  
2020-06-07 11:35:57,706 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10  
2020-06-07 11:35:57,706 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25  
2020-06-07 11:35:57,710 INFO namenode.FSNamesystem: Retry cache on namenode is enabled  
2020-06-07 11:35:57,710 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis  
2020-06-07 11:35:57,712 INFO util.GSet: Computing capacity for map NameNodeRetryCache  
2020-06-07 11:35:57,712 INFO util.GSet: VM type       = 64-bit  
2020-06-07 11:35:57,712 INFO util.GSet: 0.029999999329447746% max memory 1.9 GB = 611.9 KB  
2020-06-07 11:35:57,712 INFO util.GSet: capacity      = 2^16 = 65536 entries  
2020-06-07 11:35:57,743 INFO namenode.FSImage: Allocated new BlockPoolId: BP-1242120599-69.87.216.36-1591529757733  
2020-06-07 11:35:57,763 INFO common.Storage: Storage directory /home/hadoop/hdfs/namenode has been successfully formatted.  
2020-06-07 11:35:57,817 INFO namenode.FSImageFormatProtobuf: Saving image file /home/hadoop/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 using no compression  
2020-06-07 11:35:57,972 INFO namenode.FSImageFormatProtobuf: Image file /home/hadoop/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 of size 398 bytes saved in 0 seconds.  
2020-06-07 11:35:57,987 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0  
2020-06-07 11:35:58,000 INFO namenode.FSImage: FSImageSaver clean checkpoint: txid=0 when meet shutdown.  
2020-06-07 11:35:58,003 INFO namenode.NameNode: SHUTDOWN_MSG:  
/************************************************************  
SHUTDOWN_MSG: Shutting down NameNode at ubuntu2004/69.87.216.36  
************************************************************/

## Arranque de Hadoop
En primer lugar, inicia el NameNode y el DataNode con el siguiente comando:
```
start-dfs.sh
```
Dará una salida del tipo:
> Starting namenodes on [0.0.0.0]  
Starting datanodes  
Starting secondary namenodes [ubuntu2004]

Para poner en marcha los administradores de recursos y los nodos de YARN, ejecuta el siguiente comando:
```
start-yarn.sh
```
Salida:
> Starting resourcemanager  
Starting nodemanagers

Comprueba que todo está arrancado:
```
jps
```
Salida:
> 5047 NameNode  
5850 Jps  
5326 SecondaryNameNode  
5151 DataNode

# Configuración del cluster Hadoop

## Renombramos en el fichero de configuración "hostname" el nombre de la máquina virtual
```
sudo nano /etc/hostname
```
Se hará con las dos máquinas, tanto con la maestra como con la esclava
Después de hacerlo se reiniciarán las máquinas con 
```
sudo reboot
```
## Modificamo el archivo de configuración core-site.xml
```
sudo nano /usr/local/hadoop/etc/hadoop/core-site.xml
```
En la variable value cambiamos el valor de "0.0.0.0" por el nombre_de_la_maquina
En mi caso quedaría de la siguiente forma:
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop-master:9000</value>
  </property>
</configuration>
```
## Modificamos el archivo de configuración hdfs-site.xml
```
sudo nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```
Donde pone value pondremos el valor 2, quedaria de la siguiente forma:
```xml
 <configuration>
<property>
<name>dfs.namenode.name.dir</name><value>/usr/local/hadoop/data/nameNode</value>
</property>
<property>
<name>dfs.datanode.data.dir</name><value>/usr/local/hadoop/data/dataNode</value>
</property>
<property>
<name>dfs.replication</name>
<value>2</value>
</property>
</configuration>
```
## Modificamos el archivo de configuracion workers
```
sudo nano /usr/local/hadoop/etc/hadoop/workers
```
Donde añadiremos el nombre de las máquinas virtuales, quedaria de la siguiente manera:
```xml
192.168.56.104
192.168.56.103
```
## Copiamos la configuración del hadoop-master al esclavo
```
scp /usr/local/hadoop/etc/hadoop/* hadoop-slave1:/usr/local/hadoop/etc/hadoop/
```
Deberia salir por pantalla lo siguiente:
> hadoop@hadoop-master:~$ scp /usr/local/hadoop/etc/hadoop/* hadoop-slave1:/usr/local/hadoop/etc/hadoop/
capacity-scheduler.xml                        100% 9213     2.1MB/s   00:00    
configuration.xsl                             100% 1335    98.2KB/s   00:00    
container-executor.cfg                        100% 1940   743.6KB/s   00:00    
core-site.xml                                 100%  864   382.0KB/s   00:00    
hadoop-env.cmd                                100% 3999     1.4MB/s   00:00    
hadoop-env.sh                                 100%   16KB   2.0MB/s   00:00    
hadoop-metrics2.properties                    100% 3321     1.2MB/s   00:00    
hadoop-policy.xml                             100%   11KB   3.2MB/s   00:00    
hadoop-user-functions.sh.example              100% 3414   785.4KB/s   00:00    
hdfs-site.xml                                 100% 1052   414.3KB/s   00:00    
httpfs-env.sh                                 100% 1484   402.6KB/s   00:00    
httpfs-log4j.properties                       100% 1657   501.5KB/s   00:00    
httpfs-signature.secret                       100%   21     7.4KB/s   00:00    
httpfs-site.xml                               100%  620   269.8KB/s   00:00    
kms-acls.xml                                  100% 3518     1.3MB/s   00:00    
kms-env.sh                                    100% 1351   182.9KB/s   00:00    
kms-log4j.properties                          100% 1860   605.3KB/s   00:00    
kms-site.xml                                  100%  682   254.0KB/s   00:00    
log4j.properties                              100%   14KB   2.2MB/s   00:00    
mapred-env.cmd                                100%  951   386.8KB/s   00:00    
mapred-env.sh                                 100% 1764   461.1KB/s   00:00    
mapred-queues.xml.template                    100% 4113   637.3KB/s   00:00    
mapred-site.xml                               100%  862   361.4KB/s   00:00    
scp: local "/usr/local/hadoop/etc/hadoop/shellprofile.d" is not a regular file
scp: failed to upload file /usr/local/hadoop/etc/hadoop/shellprofile.d to /usr/local/hadoop/etc/hadoop/
ssl-client.xml.example                        100% 2316   670.4KB/s   00:00    
ssl-server.xml.example                        100% 2697   831.1KB/s   00:00    
user_ec_policies.xml.template                 100% 2642   232.2KB/s   00:00    
workers                                       100%   30    12.8KB/s   00:00    
yarn-env.cmd                                  100% 2250   487.1KB/s   00:00    
yarn-env.sh                                   100% 6272     1.6MB/s   00:00    
yarnservice-log4j.properties                  100% 2591   746.5KB/s   00:00    
yarn-site.xml 
## Se formatea el archivo HDFS
```
source /etc/environment
hdfs namenode -format
```
como el segundo comando no me funcionaba use el comando
```
source .bashrc
```
## Arrancamos el HDFS 
```
start-dfs.sh
```
Deberia mostrarse por pantalla lo siguiente:
> Starting namenodes on [hadoop-master] 
Starting datanodes  
Starting secondary namenodes [hadoop-slave1]

```
jps
```
> 5047 NameNode  
5850 Jps  
5326 SecondaryNameNode  
5151 DataNode

Salida en el esclavo al realizar el comando jps:
> 4181 Jps
5089 DataNode

## Comprobar que se han arrancado los dos DataNode en la página web
Buscamos lo siguiente en el buscador:
```
http://hadoop-master:9870
```
# Subimos una imagen iso a HDFS
## Creamos la carpeta datos
Creamos el directorio datos en el cual posteriormente vamos a subir la imagen iso
```
hdfs dfs -mkdir /datos
```
Comprobamos que el directorio se ha creado con éxito con el comando:
```
hdfs dfs -ls /
```
## Descargamos la imagen iso de ubuntu
```
https://releases.ubuntu.com/lunar/
```
## Movemos la imagen iso de descargas a nuestro usuario de hadoop
```
sudo mv /home/josedaniel/Descargas/ubuntu.iso .
```
Comprobamos que la imagen se ha movido con éxito con el comando:
```
ls
```
Copiamos la imagen al directorio datos
```
hdfs dfs -copyFromLocal ubuntu.iso /datos 
```
Comprobamos que la imagen se ha copiado con éxito
```
hdfs dfs -ls /datos
```
# Cosas aprendidas acerca de esta entrega
Los DataNodes son los nodos de almacenamiento en un clúster Hadoop HDFS.
Almacenan los datos reales en forma de bloques y los sirven cuando se solicitan.
Son responsables de informar regularmente al NameNode sobre su estado y los bloques que almacenan.
Si un DataNode falla, el NameNode puede utilizar esta información para replicar los bloques de datos perdidos en otros DataNodes.
No almacenan metadatos sobre el sistema de archivos, solo los bloques de datos.
Puedes tener múltiples DataNodes en un clúster para escalar el almacenamiento y la redundancia de datos.
