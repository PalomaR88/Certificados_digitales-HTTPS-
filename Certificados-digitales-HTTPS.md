# Práctica: Certificados digitales. HTTPS

## Certificado digital de persona físicaPermalink
**Tarea 1: Instalación del certificado**
1. Una vez que hayas obtenido tu certificado, explica brevemente como se instala en tu navegador favorito.

2. Muestra una captura de pantalla donde se vea las preferencias del navegador donde se ve instalado tu certificado.

3. ¿Cómo puedes hacer una copia de tu certificado?, ¿Como vas a realizar la copia de seguridad de tu certificado?. Razona la respuesta.

4. Investiga como exportar la clave pública de tu certificado.


**Tarea 2: Validación del certificado**
1. Instala en tu ordenador el software autofirma y desde la página de VALIDe valida tu certificado. Muestra capturas de pantalla donde se comprueba la validación.


**Tarea 3: Firma electrónica**
1. Utilizando la página VALIDe y el programa autofirma, firma un documento con tu certificado y envíalo por correo a un compañero.

2. Tu debes recibir otro documento firmado por un compañero y utilizando las herramientas anteriores debes visualizar la firma (Visualizar Firma) y (Verificar Firma). ¿Puedes verificar la firma aunque no tengas la clave pública de tu compañero?, ¿Es necesario estar conectado a internet para hacer la validación de la firma?. Razona tus respuestas.

3. Entre dos compañeros, firmar los dos un documento, verificar la firma para comprobar que está firmado por los dos.

**Tarea 4: Autentificación**
1. Utilizando tu certificado accede a alguna página de la administración pública )cita médica, becas, puntos del carnet,…). Entrega capturas de pantalla donde se demuestre el acceso a ellas.


## HTTPS / SSL
Antes de hacer esta práctica vamos a crear una página web (puedes usar una página estática o instalar una aplicación web) en un servidor web apache2 que se acceda con el nombre tunombre.iesgn.org.
sudo cp 000-default.conf paloma.iesgn.org
~~~
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/paloma
~~~


**Tarea 1: Certificado autofirmado**
Esta práctica la vamos a realizar con un compañero. En un primer momento un alumno creará una Autoridad Certficadora y firmará un certificado para la página del otro alumno. Posteriormente se volverá a realizar la práctica con los roles cambiados.

El alumno que hace de Autoridad Certificadora deberá entregar una documentación donde explique los siguientes puntos:

1. Crear su autoridad certificadora (generar el certificado digital de la CA).



2. Debe recibir el fichero CSR (Solicitud de Firmar un Certificado) de su compañero, debe firmarlo y enviar el certificado generado a su compañero.

3. ¿Qué otra información debe aportar a tu compañero para que éste configure de forma adecuada su servidor web con el certificado generado?

El alumno que hace de administrador del servidor web, debe entregar una documentación que describa los siguientes puntos:

1. Crea una clave privada RSA de 4096 bits para identificar el servidor.

2. Utiliza la clave anterior para generar un CSR, considerando que deseas acceder al servidor tanto con el FQDN (tunombre.iesgn.org) como con el nombre de host (implica el uso de las extensiones Alt Name).

Se crea la clave:
~~~
vagrant@servidor:~$ sudo openssl genrsa -out paloma.iesgn.org.key 4096
~~~

Se crea el fichero paloma.iesgn.org.conf y configura:
~~~
[ req ]
default_bits       = 4096
default_keyfile    = paloma.iesgn.org.key
distinguished_name = req_distinguished_name
req_extensions     = req_ext

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = sp
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Seville
localityName                = Locality Name (eg, city)
localityName_default        = Dos Hermanas
organizationName            = Organization Name (eg, company)
organizationName_default    = paloma 
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64

[ req_ext ]
subjectAltName = @alt_names

[alt_names]
DNS.1   = servidor
~~~

Y se crea el certificado llamando al fichero de configuración que se ha creado:
~~~
vagrant@servidor:~$ sudo openssl req -new -nodes -sha256 -config paloma.iesgn.org.conf -out paloma.iesgn.org.csr
Generating a RSA private key
.......................................................................................................................................................................................................................................................................++++
.............++++
writing new private key to 'paloma.iesgn.org.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [sp]:
State or Province Name (full name) [Seville]:
Locality Name (eg, city) [Dos Hermanas]:
Organization Name (eg, company) [paloma]:
Common Name (e.g. server FQDN or YOUR name) []:paloma.iesgn.org 
~~~

![Imagen1](Img-tarea2-serv.png)


3. Envía la solicitud de firma a la entidad certificadora (su compañero).
Como este ejercicio se está realizando a través de máquinas virtuales en vagrant, vamos a enviar el fichero con el comando scp a la máquina anfitriona y de ahí a la máquina que actúa de entidad certificadora. 


4. Recibe como respuesta un certificado X.509 para el servidor firmado y el certificado de la autoridad certificadora.


5. Configura tu servidor web con https en el puerto 443, haciendo que las peticiones http se redireccionen a https (forzar https).
Se crea un nuevo fichero de configuración que lo hemos llamado paloma.iesgn.ssl.conf, que es una copia del fichero de configuración de nuestro sitio web, indicando la llave y los certificados:
~~~
<IfModule mod_ssl.c>
	<VirtualHost _default_:443>
		ServerAdmin webmaster@localhost
		ServerName paloma.iesgn.org
		DocumentRoot /var/www/html/paloma

		SSLEngine on

		SSLCertificateFile /etc/ssl/certs/paloma.iesgn.org-firmado.crt
		SSLCertificateKeyFile /etc/ssl/private/paloma.iesgn.org.key
		SSLCertificateChainFile /etc/ssl/certs/paloma.iesgn.org.csr

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

	</VirtualHost>
</IfModule>
~~~


Se inicia apache con el nuevo .conf:
~~~
vagrant@servidor:~$ sudo a2ensite paloma.iesgn.ssl
Enabling site paloma.iesgn.ssl.
To activate the new configuration, you need to run:
  systemctl reload apache2
vagrant@servidor:~$ sudo systemctl restart apache2
~~~

Accedemos a la misma dirección que antes pero con https://
![Imagen] (Img_Tarea3-serv.png)

Se importa el certificado de la entidad autentificadora:
![imagen_crsEA] (Img_Tarea3A.png)

Con el certificado importado aparece ya la señal de conexión segura:
![imagen_conexionsegura] (Img_Tarea3B.png)


Se accede al fichero de configuración paloma.iesgn.conf apra realizar la redirección a https añadiendo las siguientes líneas:
~~~
        ServerName paloma.iesgn.org
        <Location/>
                Redirect permanent / https://paloma.iesgn.org
        </Location>
~~~


**Tarea 2: Certificados digital con CAcert**

El lema de CAcert es Free digital certificates for everyone y es que la utilización de certificados emitidos por CA comerciales no es posible para todos los sitios de Internet debido a su coste, lo que los limita su uso a transacciones económicas o sitios con datos relevantes. CAcert es una organización sin ánimo de lucro que mantiene una infraestructura equivalente a una CA comercial aunque con ciertas limitaciones.

Vamos a cambiar el certificado de la página que has desarrollado en el punto anterior para usar el nuevo certificado emitido por CAcert.

Los pasos que hay que dar para utilizar un certificado X.509 emitido por CAcert son los siguientes:

- Darse de alta como usuario en el sitio web.
-         ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/paloma
Dar de alta el dominio para el que queremos obtener el certificado. (opción Domains -> Add)
    CAcert verifica que podemos hacer uso legítimo del dominio enviando un mensaje de correo electrónico.
    Dar de alta el certificado de un servidor mediante una solicitud de firma certificado (CSR).
    Configurar el servidor web con el certificado X.509 emitido por la CA.
    Al acceder a la página debemos evitar el mensaje de error de “Conexión segura fallida”.
    ¿Qué fecha de caducidad tiene el certificado? ¿Qué tendrás que hacer cuando termine ese tiempo?

Escribe una documentación donde expliques el proceso y muestra al profesor su funcionamiento.

