---
id: docker
title: Deploying with Docker
---

### Steps:

1. Install SigNoz backend as instructed in this page
2. Instrument your application as instructed in [Instructions Page](/docs/instrumentation/overview)

### Docker Installation Tutorial

1. To clone the SigNoz repository and enter the new directory, run:

```console
git clone https://github.com/SigNoz/signoz.git && cd signoz/deploy/
```

2. **To run SigNoz**:

Check that you are in `signoz/deploy` folder. Now run

```
./install.sh
```

This should install a tiny instance setting which runs with **4GB of memory**. This is just for demo/testing purpose and not to be used in production.

To test if everything is fine, run the following command

```
docker ps
```

Output should look like this ( Should have 14 image names as shown below )

```
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                   PORTS                                                                                                                                                                              NAMES
d23bcc11b54e   signoz/flattener-processor:0.1.1               "./flattener"            2 minutes ago   Up 2 minutes             0.0.0.0:8000->8000/tcp                                                                                                                                                             flattener-processor
3fe13c96c4a5   otel/opentelemetry-collector:0.18.0            "/otelcol --config=/…"   2 minutes ago   Up 2 minutes             0.0.0.0:1777->1777/tcp, 0.0.0.0:14268->14268/tcp, 0.0.0.0:55679->55679/tcp, 0.0.0.0:55681->55681/tcp, 0.0.0.0:8887->8888/tcp, 0.0.0.0:49166->13133/tcp, 0.0.0.0:49165->55678/tcp   docker_otel-collector_1
7fab07ba551b   signoz/frontend:0.1.8                          "nginx -g 'daemon of…"   4 minutes ago   Up 4 minutes             80/tcp, 0.0.0.0:3000->3000/tcp                                                                                                                                                     frontend
fd21e0fb04ee   signoz/query-service:0.1.3                     "./query-service"        4 minutes ago   Up 4 minutes             0.0.0.0:8080->8080/tcp                                                                                                                                                             query-service
485b9a6b1f68   apache/druid:0.20.0                            "/druid.sh router"       4 minutes ago   Up 4 minutes             0.0.0.0:8888->8888/tcp                                                                                                                                                             router
061116e08ef4   apache/druid:0.20.0                            "/druid.sh historical"   4 minutes ago   Up 4 minutes             0.0.0.0:8083->8083/tcp                                                                                                                                                             historical
89100a85e4f2   apache/druid:0.20.0                            "/druid.sh middleMan…"   4 minutes ago   Up 4 minutes             0.0.0.0:8091->8091/tcp                                                                                                                                                             middlemanager
c18b08e8c0c1   apache/druid:0.20.0                            "/druid.sh broker"       4 minutes ago   Up 4 minutes             0.0.0.0:8082->8082/tcp                                                                                                                                                             broker
3536502096f6   apache/druid:0.20.0                            "/druid.sh coordinat…"   4 minutes ago   Up 4 minutes             0.0.0.0:8081->8081/tcp                                                                                                                                                             coordinator
7b22c4fc9aa4   bitnami/kafka:2.7.0-debian-10-r1               "/opt/bitnami/script…"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:9092->9092/tcp                                                                                                                                                             docker_kafka_1
91e375b8c19b   jaegertracing/example-hotrod:latest            "/go/bin/hotrod-linu…"   4 minutes ago   Up 4 minutes             8081-8083/tcp, 0.0.0.0:9000->8080/tcp                                                                                                                                              hotrod
c38e34c3f272   postgres:latest                                "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes             5432/tcp                                                                                                                                                                           postgres
9b656aa85d7d   grubykarol/locust:1.2.3-python3.9-alpine3.12   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes             5557-5558/tcp, 0.0.0.0:8089->8089/tcp                                                                                                                                              load-hotrod
b939dd7dcc47   bitnami/zookeeper:3.6.2-debian-10-r100         "/opt/bitnami/script…"   4 minutes ago   Up 4 minutes             2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, 8080/tcp                                                                                                                               docker_zookeeper_1
```

Once `./install.sh` runs successfully, the UI should be accessible at port 3000 on the domain you set up or the IP of your instance.

:::info
Wait for 2-3 mins for the data to be available to frontend. If you are running on local machine, checkout `http://localhost:3000`.
You would want to open port 3000 to be accessible from outside world if you want to use public url of machine.
:::

:::note
If you face any issues here, don't worry - just check out the [troubleshooting steps](/docs/deployment/docker#troubleshooting) or join our [slack communty](https://join.slack.com/t/signoz-community/shared_invite/zt-lrjknbbp-J_mI13rlw8pGF4EWBnorJA)
:::

### Production Settings

A standard instance of SigNoz needs around **8GB of memory**. The `docker-compose.yaml` file at `deploy/docker/` can handle around 100RPS or 5K events/sec. Email at ankit@signoz.io or join [Slack](https://join.slack.com/t/signoz-community/shared_invite/zt-lrjknbbp-J_mI13rlw8pGF4EWBnorJA) for help in setting this up.

### Troubleshooting of common issues

1. `docker ps` will show all containers created by SigNoz. Check if `broker`, `otel-collector` and `historical` containers are running. They do not come up if there is a memory problem. You may want to increase alloted memory.
2. If you are still facing issues, try re-running `./install.sh`. This will retry installing containers which failed the first time.
3. If you are facing issues like `Request failed with status code 400` in frontend, then open `http://localhost:8888` or port 8888 on your IP .This is druid console. Check if **Datasource** named `flattened_spans` has come up. If there is no **Ingestion Supervsor** running, then run `./install.sh` again to bring them up.
4. If you couldn't spot issues, feel free to join our [slack community](https://join.slack.com/t/signoz-community/shared_invite/zt-lrjknbbp-J_mI13rlw8pGF4EWBnorJA) or shoot an email at ankit@signoz.io. We are generally always there.

### Troubleshooting for Mac users

You need to check the memory allocated to docker. Follow below steps:

a) Choose the Docker menu whale menu > Preferences from the menu bar and configure the runtime options described below.

![Docker Preferences](https://docs.docker.com/docker-for-mac/images/menu/prefs.png)

b) Choose Resources from Preferences Menu and change Memory to 3GB

![Docker Resource Preferences](../../static/img/docker_preferences.png)

### Configure docker-compose.yml

The current `docker-compose.yaml` includes sample application ([HotR.O.D](https://github.com/jaegertracing/jaeger/tree/master/examples/hotrod)) that generates tracing data. To see your own application data, follow the steps below

### How to instrument your own applications

[Checkout Instrumentation Section](/docs/instrumentation/overview)