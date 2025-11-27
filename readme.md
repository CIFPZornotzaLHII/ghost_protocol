# 🕵️‍♂️ OPERACIÓN: GHOST PROTOCOL

![alt text](image.png)

--- 

- [🕵️‍♂️ OPERACIÓN: GHOST PROTOCOL](#️️-operación-ghost-protocol)
  - [Briefing](#briefing)
  - [🛑 MISIÓN 1: Limpieza y Seguridad (.env) (45 min)](#-misión-1-limpieza-y-seguridad-env-45-min)
  - [🔗 MISIÓN 2: La Tubería de Datos (45 min)](#-misión-2-la-tubería-de-datos-45-min)
  - [🕵️‍♂️ MISIÓN 3: El Análisis Forense (30 min)](#️️-misión-3-el-análisis-forense-30-min)

---

**Clasificación:** TOP SECRET
**Tiempo límite:** 120 Minutos
**Estado:** CRÍTICO

## Briefing
¡Atención, equipo\!

La redacción de *The Daily Leak* ha sido comprometida. Hemos perdido el acceso a nuestros servidores físicos. Tenemos a 20 periodistas de investigación esperando en pisos francos para subir pruebas de corrupción, redactar artículos y analizar vídeos, pero **no tienen herramientas**.

El anterior SysAdmin intentó levantar una infraestructura de emergencia en Docker antes de desaparecer, pero dejó el trabajo a medias, lleno de errores y sin cumplir los protocolos de seguridad.

Vuestra misión es reactivar el sistema. Si falláis, la exclusiva se pierde.

-----

## 🛑 MISIÓN 1: Limpieza y Seguridad (.env) (45 min)

**Informe de situación:**
El archivo `docker-compose.yml` que nos dejó el anterior admin es un desastre.

1.  Tiene errores de sintaxis que impiden que arranque.
2.  Tiene **valores "hardcodeados"**. Por política de seguridad de la empresa, **NO** puede haber puertos ni rutas escritas directamente en el compose. Debéis usar un archivo `.env`.
3.  Algunos nombres de imágenes son sospechosos.

**Objetivos:**

1.  Cread un archivo `.env` donde defináis las siguientes variables (ejemplo):
      * `PORT_FILEBROWSER=8095`
      * `PORT_JELLYFIN=8096`
      * `PUID=1000`
      * `RUTA_BASE=./redaccion_data`
      * *(Y el resto de puertos necesarios)*
2.  Sustituid los valores fijos en el `docker-compose.yml` por las variables (ejemplo: `${PORT_FILEBROWSER}:80`).
3.  **Detectad y reparad los 3 sabotajes** que impiden el despliegue.

**Recursos (El archivo corrupto):**
*Copiad esto en vuestro editor. ¡Cuidado, está roto\!*

```yaml
version: "3.8"

services:
  uploader:
    image: filebrowser/filebrowser:v2.23.0
    container_name: uploader
    volumes:
      - ${RUTA_BASE}:/srv
      - filebrowser_db:/database
    ports:
      - "80:80"   # ERROR DE SEGURIDAD: Puerto 80 reservado por sistema, cambialo a variable
    restart: unless-stopped

  firefox:
    image: jlesage/firefox-gui  # OJO: ¿Esta imagen existe así tal cual? Verificad en Docker Hub.
    ports:
      - "${PORT_FIREFOX}:5800"
    volumes:
      - firefox_config:/config:rw

  obsidian:
    image: lscr.io/linuxserver/obsidian:latest
      container_name: obsidian  # ERROR DE SINTAXIS: Mala indentación
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=Etc/UTC
    volumes:
      - obsidian_config:/config
    ports:
      - "3000:3000" # ERROR: Conflictos de puerto si no se parametrizan. Usad variables.
      - "3001:3001" 
    shm_size: "1gb"
    restart: unless-stopped

  libreoffice:
    image: lscr.io/linuxserver/libreoffice:latest
    container_name: libreoffice
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=Etc/UTC
    volumes:
      - libreoffice_config:/config
    ports:
      - "3000:3000" # ERROR LÓGICO: ¡Este puerto ya lo usa Obsidian!
      - "3001:3001" 
    shm_size: "1gb"
    restart: unless-stopped

  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    ports:
      - "${PORT_JELLYFIN}:8096"
    environment:
      - TZ=Etc/UTC
    volumes:
      - jellyfin_config:/config
    restart: unless-stopped

volumes:
  filebrowser_db:
  firefox_config:
  obsidian_config:
  libreoffice_config:
  jellyfin_config:
```

> **🎖️ Checkpoint:** La misión se supera cuando ejecutáis `docker compose up -d` y TODOS los contenedores están en verde y accesibles.

> **📂 SUBIDA DE EVIDENCIA \#1: EL DESPLIEGUE**
> Para confirmar que la infraestructura es segura, subid al campus una **captura de pantalla** que muestre:
>
> 1.  Vuestro terminal ejecutando el comando `docker ps` (deben verse los 5 contenedores en estado "Up").
> 2.  El contenido de vuestro archivo `.env` abierto en el editor (para verificar que no hay secretos hardcodeados).

-----

## 🔗 MISIÓN 2: La Tubería de Datos (45 min)

**Informe de situación:**
El sistema arranca, pero es inútil.
Un periodista sube una foto a través del "Gestor de Archivos" (Uploader), pero cuando abre LibreOffice para pegarla en su artículo... ¡la foto no está\! Y el editor de vídeo (Jellyfin) tampoco ve nada.

Cada contenedor es una isla y necesitamos un puente.

**Objetivos:**

1.  Modificad el `docker-compose.yml`.
2.  Todos los servicios (`uploader`, `firefox`, `obsidian`, `libreoffice`, `jellyfin`) deben tener acceso a una carpeta común en vuestro ordenador llamada `./redaccion_data`.
3.  **Reto Técnico:** Tened cuidado con las rutas internas de cada contenedor:
      * En **Filebrowser** la data va en `/srv`.
      * En **Jellyfin** la data va en `/media`.
      * En **LibreOffice** y **Obsidian**, investigad dónde guardan los documentos (pista: suele ser `/config` o una carpeta `/data` que debéis crear).

**Prueba de fuego:**
Debéis ser capaces de subir un archivo `.mp4` desde Filebrowser y reproducirlo inmediatamente en Jellyfin sin mover nada más.

> **🎖️ Checkpoint:** Enseñad al instructor cómo un archivo creado en LibreOffice aparece automáticamente en el Filebrowser.

> **📂 SUBIDA DE EVIDENCIA \#2: LA CONEXIÓN**
> Necesitamos probar que la tubería de datos fluye. Subid **dos capturas de pantalla** (o un collage):
>
> 1.  **Captura A:** LibreOffice abierto escribiendo un documento llamado "CONFIDENCIAL\_[VUESTRO\_NOMBRE].odt".
> 2.  **Captura B:** El navegador web mostrando la interfaz de **Filebrowser** donde se vea claramente ese mismo archivo "CONFIDENCIAL\_[VUESTRO\_NOMBRE].odt" ya disponible para descarga.

-----

## 🕵️‍♂️ MISIÓN 3: El Análisis Forense (30 min)

**Informe de situación:**
¡Buen trabajo con la infraestructura\! Pero no hay tiempo para celebraciones.
Nuestros informantes han dejado un "paquete digital" en una ubicación remota. Contiene la prueba definitiva del escándalo, pero está en un formato de vídeo que necesita ser analizado frame a frame.

**Vuestra tarea es interceptar, almacenar y analizar la evidencia sin dejar rastro en vuestro ordenador personal (Host). Todo debe ocurrir dentro de los contenedores.**

**Objetivos:**

1.  **La Intercepción (Firefox):**

      * Abrid el navegador seguro (Firefox).
      * Acceded a la siguiente URL: `https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/1080/Big_Buck_Bunny_1080_10s_1MB.mp4`
      * Descargad el vídeo y guardadlo **exclusivamente** en la carpeta compartida que configurasteis (`/config/downloads` o donde apuntara vuestro volumen).

2.  **La Cadena de Custodia (Filebrowser):**

      * Entrad al Gestor de Archivos.
      * Verificad que el vídeo ha aparecido ahí.
      * Renombrad el archivo a: `EVIDENCIA_FINAL_EQUIPO_[NUMERO].mp4`.
      * Cread una carpeta llamada `CASO_CERRADO` y moved el vídeo dentro.

3.  **El Análisis (Jellyfin):**

      * Entrad a vuestro servidor multimedia Jellyfin.
      * Si hicisteis bien la Misión 2, el vídeo debería aparecer automáticamente en la biblioteca (quizás tengáis que dar a "Escanear Biblioteca").
      * Reproducid el vídeo.

4.  **El Informe Final (Obsidian):**

      * Abrid Obsidian.
      * Cread una nota llamada **"CONCLUSIÓN"**.
      * Escribid: *"La evidencia ha sido asegurada correctamente en el servidor. El sistema es estable."*
      * **RETROALIMENTACIÓN VISUAL:** Insertad (embebed) el video o una captura de pantalla del video dentro de la nota de Obsidian (si la configuración lo permite) o simplemente dejad el enlace al archivo local.

> **📂 SUBIDA DE EVIDENCIA \#3: INFORME FINAL**
> Para cerrar el caso, subid una **captura de pantalla completa de vuestro escritorio** mostrando:
>
> 1.  La nota de **Obsidian** abierta con el texto de "CONCLUSIÓN".
> 2.  De fondo (o al lado), **Jellyfin** reproduciendo el vídeo `EVIDENCIA_FINAL_EQUIPO_[NUMERO].mp4`.
>     *Esto demostrará que la persistencia de datos, la red y las aplicaciones funcionan simultáneamente.*
