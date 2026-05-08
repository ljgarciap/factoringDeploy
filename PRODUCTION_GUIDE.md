# Guía de Mantenimiento de Producción - Factoring

Esta guía contiene los procedimientos críticos para mantener el ambiente de producción estable y realizar tareas de mantenimiento sin romper el sistema de autenticación.

## 🔄 Reseteo de Base de Datos (Seguro)

Si necesitas limpiar la base de datos para borrar pruebas y dejar solo los datos iniciales (Seeders), ejecuta estos comandos en orden:

```bash
cd ~/apps/factoring/deployment_docker

# 1. Borrar archivos de migración temporales (si existen)
docker compose exec backend sh -c "rm -f database/migrations/*_create_oauth_*.php"

# 2. Resetear base de datos y ejecutar seeders
docker compose exec backend php artisan migrate:fresh --seed

# 3. Forzar creación de tablas de Passport (apuntando a la fuente en vendor)
docker compose exec backend php artisan migrate --path=vendor/laravel/passport/database/migrations

# 4. Crear llaves y clientes de acceso OAuth (Modo automático)
docker compose exec backend php artisan passport:keys --force
docker compose exec backend php artisan passport:client --personal --no-interaction
docker compose exec backend php artisan passport:client --password --provider="users" --no-interaction
```

## 📂 Exportación de Excel (Directa)

La ruta de exportación de historial ha sido liberada del middleware `auth:api` para permitir descargas directas desde el navegador:
- **URL**: `GET /api/history/{categoria}/export`
- **Nota**: Esta ruta ya no requiere Token Bearer en la cabecera, permitiendo el uso de `window.open()` o links directos en el frontend.

## 🛠️ Limpieza de Caché y Logs

Si realizas cambios en el archivo `.env` o en las rutas, ejecuta:

```bash
docker compose exec backend php artisan config:clear
docker compose exec backend php artisan route:clear
docker compose exec backend php artisan view:clear
```

Para vaciar los logs de Laravel:
```bash
docker compose exec backend sh -c "echo '' > storage/logs/laravel.log"
```

## 🤖 Mantenimiento de n8n

El flujo de trabajo de n8n está configurado para conectarse internamente con Laravel. Si el flujo se detiene:
1. Verifica que el nodo "Mistral" esté devolviendo un JSON válido.
2. El código del nodo "Extract" está blindado contra fechas vacías (usa `00/00/0000` por defecto).
3. Asegúrate de que el token `Authorization: Bearer MiTokenSuperSecreto123` coincida entre n8n y el `.env` del backend.
