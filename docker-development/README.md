# Docker para desarrollo de aplicaciones
Este es tomado del repositorio de docker de [Platzi](https://github.com/platzi/docker.git).  
  
En este repositorio, se cuenta con un proyecto de NodeJS.  
## 1. Explorando el Dockerfile  
El dockerfile tiene la siguiente estructura:  
```dockerfile
FROM node:12

COPY [".", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

EXPOSE 3000

CMD ["node", "index.js"]
``` 
- Como ya vimos en la carpeta imagenes, contamos aqui con el comando `FROM` que nos indica que debemos tomar la imagen de node en su version 12.
- Adicional, realizamos un `COPY` de todo lo que se encuentra en la ruta hacia la ruta `/usr/src/`. 
- Seleccionamos como directorio (similar a un `cd` en bash) y colocamos como ruta donde copiamos toda la informacion.
- Con el comando `RUN` ejecutamos la instruccion `npm install` para que instale la libreria npm.
- El comando `EXPOSE` abre el puerto 3000 para el contenedor.
- `CMD` es una instruccion que indicamos para que ejecute el comando, este sigue la siguiente estructura:  
```dockerfile
CMD [ejecutable, comandos/archivos/etc]
```  
## 2. Corriendo el contenedor

Para este proyecto en particular, se toma la imagen creada anteriormente. En este ejemplo hicimos:  
```bash 
docker build -t platziapp .
```  
Lo siguiente es crear un contenedor a partir de la imagen `platziapp`. Para ello realizamos lo siguiente:  
```bash 
docker run --rm 
```  
- El comando `--rm` sirve para decirle que cuando se detenga este contenedor, no lo conserve en el historial y lo borre directamente.  

El archivo dockerfile en este repositorio esta estructurado de manera ineficiente ya que no reaprovecha todas las capas. Es por eso, que debemos cambiar la estructura a la siguiente.  

```dockerfile
## Instala la nueva version de NodeJS 
FROM node:14 
## Cambia el copy para la instalacion
COPY ["package.json","package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

COPY [".", "/usr/src/"]

EXPOSE 3000

## Antes: CMD ["node", "index.js"]
## Ahora cambiamos el comando con npx y nodemon para que los cambios
## se reflejen en tiempo real
CMD ["npx","nodemon","index.js"]
```  
Ahora bien, con estos cambios, podemos monitorear en tiempo real los cambios que hagamos en el codigo con nuestro contenedor. Usaremos un mount con el comando visto anteriormente:  
```bash
docker run --rm -d -p 3000:3000 -v $(pwd)/index.js:/usr/src/index.js platziapp
```  
Con esta instruccion le estamos diciendo que tome como volumen el archivo index.js para que haga tracking de todos los cambios.  
## 3. Docker networking  
El comando `docker network ls` permite listar todas las redes que tenemos. La brdidge, esta tiene por default para imagenes anteriores. Adicional tenemos la host que es la de la maquina.  
Para crear una nueva red y que pueda adherirse a otras imagenes:  
```bash
docker network create --attachable [nombre-red]
```
`--attachable` es el flag necesario para permitir que otras imagenes se adhieran a esta red previamente creada.  

Para adherir a nuestra imagen:  
```bash
docker network connect [red] [nombre-contenedor]
```  
Una vez realizado este paso, podremos iniciar un contenedor asignandole como variable de entorno con el flag `--env` el host de mi base de datos.  
```bash
docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://db:27017/test platziapp
```  
## 4. Docker - Compose  
Docker compose es una herramienta que ayuda a la orquestacion de las imagenes. En distribuciones de Linux debe instalarse a parte como un plugin, en otros SO viene por defecto.  
Este utiliza archivos `.yml`. El cual sigue una estrucura para poder realizar la orquestacion de estos containers. La estructura basica que veremos es la siguiente:  
```yaml
services:
    container-1:
        image: [nombre-imagen]
        environment:
            VARIABLE_1: ""
            VARIABLE_2: ""
        depends_on:
            - container-2
        ports:
            - "9999:9999"
        networks:
            - networkName
        
    container-2:
        image: [nombre-imagen]
        networks:
            - networkName
    networks:
        networkName:
            name: networkName
            external: true

```  
En esta estructura de ejemplo, estamos subiendo dos containers como servicios, siendo que el container-1 depende del container-2, cada palabra reservada describe que hara en cada uno. Siendo que en environment definimos las variables de entorno necesarias para que docker-compose tome el valor que usaremos en nuestra aplicacion.  

Adicional, podemos tambien definir conexiones en caso de que tengamos creado una en especifico para nuestra app.

Para correr el docker compose, usamos el siguiente comando:  
```bash
docker compose up
```  
**OJO**: Es estando dentro del directorio donde tenemos el archivo docker-compose.  
### 4.1. Comandos que podemos utilizar: 
Estos asumen que se usa la version mas reciente de Docker. Si se quiere replicar este ejercicio, se pueden reemplazar los comandos `docker compose` por `docker-compose`.

```bash
docker compose up -d ## Subimos el compose dejando en el backend la ejecucion
docker compose logs [servicio] ## Queremos ver los logs de cada servcio
docker compose logs [servicio] -f ## hacer un follow a los logs en la terminal
docker compose exec [servico] [comando binario] ## Ejecutar el bash por ejemplo.
docker compose down ## destruye los contenedores.
```  
En el caso del `exec` no hace falta el modo interactivo.  


### 4.2. Desarrollando con Docker Compose  
El script inicial que tenemos guardado para la configuracion es el siguiente:  
```yaml
services:
  app:
    image: platziapp
    environment:
      MONGO_URL: "mongodb://db:27017/test"
    depends_on:
      - db
    ports:
      - "3000:3000"
    networks:
      - platzinet

  db:
    image: mongo
    networks:
      - platzinet

networks:
  platzinet:
    name: platzinet
    external: true
```  
Si queremos modificar este script para que en vez de partir de una imagen, haga un build del `Dockerfile` que tenemos preparado, podemos indicarselo con la instruccion
```yaml
build: .
```
Reemplazandola directamente de la instruccion `image`:
```yaml
services:
  app:
    build: .
    environment:
      MONGO_URL: "mongodb://db:27017/test"
    depends_on:
      - db
    ports:
      - "3000:3000"
    networks:
      - platzinet
    volumes:
      - .:/usr/src
      - /usr/src/node_modules
    command: npx nodemon index.js

  db:
    image: mongo
    networks:
      - platzinet

networks:
  platzinet:
    name: platzinet
    external: true
```  
Para que los contenedores atrapen los cambios que yo realice en alguna de las imagenes, podemos utilizar la siguiente instruccion:  
```bash
docker compose build [nombre-servicio]
docker compose up -d
```  
El primer comando volvera a construir el contenedor de app y luego docker compose up va tomar los cambios que se realizaron.