# Introducción


Este repositorio contiene lo necesario para crear varios contenedores linux con servicios web en la misma dirección IP y diferentes nombres DNS. Para conseguirlo monto un contenedor que reparte el tráfico (squid, basado en un proxy inverso) y varios contenedores con el servicio Web para cada nombre DNS.

                                                         +-------+  +-----------+
                                                         |fluentd|  |   mysql   |
                                                         +-------+  +-----------+
                                                             |         |
                                             +-----------+   |         |
                    +-- www.tld.org -->  80: | wordpress |---+---------+
        +-------+  /                         +-----------+            
    80: | squid |---+                                                      
        +-------+ |
                  |\                         +-------------------+     
                  | +-- blog.tld.org --> 80: | apache/nibbleblog |     
                  |                          +-------------------+     
                   \                          +--------+
                    +-- test.tld.org -->  80: | apache |
                                              +--------+


Consulta este [apunte técnico sobre varios servicios en contenedores Docker](http://www.luispa.com/?p=172) para acceder a otros contenedores Docker y sus fuentes en GitHub.


# Instalación

Se apoya en [Docker](https://www.docker.com/) y [fig](http://www.fig.sh/index.html). En el fichero fig-template.yml tienes un ejemplo con varios contenedores. El principal es un servidor squid que escuchará en el puerto 80 del HOST, reencamina las peticiones a servidores Web Apache o PHP (wordpress) con direcciones IP internas no expuestas. La redirección se realiza sobre la base de la dirección dns en el URL.

Utilizo varias imágienes que he creado: 

* [luispa/base-squid](https://registry.hub.docker.com/u/luispa/base-squid/)
* [luispa/base-apache](https://registry.hub.docker.com/u/luispa/base-apache/)
* [mysql](https://registry.hub.docker.com/_/mysql/)
* [wordpress](https://registry.hub.docker.com/_/wordpress/)


Para instalar y crear los contenedores haz un clone en tu Host

    $  git clone https://github.com/LuisPalacios/servicio-web.git

Renombra, edita y adapta a tu gusto y necesidades los puertos y los nombres dns. 

    $ mv fig.sample.yml fig.yml

Arranca el servicio con el comando siguiente

    $ fig up -d



# Configuración

Si analizas mi fichero fig.yml verás que me apoyo en un repositorio persistente en el Host donde tengo los "datos" de los web servers, de wordpress y el fichero de configuración de proxy inverso de squid.

    /Apps/data/web
              +--/blog.tld.org
              +--/www.tld.org  
              +--/test.tld.org
              +--/squid/squid_reverse_vhosts.conf  (ejemplo bajo 'sample')


## Configuración de ```squid```

El Contenedor "squid" va a buscar el fichero /Apps/data/web/squid/squid_reverse_vhosts.conf, se trata de un fichero temporal que uso para añadir al principio del /etc/squid3/squid3.conf, de modo que se realice correctamente el reverse proxy. Tienes un ejemplo en el directorio "sample"


## Configuración de ```blog.tld.org```

El web server "blog" espera encontrar su contenido bajo /Apps/data/web/blog.tld.org

Como blog server empleo [nibbleblog](http://www.nibbleblog.com/) y como tenía ya contenido antiguo he hecho un backup y lo he restaurado


## Configuración de ```www.tld.org```

Este nombre DNS lo conecto con el servidor wordpress, que realmente se compone de dos contenedores, el primero es "wordpress" y en el backend tiene su base de datos en otro contenedor "mysqld".

En este ejemplo reservaría el directorio ```/Apps/data/web/www.tld.org``` para ubicar los datos persistentes 
