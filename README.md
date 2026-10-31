# Despliegue de Nginx (Proxy Inverso y Multi-Sitio) con Docker Compose

## ¿Qué es Nginx y para qué sirve?
Nginx (pronunciado *Engine-X*) es uno de los servidores web y proxy inversos más rápidos y ligeros del mundo. Sirve como la puerta de entrada principal para el tráfico de internet hacia nuestro servidor.

Básicamente, Nginx recibe las peticiones de los usuarios (cuando escriben una dirección web en su navegador) y decide de manera inteligente a qué carpeta o a qué servicio interno debe enviar a ese usuario.

## ¿En qué puede ayudarnos?
Para un equipo enfocado en infraestructura, soporte técnico y gestión de plataformas, Nginx nos permite:
* **Enrutamiento Inteligente (Proxy Inverso):** Recibir el tráfico en el puerto 80 (HTTP) o 443 (HTTPS) y redirigirlo hacia otras herramientas internas sin tener que abrir decenas de puertos en el firewall.
* **Alojamiento Multi-Sitio:** Mantener múltiples páginas web estáticas (HTML/CSS) de distintos clientes alojadas en el mismo servidor VPS, consumiendo mínimos recursos.
* **Organización:** Mantener un control centralizado de los dominios desde un único contenedor.

## Estructura de las carpetas
Este repositorio contiene la configuración para desplegar Nginx de manera segura. La estructura se divide de la siguiente manera:

```text
/nginx-proxy
├── .env                 # Contiene los puertos del servidor. Nunca se sube a internet.
├── .env.example         # Plantilla de ejemplo para crear tu archivo .env real.
├── docker-compose.yml   # El archivo orquestador que descarga y levanta Nginx.
├── /conf.d              # REGLAS: Aquí le decimos a Nginx qué dominios existen y a qué carpeta apuntan.
├── /html                # SITIOS WEB: Aquí creamos las subcarpetas para guardar el código de cada página.
└── /logs                # REGISTROS: Directorio donde Nginx guardará automáticamente el registro de visitas y errores.
```

## Cómo levantar el proyecto

1. Clona este repositorio o descarga los archivos en tu VPS.
2. Crea un archivo llamado `.env` basándote en la plantilla `.env.example` (nunca subas tus puertos reales al repositorio público). El archivo `.env` debe contener:
   ```env
   HTTP_PORT=80
   HTTPS_PORT=443
   TZ=America/La_Paz
   ```
3. Abre los puertos en el firewall de tu servidor:
   ```bash
   sudo firewall-cmd --permanent --add-port=80/tcp
   sudo firewall-cmd --permanent --add-port=443/tcp
   sudo firewall-cmd --reload
   ```
4. Levanta el servidor ejecutando el siguiente comando en segundo plano:
   ```bash
   docker compose up -d
   ```

## Cómo alojar nuevas páginas web

Cuando tengas un nuevo cliente y quieras alojar su página web estática, debes seguir dos pasos:

1. **Guardar los archivos:** Ve a la carpeta `html/` y crea una carpeta con el nombre de tu cliente (Ej. `html/empresa1/`). Pon ahí todos sus archivos y su `index.html`.
2. **Crear la regla de Nginx:** Ve a la carpeta `conf.d/` y crea el archivo `empresa1.conf` con esta estructura:
   ```nginx
   server {
       listen 80;
       
       # El dominio que el cliente usará en internet
       server_name [www.sudominio.com](https://www.sudominio.com) sudominio.com; 

       location / {
           # La ruta a la carpeta donde guardaste los archivos en el Paso 1
           root /usr/share/nginx/html/empresa1; 
           index index.html index.htm;
       }
   }
   ```

## ⚠️ Aplicar cambios (Zero-Downtime)

**Regla crítica:** Si agregas una nueva página o modificas un archivo `.conf`, **NUNCA** uses `docker compose restart` ni apagues el contenedor. Si lo haces, todas las demás páginas en producción se caerán.

Para que Nginx aplique las configuraciones nuevas suavemente y sin cortes de servicio, ejecuta siempre este comando:
```bash
docker exec nginx_proxy nginx -s reload
```