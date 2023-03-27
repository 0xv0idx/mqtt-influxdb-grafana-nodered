# mqtt-influxdb-grafana-nodered
MQTT, InfluxDB, Grafana, Node-RED Docker compose reposu

1.adım : gerekli config log data dosyaları için klasörlerin oluşturulması ve izinlerinin verilmesi

mkdir -m 777 -p ~/mosquitto/config && \
mkdir -m 777 -p ~/mosquitto/data && \
mkdir -m 777 -p ~/mosquitto/log && \
mkdir -m 777 -p ~/influxdb/data && \
mkdir -m 777 -p ~/influxdb/conf && \
mkdir -m 777 -p ~/grafana/data && \
mkdir -m 777 -p ~/grafana/conf && \
mkdir -m 777 -p ~/grafana/log && \
mkdir -m 777 -p ~/node-red/data

2.adım: mosquittonun portunun ayarlanması ve kullanıcı adı şifre ile erişimi

cat > ~/mosquitto/config/mosquitto.conf <<EOF
listener 1883
#allow_anonymous false
#password_file /mosquitto/config/password.txt
EOF

3.adım: grafana'nın portunun ayarlanması ve grafana.ini'ye yazılması

cat > ~/grafana/conf/grafana.ini <<EOF
[server]
http_port = 80
EOF

4.adım: docker-compose.yml dosyasının oluşturulması

cat > docker-compose.yml <<EOF
version: "2"
services:
  influxdb:
    container_name: influxdb
    image: influxdb
    environment:
      - TZ=Europe/Istanbul
    ports:
      - "8086:8086"
    volumes:
      - ~/influxdb/data:/var/lib/influxdb
    networks:
      - influxdb-net
    restart: always
    
  grafana:
    container_name: grafana
    image: grafana/grafana
    environment:
      - TZ=Europe/Istanbul
    ports:
      - "80:80"
    volumes:
      - ~/grafana/data:/var/lib/grafana
      - ~/grafana/log:/var/log/grafana
      - ~/grafana/conf/grafana.ini:/etc/grafana/grafana.ini
    links:
      - influxdb
    networks:
      - grafana-net
    restart: always

  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto:latest
    environment:
      - TZ=Europe/Istanbul
    volumes:
      - ~/mosquitto/:/mosquitto  
    ports:
      - 1883:1883
    networks:
      - mosquitto-net
    restart: always

  node-red:
    container_name: node-red
    image: nodered/node-red:latest
    environment:
      - TZ=Europe/Istanbul
    ports:
      - "1880:1880"
    volumes:
      - ~/node-red/data:/data
    networks:
      - node-red-net
    restart: always

networks:
  node-red-net:
  influxdb-net:
  mosquitto-net:
  grafana-net:
EOF

5.adım:

docker compose up -d

6.adım:

docker exec -it mosquitto mosquitto_passwd -c /mosquitto/config/password.txt IoT

7.adım: moquitto şifre dosyasının oluşturulması

cat > ~/mosquitto/config/mosquitto.conf <<EOF
listener 1883
allow_anonymous false
password_file /mosquitto/config/password.txt
EOF

8.adım:

docker restart mosquitto

9.adım:

docker exec -it node-red node-red-admin hash-pw

10.adım (oluşturulan hash passwordun nore red ayarlarına eklenmesi)

nano +76,5 ~/node-red/data/settings.js

11.adım

docker restart node-red
