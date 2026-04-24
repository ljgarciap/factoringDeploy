# Etapa 1: Build
FROM node:20 as build-stage

WORKDIR /app

# Copiamos archivos de dependencias
COPY ../../factoringFrontend/package*.json ./
RUN npm install

# Copiamos el resto del código y compilamos
COPY ../../factoringFrontend/ .
RUN npm run build -- --configuration production

# Etapa 2: Serve
FROM nginx:stable-alpine

# Copiamos el build a la carpeta de nginx
COPY --from=build-stage /app/dist/factoringFrontend /usr/share/nginx/html

# Copiamos configuración de Nginx para Angular (Rutas)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
