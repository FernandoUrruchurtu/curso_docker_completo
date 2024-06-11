FROM node:14
## Instala la nueva version de Node
## Cambia el codigo
COPY ["package.json","package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

COPY [".", "/usr/src/"]

EXPOSE 3000
## Antes: CMD ["node", "index.js"]
## Ahora cambiamos el comando con npx y nodemon para que los cambios
## se reflejen en tiempo 
#CMD ["npx","nodemon", "index.js"]

## dejamos como estaba antes:
CMD ["node", "index.js"]