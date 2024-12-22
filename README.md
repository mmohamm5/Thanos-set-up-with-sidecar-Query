# Thanos-set-up-with-sidecar-Query
## Initial Prometheus Setup
# Step 1 - Start initial Prometheus servers
Thanos is meant to scale and extend vanilla Prometheus. This means that you can gradually, without disruption, deploy Thanos on top of your existing Prometheus setup.

Let's start our tutorial by spinning up three Prometheus servers. Why three? The real advantage of Thanos is when you need to scale out Prometheus from a single replica. Some reason for scale-out might be:

Adding functional sharding because of metrics high cardinality

Need for high availability of Prometheus e.g: Rolling upgrades

Aggregating queries from multiple clusters

For this course, let's imagine the following situation:
![image](https://github.com/user-attachments/assets/371ef63c-e1c7-4107-b5ad-20754c50402e)


We have one Prometheus server in some eu1 cluster.

We have 2 replica Prometheus servers in some us1 cluster that scrapes the same targets.

Let's start this initial Prometheus setup for now.
# Prometheus Configuration Files
Now, we will prepare configuration files for all Prometheus instances.

Click on the box and it will get copied

Switch on to the Editor tab and make a prometheus0_eu1.yml file and paste the above code in it.

First, for the EU Prometheus server that scrapes itself:
``` global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: eu1
    replica: 0

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['172.17.0.1:9090']
