# Informe de Investigación: Docker

**Docker y su importancia en la Arquitectura de Software**

Lucas Perez  
Ingeniería de software NRC: 1508  
Docente: Luis Giovanny Carreno Ortiz  
16/10/2026

---

## Introducción

Hoy las aplicaciones rara vez son un solo programa: suelen combinar una API, una base de datos, un sistema de caché y, a veces, un broker de mensajes. Instalar y configurar todas estas piezas de la misma forma en cada computador y servidor es difícil, y las diferencias entre entornos provocan errores que no vienen del código. Docker surgió para resolver este problema empaquetando cada aplicación con todo lo que necesita. Este informe explica qué es Docker, sus componentes, funcionalidades y comandos principales, el problema que resuelve y por qué es una pieza clave para implementar arquitecturas como microservicios, SOA y orientada a eventos.

## Desarrollo

### 1. ¿Qué es Docker?

Docker es una plataforma que permite empaquetar una aplicación junto con todo lo que necesita para funcionar (código, librerías, dependencias y configuración) en una unidad llamada **contenedor**, que se ejecuta igual en cualquier máquina con Docker.

Un contenedor es un proceso aislado que corre sobre el sistema operativo del host. No incluye un sistema operativo completo, sino solo los archivos que la aplicación necesita. Comparte el **kernel** del host porque Docker aprovecha funciones del propio kernel de Linux para aislar procesos: los *namespaces* hacen que cada contenedor vea su propio sistema de archivos, red y procesos, y los *cgroups* limitan cuánta CPU y memoria puede usar. Como el kernel ya está en ejecución, el contenedor no necesita arrancar uno propio.

**Contenedor vs. máquina virtual:** una VM virtualiza el hardware y cada una carga un sistema operativo completo (varios GB, arranque en minutos). Un contenedor virtualiza a nivel de sistema operativo y comparte el kernel, por lo que pesa pocos MB, arranca en segundos y consume mucha menos memoria. La contrapartida es que una VM ofrece un aislamiento más fuerte.

```
   MÁQUINA VIRTUAL                 CONTENEDOR
[App A] [App B]                [App A] [App B]
[SO guest][SO guest]           [   Docker Engine  ]
[   Hipervisor    ]            [ SO host (kernel) ]
[    SO host      ]            [     Hardware     ]
[    Hardware     ]
```

### 2. ¿Cuáles son sus componentes?

- **CLI (`docker`):** herramienta de línea de comandos con la que el usuario da órdenes. No ejecuta nada por sí misma: envía cada comando al daemon mediante una API REST.
- **Daemon (`dockerd`):** el motor que corre en segundo plano y hace el trabajo real: construye imágenes, crea contenedores y gestiona redes, volúmenes y la comunicación con registros.
- **Dockerfile:** archivo de texto con las instrucciones para construir una imagen. Es la "receta".
- **Imagen:** plantilla de solo lectura, compuesta por capas, con todo lo necesario para ejecutar la aplicación. Es como una "clase".
- **Contenedor:** instancia en ejecución de una imagen. De una imagen pueden crearse muchos contenedores, como objetos de una clase.
- **Registro (ej. Docker Hub):** repositorio donde se almacenan y comparten imágenes.

```
CLI --API REST--> Daemon (dockerd)
                    |
   Dockerfile --docker build--> Imagen --docker run--> Contenedor
                                 ^  |
                     docker pull |  | docker push
                                 |  v
                         Registro (Docker Hub)
```

La cadena funciona así: se escribe un **Dockerfile**; `docker build` lo convierte en una **imagen**; `docker run` crea un **contenedor** a partir de ella. El **registro** entra en dos momentos: con `docker push` se publica la imagen para que otros la usen, y con `docker pull` se descarga una imagen ya hecha (por ejemplo `mysql:8.0`) sin necesidad de escribir un Dockerfile.

### 3. ¿Cuáles son sus principales funcionalidades?

- **Portabilidad:** una imagen se ejecuta igual en cualquier máquina con Docker. *Ejemplo:* construyo mi API en Windows, la subo a Docker Hub y el docente la ejecuta en su Mac con `docker run usuario/mi-api` sin instalar nada más.
- **Aislamiento:** cada contenedor tiene sus propias dependencias, así que las aplicaciones no interfieren. *Ejemplo:* en un mismo servidor corren una app que necesita Python 2.7 y otra con Python 3.12.
- **Versionado de imágenes:** las etiquetas fijan qué versión se usa y permiten volver atrás. *Ejemplo:* si `mi-api:2.1` falla en producción, se vuelve a desplegar `mi-api:2.0` en segundos.
- **Reproducibilidad del entorno:** el entorno queda descrito como código. *Ejemplo:* un compañero nuevo clona el repositorio, ejecuta `docker compose up` y tiene la API, PostgreSQL y Redis funcionando en minutos.
- **Escalabilidad:** los contenedores arrancan rápido, así que se pueden levantar varias instancias. *Ejemplo:* `docker compose up --scale api=4` levanta cuatro instancias de la API.
- **Base para CI/CD:** los pipelines pueden construir, probar y publicar imágenes automáticamente. *Ejemplo:* con GitHub Actions, cada push a `main` construye la imagen, ejecuta las pruebas y la sube al registro.

### 4. ¿Cuáles son sus principales comandos?

**a) Tabla de comandos**

| Comando | ¿Qué hace? | Ejemplo |
|---|---|---|
| `docker pull` | Descarga una imagen de un registro | `docker pull mysql:8.0` |
| `docker run` | Crea y ejecuta un contenedor desde una imagen | `docker run -d -p 8080:80 --name web nginx` |
| `docker ps` | Lista los contenedores en ejecución | `docker ps` |
| `docker ps -a` | Lista todos los contenedores, incluidos los detenidos | `docker ps -a` |
| `docker logs` | Muestra la salida de un contenedor | `docker logs -f web` |
| `docker exec` | Ejecuta un comando dentro de un contenedor activo | `docker exec -it mi-mysql bash` |
| `docker stop` / `start` | Detiene / inicia un contenedor | `docker stop web` |
| `docker rm` | Elimina un contenedor detenido | `docker rm web` |
| `docker images` | Lista las imágenes locales | `docker images` |
| `docker rmi` | Elimina una imagen local | `docker rmi nginx:latest` |
| `docker build` | Construye una imagen desde un Dockerfile | `docker build -t mi-api:1.0 .` |
| `docker compose up` | Levanta los servicios de `docker-compose.yml` | `docker compose up -d` |
| `docker compose down` | Detiene y elimina los servicios de Compose | `docker compose down` |
| `docker push` | Sube una imagen a un registro | `docker push usuario/mi-api:1.0` |

**b) Tres comandos en detalle**

**`docker run`** crea y arranca un contenedor. Sus parámetros más comunes son: `-d` para ejecutarlo en segundo plano (servicios que deben quedarse activos); `-p host:contenedor` para publicar un puerto y poder acceder al servicio; `-e VAR=valor` para pasar configuración como contraseñas sin modificar la imagen; `-v volumen:ruta` para que los datos persistan aunque se borre el contenedor; y `--name` para darle un nombre legible. Lo usaría para levantar una base de datos de desarrollo:

```bash
docker run -d --name mi-mysql -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=secreto -v datos_mysql:/var/lib/mysql mysql:8.0
```

**`docker exec`** ejecuta un comando dentro de un contenedor que ya está corriendo. Se usa casi siempre con `-it` (`-i` mantiene la entrada abierta y `-t` asigna una terminal); también acepta `-u` para elegir el usuario y `-w` para el directorio de trabajo. Lo usaría para depurar o revisar datos: `docker exec -it mi-mysql mysql -u root -p`.

**`docker build`** construye una imagen a partir de un Dockerfile. Usa `-t nombre:etiqueta` para nombrar y versionar la imagen, el `.` final para indicar la carpeta de contexto, `-f` para usar otro Dockerfile (ej. `Dockerfile.prod`) y `--no-cache` para reconstruir desde cero. Lo usaría para empaquetar mi API antes de publicarla:

```bash
docker build -t usuario/mi-api:1.0 .
docker push usuario/mi-api:1.0
```

### 5. ¿Qué problema resuelve?

El problema clásico es el de **"en mi máquina funciona"**: un desarrollador prueba su aplicación y todo va bien, pero en el computador de un compañero o en el servidor falla. El código es el mismo; lo que cambia es el entorno. Las causas típicas son: **versiones de dependencias** distintas (Node 20 en un lado, Node 16 en otro); **sistema operativo** diferente (rutas, mayúsculas en nombres de archivo, librerías del sistema); **configuración** distinta (variables de entorno, puertos, zona horaria); e **instalación manual**, donde cada persona sigue la guía a su manera y cada máquina termina siendo un caso único.

**Antes:** un equipo de tres estudiantes desarrolla una tienda en línea con Python 3.11, PostgreSQL 15 y Redis. Una integrante tiene PostgreSQL 13 de otro curso, otro no logra instalar Redis en su Mac, y el servidor de pruebas tiene Python 3.8, así que la API falla. Pierden dos días configurando entornos.

**Después:** el equipo agrega un Dockerfile y un `docker-compose.yml` que define la API, `postgres:15` y `redis:7`. Cada integrante solo instala Docker y ejecuta `docker compose up`. Todos usan exactamente las mismas versiones, y el servidor de pruebas corre las mismas imágenes.

### 6. ¿Por qué es importante dentro de una Arquitectura de Software?

**Un servicio por contenedor.** Que la API, la base de datos y el broker de eventos corran cada uno en su contenedor permite que tengan dependencias independientes, que se actualicen o reinicien sin afectar a los demás, que un fallo quede contenido y que cada uno se escale según su propia carga. Es la aplicación práctica de la **separación de responsabilidades** y el **bajo acoplamiento**.

**Despliegue consistente.** La misma imagen que se construye en desarrollo pasa a pruebas y luego a producción; entre entornos solo cambia la configuración externa (variables de entorno, secretos). Esto elimina los errores que aparecen "solo en producción" y hace que el rollback sea tan simple como volver a la etiqueta anterior.

**Escalabilidad y nube.** Como los contenedores arrancan en segundos, se pueden añadir instancias cuando sube la demanda (escalado horizontal), y orquestadores como Kubernetes lo automatizan. Además, todos los grandes proveedores (AWS, Azure, Google Cloud) ejecutan contenedores de forma nativa, por lo que una aplicación contenerizada se mueve entre ellos con cambios mínimos.

**Relación con los estilos arquitectónicos:**

- **Microservicios:** Docker es su base práctica. Cada microservicio tiene su propia imagen, lenguaje y base de datos, y se despliega y escala por separado.
- **SOA:** los servicios, más grandes y comunicados por contratos (a veces mediante un ESB), también se benefician de desplegarse en contenedores aislados y reproducibles.
- **Orientada a eventos:** brokers como Kafka o RabbitMQ se levantan fácilmente como contenedores, y los consumidores se pueden escalar según el volumen de eventos en cola.

## Conclusión

Docker no define una arquitectura, pero hace posible implementarla: convierte decisiones como separar servicios, desplegar de forma independiente o escalar horizontalmente en algo que realmente se puede construir y operar. Lo que más destaco es que resuelve un problema muy cotidiano, el de "en mi máquina funciona", al describir el entorno como código.

Por ejemplo, lo usaría en el proyecto final del curso, donde trabajamos en equipo con una API, una base de datos y un servicio de notificaciones. Con un `docker-compose.yml` cada integrante podría levantar todo el sistema con un solo comando, evitaríamos perder tiempo en instalaciones, y el despliegue final sería más confiable porque usaríamos las mismas imágenes que probamos durante el desarrollo.

## Referencias

1. Docker Inc. (s.f.). *What is Docker?* Docker Documentation. https://docs.docker.com/get-started/docker-overview/
2. Docker Inc. (s.f.). *Docker CLI reference*. Docker Documentation. https://docs.docker.com/reference/cli/docker/
3. Docker Inc. (s.f.). *Dockerfile reference*. Docker Documentation. https://docs.docker.com/reference/dockerfile/
4. Docker Inc. (s.f.). *Docker Compose overview*. Docker Documentation. https://docs.docker.com/compose/
5. Merkel, D. (2014). Docker: lightweight Linux containers for consistent development and deployment. *Linux Journal*, 2014(239).
6. Lewis, J. y Fowler, M. (2014). *Microservices*. https://martinfowler.com/articles/microservices.html
7. Newman, S. (2021). *Building Microservices* (2.ª ed.). O'Reilly Media.
