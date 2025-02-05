---
title: Monitoring your FastAPI application with OpenTelemetry
slug: opentelemetry-fastapi
date: 2024-02-05
tags: [OpenTelemetry Instrumentation, Python]
authors: ankit_anand
description: OpenTelemetry FastAPI client libraries can help you monitor your FastAPI applications for performance issues. In this article, learn how to set up monitoring for FastAPI web framework using OpenTelemetry.
image: /img/blog/2024/02/opentelemetry-fastapi-cover.webp
keywords:
  - opentelemetry
  - opentelemetry python
  - opentelemetry fastapi
  - distributed tracing
  - observability
  - fastapi monitoring
  - fastapi instrumentation
  - signoz
---

import { LiteYoutubeEmbed } from "react-lite-yt-embed";

<head>
  <link rel="canonical" href="https://signoz.io/blog/opentelemetry-fastapi/"/>
</head>

FastAPI is a modern Python web framework based on standard Python type hints that makes it easy to build APIs. It's a relatively new framework, having been released in 2018 but has now been adopted by big companies like Uber, Netflix, and Microsoft. Using OpenTelemetry, you can monitor your FastAPI applications for performance by collecting telemetry signals like traces.

<!--truncate-->

![Cover Image](/img/blog/2024/02/opentelemetry-fastapi-cover.webp)

FastAPI is one of the fastest Python web frameworks currently available and is really efficient when it comes to writing code. It is based on ASGI specification, unlike other Python frameworks like Flask, which is based on WSGI specification.

Instrumentation is the biggest challenge engineering teams face when starting out with monitoring their application performance. <a href = "https://opentelemetry.io/" rel="noopener noreferrer nofollow" target="_blank" >OpenTelemetry</a> is the leading open-source standard that is solving the problem of instrumentation. It is currently an incubating project under the <a href = "https://www.cncf.io/" rel="noopener noreferrer nofollow" target="_blank" >Cloud Native Computing Foundation</a>.

It is a set of tools, APIs, and SDKs used to instrument applications to create and manage telemetry data(Logs, metrics, and traces). It aims to make telemetry data(logs, metrics, and traces) a built-in feature of cloud-native software applications.

 One of the biggest advantages of using OpenTelemetry is that it is vendor-agnostic. It can export data in multiple formats which you can send to a backend of your choice.

In this article, we will use [SigNoz](https://signoz.io/) as a backend. SigNoz is an open-source APM tool that can be used for both metrics and distributed tracing.

Let's get started and see how to use OpenTelemetry for a FastAPI application.

## Running a FastAPI application with OpenTelemetry

OpenTelemetry is a great choice to instrument ASGI frameworks. As it is open-source and vendor-agnostic, the data can be sent to any backend of your choice.

### Setting up SigNoz

You need a backend to which you can send the collected data for monitoring and visualization. [SigNoz](https://signoz.io/) is an OpenTelemetry-native APM that is well-suited for visualizing OpenTelemetry data.

SigNoz cloud is the easiest way to run SigNoz. You can sign up [here](https://signoz.io/teams/) for a free account and get 30 days of unlimited access to all features.

[![Try SigNoz Cloud CTA](/img/blog/2024/01/opentelemetry-collector-try-signoz-cloud-cta.webp)](https://signoz.io/teams/)

You can also install and self-host SigNoz yourself. Check out the [docs](https://signoz.io/docs/install/) for installing self-host SigNoz.



<!-- You can get started with SigNoz using just three commands at your terminal.

```jsx
git clone -b main https://github.com/SigNoz/signoz.git
cd signoz/deploy/
./install.sh
```
<br></br>

For detailed instructions, you can visit our documentation.

[![Deployment Docs](/img/blog/common/deploy_docker_documentation.webp)](https://signoz.io/docs/install/)

When you are done installing SigNoz, you can access the UI at: [http://localhost:3301](http://localhost:3301/application)

The application list shown in the dashboard is from a sample app called HOT R.O.D that comes bundled with the SigNoz installation package.



<figure data-zoomable align='center'>
    <img src="/img/blog/common/signoz_dashboard_homepage.webp" alt="Homepage SigNoz dashboard"/>
    <figcaption><i>SigNoz dashboard - It shows services from a sample app that comes bundled with the application</i></figcaption>
</figure>

<br></br> -->

### Instrumenting a sample FastAPI application with OpenTelemetry

**Prerequisites**<br></br>
Python 3.8 or newer

Download the <a href = "https://www.python.org/downloads/" rel="noopener noreferrer nofollow" target="_blank" >latest version</a> of Python.

**Step 1. Running sample FastAPI app**<br></br>
We will be using the FastAPI app at this <a href = "https://github.com/SigNoz/sample-fastAPI-app" rel="noopener noreferrer nofollow" target="_blank" >Github repo</a>. All the required OpenTelemetry packages are contained within the `requirements.txt` file under `app` folder in this sample app. Go to the `app` folder first.

```bash
git clone https://github.com/SigNoz/sample-fastAPI-app.git
cd sample-fastapi-app/
cd app
```
It’s a good practice to create virtual environments for running Python apps, so we will using a virtual python environment for this sample fastAPI app.

#### Create a Virtual Environment
```bash
python3 -m venv .venv
source .venv/bin/activate
```
<br></br>

**Step 2. Run instructions for sending data to SigNoz**<br></br>
The `requirements.txt` file contains all the necessary OpenTelemetry Python packages needed for instrumentation. In order to install those packages, run the following command:

```bash
python -m pip install -r requirements.txt
```
The dependencies included are briefly explained below:

opentelemetry-distro - The distro provides a mechanism to automatically configure some of the more common options for users. It helps to get started with OpenTelemetry auto-instrumentation quickly.

opentelemetry-exporter-otlp - This library provides a way to install all OTLP exporters. You will need an exporter to send the data to SigNoz.

:::note
💡 The `opentelemetry-exporter-otlp` is a convenience wrapper package to install all OTLP exporters. Currently, it installs:

- opentelemetry-exporter-otlp-proto-http

- opentelemetry-exporter-otlp-proto-grpc

- (soon) opentelemetry-exporter-otlp-json-http

The `opentelemetry-exporter-otlp-proto-grpc` package installs the gRPC exporter which depends on the `grpcio` package. The installation of `grpcio` may fail on some platforms for various reasons. If you run into such issues, or you don't want to use gRPC, you can install the HTTP exporter instead by installing the `opentelemetry-exporter-otlp-proto-http` package. You need to set the `OTEL_EXPORTER_OTLP_PROTOCOL` environment variable to `http/protobuf` to use the HTTP exporter.
:::

<br></br>

**Step 3. Install application specific packages**<br></br>
This step is required to install packages specific to the application. This command figures out which instrumentation packages the user might want to install and installs it for them:

```bash
opentelemetry-bootstrap --action=install
```
<br></br>

:::note
Please make sure that you have installed all the dependencies of your application before running the above command. The command will not install instrumentation for the dependencies which are not installed.
:::

**Step 4. Configure environment variables to run app and send data to SigNoz**<br></br>
You're almost done. In the last step, you just need to configure a few environment variables for your OTLP exporters. Environment variables that need to be configured:


```bash
OTEL_RESOURCE_ATTRIBUTES=service.name=<service_name> \
OTEL_EXPORTER_OTLP_ENDPOINT="https://ingest.{region}.signoz.cloud:443" \
OTEL_EXPORTER_OTLP_HEADERS="signoz-access-token=SIGNOZ_INGESTION_KEY" \
OTEL_EXPORTER_OTLP_PROTOCOL=grpc \
opentelemetry-instrument <your_run_command>
```

- <service_name> is the name of the service you want
- <your_run_command> can be python3 app.py or python manage.py runserver --noreload
- Replace SIGNOZ_INGESTION_KEY with the api token provided by SigNoz. You can find it in the email sent by SigNoz with your cloud account details.

You will be able to get ingestion details in SigNoz cloud account under settings --> ingestion settings.

<figure data-zoomable align='center'>
    <img src="/img/blog/common/ingestion-key-details.webp" alt="Ingestion key details"/>
    <figcaption><i>Ingestion details in SigNoz dashboard</i></figcaption>
</figure>

<br></br>

:::note
Don’t run app in reloader/hot-reload mode as it breaks instrumentation. For example, if you use `--reload` or `reload=True`, it enables the reloader mode which breaks OpenTelemetry isntrumentation.
:::

For our sample FastAPI application, the run command will look like:

```bash
OTEL_RESOURCE_ATTRIBUTES=service.name=sample-fastapi-app \
OTEL_EXPORTER_OTLP_ENDPOINT="https://ingest.{region}.signoz.cloud:443" \
OTEL_EXPORTER_OTLP_HEADERS="signoz-access-token=SIGNOZ_INGESTION_KEY" \
OTEL_EXPORTER_OTLP_PROTOCOL=grpc \
opentelemetry-instrument uvicorn main:app --host localhost --port 5002
```

:::note

The uvicorn run command with multiple workers has yet to be supported. Alternatively, you can use gunicorn with the worker class `uvicorn.workers.Uvicorn[H11]Worker`

:::

In that case, the final command will be:

```bash
OTEL_RESOURCE_ATTRIBUTES=service.name=sample-fastapi-app \
OTEL_EXPORTER_OTLP_ENDPOINT="https://ingest.{region}.signoz.cloud:443" \
OTEL_EXPORTER_OTLP_HEADERS="signoz-access-token=SIGNOZ_INGESTION_KEY" \
OTEL_EXPORTER_OTLP_PROTOCOL=grpc \
opentelemetry-instrument gunicorn main:app --workers 2 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

And, congratulations! You have instrumented your sample FastAPI app. You can check if your app is running or not by hitting the endpoint at [http://localhost:5002/](http://localhost:5002/).


You need to generate some load on your app so that there is data to be captured by OpenTelemetry. You can use locust for this load testing.

```bash
pip3 install locust
```

<br></br>

```bash
locust -f locust.py --headless --users 10 --spawn-rate 1 -H http://localhost:5002
```
<br></br>

Or you can also hit the endpoint [http://localhost:5002/](http://localhost:5002/) manually to generate some load. 

You will find `sample-fastapi-app` in the list of sample applications being monitored by SigNoz.


<figure data-zoomable align='center'>
    <img src="/img/blog/2021/11/list_of_apps_fastapi.webp" alt="FastAPI in the list of applications"/>
    <figcaption><i>FastAPI in the list of applications being monitored by SigNoz</i></figcaption>
</figure>

<br></br>

If you want to run the application with a docker image, refer to the section below for instructions.

### Run with docker
You can use the below instructions if you want to run your app as a docker image, below are the instructions.<br></br>

**Build docker image**
```jsx
docker build -t sample-fastapi-app .
```
<br></br>

**Setting environment variables**<br></br>
You need to set some environment variables while running the application with OpenTelemetry and send collected data to SigNoz. You can do so with the following commands at the terminal:

```bash
# If you have your SigNoz IP Address, replace <IP of SigNoz> with your IP Address. 

docker run -d --name fastapi-container \
-e OTEL_METRICS_EXPORTER='none' \
-e OTEL_RESOURCE_ATTRIBUTES='service.name=fastapiApp' \
-e OTEL_EXPORTER_OTLP_ENDPOINT="https://ingest.{region}.signoz.cloud:443" \
-e OTEL_EXPORTER_OTLP_HEADERS="signoz-access-token=SIGNOZ_INGESTION_KEY" \
-e OTEL_EXPORTER_OTLP_PROTOCOL=grpc \
-p 5002:5002 sample-fastapi-app
```
<br></br>

If you're using docker compose setup:

```bash
# If you are running signoz through official docker compose setup, run `docker network ls` and find clickhouse network id. It will be something like this clickhouse-setup_default 
# and pass network id by using --net <network ID>

docker run -d --name fastapi-container \ 
--net clickhouse-setup_default  \ 
--link clickhouse-setup_otel-collector_1 \
-e OTEL_METRICS_EXPORTER='none' \
-e OTEL_RESOURCE_ATTRIBUTES='service.name=fastapiApp' \
-e OTEL_EXPORTER_OTLP_ENDPOINT="https://ingest.{region}.signoz.cloud:443" \
-e OTEL_EXPORTER_OTLP_HEADERS="signoz-access-token=SIGNOZ_INGESTION_KEY" \
-e OTEL_EXPORTER_OTLP_PROTOCOL=grpc \
-p 5002:5002 sample-fastapi-app
```

<br></br>


## Monitor FastAPI application with SigNoz

SigNoz makes it easy to visualize metrics and traces captured through OpenTelemetry instrumentation.

SigNoz comes with out of box RED metrics charts and visualization. RED metrics stands for:

- Rate of requests
- Error rate of requests
- Duration taken by requests


<figure data-zoomable align='center'>
    <img src="/img/blog/common/signoz_charts_application_metrics.webp" alt="SigNoz charts and metrics"/>
    <figcaption><i>Measure things like application latency, requests per sec, error percentage and see your top endpoints with SigNoz</i></figcaption>
</figure>

<br></br>

You can then choose a particular timestamp where latency is high to drill down to traces around that timestamp.

<br></br>

<figure data-zoomable align='center'>
    <img src="/img/blog/common/trace_filter_apply_aggregates.webp" alt="List of traces on SigNoz dashboard"/>
    <figcaption><i>View of traces at a particular timestamp</i></figcaption>
</figure>

<br></br>

You can use flamegraphs to exactly identify the issue causing the latency.

<br></br>

<figure data-zoomable align='center'>
    <img src="/img/blog/common/signoz_flamegraphs.webp" alt="Flamegraphs used to visualize spans of distributed tracing in SigNoz UI"/>
    <figcaption><i>View of traces at a particular timestamp</i></figcaption>
</figure>

<br></br>

You can also build custom metrics dashboard for your infrastructure.

<br></br>

<figure data-zoomable align='center'>
    <img src="/img/blog/common/signoz_custom_dashboard-min.webp" alt="Custom metrics dashboard"/>
    <figcaption><i>You can also build a custom metrics dashboard for your infrastructure</i></figcaption>
</figure>

<br></br>

## Conclusion
OpenTelemetry makes it very convenient to instrument your FastAPI application. You can then use an open-source APM tool like SigNoz to analyze the performance of your app. As SigNoz offers a full-stack observability tool, you don't have to use multiple tools for your monitoring needs.

You can try out SigNoz by visiting its GitHub repo 👇

[![SigNoz GitHub repo](/img/blog/common/signoz_github.webp)](https://github.com/SigNoz/signoz)

If you have any questions or need any help in setting things up, join our slack community and ping us in `#support` channel.

[![SigNoz Slack community](/img/blog/common/join_slack_cta.webp)](https://signoz.io/slack)

---

Read more about OpenTelemetry 👇

[Things you need to know about OpenTelemetry tracing](https://signoz.io/blog/opentelemetry-tracing/)

[OpenTelemetry collector - architecture and configuration guide](https://signoz.io/blog/opentelemetry-collector-complete-guide/)

