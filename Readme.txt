HERRAMIENTAS:
- Jmeter
- InfluxDB
- Prometheus
- CAdvisor
- Grafana
- Docker


-------------------------------------------------------------------------------

OBJETIVO:
-Monitorear las pruebas de Jmeter a traves de InfluxDB y usar Grafana como herramienta de visualización.
-Monitorear las métricas de recursos de los contenedores en ejecución de las imágenes usando CAdvisor como una fuente de datos externa (exporter), enviándolas y almacenándolas en Prometheus para luego ser utilizado como data source en Grafana


-------------------------------------------------------------------------------

INFORMACION ADICIONAL:
-Crear un docker compose que contenga las imágenes de InfluxDB, Prometheus, CAdvisor y Grafana.
-InfluxDB recopilara la data de Jmeter y se conectara con Grafana para mostrar las métricas de Jmeter.
-CAdvisor recopilará la información del comportamiento de los contenedores que se levantaron, luego Prometheus utilizara la data generada por CAdvisor para añadirlo a su suite de métricas y finalmente Prometheus se integrará a Grafana como data source.
-Grafana tendremos como data source a InfluxDB que tiene data de Jmeter, y a Prometheus que tiene data de CAdvisor y de sí mismo.


-------------------------------------------------------------------------------

REQUISITOS:
-Jmeter
-Docker Desktop
-Docker Compose


-------------------------------------------------------------------------------

ESTRUCTURA DE CARPETA
mi-proyecto/
├── docker-compose.yml
├── prometheus.yml
├── scripts/
│   └── script.jmx
└── results/


-------------------------------------------------------------------------------

PASOS:
1. Crear el Docker Compose con las imágenes de Jmeter, InfluxDB, Prometheus, CAdvisor y Grafana. Configurar correctamente los volumes y puertos.
2. Tener el script listo con el Backend Listener configurado a InfluxDB, como influxdbUrl colocar http://influxdb:8086/write?db=jmeter_db
3. Guardar el script en una carpeta "scripts" dentro de la misma carpeta donde esta el docker compose
4. Levantar la infraestructura usando el Docker Compose en la consola de Docker Desktop "docker compose up -d influxdb cadvisor prometheus grafana"
	- "-d" es para liberar la terminal y se pueda seguir escribiendo mientras todo corre por atrás.
5. Ejecutar la prueba de Jmeter usando el siguiente comando "docker compose run --rm jmeter" y parar usando CTRL+C (o forzarlo usando "docker compose stop jmeter")


Verificar estado de los contenedores: "docker compose ps"


-------------------------------------------------------------------------------

ANEXOS

PROMETHEUS.YML
global:
  scrape_interval: 5s # Frecuencia de recolección

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']


DOCKER-COMPOSE.YML

version: '3.8'

services:
  influxdb:
    image: influxdb:1.8
    container_name: influxdb
    ports:
      - "8086:8086"
    volumes:
      - ./influxdb_data:/var/lib/influxdb
    environment:
      - INFLUXDB_DB=jmeter_db

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    depends_on:
      - influxdb
      - prometheus

  jmeter:
    image: justb4/jmeter:5.5
    container_name: jmeter
    volumes:
      - ./scripts:/scripts
      - ./results:/results
    # Este comando se ejecutará cuando hagamos 'docker compose run'
    entrypoint: ["jmeter", "-n", "-t", "/scripts/test_demoblaze.jmx", "-l", "/results/resultado.jtl"]





