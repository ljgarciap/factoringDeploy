# Etapa 1: Build
FROM node:20 as build-stage

WORKDIR /app
ARG FRONTEND_DIR=factoringFrontend

# Copiamos archivos de dependencias
COPY ${FRONTEND_DIR}/package*.json ./
RUN npm install --legacy-peer-deps

# Copiamos el resto del código y compilamos
COPY ${FRONTEND_DIR}/ .
RUN npm run build -- --configuration production

# Etapa 2: Serve
FROM nginx:stable-alpine

# Copiamos el build a la carpeta de nginx
COPY --from=build-stage /app/dist/factoring-frontend/browser /usr/share/nginx/html

# Copiamos configuración de Nginx para Angular (Rutas)
COPY deployment_docker/frontend/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
