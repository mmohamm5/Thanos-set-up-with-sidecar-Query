# Thanos Set-Up with Sidecar & Query

## Introduction: Global View and Seamless HA for Prometheus

Thanos is meant to scale and extend vanilla Prometheus. This means that you can gradually, without disruption, deploy Thanos on top of your existing Prometheus setup.

Let's start our tutorial by spinning up three Prometheus servers. Why three? The real advantage of Thanos is when you need to scale out Prometheus from a single replica. Some reasons for scale-out might be:

- Adding functional sharding because of metrics high cardinality
- Need for high availability of Prometheus (e.g., rolling upgrades)
- Aggregating queries from multiple clusters

For this course, let's imagine the following situation:

- We have one Prometheus server in the `eu1` cluster.
- We have 2 replica Prometheus servers in the `us1` cluster that scrape the same targets.

Let's start this initial Prometheus setup.

## Prometheus Configuration Files

Now, we will prepare configuration files for all Prometheus instances.

### **EU Prometheus Server Configuration**

Create `prometheus0_eu1.yml`:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: eu1
    replica: 0

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['172.17.0.1:9090']
```

## US Prometheus Servers Configuration
Create prometheus0_us1.yml:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: us1
    replica: 0

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['172.17.0.1:9091','172.17.0.1:9092']
```

Create prometheus1_us1.yml:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: us1
    replica: 1

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['172.17.0.1:9091','172.17.0.1:9092']
```
NOTE: Every Prometheus instance must have a globally unique set of identifying labels. These labels are important as they represent a certain "stream" of data.

## Starting Prometheus Instances
Now, we will start three Prometheus instances with the following commands:

Prepare "persistent volumes":

```yaml
mkdir -p prometheus0_eu1_data prometheus0_us1_data prometheus1_us1_data
```

## Deploying EU1

```yaml
docker run -d --net=host --rm \
    -v $(pwd)/prometheus0_eu1.yml:/etc/prometheus/prometheus.yml \
    -v $(pwd)/prometheus0_eu1_data:/prometheus \
    -u root \
    --name prometheus-0-eu1 \
    quay.io/prometheus/prometheus:v2.38.0 \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/prometheus \
    --web.listen-address=:9090 \
    --web.enable-lifecycle \
    --web.enable-admin-api && echo "Prometheus EU1 started!"
```
## Deploying US1


```yaml
docker run -d --net=host --rm \
    -v $(pwd)/prometheus0_us1.yml:/etc/prometheus/prometheus.yml \
    -v $(pwd)/prometheus0_us1_data:/prometheus \
    -u root \
    --name prometheus-0-us1 \
    quay.io/prometheus/prometheus:v2.38.0 \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/prometheus \
    --web.listen-address=:9091 \
    --web.enable-lifecycle \
    --web.enable-admin-api && echo "Prometheus 0 US1 started!"
```


```yaml
docker run -d --net=host --rm \
    -v $(pwd)/prometheus1_us1.yml:/etc/prometheus/prometheus.yml \
    -v $(pwd)/prometheus1_us1_data:/prometheus \
    -u root \
    --name prometheus-1-us1 \
    quay.io/prometheus/prometheus:v2.38.0 \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/prometheus \
    --web.listen-address=:9092 \
    --web.enable-lifecycle \
    --web.enable-admin-api && echo "Prometheus 1 US1 started!"

```
## Installing Thanos Sidecar
Each Prometheus instance needs a Thanos sidecar. Deploy the sidecar using the following commands:

```yaml
docker run -d --net=host --rm \
    -v $(pwd)/prometheus0_eu1.yml:/etc/prometheus/prometheus.yml \
    --name prometheus-0-sidecar-eu1 \
    -u root \
    quay.io/thanos/thanos:v0.28.0 \
    sidecar \
    --http-address 0.0.0.0:19090 \
    --grpc-address 0.0.0.0:19190 \
    --reloader.config-file /etc/prometheus/prometheus.yml \
    --prometheus.url http://172.17.0.1:9090 && echo "Started sidecar for Prometheus 0 EU1"
```
Repeat similar steps for US1 Prometheus instances.

## Deploying Thanos Querier
Finally, deploy Thanos Querier to enable global queries:

```yaml
docker run -d --net=host --rm \
    --name querier \
    quay.io/thanos/thanos:v0.28.0 \
    query \
    --http-address 0.0.0.0:29090 \
    --query.replica-label replica \
    --store 172.17.0.1:19190 \
    --store 172.17.0.1:19191 \
    --store 172.17.0.1:19192 && echo "Started Thanos Querier"
```

## Setup Verification
Access Prometheus UI: http://localhost:9090
Access Thanos Query UI: http://localhost:29090
Run sum(prometheus_tsdb_head_series) in Thanos UI to verify global data availability.

## Summary
Set up Prometheus with HA across multiple clusters.
Deployed Thanos Sidecars to connect Prometheus to Thanos.
Deployed Thanos Querier for a unified global view of metrics.
## Resources
Thanos Documentation
Prometheus Documentation
## Contributing
Contributions are welcome! Please open an issue or submit a pull request.

## License
This project is licensed under the MIT License. See the LICENSE file for details.


```yaml

This is your complete `README.md` file formatted for GitHub. Just copy and paste it into your repository. 🚀 Let me know if you need any modifications!

```
