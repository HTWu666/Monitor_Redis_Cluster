# Monitor_Redis_Cluster
## Setup Redis
### Installation
To install Redis on your system, use the following command. This will install the Redis server which is the first step to setting up your Redis Cluster.
```
sudo apt install redis-server
```
### Configuration
Configuring your Redis instance for clustering involves setting several options in the Redis configuration file. 
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
To start a Redis server with the specified configuration file, use the command below. Replace /path/to/your/redis.conf with the actual path to your configuration file.
```
redis-server /path/to/your/redis.conf
```
### Create Redis Cluster
Creating a Redis Cluster involves starting multiple Redis instances configured for clustering and then using the redis-cli command to create the cluster:
```
redis-cli --cluster create <redis_node_1_IP>:<redis_node_1_port> <redis_node_2_IP>:<redis_node_2_port> <redis_node_3_IP>:<redis_node_3_port> <redis_node_4_IP>:<redis_node_4_port> <redis_node_5_IP>:<redis_node_5_port> <redis_node_6_IP>:<redis_node_6_port> --cluster-replicas 1
```
This command initiates a cluster with the specified nodes, ensuring there is one replica for each primary node.
### Check Cluster Status
To verify the status and configuration of your Redis Cluster, use the following commands on any of the cluster nodes:
```
redis-cli -p 7000 CLUSTER NODES
redis-cli -p 7000 CLUSTER INFO
```
These commands provide detailed information about the nodes in the cluster and the cluster's overall status, respectively.
## Setup Redis Exporter
Redis Exporter is a tool that allows you to export Redis metrics to Prometheus. To run Redis Exporter in a Docker container, use the following command:
```
docker run -d --name redis_exporter --network="host" -e REDIS_ADDR=redis://127.0.0.1:<redis_expose_port> oliver006/redis_exporter
```
This command starts the Redis Exporter, making it available for Prometheus to scrape Redis metrics.

## Setup Prometheus
### Configuration
Prometheus needs to be configured to scrape metrics from Redis Exporter. Below is an example configuration that defines multiple scrape jobs for different Redis Exporters:
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
Each job corresponds to a Redis Exporter instance, allowing Prometheus to collect metrics from each part of your Redis Cluster.
### Launch
To start Prometheus with your custom configuration file, use the Docker command below:
```
docker run -d --network monitoring -p 9090:9090 -v /home/huitingwu/garmin/prometheus.yml:/etc/prometheus/prometheus.yml --name prometheus prom/prometheus
```
This command runs Prometheus in a Docker container, using the specified configuration file to scrape metrics.
## Setup Grafana
### Launch
Grafana is a powerful visualization tool for displaying metrics. To start Grafana in a Docker container, use the command:
```
 docker run -d --name grafana --network monitoring -p 3000:3000 -v <path_to_store_grafana_data_on_your_computer>:/var/lib/grafana --user $(id -u):$(id -g) grafana/grafana
```
This makes Grafana available for visualizing the metrics collected by Prometheus.
### Others
After launching Grafana, you'll need to configure it:
1. Choose Prometheus as the data source: This allows Grafana to fetch and display data collected by Prometheus.
2. Setup dashboard: Create or import dashboards in Grafana to visualize Redis metrics in a meaningful way

## Start monitoring
With all components set up, your monitoring system is now operational. Prometheus will collect metrics from Redis via Redis Exporter, and Grafana will visualize these metrics, providing insights into the performance and health of your Redis Cluster.
