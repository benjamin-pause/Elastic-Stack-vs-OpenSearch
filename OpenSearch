version: "3.8"

# ============================================================
#  OPENSEARCH STACK — OpenSearch + OpenSearch Dashboards
# ============================================================

services:

  # ----------------------------------------------------------
  # OpenSearch — Fork open source d'Elasticsearch (AWS)
  # ----------------------------------------------------------
  opensearch:
    image: opensearchproject/opensearch:2.13.0
    container_name: opensearch
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch
      - discovery.type=single-node
      - OPENSEARCH_INITIAL_ADMIN_PASSWORD=${OPENSEARCH_PASSWORD:-Admin@1234!}
      - bootstrap.memory_lock=true
      # Désactiver les plugins de démo (laisser en false en prod avec vrais certs)
      - plugins.security.ssl.http.enabled=false
      - plugins.security.disabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - os_data:/usr/share/opensearch/data
    ports:
      - "9201:9200"   # Port décalé pour éviter le conflit avec Elastic (9200)
    networks:
      - opensearch_net
    healthcheck:
      test: ["CMD-SHELL", "curl -s -u admin:${OPENSEARCH_PASSWORD:-Admin@1234!} http://localhost:9200/_cluster/health | grep -q '\"status\":\"green\\|yellow\"'"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ----------------------------------------------------------
  # OpenSearch Dashboards — Interface web (équivalent Kibana)
  # ----------------------------------------------------------
  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:2.13.0
    container_name: opensearch-dashboards
    environment:
      - OPENSEARCH_HOSTS=["http://opensearch:9200"]
      - DISABLE_SECURITY_DASHBOARDS_PLUGIN=true   # Adapter si TLS activé
    ports:
      - "5602:5601"   # Port décalé pour éviter le conflit avec Kibana (5601)
    networks:
      - opensearch_net
    depends_on:
      opensearch:
        condition: service_healthy

# ----------------------------------------------------------
# Volumes persistants
# ----------------------------------------------------------
volumes:
  os_data:
    driver: local

# ----------------------------------------------------------
# Réseau isolé pour la stack OpenSearch
# ----------------------------------------------------------
networks:
  opensearch_net:
    driver: bridge
