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

El docker 