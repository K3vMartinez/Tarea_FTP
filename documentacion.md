# Servicio de transferencia de ficheros
## Infraestructura
Reutilizaremos las MV de la práctica de **ssh**. Dos MV dentro de una **red NAT**:

* **Servidor**: con un Ubuntu server sin entorno gráfico.
    * Usuario: **sergio**, contraseña: **sergio**.
* **Casa**: con un Lubuntu con el entorno gráfico por defecto (LXQt).
    * Usuario: **carmen**, contraseña: **carmen**.

Desde el equipo **Casa** nos conectaremos al equipo **Servidor** mediante una conexión **ssh** autentificándonos mediante claves asimétricas **ed25519**.

![MV](./img/img01.png)

![MV](./img/img02.png)

![MV](./img/img03.png)


## Instalación y uso básico

1. Acceder al servidor:
```bash
ssh -i ~/.ssh/id_ed25519 10.0.2.4
```
![Acceso al servidor desde Casa](./img/img04.png)

2. Instalar proftpd:
```bash
sudo apt update
sudo apt install proftpd
```
![Instalar proftpd](./img/img05.png)![Instalar proftpd](./img/img06.png)![Instalar proftpd](./img/img07.png)

3. Realizar algunos cambios en el archivo de configuración:
```bash
sudo nano /etc/proftpd/proftpd.conf
```
![Configuración proftpd](./img/img08.png)

* Cambiar el nombre del servidor.

![CambioNombre](./img/img09.png)
![CambioNombre](./img/img10.png)

* Desactivar el protocolo IP versión 6.

![DesactivarIPv6](./img/img11.png)
![DesactivarIPv6](./img/img12.png)

* No mostrar el mensaje de bienvenida hasta que el usuario no se haya autentificado correctamente.

![NoMostrarMensaje](./img/img13.png)
![NoMostrarMensaje](./img/img14.png)


> Puedes obtener más información sobre las distintas directivas de configuración en:
http://www.proftpd.org/docs/directives/configuration_full.html. Revisa las directivas:
**DeferWelcome**, **DisplayConnect**, **DisplayLogin**, **DisplayChdir**, **DisplayGoAway**,
**DisplayQuit**, **AccessGrantMsg** y **AccessDenyMsg**.

4. Comprobar estado del servicio **proftpd**:
```bash
sudo systemctl status proftpd
```

![Estado proftpd](./img/img15.png)

5. Con los siguientes comandos lo activaremos para que se inicie al arrancar el servidor y lo iniciaremos:
```bash
sudo systemctl enable proftpd
sudo systemctl start proftpd
```
![Activar e iniciar servidor](./img/img16.png)
![ResetearServidor](./img/img16_1.png)

> Otras comandos del servicio son:
>```bash
>sudo systemctl enable proftpd
>sudo systemctl start proftpd
>sudo systemctl stop proftpd
>sudo systemctl restart proftpd
>sudo systemctl status proftpd
>sudo systemctl reload proftpd
>sudo systemctl show proftpd
>```

6. Reglas firewall:
```bash
sudo ufw enable
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
sudo ufw status
```
![Reglas firewall](./img/img17.png)

7. Probar desde el cliente qué puertos tiene abiertos el servidor, en nuestro ejemplo desde el equipo **Casa** ejecutaremos:
```bash
nmap 10.0.2.4 -p 1-1024
```
![Instalar nmap](./img/img18.png)![Instalar nmap](./img/img19.png)
![Puertos abiertos](./img/img20.png)

Si no tienes instalada esta utilidad, instalalá con: **sudo apt install nmap**. Esta comprobación
también se puede hacer desde el propio servidor, pero es menos fiable que desde otro equipo ya que
puede conectarse por localhost.

8. Conexión FTP desde el cliente, en nuestro ejemplo el equipo **Casa**. (-A: forzar modo activo):
```bash
ftp -A 10.0.2.4
```
![Conexión FTP](./img/img21.png)

>Para evitar problemas con el firewall en todo momento utilizaremos el modo de transferencia 'activo'. Este modo utiliza unicamente los puertos 20 y 21 del servidor.

9. Comprobar en el equipo **Servidor** qué conexiones están establecidas con otros equipos:
```bash
ss | grep tcp
```
![Conexiones establecidas](./img/img22.png)

10. Instalar el cliente gráfico **Filezilla** en el equipo **Casa**. Crea una nueva conexión en el **Gestor de sitios**. Conectarse al equipo **Servidor** con el protocolo FTP y cifrado FTP plano. Recuerda conectar el modo **activo** en la pestaña **Opciones de transferencia** del Gestor de sitios.

![Instalación filezilla](./img/img23.png)
![Instalación filezilla](./img/img24.png)

## Configurar una cuenta anónima
1. Volvemos a realizar algunos cambios en el archivo de configuración:
```bash
sudo nano /etc/proftpd/proftpd.conf
```
![Cambios en la configuracion](./img/img25.png)

Descomentamos todo el bloque <Anonymous ~ftp> de modo que quede de la siguiente forma:

```bash
<Anonymous ~ftp>
User ftp
Group nogroup
# We want clients to be able to login with "anonymous" as well as
"ftp"
UserAlias anonymous ftp
# Cosmetic changes, all files belongs to ftp user
DirFakeUser on ftp
DirFakeGroup on ftp
RequireValidShell off
# Limit the maximum number of anonymous logins
MaxClients 10
# We want 'welcome.msg' displayed at login, and '.message' displayed
# in each newly chdired directory.
DisplayLogin welcome.msg
DisplayChdir .message
# Limit WRITE everywhere in the anonymous chroot
<Directory *>
<Limit WRITE>
DenyAll
</Limit>
</Directory>
# Uncomment this if you're brave.
# <Directory incoming>
# # Umask 022 is a good standard umask to prevent new files and>
# # (second parm) from being group and world writable.
# Umask022 022
# <Limit READ WRITE>
# DenyAll
# </Limit>
# <Limit STOR>
# AllowAll
# </Limit>
# </Directory>
</Anonymous>
```

![Bloque anonymous](./img/img26.png)

![Bloque anonymous](./img/img27.png)


2. Reiniciaremos el servicio **proftpd**:
```bash
sudo systemctl restart proftpd
```
![Reinicio](./img/img28.png)

3. Puedes probar que los archivos de configuración son correctos con el siguiente comando:
```bash
sudo /usr/sbin/proftpd --configtest -c /etc/proftpd/proftpd.conf
```
![Archivos de configuracion](./img/img29.png)

4. Prueba que puedes acceder al servidor sin escribir contraseñas usando los dos usuarios anónimos:
anonymous y ftp. Realiza la prueba tanto desde el teminal como desde la aplicación gráfica filezilla.
>Realmente al instalar ProFTP se crea el usuario **ftp**, anonymous es un alias de este usuario.
Puedes verlo en la directiva **UserAlias** del bloque **<Anonymous >** del archivo de
configuración. Además puedes ver que este usuario existen en el sistema en el archivo de
usuarios: **sudo cat /etc/passwd**.

![Conexion desde terminal](./img/img30.png)

![Conexion desde interfaz gráfica](./img/img31.png)
![Conexion desde interfaz gráfica](./img/img32.png)
![Conexion desde interfaz gráfica](./img/img32_1.png)
![Conexion desde interfaz gráfica](./img/img32_2.png)
![Conexion desde interfaz gráfica](./img/img33.png)
![Conexion desde interfaz gráfica](./img/img34.png)
![Conexion desde interfaz gráfica](./img/img34_1.png)
![Conexion desde interfaz gráfica](./img/img34_2.png)
![Conexion desde interfaz gráfica](./img/img35.png)
![Conexion desde interfaz gráfica](./img/img36.png)
![Conexion desde interfaz gráfica](./img/img37.png)
![Conexion desde interfaz gráfica](./img/img38.png)
![Conexion desde interfaz gráfica](./img/img39.png)
![Conexion desde interfaz gráfica](./img/img40.png)
![Conexion desde interfaz gráfica](./img/img41.png)

5. Si queremos que el usuario anónimo tenga una carpeta diferente, tenemos que crear dicha carpeta, por ejemplo:
```bash
sudo mkdir -p /var/ftp/anonimo
sudo chown ftp.nogroup /var/ftp/anonimo
```
![Carpeta diferente](./img/img42.png)

E indicarlo en el fichero de configuración, cambiando la etiqueta **<Anonymous ~ftp>** por
**<Anonymous /nombre-de-carpeta>**. Por ejemplo: **<Anonymous /var/ftp/anonimo>**

![Carpeta diferente](./img/img43.png)
![Carpeta diferente](./img/img44.png)
![Carpeta diferente](./img/img45.png)
![Carpeta diferente](./img/img46.png)

6. Para saber que estamos en esta nueva carpeta, le vamos a crear un fichero dentro:
```bash
sudo touch /var/ftp/anonimo/UsuarioAnonimo.txt
```
> Otra opción sería crear/modificar el archivo **welcome.msg** que nos indica la directiva
**DisplayLogin**.

![Creacion de fichero dentro](./img/img47.png)

7. Recuerda reinicia el servicio **proftpd**:
```bash
sudo systemctl restart proftpd
```
![Reinicio](./img/img48.png)

8. Puedes probar que los archivos de configuración son correctos con el siguiente comando:
```bash
sudo /usr/sbin/proftpd --configtest -c /etc/proftpd/proftpd.conf
```
![Prueba](./img/img49.png)

9. Vuelve a probar a acceder al servidor usando los usuarios: anonymous y ftp. Tanto desde el teminal como desde la aplicación gráfica filezilla. Esta vez debe aparecer el nuevo directorio de trabajo.

![Comprobacion](./img/img50.png)
![Comprobacion](./img/img51.png)


## Configuración de usuarios virtuales
Los usuarios virtuales son aquellos que no son usuarios de sistema pero si tienen acceso a algunos recursos
a través del servicio FTP. Los crearemos usando el **comando ftpasswd**. Los usuarios y las contraseñas se
almacenan en un fichero que vamos a crear en **/etc/proftpd/ftpd.passwd**:

1. Primero vamos a crear las carpetas de trabajo que le vamos a asignar a cada usuario virtual:
```bash
sudo mkdir /var/ftp/ftp01
sudo mkdir /var/ftp/ftp02
sudo chown ftp.nogroup /var/ftp/ftp01 /var/ftp/ftp02
```
![Creacion de carpetas](./img/img52.png)

2. Ahora nos situamos en el directorio de configuración de proftpd y crearmos un archivo vacío para generar los usuarios virtuales.
```bash
cd /etc/proftpd/
sudo touch ftpd.passwd
```
![Creacion archivo vacío](./img/img53.png)

3. Después creamos los usuarios virtuales ejecutando el comando ftpasswd:
```bash
sudo ftpasswd --passwd --name=ftp01 --uid=3001 --gid=3001 --
home=/var/ftp/ftp01 --shell=/bin/false
$ sudo ftpasswd --passwd --name=ftp02 --uid=3002 --gid=3002 --
home=/var/ftp/ftp02 --shell=/bin/false
```
Nos pedirá que introduzcamos la contraseña de los usuarios virtuales.

![Usuarios virtuales](./img/img54.png)
![Usuarios virtuales](./img/img54_1.png)

4. Para saber que estamos en nuestra carpeta, le vamos a crear un fichero dentro de cada una:
```bash
sudo touch /var/ftp/ftp01/soyusuario1.txt
sudo touch /var/ftp/ftp02/soyusuario2.txt
```
![Creacion de ficheros](./img/img55.png)

> Otra opción sería crear en dicha carpeta un archivo **.message** tal como se indica en la
directiva **DisplayChdir** del archivo de configuración.

5. A continuación necesimos modificar el archivo de configuración:
```bash
sudo nano /etc/proftpd/proftpd.conf
``` 
![Configuracion](./img/img56.png)
Descomentamos las líneas:
```bash
DefaultRoot ~
RequireValidShell off
```
![Configuracion](./img/img57.png)
![Configuracion](./img/img58.png)
![Configuracion](./img/img58_1.png)
![Configuracion](./img/img59.png)
![Configuracion](./img/img60.png)
![Configuracion](./img/img59_1.png)
Y al final del archivo incluimos la siguiente directiva:
```bash
AuthUserFile /etc/proftpd/ftpd.passwd
```
![Configuracion](./img/img61.png)
6. Guardamos y reiniciamos el servicio **proftpd**:
```bash
sudo systemctl restart proftpd
```

![Configuracion](./img/img62.png)


7. Puedes probar que los archivos de configuración son correctos con el siguiente comando:
```bash
sudo /usr/sbin/proftpd --configtest -c /etc/proftpd/proftpd.conf
```
![Configuracion](./img/img63.png)

8. Vuelve a probar a acceder al servidor usando los usuarios virtuales, tanto desde el teminal como
desde la aplicación gráfica filezilla. Debe aparecer el archivo correspondiente de su directorio de
trabajo.

![Acceso](./img/img64.png)
![Acceso](./img/img64_1.png)
![Acceso](./img/img65.png)
![Acceso](./img/img66.png)
![Acceso](./img/img67.png)
![Acceso](./img/img68.png)

## Configuración de FTP con TLS/SSL
1. Para utilizar una conexión segura primero debemos instalar en el servidor el paquete para hacer funcionar el módulo de criptografía en proftpd.
```bash
sudo apt install proftpd-mod-crypto
```
![Instalacion](./img/img69.png)

2. Ahora debemos activar dicho paquete en el archivo de configuración:
**/etc/proftpd/modules.conf**. Descomentar la directiva: **LoadModule mod_tls.c**

![Instalacion](./img/img69_1.png)
![Instalacion](./img/img69_2.png)

3. A continuación debemos instalar el paquete openssl para generar un certificado autofirmado.
```bash
sudo apt install openssl
```
![Instalacion](./img/img70.png)

4. Una vez instalado vamos a generar una **clave RSA** de **2048 bit**. Guardaremos la clave privada en
el directorio **/etc/ssl/private/proftpd.key** y la clave pública en
**/etc/ssl/certs/proftpd.crt**. Le daremos una caducidad de 365 días.
```bash
openssl req -x509 -newkey rsa:2048 -sha256 -keyout
/etc/ssl/private/proftpd.key -out /etc/ssl/certs/proftpd.crt -nodes -days 365
```
![Instalacion](./img/img71.png)

Al crear el certificado nos pedirá cierta información que tendremos que ir rellenando.
Ponemos los permisos adecuados a estos certificados:
```bash
chmod 600 /etc/ssl/private/proftpd.key
chmod 600 /etc/ssl/certs/proftpd.crt
```
![Instalacion](./img/img72.png)

5. A continuación debemos incluir el archivo **tls.conf** en el archivo **proftpd.conf**. Editamos el
archivo de configuración: **/etc/proftpd/proftpd.conf**. Descomentar la línea: **Include /etc/proftpd/tls.conf**

![Configuracion](./img/img73.png)
![Configuracion](./img/img74.png)
![Configuracion](./img/img75.png)

6. Ahora abrimos el archivo de configuración de TLS /etc/proftpd/tls.conf y descomentamos las
siguientes líneas:
```bash
TLSEngine on
TLSLog /var/log/proftpd/tls.log
TLSProtocol SSLv23
...
TLSRSACertificateFile /etc/ssl/certs/proftpd.crt
TLSRSACertificateKeyFile /etc/ssl/private/proftpd.key
...
TLSRequired on
```
Con la directiva **TLSRequired on** establecemos que toda la comunicación con el servidor debe ser
segura.

![Configuracion](./img/img76.png)
![Configuracion](./img/img77.png)
![Configuracion](./img/img78.png)


7. Reiniciamos el servicio **proftpd**:
```bash
sudo systemctl restart proftpd
```
![Reset](./img/img79.png)

8. Puedes probar que los archivos de configuración son correctos con el siguiente comando:
```bash
sudo /usr/sbin/proftpd --configtest -c /etc/proftpd/proftpd.conf
```
![Comprobacion](./img/img80.png)

9. Puedes probar que se establece la comunicación TLS entre el cliente **Casa** y el **Servidor*** ejecutando
el siguiente comando desde un terminal del equipo **Casa**:
```bash
openssl s_client -connect 10.0.2.4:21 -starttls ftp
```
![Comunicacion](./img/img81.png)
![Comunicacion](./img/img81_1.png)
![Comunicacion](./img/img81_2.png)
![Comunicacion](./img/img81_3.png)

10. Vuelve a probar a acceder al servidor usando la conexión segura. Primero con la aplicación gráfica
filezilla. Crea una nueva conexión en el gestor de sitios con el protocolo FTP y el cifrado 'Requiere
FTP explícito sobre TLS'. Recuerda activar el modo de transferencia activa en la pestaña 'Opciones de
transferencia'.
![Comunicacion](./img/img82.png)
![Comunicacion](./img/img83.png)
![Comunicacion](./img/img84.png)
![Comunicacion](./img/img85.png)

11. Para probar con el terminal debemos instalarnos un nuevo paquete ya que el comando básico **ftp** no
permite certificados. Instala el paquete **lftp**:
```bash
sudo apt install `lftp`
```
![Instalacion](./img/img86.png)

12. Ahora configura el comando **lftp** creando un archivo de configuración **nano ~/.lftprc** con el
siguiente contenido:
```bash
set ftp:passive-mode off
set ftp:ssl-auth TLS
set ftp:ssl-force true
set ftp:ssl-protect-list yes
set ftp:ssl-protect-data yes
set ftp:ssl-protect-fxp yes
set ssl:verify-certificate no
```
Configuramos: el modo de transferencia en activo, forzamos el uso de ssl y la autentificación con TLS.
Además como nuestro certificado en autofirmado establecemos que no se verifique el certificado.

![Configuracion](./img/img87.png)
![Configuracion](./img/img88.png)


13. Prueba a acceder desde el terminal con:
```bash
lftp jose@10.0.2.4
```
![Prueba](./img/img89.png)
