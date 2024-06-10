# Creacion de una imagen  
Para crear imagenes, debemos crear un Dockerfile dentro de la carpeta que elijamos. Estando en la carpeta de trabajo, desde la consola usamos:  
```bash
touch Dockerfile
```  
Hay que resaltar que la estructura del dockerfile siempre es con D en mayuscula. `Dockerfile`

## 1. Dockerfile (basico)
Con el dockerfile creado, escribimos lo siguiente para el ejercicio:  
```dockerfile
FROM ubuntu:latest

RUN touch /usr/src/hola-platzi.txt
```  
A este comando, le estamos indicando que tome desde la imagen `ubuntu:latest`, es decir la ultima version con la palabra reservada 
```dockerfile 
FROM [nombre-imagen]:[tag de version]
```
Luego, la palabra reservada `RUN` indica que ejecutaremos un comando cuando se inicie la imagen, por ejemplo:  
```dockerfile
RUN [comando bash] touch /usr/src/hola-platzi.txt
``` 
Ahora bien, para convertir esto en una imagen, hacemos lo siguiente:  
```bash
docker build -t [nombre-imagen]:[version-tag] .
```  
El punto al final, por su parte, le da el contexto para poder crear la imagen a partir del dockerfile.  
Todas las imagenes por si solas, son un conjunto de capas a las que se apuntan con cada comando.  
Ahora bien, para hacer un login en docker con el docker hub, ejecutamos `docker login` lo cual se configuro previamente, y para apuntar a nuestro repositorio, ejecutamos el siguiente comando:
```bash
docker tag [nombre-imagen]:[tag-version] [username]/[imagen]:[tag-version]
```  
El comando tag lo que hace es realizar un cambio en la imagen existente y asignarle un nuevo tag.  
Por cada capa se crea un espacio, por lo que es importante tener en cuenta que debe optimizarse siempre.