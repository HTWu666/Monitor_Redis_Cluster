# Monitor_Redis_Cluster
## Setup Redis
### Installation
```
sudo apt install redis-server
```
### Configuration
```
# Set the Redis listening port
port 7000

# Enable Redis Cluster mode
cluster-enabled yes

# Specify the Redis Cluster configuration file name, automatically maintained by Redis, matching the port
cluster-config-file nodes-7000.conf

# Timeout in milliseconds before considering a node as failed
cluster-node-timeout 5000

# Enable AOF persistence mode
appendonly yes

# Set AOF file sync mode to once every second to disk, balancing data safety and performance
appendfsync everysec

# RDB persistence strategy: Generate RDB snapshot if at least 1 key changes in 900 seconds, 10 keys in 300 seconds, or 10000 keys in 60 seconds
save 900 1
save 300 10
save 60 10000
```
### Launch
```
redis-server /path/to/your/redis.conf
```
### Create Redis Cluster
To crate redis cluster, you need to launch at least 6 redis instance and use the following command to create cluster.
```
redis-cli --cluster create <redis_node_1_IP>:<redis_node_1_port> <redis_node_2_IP>:<redis_node_2_port> <redis_node_3_IP>:<redis_node_3_port> <redis_node_4_IP>:<redis_node_4_port> <redis_node_5_IP>:<redis_node_5_port> <redis_node_6_IP>:<redis_node_6_port> --cluster-replicas 1
```
### Check Cluster Status
Choose one of the redis node and use the following command to check the cluster status.
```
redis-cli -p 7000 CLUSTER NODES
redis-cli -p 7000 CLUSTER INFO
```
## Setup Redis Exporter
```
docker run -d --name redis_exporter --network="host" -e REDIS_ADDR=redis://127.0.0.1:<redis_expose_port> oliver006/redis_exporter
```
## Setup Prometheus
### Configuration
```
# file: prometheus.yml

scrape_configs:
  - job_name: 'redis_exporter_1'
    static_configs:
      - targets: ['<redis_exporter_1_IP>:<redis_exporter_1_port>']
  - job_name: 'redis_exporter_2'
    static_configs:
      - targets: ['<redis_exporter_2_IP>:<redis_exporter_2_port>']
  - job_name: 'redis_exporter_3'
    static_configs:
      - targets: ['<redis_exporter_3_IP>:<redis_exporter_3_port>']
  - job_name: 'redis_exporter_4'
    static_configs:
      - targets: ['<redis_exporter_4_IP>:<redis_exporter_4_port>']
  - job_name: 'redis_exporter_5'
    static_configs:
      - targets: ['<redis_exporter_5_IP>:<redis_exporter_5_port>']
  - job_name: 'redis_exporter_6'
    static_configs:
      - targets: ['<redis_exporter_6_IP>:<redis_exporter_6_port>']
```
### Launch
```
docker run -d --network monitoring -p 9090:9090 -v /home/huitingwu/garmin/prometheus.yml:/etc/prometheus/prometheus.yml --name prometheus prom/prometheus
```
## Setup Grafana
### Launch
```
 docker run -d --name grafana --network monitoring -p 3000:3000 -v <path_to_store_grafana_data_on_your_computer>:/var/lib/grafana --user $(id -u):$(id -g) grafana/grafana
```
### Others
1. Choose prometheus as data source
2. Setup dashboard

## Start monitoring
The setup is done.
