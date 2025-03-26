# Configuración de Seguridad en MongoDB

Este documento detalla los pasos para habilitar la autenticación en MongoDB, crear usuarios con roles específicos y configurar conexiones seguras mediante SSL/TLS.

## 1️⃣ Habilitar la Autenticación en MongoDB

Para reforzar la seguridad, es necesario modificar la configuración de MongoDB en la sección `security` en el  archivo de configuración (`mongod.cfg`).
![Texto alternativo](/asset/cfgConfIncio.png)

#### Aplicar los cambios
Guarda los cambios y reinicia MongoDB con los siguientes comandos: ```net stop MongoDB ,
net start MongoDB```

![Texto alternativo](/asset/detener%20y%20iniciar(2).png)

## 2️⃣ Crear Usuarios con Roles y Permisos Específicos

Se deben crear usuarios con distintos niveles de acceso según las necesidades de la base de datos.
#### Conectarse a la base de datos
Accede a MongoDB: ```mongosh```
![Texto alternativo](/asset/mongosh(3).png)

Acceder a MongoDB con privilegios de administrador:``` use admin```
![Texto alternativo](/asset/use%20admin(4.1).png)

#### Crear un usuario administrador

Ejecuta el siguiente comando para crear un usuario administrador con permisos globales: ``` db.createUser({user: "adminUser", pwd: "AdminPassword123!", roles: [{ role: "userAdminAnyDatabase", db: "admin" }]});```

![Texto alternativo](/asset/CreateUser(5).png)

Acceder a miBaseDatos: ```use miBaseDatos```
![Texto alternativo](/asset/UseMibase(5).png)


#### Crear usuarios con permisos específicos

Usuario con permisos de solo lectura: ```db.createUser({user: "readUser", pwd: "ReadPassword123!", roles: [{ role: "read", db: "miBaseDatos" }]});```
![Texto alternativo](/asset/createRead(6).png)

Usuario con permisos de escritura: ```db.createUser({user: "writeUser",pwd: "WritePassword123!",roles: [{ role: "readWrite", db: "miBaseDatos" }]});```
![Texto alternativo](/asset/UserWrite(7).png)



## 3️⃣ Configurar Conexiones Seguras con SSL/TLS

Para mejorar la seguridad, es recomendable habilitar conexiones seguras mediante SSL/TLS.

#### Problema encontrado

Al ejecutar el comando:```where openssl```
![Texto alternativo](/asset/ErrorOpenSSL.png)

#### Solución
Agregar manualmente OpenSSL al PATH.

1. Abre la configuración de las variables de entorno del sistema.
![Texto alternativo](/asset/Entorno(1).png)

2. Haz clic en "Variables de entorno", luego selecciona "Path" en la sección "Variables del sistema".
![Texto alternativo](/asset/phat.png)

3. Haz clic en "Nuevo" y agrega la ruta de MongoDB:  ```C:\Program Files\OpenSSL-Win64\bin. ```
![Texto alternativo](/asset/AgreEntorno.png)

#### Configurar MongoDB para usar SSL
Modifica el archivo de configuración (`mongod.cfg`) agregando las siguientes líneas en la sección net: ```net: ssl: mode: requireSSL 
PEMKeyFile: C:\mongodb.pem```

![Texto alternativo](/asset/LicenciasSSL.png)

#### Aplicar los cambios
Reinicia MongoDB para que la nueva configuración tenga efecto: ```net stop MongoDB ,
net start MongoDB```
![Texto alternativo](/asset/reiniciar(8).png)


#### Validación
Para asegurarte de que la conexión segura funciona correctamente, conectarse a MongoDB usando SSL y credenciales.

1. Crear credenciales con el comado: ``` openssl req -newkey rsa:2048 -nodes -keyout mongodb-key.pem -x509 -days 365 -out mongodb-cert.pem```
![Texto alternativo](/asset/CrearCredencial.png)

2. Completar los campos requeridos para crear las credenciales de usuario.
![Texto alternativo](/asset/RellanarCredencial.png)

3. validar la coneccion con ssl:  ```mongosh --tls --host localhost --tlsCertificateKeyFile "C:\Program Files\MongoDB\Server\8.0\bin\mongodb.pem" -u adminUser -p 'AdminPassword123!' --authenticationDatabase admin```
![Texto alternativo](/asset/validacionError.png)

## 📌 Notas Adicionales:

#### Conclusión sobre el error "MongoServerSelectionError: read ECONNRESET"

El error "MongoServerSelectionError: read ECONNRESET" ocurrió al intentar establecer una conexión segura (TLS) con el servidor MongoDB. Este tipo de error generalmente indica que la conexión fue interrumpida inesperadamente, lo que puede suceder por diversas razones, como una configuración incorrecta del servidor, un problema con la red, o un problema en el certificado utilizado para la conexión TLS.

##### Se intentaron varias soluciones para resolver este error, pero no se logró establecer la conexión. Las posibles soluciones intentadas fueron:

```Verificación del servicio de MongoDB```: Se verificó si MongoDB estaba en ejecución y el servicio estaba activo. A pesar de estar en ejecución, el error persiste.

```Revisión de la configuración de TLS```: Se revisó el archivo de certificado (mongodb.pem) para asegurar que estaba bien configurado. Sin embargo, el certificado parece no ser el problema principal, ya que el error persiste.

```Comprobación del puerto y la dirección del servidor```: Se intentó verificar si el servidor MongoDB estaba correctamente escuchando en el puerto 27017 y si no había bloqueos en la red. Sin embargo, no se observó ningún problema en la red.

```Deshabilitar la verificación TLS```: Como solución temporal, se intentó deshabilitar la verificación de certificados TLS mediante la opción --tlsAllowInvalidCertificates, pero el problema persiste, indicando que la raíz del problema puede estar en la configuración de la conexión o en la red.

```Reinicio de MongoDB```: Se intentó reiniciar el servicio de MongoDB, pero nuevamente, el error no se resolvió.



