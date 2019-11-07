En /etc/apache2/sites-available/, en el fichero de configuración, se duplica el fichero y crea uno con nombre-ssl.conf que hay que activarlo.

También se puede hacer en un mismo fichero, pero se va a explicar como hacerlo en ficheros diferentes. 

La configuración es igual al otro, con la diferencia de 3 líneas:
~~~
SSLEngine on
SSLCertificateFile /etc/ssl/certs/ssl-cer-snakeoil.pem
SSLCertificatekey... /etc/ssl/certs/ssl-cer-snakeoil.key
~~~


