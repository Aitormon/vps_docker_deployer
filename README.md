Configuración vps
Pasos a seguir:

Conexión al servidor y claves ssh
Conseguimos la dirección IP de nuestro servidor y nos conectamos mediante ssh
ssh <usuario>@<host>
abrimos una sesión nueva de terminal en nuestro PC y creamos la clave ssh mediante el siguiénte código:
ssh-keygen -t ed25519 -C "<comentario>"
Añadimos la clave a nuestro agente ssh mediante el siguiente código:
eval "$(ssh-agent -s)"

ssh-add ~/.ssh/<clave_ssh>
Si el código anterior no funciona de forma permanente, podemos crear el archivo ~/.ssh/config con el siguiente contenido:
Host <host>
	IdentityFile ~/.ssh/<clave_ssh>
Añadimos esa clave a la lista de claves conocidas del servidor mediante el siguiente código:
ssh-copy-id -i ~/.ssh/<clave_ssh>.pub <user>@<host>
Si queremos, podemos añadir una clave ssh en el servidor para acceder a GitHub. Los pasos a seguir son:
pasos 2 y 3 del apartado anterior
copiamos la clave pública en el portapapeles, la podemos mostrar por pantalla y copiarla
cat ~/.ssh/<clave_ssh>.pub
añadimos la clave a nuestra cuenta de GitHub
comprobamos que todo funciona correctamente mediante el siguiente código:
ssh -T git@github.com
Instalación de Docker
actualizamos el servidor con los siguientes comandos
sudo apt update  

#opcional

sudo apt upgrade  
seguimos los pasos que encontramos en la página oficial
# Update the apt package index and install packages to allow apt to use a repository over HTTPS:

 sudo apt-get update

 sudo apt-get install ca-certificates curl gnupg


# Add Docker’s official GPG key:

 sudo install -m 0755 -d /etc/apt/keyrings

 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

 sudo chmod a+r /etc/apt/keyrings/docker.gpg


# Use the following command to set up the repository:

 echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the apt package index:

 sudo apt-get update

# Install Docker Engine, containerd, and Docker Compose.
# To install the latest version, run:

 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


Añadimos nuestro usuario al grupo de docker para poder ejecutar comandos sin sudo
sudo groupadd docker

sudo usermod -aG docker $USER

newgrp docker
NGINX reverse proxy y Letsencrypt SSL
Puesta en marcha
Seguimos los pasos de la siguiente página

Creamos un repositorio en github y añadimos el archivo docker-compose.yml al repositorio.
version: '3'

services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - dhparam:/etc/nginx/dhparam
      - certs:/etc/nginx/certs:ro
      - html:/usr/share/nginx/html
      - /var/run/docker.sock:/tmp/docker.sock:ro
    networks:
      - proxy
    restart: always

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nginx-proxy-le
    volumes_from:
      - nginx-proxy
    volumes:
      - certs:/etc/nginx/certs:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - acme:/etc/acme.sh
      - html:/usr/share/nginx/html
    restart: always

volumes:
  conf:
  vhost:
  dhparam:
  certs:
  acme:
  html:

networks:
  proxy:
    name: nginx-proxy
    external: true
Descargamos el repositorio en el servidor

Creamos la red nginx-proxy en el servidor

docker network create nginx-proxy
Ponemos en marcha los contenedores
docker compose up -d
Creación de un contenedor para el frontend
version: '3'

services:
  nginx:
    image: nginx:latest
    expose:
      - "80"
    environment:
      - VIRTUAL_HOST=<dominio>
      - LETSENCRYPT_HOST=<dominio>
      - LETSENCRYPT_EMAIL=<email>
    volumes:
      - <dirección_completa_al_proyecto>:/usr/share/nginx/html:ro
      ##En caso de que tenga rutass variables o se quiera configurar nginx##
      #- ../nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - proxy
    restart: unless-stopped

networks:
  proxy:
    name: nginx-proxy
    external: true
    external: true
