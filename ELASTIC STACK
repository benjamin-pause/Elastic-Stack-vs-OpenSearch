version: "3.8"

# ============================================================
#  ELASTIC STACK — Elasticsearch + Kibana + Fleet Server
#  + Filebeat + Metricbeat + Auditbeat
# ============================================================

services:

  # ----------------------------------------------------------
  # Elasticsearch — Moteur de recherche / stockage des données
  # ----------------------------------------------------------
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.0
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=es-cluster
      - discovery.type=single-node
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD:-changeme}
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=false   # TLS HTTP désactivé (ajuster en prod)
      - xpack.license.self_generated.type=basic
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"   # API REST
      # Port 9300 (transport interne) — non exposé à l'hôte
    networks:
      - elastic_net
    healthcheck:
      test: ["CMD-SHELL", "curl -s -u elastic:${ELASTIC_PASSWORD:-changeme} http://localhost:9200/_cluster/health | grep -q '\"status\":\"green\\|yellow\"'"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ----------------------------------------------------------
  # Kibana — Interface web de visualisation
  # ----------------------------------------------------------
  kibana:
    image: docker.elastic.co/kibana/kibana:8.13.0
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD:-changeme}
      - xpack.fleet.enabled=true
    ports:
      - "5601:5601"   # Interface web
    networks:
      - elastic_net
    depends_on:
      elasticsearch:
        condition: service_healthy

  # ----------------------------------------------------------
  # Fleet Server — Gestion centralisée des agents Elastic
  # ----------------------------------------------------------
  fleet-server:
    image: docker.elastic.co/beats/elastic-agent:8.13.0
    container_name: fleet-server
    environment:
      - FLEET_SERVER_ENABLE=true
      - FLEET_SERVER_ELASTICSEARCH_HOST=http://elasticsearch:9200
      - FLEET_SERVER_SERVICE_TOKEN=${FLEET_TOKEN:-}
      - FLEET_SERVER_POLICY_ID=fleet-server-policy
    ports:
      - "8220:8220"   # Communication Fleet agents
    networks:
      - elastic_net
    depends_on:
      - kibana

  # ----------------------------------------------------------
  # Filebeat — Collecte des logs (Docker, système)
  # ----------------------------------------------------------
  filebeat:
    image: docker.elastic.co/beats/filebeat:8.13.0
    container_name: filebeat
    user: root
    environment:
      - ELASTICSEARCH_HOST=http://elasticsearch:9200
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD:-changeme}
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - elastic_net
    depends_on:
      elasticsearch:
        condition: service_healthy

  # ----------------------------------------------------------
  # Metricbeat — Métriques système (CPU, RAM, disque...)
  # ----------------------------------------------------------
  metricbeat:
    image: docker.elastic.co/beats/metricbeat:8.13.0
    container_name: metricbeat
    user: root
    environment:
      - ELASTICSEARCH_HOST=http://elasticsearch:9200
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD:-changeme}
    volumes:
      - ./metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:ro
      - /proc:/hostfs/proc:ro
      - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
      - /:/hostfs:ro
    networks:
      - elastic_net
    depends_on:
      elasticsearch:
        condition: service_healthy

  # ----------------------------------------------------------
  # Auditbeat — Audit Linux (accès fichiers, appels système)
  # ----------------------------------------------------------
  auditbeat:
    image: docker.elastic.co/beats/auditbeat:8.13.0
    container_name: auditbeat
    user: root
    cap_add:
      - AUDIT_CONTROL
      - AUDIT_READ
    pid: host
    environment:
      - ELASTICSEARCH_HOST=http://elasticsearch:9200
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD:-changeme}
    volumes:
      - ./auditbeat.yml:/usr/share/auditbeat/auditbeat.yml:ro
    networks:
      - elastic_net
    depends_on:
      elasticsearch:
        condition: service_healthy

# ----------------------------------------------------------
# Volumes persistants
# ----------------------------------------------------------
volumes:
  es_data:
    driver: local

# ----------------------------------------------------------
# Réseau isolé pour la stack Elastic
# ----------------------------------------------------------
networks:
  elastic_net:
    driver: bridge
