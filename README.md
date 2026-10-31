🚀 Servidor de Páginas Web Múltiples con Nginx (Docker)

Este repositorio contiene la infraestructura automatizada mediante Docker para gestionar todo el tráfico web que entra al servidor. Su propósito es permitirte alojar varias páginas web estáticas (HTML, CSS, JS) en un mismo servidor, sin que interfieran entre sí y consumiendo mínimos recursos.

📖 1. ¿Qué es Nginx y para qué sirve?

Nginx (se pronuncia Engine-X) es uno de los servidores web más rápidos y ligeros del mundo.

Imagina que tu servidor VPS es un edificio con una sola puerta de entrada principal para internet (los puertos 80 y 443). Nginx es el recepcionista de ese edificio.
Cuando un visitante escribe en su navegador www.empresa1.com, esa petición llega al servidor. Nginx lee lo que el visitante escribió, busca rápidamente en sus carpetas la página web correspondiente a ese dominio, y se la muestra al usuario casi al instante.

📁 2. Estructura de las Carpetas

/nginx-proxy
├── .env                 # Contiene los puertos del servidor. Nunca se sube a internet.
├── .env.example         # Plantilla de ejemplo para crear tu archivo .env real.
├── docker-compose.yml   # El archivo orquestador que descarga y levanta Nginx.
├── /conf.d              # REGLAS: Aquí le decimos a Nginx qué dominios existen y a qué carpeta apuntan (.conf).
├── /html                # SITIOS WEB: Aquí creamos las carpetas para guardar el código de cada página.
└── /logs                # REGISTROS: Aquí Nginx anota automáticamente las visitas y los errores.


🚀 3. ¿Cómo levantar el servidor por primera vez?

Si es un servidor VPS nuevo y vas a encender Nginx por primera vez, sigue estos pasos:

Prepara las variables de entorno: Copia la plantilla para crear tu configuración real.

cp .env.example .env


Abre los puertos: Asegúrate de que el firewall del servidor permita el tráfico web (Puertos 80 y 443).

Levanta Nginx: Ejecuta el siguiente comando para que Docker lo inicie en segundo plano (el parámetro -d evita que se bloquee tu terminal).

docker compose up -d


Verifica el estado:

docker compose ps


(Si la columna de estado dice "Up" o "Running", Nginx ya está activo y esperando a que agregues páginas web).

🛠️ 4. ¿Cómo agregar una nueva página web?

Cuando tengas un nuevo cliente o proyecto, debes seguir estos dos pasos exactos para que su página aparezca en internet:

Paso 1: Guardar los archivos de la página

Ve a la carpeta html/ y crea una carpeta con el nombre de tu proyecto o cliente (Ejemplo: empresa1).
Guarda ahí todos los archivos de su página web (index.html, imágenes, estilos, etc.).

Paso 2: Crear la regla de enrutamiento

Ve a la carpeta conf.d/ y crea un archivo de texto con el nombre del cliente (Ejemplo: empresa1.conf). Pega este código dentro y cambia los datos por los reales:

server {
    listen 80;
    
    # 1. El dominio real que el usuario escribirá en internet:
    server_name www.sudominio.com sudominio.com; 

    location / {
        # 2. La ruta a la carpeta donde guardaste los archivos en el Paso 1:
        root /usr/share/nginx/html/empresa1; 
        
        index index.html index.htm;
    }
}


(Importante: El dominio del cliente debe tener un registro "A" en su proveedor de dominio que apunte a la IP de tu VPS).

⚡ 5. ¿Cómo aplicar los cambios sin apagar el servidor? (Zero-Downtime)

⚠️ REGLA DE ORO: Cuando agregas un nuevo archivo .conf o modificas uno existente, NUNCA APAGUES NI REINICIES DOCKER (No uses docker compose restart). Si lo haces, Nginx se apagará por completo y las páginas del resto de tus clientes se caerán por unos segundos.

Para que Nginx lea las nuevas páginas suavemente y sin cortes, ejecuta este comando:

docker exec nginx_proxy nginx -s reload


¿Qué hace este comando?
Entra directamente al contenedor que ya está en funcionamiento y le dice al motor de Nginx que vuelva a leer la carpeta conf.d/. Si cometiste un error de sintaxis en el archivo nuevo, te lo avisará en la terminal, pero las páginas que ya funcionaban seguirán activas y sin interrupción.