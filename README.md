# kubernetes

---

# WordCount & PageRank with PySpark on Google Cloud's Kubernetes

This project guides users through executing WordCount and PageRank algorithms on datasets at scale, leveraging Kubernetes on the Google Cloud Platform (GCP) and Apache Spark.

## Table of Contents
- [Overview](#overview)
- [Design](#design)
  - [WordCount](#wordcount)
  - [PageRank](#pagerank)
- [Setup & Deployment](#setup--deployment)
- [Implementation](#implementation)
  - [WordCount](#wordcount-1)
  - [PageRank](#pagerank-1)
- [Output](#output)

## Overview

### WordCount:
Count occurrences of unique words in a text or document, leveraging Spark's parallel processing on Kubernetes.

### PageRank:
Measure the importance of web pages for search engine ranking by assigning numerical weight based on the quality and quantity of incoming links, again exploiting Spark's distributed computing.

**[Google Slides on WordCount & PageRank](link_to_your_google_slides)**

## Design

### WordCount:
A depiction of how words are counted across multiple nodes can be found [here](image_link).

### PageRank:

The algorithm's steps are as follows:
1. **Initialization:** Every page in the web graph starts with a rank of 1.0.
2. **Iterative Contribution:** On each iteration, each page (p) sends a contribution of its rank divided by its number of neighbors (links).
3. **Rank Update:** After receiving contributions, each page's rank is updated using: `rank = 0.15 + 0.85 * contributionsReceived`.

> Note:
> - The damping factor (0.85) ensures balance, preventing undue emphasis on highly connected pages.
> - More input pages and higher PageRank input pages are beneficial.

## Setup & Deployment

1. **Cloud Shell Activation & GCP Authentication:**
    ```bash
    $ gcloud auth login
    ```

2. **Helm Configuration:**
    ```bash
    $ helm repo add stable https://charts.helm.sh/stable
    $ helm install nfs stable/nfs-server-provisioner \
        --set persistence.enabled=true,persistence.size=5Gi
    ```

3. **Set Up PersistentVolumeClaim (PVC):**
    ```bash
    $ vim spark-pvc.yaml # Create or edit the YAML file.
    $ kubectl apply -f spark-pvc.yaml
    ```

4. **Prepare Spark JAR & Test File:**
    ```bash
    $ docker run -v /tmp:/tmp -it bitnami/spark -- find /opt/bitnami/spark/examples/jars/ -name spark-examples* -exec cp {} /tmp/my.jar \;
    $ echo "how much wood could a woodpecker chuck if a woodpecker could chuck wood" > /tmp/test.txt
    $ kubectl cp /tmp/my.jar spark-data-pod:/data/my.jar
    $ kubectl cp /tmp/test.txt spark-data-pod:/data/test.txt
    ```

5. **Spark Deployment:**
    ```bash
    $ vim spark-chart.yaml # Create or edit the YAML file.
    $ helm repo add bitnami https://charts.bitnami.com/bitnami
    $ helm install spark bitnami/spark -f spark-chart.yaml
    ```

## Implementation

### WordCount:

1. **Job Submission:**
    Replace `<SPARK_MASTER_IP>` with your Spark master IP.
    ```bash
    $ kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
        --image docker.io/bitnami/spark:3.4.1-debian-11-r3 \
        -- spark-submit --master spark://<SPARK_MASTER_IP>:7077 \
        --deploy-mode cluster \
        --class org.apache.spark.examples.JavaWordCount \
        /data/my.jar /data/test.txt
    ```

2. **View Output:**
    Replace `<SPARK_WORKER_IP>` and `<Submission ID>` accordingly.
    ```bash
    $ kubectl get pods -o wide | grep <SPARK_WORKER_IP>
    $ kubectl exec -it spark-worker-1 -- bash
    $ cd /opt/bitnami/spark/work
    $ cat <Submission ID>/stdout
    ```

### PageRank:

1. **Job Submission:**
    ```bash
    $ kubectl exec -it spark-master-0 -- bash
    $ cd /opt/bitnami/spark/examples/src/main/python
    $ spark-submit pagerank.py /opt 2
    ```

## Output

