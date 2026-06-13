# Respuestas del laboratorio de observabilidad

## Instrucciones para validar el trabajo

1. Levantar el stack con `docker compose up -d --build`.
2. Revisar que todos los servicios estén arriba con `docker compose ps`.
3. Entrar a Grafana en `http://localhost:3000` con `admin` / `admin` y confirmar las fuentes Prometheus y Loki.
4. Generar tráfico desde `http://localhost:8080` usando el botón `Saludar (API)`.
5. Crear el dashboard con los paneles de CPU del backend, CPU del host, logs de aplicación y logs de infraestructura.
6. Configurar la alerta `CPU backend > 50%` con evaluación cada `10s` y pending period de `30s`.
7. Probar la alerta con el botón `Generar carga de CPU (30s)` o con `curl "http://localhost:3001/load?seconds=60"`.
8. Verificar el ciclo de alerta a log usando el webhook `http://backend:3001/alerts`.

## Explicación breve del stack

Prometheus recolecta las métricas que exponen el backend, el frontend, cAdvisor y node-exporter.
Loki guarda los logs de los contenedores, porque esos eventos no se representan bien como métricas numéricas.
Alloy lee los logs desde Docker y les agrega etiquetas como `tier=application` o `tier=infrastructure`.
Grafana conecta Prometheus y Loki para visualizar métricas, revisar logs y configurar alertas.
cAdvisor permite observar el consumo de recursos por contenedor, como el CPU del backend.
node-exporter muestra métricas del host completo, por eso sirve para comparar la infraestructura contra una app específica.

## Preguntas

### 1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?

Porque Prometheus trabaja con métricas numéricas en el tiempo, como CPU, memoria o cantidad de peticiones. En cambio, Loki guarda líneas de log con mensajes, errores y eventos del backend, frontend e infraestructura. En este laboratorio se necesitan ambos: Prometheus ayuda a detectar que sube el CPU, y Loki permite revisar qué estaba pasando en los servicios cuando ocurrió.

### 2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

La ventaja es que Grafana arranca ya conectado a Prometheus y Loki sin depender de pasos manuales. Si se borra el entorno y se vuelve a levantar con Docker Compose, las fuentes aparecen otra vez iguales. Eso reduce errores de configuración y hace que el laboratorio sea reproducible.

### 3. El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

Pueden ser distintos porque el CPU del contenedor mide solo el consumo del backend `lab-backend`, mientras que el CPU del host mezcla todo lo que ocurre en la máquina: Docker, navegador, Grafana, Prometheus y otros procesos. Para alertar sobre una aplicación concreta usaría el panel del contenedor, porque apunta directamente al servicio que quiero vigilar. El CPU del host lo usaría como apoyo para saber si el problema es general de la máquina.

### 4. ¿Qué diferencia hay entre el `evaluation interval` y el `pending period` de una alarma?

El `evaluation interval` indica cada cuánto Grafana revisa la condición de la alerta. En este laboratorio se usa cada `10s`, así que Grafana evalúa seguido si el CPU supera el 50%. El `pending period` es el tiempo que la condición debe mantenerse antes de pasar a `Firing`; con `30s`, se evitan alertas por picos muy cortos.

## Capturas sugeridas para el PDF

1. `docker compose ps` mostrando los contenedores arriba.
2. Frontend en `http://localhost:8080`.
3. Backend en `http://localhost:3001/metrics`.
4. Grafana con las fuentes Prometheus y Loki configuradas.
5. Panel `CPU contenedor backend (%)` con umbral en 50.
6. Panel `CPU del host (%)`.
7. Panel `Logs de aplicación (API + frontend)` con `{tier="application"} | json`.
8. Panel `Logs de infraestructura` con `{tier="infrastructure"}`.
9. Regla `CPU backend > 50%` configurada.
10. Estado `Firing` después de generar carga.
11. Log generado por el webhook `http://backend:3001/alerts`.
12. Al final del PDF, la URL del repositorio de GitHub.
