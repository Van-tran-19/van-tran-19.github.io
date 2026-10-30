---
title: "Building a Small ChatGPT Clone: Phase 1 - Deploying an Echo Bot Across IaaS, PaaS, and Serverless"
date: 2026-10-29 17:00:00 +0200
categories: [CloudComputing, phase1]
tags: [GCP, CloudRun, CloudFunctions, ComputeEngine, Docker, Flask, DevOps, Serverless]
toc: true
---
# Deploying an Echo Bot Across IaaS, PaaS, and Serverless

The goal is to deploy one tiny web service three ways

- on an IaaS VM
- on a container PaaS (Cloud Run)
- on a serverless function (Cloud Functions)

and measure the trade-offs first-hand (image size, cold start, scaling, cost, effort).

I will create a small Flask web application called an echo bot. The application has one endpoint:

```
GET /echo?msg=hello
```

**What is this?** 

This is a URL request sent to the web application. It contains 3 parts: 

- `GET` is a an HTTP method that retrieves information.
- `echo` which is a function inside the flask app that will handle the request.
- `?msg=hello` which is the query parameter.

If a user sends the message `"hello"`, the application returns:

```
{
  "echo": "hello",
  "length": 5
}
```

## Key concepts

#### **What is Docker multi-stage builds?**

A Docker multi-stage build uses multiple build stages in a single Dockerfile so that you can build an application with all necessary tools in one stage and copy only the final artefacts into a small, lightweight production image. 

The main goal is to create a smaller, cleaner production image.

Without multi-stage builds, your final image may contain:

- Build tools
- Compilers
- Development dependencies
- Temporary files
- Cache files

With multi-stage builds, these stay in the build stage and are not included in the final image. Only the application and its runtime are copied into the final image.

Without multi-stage: 

```
Docker Image
│
├── Python
├── Flask
├── Build tools
├── Compiler
├── Cache
├── Temporary files
└── Your app
```

With multi-stage: 

```
Build Stage
│
├── Python
├── Build tools
├── Compiler
└── Application build

        ↓

Production Stage
│
├── Python Runtime
├── Flask
└── Your app
```

#### **Why gunicorn for Flask in a container?**

[Gunicorn](https://gunicorn.org/) is a pure Python WSGI server with simple configuration and multiple worker implementations for performance tuning.

- It tends to integrate easily with hosting platforms.
- It does not support Windows (but does run on WSL).
- It is easy to install as it does not require additional dependencies or compilation.
- It has built-in async worker support using gevent.

WSGI stands for Web Server Gateway Interface, it is a simple calling convention for web servers to forward requests to web applications or frameworks written in the Python programming language. 

Gunicorn stands for Green Unicorn. Instead of: 

```bash
python3 app.py
#we have: 
gunicorn app:app #where app = Flask(__name__)
```

Gunicorn handles multiple requests, it is more robust, better for cloud run. Cloud Run expects a process that:

- starts quickly
- listens on a port
- handles concurrent requests

Gunicorn is commonly used because it fits this model perfectly and it’s an industry standard. 

#### **What is Cloud Run?**

Cloud Run is Google's fully managed serverless platform for running containerised applications. You deploy a container or source code, and Google automatically handles infrastructure, scaling, networking, and HTTPS, while allowing services to scale from zero and charging primarily when the application is actually running.

#### **What are Cloud functions?**

Cloud Functions is Google's Function-as-a-Service (FaaS) platform. In its 2nd generation, functions are deployed as Cloud Run services, giving developers a fully managed serverless environment with automatic scaling, higher concurrency, longer execution times, more event sources, and improved performance compared with 1st generation Cloud Functions.

#### **What is artifact registry?**

Artifact Registry is Google Cloud's managed repository service for storing and managing container images and software packages, serving as the bridge between building an application and deploying it to services such as Cloud Run. 

Google Artifact Registry supports multiple package formats, including:

- Docker/OCI container images
- Python packages
- Maven packages (Java)
- npm packages (JavaScript)
- Apt packages
- Yum packages

This makes it a general-purpose artefact repository, not just a container registry.

```bash
Source Code
      ↓
Docker Build
      ↓
Artifact Registry
      ↓
Cloud Run
```

Without a registry, Cloud Run would have nowhere to pull the container image from.

#### **What is a cold start?**

A cold start is the extra latency that occurs when Cloud Run must create and initialise a new container instance before processing a request because no ready instance is available. 

If a user sends an HTTP request, Cloud Run must:

1. Start a container instance
2. Start Python
3. Start Flask/Gunicorn
4. Load your application
5. Process the request

This amount of time is called a cold start. 

#### **What is compute Engine VMs?**

Compute Engine is the service that lets you create and run Virtual Machines (VMs) on Google's infrastructure. A VM is essentially a computer running in Google's data centres that you control: you choose the operating system, install software, run applications, and manage it like a Linux server.

## Practise

### Starting

At the beginning, I will run this command: 

```bash
export PROJECT=$(gcloud config get-value project)
export REGION=us-central1
export ZONE=us-central1-a
export IMAGE=$REGION-docker.pkg.dev/$PROJECT/eurecomgpt/echo-bot:v1
#Output: 
Your active configuration is: [cloudshell-15310]
```

**What each line does?** 

- `export PROJECT=$(gcloud config get-value project)`  retrieves the active GCP project from your gcloud configuration
- `export REGION=us-central1` defines the region I will use
- `export ZONE=us-central1-a` defines the availability inside that region because creation of VM in that region is free
- `export IMAGE=$REGION-docker.pkg.dev/$PROJECT/eurecomgpt/echo-bot:v1` creates the full Artifact Registry path that will later store your Docker image

### **Task 1 — Enable the APIs**

Artifact Registry, Cloud Run, Cloud Functions, Cloud Build and Compute Engine are all enabled on the project.

```bash
gcloud services enable \
    artifactregistry.googleapis.com \
    run.googleapis.com \
    cloudfunctions.googleapis.com \
    cloudbuild.googleapis.com \
    compute.googleapis.com
#output: 
Operation "operations/acf.p2-89248238899-59b4de2e-4b2e-4f51-95e7-752f7b41b239" finished successfully.
```

After running it, Google makes these services available for the project. 

### **Task 2 — Implement the code and run the unit tests**

Before getting hands dirty into code, I need a quick learning phase on Flask. 

**What is flask?** 

Flask is a lightweight, open-source Python micro web framework. It gives developers the essential tools to build web applications and APIs quickly, relying on a minimalist core while letting you choose your own databases and libraries. 

**What a route is?**

A route maps a URL to a Python function. 

```python
@app.route("/echo")
def echo():
    return "hello"
```

When someone visits [`http://localhost:8080/echo`](http://localhost:8080/echo) , flask runs `echo()` . 

**What are query parameters (request.args)?**

Query parameters are small pieces of data added to a URL to send information to a web application. The general format is `?key=value` . You can combine several queries at the same time by adding new key value: `?key=value&key=value` . 

**What is a JSON request?** 

A JSON request is a request where the client sends data to the server in JSON format.

| JSON request | what |
| --- | --- |
| GET | `request.args.get("msg")` , it’s used to retrieve information.  |
| POST | Instead of putting the data in the URL, you put it in the **request body**. |

**What is JSON response (jsonify)?** 

`jsonify` is a Flask function that converts Python data into a proper JSON HTTP response. Inside Python, that's a dictionary, not JSON. If you want to send it back through the web API, Flask needs to:

1. Convert it to JSON text
2. Set the correct HTTP header:

`jsonify` does both automatically.

This is my `app.py` : 

```python
#!/usr/bin/env python3
"""EurecomGPT echo bot — a ~50-line Flask service.
"""

from __future__ import annotations
from flask import Flask, jsonify, request
#Flask creates the web app, request contains everything the client sent
app = Flask(__name__)
#this part is acutally the creation of the app

# --------------------------------------------------------------------------- #
# Functions YOU implement (pure — no Flask needed; unit-tested offline)
# --------------------------------------------------------------------------- #
def build_echo(message: str) -> dict:
    return {"echo": message, "length": len(message)}

def extract_message(args, json_body) -> str | None:
    """Pull the user's message out of a request.

    Accept either a query parameter ``msg`` (GET) or a JSON body ``{"message": ...}``
    (POST). Return the string, or ``None`` if neither is present.

    Args:
        args: request.args (a mapping with ``.get``), e.g. ``{"msg": "hi"}``.
        json_body: parsed JSON body or ``None``, e.g. ``{"message": "hi"}``.
    """
    # TODO: return args["msg"] if present, else json_body["message"] if present,
    #       else None. Use .get() so missing keys don't raise.
    if args.get("msg") is not None or json_body != None and json_body.get("message") is not None:
        return args.get("msg") or json_body.get("message")
    else: 
        return None
    

# --------------------------------------------------------------------------- #
# Routes (provided — they call the functions above)
# --------------------------------------------------------------------------- #
@app.get("/")
def index():
    return jsonify(
        {
            "service": "eurecomgpt-echo",
            "usage": "GET /echo?msg=hello  or  POST /echo {\"message\": \"hello\"}",
        }
    )

@app.get("/health")
def health():
    return "ok", 200
#which means that if someone visits health, the respnse will be 'ok'

@app.route("/echo", methods=["GET", "POST"])
def echo():
    message = extract_message(request.args, request.get_json(silent=True))
    if message is None:
        return jsonify({"error": "provide ?msg=... or JSON {\"message\": ...}"}), 400
    return jsonify(build_echo(message))

if __name__ == "__main__":
    # Local dev server only; in the container we run gunicorn (see Dockerfile).
    import os

    app.run(host="0.0.0.0", port=int(os.environ.get("PORT", 8080)))

```

During this task, my mission is to write a small function in `app.py` .  This flask app is an echo API. The user sends a message `hello`  and the API returns 

```python
{
  "echo": "hello",
  "length": 5
}
```

#### Learning

```python
args = {"msg": "hello"}
args.get("msg")
#output : "hello"
```

One mistake: `None.get(””)` is invalid. 

The second TODO of this section is to complete my docker: 

```docker
### Docker file
# Multi-stage build for the echo bot.
#   Stage 1 (builder) installs the Python deps into a throwaway layer.
#   Stage 2 (runtime) is a slim image that copies only the installed packages + app.
# The two stages keep the final image small (target < 400 MB) — inspect it with
# `docker images` and discuss the size in your report.
#
# Two lines are left for YOU to complete (search for TODO).

# ---- Stage 1: builder ----
FROM python:3.11-slim AS builder
#Starts a temporary environment based on Python 3.11.
WORKDIR /app
COPY requirements.txt .
#Docker needs to install the required Python packages.
# TODO: install the requirements into the prefix /install so stage 2 can copy them.
#       Hint: pip install --no-cache-dir --prefix=/install -r requirements.txt

# ---- Stage 2: runtime ----
FROM python:3.11-slim
#This is the image that will actually be deployed.
WORKDIR /app
# Bring in the packages installed in the builder (site-packages + console scripts).
COPY --from=builder /install /usr/local
COPY app.py .
ENV PORT=8080
EXPOSE 8080
# TODO: set the CMD to launch gunicorn, binding 0.0.0.0:8080 and serving "app:app". Use 1 worker and 8 threads.
#       Use the JSON/exec form: CMD ["gunicorn", "--bind", "0.0.0.0:8080", ...]

```

**What is a dockerfile?** 

A Dockerfile is simply a recipe for creating a Docker image.

**What is NOT copied?**

`Caches, Temporary files, Build files` which keep the image smaller. 

**What is Builder stage?** 

Its only purpose is to prepare the application, it prepares dependencies. 

**First TODO**

A Dockerfile command that executes a shell command is `RUN` . The hint gives almost everything, my proposition is : 

```docker
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
```

**Second TODO**

Let’s first decode the requirement of stage 2. 

**What is CMD?** 

`CMD` tells Docker:

> "When this container starts, run this command." which means that `python3 app.py` runs automatically when I run `docker run my-image` .  We can see that at the bottom of app.py: they have:
> 

```python
if __name__ == "__main__":
    app.run(...)
```

It lets you run the app, but this command is useful for development stage, not for production. Instead of this command, we will use gunicorn. 

**What does app:app mean?** 

Gunicorn expects `module:object`. This is actually: 

```python
app.py   → module = app
app = Flask(...) → object = app
```

**What does `0.0.0.0:8080` mean?** 

Binding means listening for incoming requests on port 8080 and `0.0.0.0` means listening on all network interfaces because without this, Cloud Run and VM won't be able to reach the service.

**What are workers?** 

Workers are processes that gunicorn can start. 

```docker
gunicorn --workers 4
#will create
Worker 1
Worker 2
Worker 3
Worker 4
```

**What are threads?** 

8 threads means that this single worker can handle multiple requests concurrently.

While workers are like cookers in restaurants, threads will be the hands workers to execute task. 

**What is JSON/exec form?** 

It’s just `CMD ["something", "arg1", "arg2"]` and not `CMD gunicorn …` . 

**Issues**

`RUN` = execute something when building the image

`CMD` = execute something when starting the container

**Proposition for the TODO**

```docker
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "--workers" ,"1", "--threads", "8", "app:app"]
```

**How can I know the correct order for each attributes?** 

By running `gunicorn --help` . Attributes are started with options and are finished with values. 

```python
program  option      value
gunicorn --workers   1
```

**Test of the server flask**

By running this command: 

```bash
python phase-1-echo-bot/app.py 
#output: 
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8080
 * Running on http://10.88.0.4:8080
Press CTRL+C to quit
127.0.0.1 - - [28/Oct/2026 09:17:37] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [28/Oct/2026 09:17:37] "GET /favicon.ico HTTP/1.1" 404 -
```

<img width="959" height="77" alt="image" src="https://github.com/user-attachments/assets/9e19125e-fd4e-4c36-8ff0-eb855bdd8d08" />

I see the server flask. 

### **Task 3 — Build the image and push it to Artifact Registry**

Objective. A Docker repository named `eurecomgpt` in `$REGION`, containing your image built from this phase's `Dockerfile` and tagged `$IMAGE`. 

I will run this command first: 

```bash
gcloud artifacts docker images list $REGION-docker.pkg.dev/$PROJECT/eurecomgpt
#output: 
(.venv) gabriel_lapointe@cloudshell:~/cloud (project-54f8e752-5266-4cfd-9ff)$ gcloud artifacts docker images list $REGION-docker.pkg.dev/$PROJECT/eurecomgpt
ERROR: (gcloud.artifacts.docker.images.list) NOT_FOUND: Requested entity was not found. This command is authenticated as gabriel.lapointe@eurecom.fr which is the active account specified by the [core/account] property.
(.venv) gabriel_lapointe@cloudshell:~/cloud (project-54f8e752-5266-4cfd-9ff)$ 
```

The repository doesn’t exist yet. Before listing images, I must create a repo. So, first I will check if eurecomgpt exists: 

```bash
gcloud artifacts repositories list --location=$REGION
#output: 
Listing items under project project-54f8e752-5266-4cfd-9ff, location us-central1.
Listed 0 items.
#eurecomgpt doesn't exist, I must create it: 
gcloud artifacts repositories create eurecomgpt \
    --repository-format=docker \
    --location=$REGION
#output: 
Create request issued for: [eurecomgpt]
Waiting for operation [projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/operations/a3b19451-61ef-4c1a-872f-50d0a7eca088] to complete...done.
Created repository [eurecomgpt].
```

Now, lets configure docker authentication. Right now, docker can build images locally but Docker does not automatically have permission to upload images to Google Cloud.

```bash
gcloud auth configure-docker us-central1-docker.pkg.dev
#output
WARNING: Your config file at [/home/gabriel_lapointe/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud",
    "africa-south1-docker.pkg.dev": "gcloud",
    "asia-docker.pkg.dev": "gcloud",
    "asia-east1-docker.pkg.dev": "gcloud",
    "asia-east2-docker.pkg.dev": "gcloud",
    "asia-northeast1-docker.pkg.dev": "gcloud",
    "asia-northeast2-docker.pkg.dev": "gcloud",
    "asia-northeast3-docker.pkg.dev": "gcloud",
    "asia-south1-docker.pkg.dev": "gcloud",
    "asia-south2-docker.pkg.dev": "gcloud",
    "asia-southeast1-docker.pkg.dev": "gcloud",
    "asia-southeast2-docker.pkg.dev": "gcloud",
    "asia-southeast3-docker.pkg.dev": "gcloud",
    "australia-southeast1-docker.pkg.dev": "gcloud",
    "australia-southeast2-docker.pkg.dev": "gcloud",
    "docker.africa-south1.rep.pkg.dev": "gcloud",
    "docker.asia-east1.rep.pkg.dev": "gcloud",
    "docker.asia-east2.rep.pkg.dev": "gcloud",
    "docker.asia-northeast1.rep.pkg.dev": "gcloud",
    "docker.asia-northeast2.rep.pkg.dev": "gcloud",
    "docker.asia-northeast3.rep.pkg.dev": "gcloud",
    "docker.asia-south1.rep.pkg.dev": "gcloud",
    "docker.asia-south2.rep.pkg.dev": "gcloud",
    "docker.asia-southeast1.rep.pkg.dev": "gcloud",
    "docker.asia-southeast2.rep.pkg.dev": "gcloud",
    "docker.australia-southeast1.rep.pkg.dev": "gcloud",
    "docker.australia-southeast2.rep.pkg.dev": "gcloud",
    "docker.europe-central2.rep.pkg.dev": "gcloud",
    "docker.europe-north1.rep.pkg.dev": "gcloud",
    "docker.europe-southwest1.rep.pkg.dev": "gcloud",
    "docker.europe-west1.rep.pkg.dev": "gcloud",
    "docker.europe-west10.rep.pkg.dev": "gcloud",
    "docker.europe-west12.rep.pkg.dev": "gcloud",
    "docker.europe-west2.rep.pkg.dev": "gcloud",
    "docker.europe-west3.rep.pkg.dev": "gcloud",
    "docker.europe-west4.rep.pkg.dev": "gcloud",
    "docker.europe-west6.rep.pkg.dev": "gcloud",
    "docker.europe-west8.rep.pkg.dev": "gcloud",
    "docker.europe-west9.rep.pkg.dev": "gcloud",
    "docker.me-central1.rep.pkg.dev": "gcloud",
    "docker.me-central2.rep.pkg.dev": "gcloud",
    "docker.me-west1.rep.pkg.dev": "gcloud",
    "docker.northamerica-northeast1.rep.pkg.dev": "gcloud",
    "docker.northamerica-northeast2.rep.pkg.dev": "gcloud",
    "docker.southamerica-east1.rep.pkg.dev": "gcloud",
    "docker.southamerica-west1.rep.pkg.dev": "gcloud",
    "docker.us-central1.rep.pkg.dev": "gcloud",
    "docker.us-central2.rep.pkg.dev": "gcloud",
    "docker.us-east1.rep.pkg.dev": "gcloud",
    "docker.us-east4.rep.pkg.dev": "gcloud",
    "docker.us-east5.rep.pkg.dev": "gcloud",
    "docker.us-east7.rep.pkg.dev": "gcloud",
    "docker.us-south1.rep.pkg.dev": "gcloud",
    "docker.us-west1.rep.pkg.dev": "gcloud",
    "docker.us-west2.rep.pkg.dev": "gcloud",
    "docker.us-west3.rep.pkg.dev": "gcloud",
    "docker.us-west4.rep.pkg.dev": "gcloud",
    "europe-central2-docker.pkg.dev": "gcloud",
    "europe-docker.pkg.dev": "gcloud",
    "europe-north1-docker.pkg.dev": "gcloud",
    "europe-north2-docker.pkg.dev": "gcloud",
    "europe-southwest1-docker.pkg.dev": "gcloud",
    "europe-west1-docker.pkg.dev": "gcloud",
    "europe-west10-docker.pkg.dev": "gcloud",
    "europe-west12-docker.pkg.dev": "gcloud",
    "europe-west2-docker.pkg.dev": "gcloud",
    "europe-west3-docker.pkg.dev": "gcloud",
    "europe-west4-docker.pkg.dev": "gcloud",
    "europe-west6-docker.pkg.dev": "gcloud",
    "europe-west8-docker.pkg.dev": "gcloud",
    "europe-west9-docker.pkg.dev": "gcloud",
    "me-central1-docker.pkg.dev": "gcloud",
    "me-central2-docker.pkg.dev": "gcloud",
    "me-west1-docker.pkg.dev": "gcloud",
    "northamerica-northeast1-docker.pkg.dev": "gcloud",
    "northamerica-northeast2-docker.pkg.dev": "gcloud",
    "northamerica-south1-docker.pkg.dev": "gcloud",
    "southamerica-east1-docker.pkg.dev": "gcloud",
    "southamerica-west1-docker.pkg.dev": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud",
    "us-central2-docker.pkg.dev": "gcloud",
    "us-docker.pkg.dev": "gcloud",
    "us-east1-docker.pkg.dev": "gcloud",
    "us-east4-docker.pkg.dev": "gcloud",
    "us-east5-docker.pkg.dev": "gcloud",
    "us-east7-docker.pkg.dev": "gcloud",
    "us-south1-docker.pkg.dev": "gcloud",
    "us-west1-docker.pkg.dev": "gcloud",
    "us-west2-docker.pkg.dev": "gcloud",
    "us-west3-docker.pkg.dev": "gcloud",
    "us-west4-docker.pkg.dev": "gcloud",
    "us-west8-docker.pkg.dev": "gcloud"
  }
}
Adding credentials for: us-central1-docker.pkg.dev
gcloud credential helpers already registered correctly.
#Docker now is well configured
```

Now, let’s build the docker image. 

```bash
docker build \
  -f phase-1-echo-bot/Dockerfile \
  -t $IMAGE \
  phase-1-echo-bot
```

The command to create an image is `docker build` . If I need help about docker build, I can run this command `docker build —help`

```bash
Usage:  docker buildx build [OPTIONS] PATH | URL | -

Start a build

Aliases:
  docker build, docker builder build, docker image build, docker buildx b

Options:
      --add-host strings              Add a custom host-to-IP mapping (format: "host:ip")
      --allow stringArray             Allow extra privileged entitlement (e.g., "network.host", "security.insecure", "device", "buildx.local.delete")
      --annotation stringArray        Add annotation to the image
      --attest stringArray            Attestation parameters (format: "type=sbom,generator=image")
      --build-arg stringArray         Set build-time variables
      --build-context stringArray     Additional build contexts (e.g., name=path)
      --builder string                Override the configured builder instance (default "default")
      --cache-from stringArray        External cache sources (e.g., "user/app:cache", "type=local,src=path/to/dir")
      --cache-to stringArray          Cache export destinations (e.g., "user/app:cache", "type=local,dest=path/to/dir")
      --call string                   Set method for evaluating build ("check", "outline", "targets") (default "build")
      --cgroup-parent string          Set the parent cgroup for the "RUN" instructions during build
      --check                         Shorthand for "--call=check"
  -D, --debug                         Enable debug logging
  -f, --file string                   Name of the Dockerfile (default: "PATH/Dockerfile")
      --iidfile string                Write the image ID to a file
      --label stringArray             Set metadata for an image
      --load                          Shorthand for "--output=type=docker"
      --metadata-file string          Write build result metadata to a file
      --network string                Set the networking mode for the "RUN" instructions during build (default "default")
      --no-cache                      Do not use cache when building the image
      --no-cache-filter stringArray   Do not cache specified stages
  -o, --output stringArray            Output destination (format: "type=local,dest=path")
      --platform stringArray          Set target platform for build
      --policy stringArray            Policy configuration (format:
                                      "filename=path[,filename=path][,reset=true|false][,disabled=true|false][,strict=true|false][,log-level=level]")
      --progress string               Set type of progress output ("auto", "none",  "plain", "quiet", "rawjson", "tty"). Use plain to show container
                                      output (default "auto")
      --provenance string             Shorthand for "--attest=type=provenance"
      --pull                          Always attempt to pull all referenced images
      --push                          Shorthand for "--output=type=registry,unpack=false"
  -q, --quiet                         Suppress the build output and print image ID on success
      --resource stringArray          Resource limits for build containers (format: "memory=2g", "cpu-quota=50000")
      --sbom string                   Shorthand for "--attest=type=sbom"
      --secret stringArray            Secret to expose to the build (format: "id=mysecret[,src=/local/secret]")
      --shm-size bytes                Shared memory size for build containers
      --ssh stringArray               SSH agent socket or keys to expose to the build (format: "default|<id>[=<socket>|<key>[,<key>]]")
  -t, --tag stringArray               Image identifier (format: "[registry/]repository[:tag]")
      --target string                 Set the target build stage to build
      --ulimit ulimit                 Ulimit options (default [])
```

To build an image, docker needs a dockerfile (-f), an image tag (-t) and finally a build context (our current echo-bot). 

```bash
docker build \
  -f phase-1-echo-bot/Dockerfile \
  -t "$IMAGE" \
  phase-1-echo-bot
#output: 
[+] Building 14.8s (11/11) FINISHED                                                                                                            docker:default
 => [internal] load build definition from Dockerfile                                                                                                     0.1s
 => => transferring dockerfile: 1.31kB                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim                                                                                      0.3s
 => [internal] load .dockerignore                                                                                                                        0.1s
 => => transferring context: 215B                                                                                                                        0.0s
 => [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:e9e0bd902ae57c30d3ffdb7019b33785cee3c7860dc1eda2dfd740354911a82c                        3.7s
 => => resolve docker.io/library/python:3.11-slim@sha256:e9e0bd902ae57c30d3ffdb7019b33785cee3c7860dc1eda2dfd740354911a82c                                0.0s
 => => sha256:361db81715472ecba5e656c314c299bd945309020483159596a8d123c2b1d580 251B / 251B                                                               0.1s
 => => sha256:24ff04c921070df4f4dc987278926d5dabde59d57b55ae3665439331f79af932 14.46MB / 14.46MB                                                         1.7s
 => => sha256:ee5aaeba9945927662bea2702989dfd74d9273796415b896363e6a9c44d9a8ce 1.29MB / 1.29MB                                                           0.3s
 => => sha256:6fc242d221a673263eabd14a9ad9584d3177c96e5cb504c891497338e2f912f5 29.86MB / 29.86MB                                                         1.3s
 => => extracting sha256:6fc242d221a673263eabd14a9ad9584d3177c96e5cb504c891497338e2f912f5                                                                1.2s
 => => extracting sha256:ee5aaeba9945927662bea2702989dfd74d9273796415b896363e6a9c44d9a8ce                                                                0.1s
 => => extracting sha256:24ff04c921070df4f4dc987278926d5dabde59d57b55ae3665439331f79af932                                                                0.8s
 => => extracting sha256:361db81715472ecba5e656c314c299bd945309020483159596a8d123c2b1d580                                                                0.0s
 => [internal] load build context                                                                                                                        0.0s
 => => transferring context: 2.92kB                                                                                                                      0.0s
 => [builder 2/4] WORKDIR /app                                                                                                                           0.7s
 => [builder 3/4] COPY requirements.txt .                                                                                                                0.0s
 => [builder 4/4] RUN pip install --no-cache-dir --prefix=/install -r requirements.txt                                                                   5.8s
 => [stage-1 3/4] COPY --from=builder /install /usr/local                                                                                                0.2s 
 => [stage-1 4/4] COPY app.py .                                                                                                                          0.0s 
 => exporting to image                                                                                                                                   3.5s 
 => => exporting layers                                                                                                                                  0.8s 
 => => exporting manifest sha256:7fb2e52d7bcf48b1b620a22fd69c9c6b91a9b60c0078180660655036e531c500                                                        0.0s 
 => => exporting config sha256:e57229b79b25e81e1541620de26a9ac7860199d7ce45eee01f6f287742831194                                                          0.0s 
 => => exporting attestation manifest sha256:d64b843a7fdad38b637aa157f1330b6aff37e5d81de564639334db9dcdba6c34                                            0.0s
 => => exporting manifest list sha256:bd5ff13133d341d398fd2ce375688f23acafd0494f0118fb53ec995699f57d19                                                   0.0s
 => => naming to us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1                                                        0.0s
 => => unpacking to us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1    
```

The image has been built successfully. The next step is to pushed the image to Artifact registry. 

```bash
docker push $IMAGE
#output
The push refers to repository [us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot]
ee5aaeba9945: Pushed 
44136fa355b3: Pushed 
6fc242d221a6: Pushed 
c6a46033c7e7: Pushed 
24ff04c92107: Pushed 
361db8171547: Pushed 
bfa2a0b5e434: Pushed 
b96bdcc02486: Pushed 
c1c3a92854ca: Pushed 
v1: digest: sha256:bd5ff13133d341d398fd2ce375688f23acafd0494f0118fb53ec995699f57d19 size: 856
```

The push is what makes your image available to Google Cloud services.

The important size is 48.98 MB which is much lower than 400MB. 

### **Task 4 — Build a network to put it on**

The goal is to create a **custom-mode VPC** named `echo-net` with one subnet `echo-subnet` in `$REGION`, plus **two** ingress firewall rules for instances carrying the network tag `echo-http`: TCP **8080** for the app, and TCP **22** so you can SSH in at all. Do **not** use the `default` network. Every other phase in this lab hides networking behind managed services; this is the one place you build it yourself.

In simpler terms, the goal is to create: 

```
echo-net (custom VPC)
│
└── echo-subnet (subnet in us-central1)
      │
      └── echo-vm (later, in Task 5)
            │
            └── tag: echo-http
and two firewalls rule: 
allow-echo-8080
    TCP 8080
    target tag: echo-http

allow-echo-ssh
    TCP 22
    target tag: echo-http 
```

**Why two firewall rules?**

The VM will run the Echo Bot on `port 8080` so external users need access to `http://<VM-IP>:8080` . `Port 8080` is where your Flask app listens. I also need `port 22` because SSH uses port 22: `gcloud compute ssh echo-vm` . 

**Why use the tag `echo-http`?**

Instead of allowing SSH to every VM, I allow SSH to only to VMs tagged echo-http. The firewall will check: Does this VM have tag echo-http? If yes, allow traffic, else: block traffic. 

#### **Learning**

Before completing this 4th task, I must first understand few concpts first. 

**What is VPC?** 

VPC stands for virtual private cloud. VPC is like my own private network inside Google Cloud. 

The VPC defines:

- who can communicate
- how machines connect
- network boundaries

**What is a Subnet?** 

Subnet is just a section of a VPC. A VPC can be divided into smaller pieces called subnets. Each subnet owns a range of IP addresses. 

**What is a network tag?** 

Imagine I set different  VMs: VM a, VM b, VM c. I don’t necessarly want every VM to accept traffic. So I will set tags for each VMs like: 

- VM a : web
- VM b : database
- VM c: web

Then I will make a firewall rule: 

```
Allow 8080
Target tag = web
```

So Google interprets that like VM a and c are ok but not VM b. 

Let’s start by creating the VPC first. 

```bash
gcloud compute networks create echo-net \
    --subnet-mode=custom
#output
Created [https://www.googleapis.com/compute/v1/projects/project-54f8e752-5266-4cfd-9ff/global/networks/echo-net].
NAME: echo-net
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 
INTERNAL_IPV6_RANGE: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network echo-net --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network echo-net --allow tcp:22,tcp:3389,icmp
```

I set the mode: custom because I mustn’t use the default network.  Now let’s create the subnet. 

The goal is to create a subnet called echo-subnet inside the VPC echo-net in the region $REGION. 

```bash
#I will try exploring --help first
gcloud compute networks subnets --help
NAME
    gcloud compute networks subnets - list, describe, and delete, and update
        Compute Engine subnetworks

SYNOPSIS
    gcloud compute networks subnets COMMAND [GCLOUD_WIDE_FLAG ...]

DESCRIPTION
    Read and manipulate VPC subnetworks.

    For more information about VPC, see the VPC documentation
    (https://cloud.google.com/vpc/docs/vpc).

    See also: Subnetworks API
    (https://cloud.google.com/compute/docs/reference/rest/v1/subnetworks).

GCLOUD WIDE FLAGS
    These flags are available to all commands: --help.

    Run $ gcloud help for details.

COMMANDS
    COMMAND is one of the following:

     add-iam-policy-binding
        Add an IAM policy binding to a Compute Engine subnetwork.

     create
        Define a subnet for a network in custom subnet mode.

     delete
        Delete Google Cloud subnetworks.

     describe
        Describe a Compute Engine subnetwork.
```

For the creation of the subnet, the right thought is 

- the subnet name? echo-subnet
- which VPC it belongs to? echo-net
- which region it belongs to? $REGION
- what IP range it owns?

**Where can I get the IP range it owns?** 

A subnet needs its own private address space. 

Google is asking:

> "Which internal IP addresses may be assigned to machines inside this subnet?"
> 

Typical private ranges are:

```
10.0.0.0/24
10.1.0.0/24
192.168.1.0/24
172.16.0.0/24
```

The subnet needs:

- a subnet name
- a parent network
- a region
- a private CIDR range

**What is CIDR range?** 

A CIDR range defines which IP addresses belong to a network or subnet. 

`10.10.0.0/24` has two ranges: `10.10.0.0` and `/24` . 

The first part identifies the network.

The `/24` tells us how many bits are used for the network and, therefore, how many addresses remain available for hosts.

```bash
gcloud compute networks subnets create echo-subnet \
            --network=echo-net --range=10.10.0.0/24 --region=$REGION
#output
Created [https://www.googleapis.com/compute/v1/projects/project-54f8e752-5266-4cfd-9ff/regions/us-central1/subnetworks/echo-subnet].
NAME: echo-subnet
REGION: us-central1
NETWORK: echo-net
RANGE: 10.10.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
```

Now, let’s create a firewall rule that allows `TCP 8080` for VMs tagged `echo-http` inside `echo-net` . 

**What information would Google need to create such a rule?**

It needs 

- Rule name: firewall
- Network: echo-net
- Protocol: TCP
- Port: 8080
- Target VMs: echo-http

**Is this rule for incoming traffic or outgoing traffic?**

```bash
Browser
   ↓
Internet
   ↓
  VM
```

The request is coming **from outside** and trying to reach your VM. It’s incoming traffic (ingress). 

Let’s explore: 

```bash
gcloud compute firewall-rules create --help
#output
NAME
    gcloud compute firewall-rules create - create a Compute Engine firewall
        rule

SYNOPSIS
    gcloud compute firewall-rules create NAME
        (--action=ACTION | --allow=PROTOCOL[:PORT[-PORT]],[...])
        [--description=DESCRIPTION]
        [--destination-ranges=CIDR_RANGE,[CIDR_RANGE,...]]
        [--direction=DIRECTION] [--disabled] [--[no-]enable-logging]
        [--logging-metadata=LOGGING_METADATA]
        [--network=NETWORK; default="default"] [--priority=PRIORITY]
        [--resource-manager-tags=[KEY=VALUE,...]]
        [--rules=PROTOCOL[:PORT[-PORT]],[...]]
        [--source-ranges=CIDR_RANGE,[CIDR_RANGE,...]]
        [--source-service-accounts=EMAIL,[EMAIL,...]]
        [--source-tags=TAG,[TAG,...]]
        [--target-service-accounts=EMAIL,[EMAIL,...]]
        [--target-tags=TAG,[TAG,...]] [GCLOUD_WIDE_FLAG ...]

DESCRIPTION
    gcloud compute firewall-rules create is used to create firewall rules to
    allow/deny incoming/outgoing traffic.

EXAMPLES
    To create a firewall rule allowing incoming TCP traffic on port 8080, run:

        $ gcloud compute firewall-rules create example-service \
            --allow=tcp:8080 \
            --description="Allow incoming traffic on TCP port 8080" \
            --direction=INGRESS

    To create a firewall rule that allows TCP traffic through port 80 and
    determines a list of specific IP address blocks that are allowed to make
    inbound connections, run:
```

My attempt is: 

```bash
gcloud compute firewall-rules create allow-echo-8080 \
    --allow=tcp:8080 \
    --network=echo-net \
    --direction=INGRESS \
    --target-tags=echo-http
#output
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/project-54f8e752-5266-4cfd-9ff/global/firewalls/allow-echo-8080].       
Creating firewall...done.                                                                                                                                    
NAME: allow-echo-8080
NETWORK: echo-net
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:8080
DENY: 
DISABLED: False
```

Let’s create the ssh firewall rule. Without a firewall rule allowing SSH traffic, the VM will exist but the connection will fail.

It’s almost identical to the pervious command: 

```bash
gcloud compute firewall-rules create allow-echo-ssh \
    --allow=tcp:22 \
    --network=echo-net \
    --direction=INGRESS \
    --target-tags=echo-http
#output
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/project-54f8e752-5266-4cfd-9ff/global/firewalls/allow-echo-ssh].        
Creating firewall...done.                                                                                                                                    
NAME: allow-echo-ssh
NETWORK: echo-net
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False
```

### **Task 5 — IaaS path: run the container on a VM in your subnet**

The goal is to create an `e2-micro` VM named `echo-vm` in `echo-subnet`, tagged `echo-http`, running your container and answering `http://<EXTERNAL_IP>:8080/echo?msg=hi`. You need to work out five things: create the instance on your own subnet with the right tag and a Debian image; give its service account permission to pull from Artifact Registry; SSH in; install and authenticate Docker on the VM; run your container detached on port 8080. Then find the VM's external IP.

**What resource are we creating?**

We are creating a VM. Let’s exploring 

```bash
gcloud compute instances create --help
#output
NAME
    gcloud compute instances create - create Compute Engine virtual machine
        instances

SYNOPSIS
    gcloud compute instances create INSTANCE_NAMES [INSTANCE_NAMES ...]
        [--accelerator=[count=COUNT],[type=TYPE]] [--async]
        [--availability-domain=AVAILABILITY_DOMAIN]
        [--no-boot-disk-auto-delete]
        [--boot-disk-device-name=BOOT_DISK_DEVICE_NAME]
        [--boot-disk-interface=BOOT_DISK_INTERFACE]
        [--boot-disk-provisioned-iops=BOOT_DISK_PROVISIONED_IOPS]
        [--boot-disk-provisioned-throughput=BOOT_DISK_PROVISIONED_THROUGHPUT]
        [--boot-disk-size=BOOT_DISK_SIZE] [--boot-disk-type=BOOT_DISK_TYPE]
        [--can-ip-forward] [--create-disk=[PROPERTY=VALUE,...]]
        [--csek-key-file=FILE] [--deletion-protection]
        [--description=DESCRIPTION]
        [--discard-local-ssds-at-termination-timestamp=DISCARD_LOCAL_SSDS_AT_TERMINATION_TIMESTAMP]
        [--disk=[auto-delete=AUTO-DELETE],[boot=BOOT],[device-name=DEVICE-NAME],
          [force-attach=FORCE-ATTACH],
          [interface=INTERFACE],[mode=MODE],[name=NAME],[scope=SCOPE]]
        [--enable-display-device] [--[no-]enable-nested-virtualization]
        [--[no-]enable-uefi-networking] [--erase-windows-vss-signature]
        [--[no-]expose-host-topology]
        [--external-ipv6-address=EXTERNAL_IPV6_ADDRESS]
        [--external-ipv6-prefix-length=EXTERNAL_IPV6_PREFIX_LENGTH]
        [--graceful-shutdown]
        [--graceful-shutdown-max-duration=GRACEFUL_SHUTDOWN_MAX_DURATION]
        [--host-error-timeout-seconds=HOST_ERROR_TIMEOUT_SECONDS]
        [--hostname=HOSTNAME] [--identity=IDENTITY]
        [--[no-]identity-certificate]
        [--instance-termination-action=INSTANCE_TERMINATION_ACTION]
        [--internal-ipv6-address=INTERNAL_IPV6_ADDRESS]
        [--internal-ipv6-prefix-length=INTERNAL_IPV6_PREFIX_LENGTH]
        [--ipv6-network-tier=IPV6_NETWORK_TIER]
```

**What is an e2-micro?** 

An e2-micro is a very small Google Compute Engine VM. To create the echo-VM, we need to know: 

- VM name: echo-VM
- Machine type: e2-micro
- Zone = us-central1-a
- Subnet: echo-subnet
- Network tag: echo-http
- Operating system image: Debian

**What is a Zone?** 

Inside a Region such as `us-central1` , they have multiple zones`us-central1-a, us-central1-b, us-central1-c` . 

**What is an OS image?** 

A VM is basically a computer. Every computer needs an operating system: macOS, windows 11, ubuntu, debian… That operating system comes from an image, which is like the template of the machine. 

```bash
gcloud compute instances create echo-vm \
    --machine-type=e2-micro \
    --zone=$ZONE \
    --subnet=echo-subnet \
    --tags=echo-http \
    --image-family=debian-12 \
    --image-project=debian-cloud
#image family = which Debian version
#image project = where Google stores official Debian images
#Google needs both pieces of information to find the operating system image.
#output
Created [https://www.googleapis.com/compute/v1/projects/project-54f8e752-5266-4cfd-9ff/zones/us-central1-a/instances/echo-vm].
NAME: echo-vm
ZONE: us-central1-a
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.2
EXTERNAL_IP: 34.9.7.189
STATUS: RUNNING
```

The next step is SSH into the VM. 

Let’s explore: 

```bash
gcloud compute ssh --help
#output
NAME
    gcloud compute ssh - SSH into a virtual machine instance

SYNOPSIS
    gcloud compute ssh [USER@]INSTANCE [--command=COMMAND]
        [--container=CONTAINER] [--dry-run] [--force-key-file-overwrite]
        [--plain] [--ssh-flag=SSH_FLAG] [--ssh-key-file=SSH_KEY_FILE]
        [--strict-host-key-checking=STRICT_HOST_KEY_CHECKING] [--troubleshoot]
        [--zone=ZONE] [--internal-ip | --tunnel-through-iap]
        [--network=NETWORK --region=REGION : --dest-group=DEST_GROUP]
        [--ssh-key-expiration=SSH_KEY_EXPIRATION
          | --ssh-key-expire-after=SSH_KEY_EXPIRE_AFTER] [GCLOUD_WIDE_FLAG ...]
        [-- SSH_ARGS ...]

DESCRIPTION
    gcloud compute ssh is a thin wrapper around the ssh(1) command that takes
    care of authentication and the translation of the instance name into an IP
    address.

    To use SSH to connect to a Windows VM, refer to this guide:
    https://cloud.google.com/compute/docs/connect/windows-ssh

    The default network comes preconfigured to allow ssh access to all VMs. If
    the default network was edited, or if not using the default network, you
    may need to explicitly enable ssh access by adding a firewall-rule:

        $ gcloud compute firewall-rules create --network=NETWORK \
            default-allow-ssh --allow=tcp:22

    gcloud compute ssh ensures that the user's public SSH key is present in the
    project's metadata. If the user does not have a public SSH key, one is
    generated using ssh-keygen(1) (if the --quiet flag is given, the generated
    key will have an empty passphrase).

    If the --region and --network flags are provided, then --plain and
```

My attempt

```bash
gcloud compute ssh echo-vm --zone=us-central1-a
#output
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/gabriel_lapointe/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  Y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/gabriel_lapointe/.ssh/google_compute_engine
Your public key has been saved in /home/gabriel_lapointe/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:IezpMky12KY1e1GEXY1YzMIIgajnLDcj6NgO2rgYpq8 gabriel_lapointe@cs-997841202679-default
The key's randomart image is:
+---[RSA 3072]----+
|   . .oo =.*oo   |
|  . ..  o.= + .  |
| .    + . ..     |
|. .  = + o       |
|.+  o O S        |
|+ *o = o .       |
|=* o= o .        |
|**.  o .         |
|E=+              |
+----[SHA256]-----+
Updating project ssh metadata...working...Updated [https://www.googleapis.com/compute/v1/projects/project-54f8e752-5266-4cfd-9ff].                           
Updating project ssh metadata...done.                                                                                                                        
Waiting for SSH key to propagate.
Warning: Permanently added 'compute.2850164232208424274' (ED25519) to the list of known hosts.
Enter passphrase for key '/home/gabriel_lapointe/.ssh/google_compute_engine': 
Enter passphrase for key '/home/gabriel_lapointe/.ssh/google_compute_engine': 
Linux echo-vm 6.1.0-54-cloud-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.190-1 (2026-10-16) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSO
```

Let’s see if docker is installed: 

```bash
gabriel_lapointe@echo-vm:~$ docker --version
-bash: docker: command not found
```

**What package contains Docker?** 

```bash
apt search docker
#output
#they have a lot..
#the right docker for ubuntu is docker.io so 
sudo apt install docker.io 
```

**Does this VM already know how to authenticate to Artifact Registry?**

Let’s configure authentication first because How will Docker prove its identity to Artifact Registry?

- Push image → writer permissions
- Pull image → reader permissions

Let’s list the auth access of the project: 

```bash
gcloud auth list
#output
gabriel_lapointe@echo-vm:~$ gcloud auth list
                  Credentialed Accounts
ACTIVE  ACCOUNT
*       89248238899-compute@developer.gserviceaccount.com

To set the active account, run:
    $ gcloud config set account `ACCOUNT`
```

This is not my personnal account but the Compute Engine Service Account. Now let’s try: 

```bash
gcloud auth configure-docker ...
```

The VM only needs the read access to artifact registry. 

```bash
gcloud compute instances describe echo-vm --zone=us-central1-a
#That's the identity that will need the reader permission.
#output
ERROR: (gcloud.compute.instances.describe) Could not fetch resource:
 - Request had insufficient authentication scopes.
```

In Google Cloud there are two layers: IAM Role and OAuth Scope. Both must allow the operation. That command needs access to the Compute Engine API. The VM's service account is:

`89248238899-compute@developer.gserviceaccount.com`

but the VM was probably created with limited scopes. So the goal is to authenticate Docker on the VM so it can pull from Artifact Registry.

****

```bash
sudo gcloud auth configure-docker us-central1-docker.pkg.dev
#output
Adding credentials for: us-central1-docker.pkg.dev
After update, the following will be written to your Docker config file located at [/root/.docker/config.json]:
 {
  "credHelpers": {
    "us-central1-docker.pkg.dev": "gcloud"
  }
}

Do you want to continue (Y/n)?  Y

Docker configuration file updated.
#and 
sudo docker pull us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
Error response from daemon: Head "https://us-central1-docker.pkg.dev/v2/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot/manifests/v1": denied: Permission 'artifactregistry.repositories.downloadArtifacts' denied on resource '//artifactregistry.googleapis.com/projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/repositories/eurecomgpt' (or it may not exist). Remediate access with this Troubleshooter URL or share it with your administrator - https://console.cloud.google.com/iam-admin/troubleshooter/summary;errorId=CiQwMWExODc3MC1hMjE0LTdjODQtOGYzZC03MmZjMDYxNDI5NmUSVXByb2plY3RzL3Byb2plY3QtNTRmOGU3NTItNTI2Ni00Y2ZkLTlmZi9sb2NhdGlvbnMvdXMtY2VudHJhbDEvcmVwb3NpdG9yaWVzL2V1cmVjb21ncHQ%!D(MISSING) .
```

Let’s configure the IAM policy for the VM (the authorization part): 

```bash
gcloud projects add-iam-policy-binding project-54f8e752-5266-4cfd-9ff \
  --member="serviceAccount:89248238899-compute@developer.gserviceaccount.com" \
  --role="roles/artifactregistry.reader"
#output
Updated IAM policy for project [project-54f8e752-5266-4cfd-9ff].
bindings:
- members:
  - serviceAccount:89248238899-compute@developer.gserviceaccount.com
  role: roles/artifactregistry.reader
- members:
  - serviceAccount:service-89248238899@gcp-sa-artifactregistry.iam.gserviceaccount.com
  role: roles/artifactregistry.serviceAgent
- members:
  - serviceAccount:89248238899@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-89248238899@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-89248238899@gcf-admin-robot.iam.gserviceaccount.com
  role: roles/cloudfunctions.standardServiceAgent
- members:
  - serviceAccount:89248238899@cloudservices.gserviceaccount.com
  role: roles/compute.instanceGroupManagerServiceAgent
- members:
  - serviceAccount:service-89248238899@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-89248238899@containerregistry.iam.gserviceaccount.com
  role: roles/containerregistry.ServiceAgent
- members:
  - user:gabriel.lapointe@eurecom.fr
  role: roles/owner
- members:
  - serviceAccount:service-89248238899@gcp-sa-pubsub.iam.gserviceaccount.com
  role: roles/pubsub.serviceAgent
- members:
  - serviceAccount:service-89248238899@serverless-robot-prod.iam.gserviceaccount.com
  role: roles/run.serviceAgent
etag: BwZe-SCDqMA=
version: 1

#then I will pull the image: 
gabriel_lapointe@cloudshell:~ (project-54f8e752-5266-4cfd-9ff)$ sudo docker pull us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
Error response from daemon: error from registry: Unauthenticated request. Unauthenticated requests do not have permission "artifactregistry.repositories.downloadArtifacts" on resource "projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/repositories/eurecomgpt" (or it may not exist)
gabriel_lapointe@cloudshell:~ (project-54f8e752-5266-4cfd-9ff)$ 
```

Docker knows who it is, Artifact Registry knows who is calling but the identity does not have permission to read images. 

Inside the VM, after changing the IAM policies, I can run this command to pull image: 

```bash
sudo docker pull us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
#output
v1: Pulling from project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot
6fc242d221a6: Pull complete 
ee5aaeba9945: Pull complete 
24ff04c92107: Pull complete 
361db8171547: Pull complete 
bfa2a0b5e434: Pull complete 
b96bdcc02486: Pull complete 
c1c3a92854ca: Pull complete 
Digest: sha256:bd5ff13133d341d398fd2ce375688f23acafd0494f0118fb53ec995699f57d19
Status: Downloaded newer image for us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
```

- Which image to run?`us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1`
- Which host port should receive traffic? `port 8080`
- Which container port is the application listening on? `8080`
- Should the container run in the background? We must run the container detached, which means in the background.

Now we have to run the docker image in the background. Let’s find the correct command

```bash
docker run --help
#what we are looking for: 
  -d, --detach                         Run container in background and print container ID
  and -p means publish a port (port mapping between the host and the container).
```

The command will be: 

```bash
docker run -d -p 8080:8080 \
  us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
#output
  us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
2bd18cbcf663933140ddd6aad7e3abd2fad464f2e5fb9d6b4bf0950f2b292fe2
```

Now, we can test the application from inside the VM:

```bash
curl localhost:8080/echo?msg=hi
#output
{"echo":"hi","length":2}
```

The next verification is usually from outside the VM.

```bash
curl ifconfig.me
#output
34.9.7.189
curl http://34.9.7.189:8080/echo?msg=hello
#output
{"echo":"hello","length":5}
curl -s -X POST http://34.9.7.189:8080/echo \
     -H 'Content-Type: application/json' \
     -d '{"message":"hello"}'
 #output
 {"echo":"hello","length":5}
```

**What is curl?** 

`curl` is a command-line program that sends HTTP requests.

### **Task 6 — Container PaaS path: deploy the same image to Cloud Run**

Let’s create a public Cloud Run service named `echo-bot` in `$REGION` running the *same* image, scaling to zero, capped at 3 instances, listening on port 8080.

**What is a public access?** 

When you enable public access (often via a flag such as `--allow-unauthenticated`), you're saying Anyone on the Internet can call the service URL: `https://echo-bot-xxxxx-uc.a.run.app` Then anyone can run `curl https://echo-bot-xxxxx-uc.a.run.app/echo?msg=hello`without authenticating.

The command should be: 

```bash
gcloud run deploy echo-bot \
  --image=$IMAGE \
  --region=$REGION \
  --port=8080 \
  --max-instances=3 \
  --allow-unauthenticated
#output
Deploying container to Cloud Run service [echo-bot] in project [project-54f8e752-5266-4cfd-9ff] region [us-central1]
OK Deploying new service... Done.                                                                                                                            
  OK Creating Revision...                                                                                                                                    
  OK Routing traffic...                                                                                                                                      
  OK Setting IAM Policy...                                                                                                                                   
Done.                                                                                                                                                        
Service [echo-bot] revision [echo-bot-00001-6kr] has been deployed and is serving 100 percent of traffic.
Service URL: https://echo-bot-89248238899.us-central1.run.app
```

**What did `--allow-unauthenticated` do?**

With this, I make the service public. 

In this part, the architecture is: 

```bash
Internet
    ↓
Cloud Run
    ↓
Container
    ↓
Echo Bot
```

**Verification**

```bash
curl "https://echo-bot-89248238899.us-central1.run.app/echo?msg=hello"
#output
{"echo":"hello","length":5}
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"message":"hello"}' \
  "https://echo-bot-89248238899.us-central1.run.app/echo"
  #output
  {"echo":"hello","length":5}
```

**Is it filled the requirements?** 

```bash
gcloud run services describe echo-bot \
    --region=us-central1
  #output
✔ Service echo-bot in region us-central1
 
URL:     https://echo-bot-89248238899.us-central1.run.app
Ingress: all
Traffic:
  100% LATEST (currently echo-bot-00001-6kr)
 
Scaling:     Auto (Min: 0, Max: 12)
SSH enabled: false
 
Last updated on 2026-10-29T14:15:58.094016Z by gabriel.lapointe@eurecom.fr:
  Revision echo-bot-00001-6kr
  Container None
    Image:           us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot:v1
    Port:            8080
    Memory:          512Mi
    CPU:             1000m
    Startup Probe:
      TCP every 240s
      Port:          8080
      Initial delay: 0s
      Timeout:       240s
      Failure threshold: 1
      Type:          Default
  Service account:   89248238899-compute@developer.gserviceaccount.com
  Concurrency:       80
  Scaling:
    Max instances:   3
  Timeout:           300s
```

### **Task 7 — Serverless path: deploy `main.py` as a Cloud Function (gen 2)**

Th goal is to create a public gen-2 Cloud Function named `echo` in `$REGION`, Python 3.11, built from this phase's source with `echo` as the entry point, HTTP-triggered.

Does your project already contain `main.py` with a function named `def echo(..):` . 

```bash
cat main.py
#output
"""Cloud Function (gen 2) entry point — the SAME echo logic as a serverless function.

This is the third deployment target. It reuses ``build_echo`` and ``extract_message``
from ``app.py``, so the exact logic you run in a container (VM + Cloud Run) also runs
here as a function — that is the point of Phase 1: one behaviour, three service models.

Deployed with ``gcloud functions deploy`` (see TASKS.md). Provided in full — no TODOs;
your job is to get it deployed and compare its cold start to the other two.
"""

from __future__ import annotations

import functions_framework
from flask import jsonify

from app import build_echo, extract_message

@functions_framework.http
def echo(request):
    message = extract_message(request.args, request.get_json(silent=True))
    if message is None:
        return jsonify({"error": "provide ?msg=... or JSON {\"message\": ...}"}), 400
    return jsonify(build_echo(message))
```

The next step is to deploy [`main.py`](http://main.py) as a gen2 cloud function. 

**What is an entry point?** 

When you deploy a Cloud Function, Google sees a file containing potentially many functions. The entry point is the function that google will run when an http request arrives. 

The command will be: 

```bash
gcloud functions deploy echo-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=echo \
  --trigger-http \
  --allow-unauthenticated
#output
#The service account policy doesn't have the right IAM policy build to do this so I must do this first: 
gcloud projects add-iam-policy-binding project-54f8e752-5266-4cfd-9ff \
  --member="serviceAccount:89248238899-compute@developer.gserviceaccount.com" \
  --role="roles/cloudbuild.builds.builder"
#then I can redeploy it again and the output is: 
Preparing function...done.                                                                                                                                   
X  Updating function (may take a while)...                                                                                                                   
  OK [Build] Logs are available at [https://console.cloud.google.com/cloud-build/builds;region=us-central1/e77040f6-5d72-4a98-a140-d745c0c64c0e?project=89248
  238899]                                                                                                                                                    
     [Service]                                                                                                                                               
  .  [ArtifactRegistry]                                                                                                                                      
  .  [Healthcheck]                                                                                                                                           
  .  [Triggercheck]                                                                                                                                          
Completed with warnings:                                                                                                                                     
  [WARNING] Cloud Run service projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/services/echo-function for the function was not found. The service was redeployed with default values.
You can view your function in the Cloud Console here: https://console.cloud.google.com/functions/details/us-central1/echo-function?project=project-54f8e752-5266-4cfd-9ff

buildConfig:
  automaticUpdatePolicy: {}
  build: projects/89248238899/locations/us-central1/builds/e77040f6-5d72-4a98-a140-d745c0c64c0e
  dockerRegistry: ARTIFACT_REGISTRY
  dockerRepository: projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/repositories/gcf-artifacts
  entryPoint: echo
  runtime: python311
  serviceAccount: projects/project-54f8e752-5266-4cfd-9ff/serviceAccounts/89248238899-compute@developer.gserviceaccount.com
  source:
    storageSource:
      bucket: gcf-v2-sources-89248238899-us-central1
      generation: '1793284827983496'
      object: echo-function/function-source.zip
  sourceProvenance:
    resolvedStorageSource:
      bucket: gcf-v2-sources-89248238899-us-central1
      generation: '1793284827983496'
      object: echo-function/function-source.zip
createTime: '2026-10-29T14:37:33.123872468Z'
environment: GEN_2
labels:
  deployment-tool: cli-gcloud
name: projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/functions/echo-function
satisfiesPzi: true
serviceConfig:
  allTrafficOnLatestRevision: true
  availableCpu: '0.1666'
  availableMemory: 256M
  environmentVariables:
    LOG_EXECUTION_ID: 'true'
  ingressSettings: ALLOW_ALL
  maxInstanceRequestConcurrency: 1
  revision: echo-function-00001-vuj
  service: projects/project-54f8e752-5266-4cfd-9ff/locations/us-central1/services/echo-function
  serviceAccountEmail: 89248238899-compute@developer.gserviceaccount.com
  timeoutSeconds: 60
  uri: https://echo-function-kpphvy3b6a-uc.a.run.app
state: ACTIVE
updateTime: '2026-10-29T14:41:28.996667286Z'
url: https://us-central1-project-54f8e752-5266-4cfd-9ff.cloudfunctions.net/echo-function

```

**Verification**

```bash
curl "https://us-central1-project-54f8e752-5266-4cfd-9ff.cloudfunctions.net/echo-function?msg=hello"
#output
{"echo":"hello","length":5}
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"message":"hello"}' \
  "https://us-central1-project-54f8e752-5266-4cfd-9ff.cloudfunctions.net/echo-function"
#output
{"echo":"hello","length":5}

```

### **Task 8 — Measure all three and write the report**

We already have the three endpoints:

VM: `http://34.9.7.189:8080`

Cloud Run: `https://echo-bot-89248238899.us-central1.run.app`

Cloud Function: `https://us-central1-project-54f8e752-5266-4cfd-9ff.cloudfunctions.net/echo-function`

I must run exactly once: 

```bash
python phase-1-echo-bot/measure.py \
    --vm http://34.9.7.189:8080 \
    --cloudrun https://echo-bot-89248238899.us-central1.run.app \
    --function https://us-central1-project-54f8e752-5266-4cfd-9ff.cloudfunctions.net/echo-function
#output
Phase 1 — deployment measurements
------------------------------------------------------------
[OK  ] vm        cold=  267.5ms  warm_median=  212.4ms
            ########################################
[OK  ] cloudrun  cold= 4744.2ms  warm_median=  143.2ms
            ###########################
[OK  ] function  cold= 1534.9ms  warm_median=  133.6ms
            #########################
------------------------------------------------------------

Wrote submission/phase1_report.json — commit it and push.
  
```

The small difference between "cold" and "warm" is mostly measurement noise and normal network latency. Cloud Run had the largest cold start because: 

```bash
Request arrives
      ↓
No running instance
      ↓
Google starts container
      ↓
Container starts Gunicorn
      ↓
Container becomes ready
      ↓
Request succeeds
```

Once the container is running, warm requests become very fast. Cloud Function is also serverless and experiences cold starts.

The VM exhibited no meaningful cold start because the VM and Docker container remained continuously running. Cloud Run and Cloud Functions both experienced cold starts due to scaling to zero after inactivity. In the recorded measurements, Cloud Run had the highest cold-start latency (4744 ms), while Cloud Functions showed a smaller cold-start latency (1535 ms). For warm requests, both serverless services performed slightly better than the VM, with warm median latencies near 130-145 ms compared to 212 ms for the VM. The VM provides predictable latency but requires managing infrastructure, whereas Cloud Run and Cloud Functions reduce operational overhead at the cost of cold-start delays.

### **Task 9 — Commit, push, confirm green CI**

### **Task 10 — Write the comparison report**

```markdown
# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
49
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->267.5<!--/answer--> | <!--answer:vm_warm_median_ms-->212.4<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->4744.2<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->143.2<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1534.9<!--/answer--> | <!--answer:function_warm_median_ms-->133.6<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
[OK ] vm cold= 267.5ms warm_median= 212.4ms

[OK ] cloudrun cold= 4744.2ms warm_median= 143.2ms

[OK ] function cold= 1534.9ms warm_median= 133.6ms
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM does not automatically create anything to absorb additional load. All requests are served by a single e2-micro instance, so capacity is limited by the CPU and memory of that machine. Cloud Run automatically creates additional container instances when traffic increases. In my deployment the maximum was capped at 3 instances using the --max-instances=3 configuration. Cloud Functions Gen 2 also scales automatically by creating additional function instances when needed. Its limit comes from platform configuration and quotas rather than from a VM size selected by the user. With one request the platforms use minimal resources, with ten or one hundred requests Cloud Run and Functions can scale out automatically while the VM cannot.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
The VM deployment required the most manual work. I had to create a VPC, subnet, firewall rules, a Compute Engine VM, install Docker, configure Artifact Registry access, pull the container image, and run the application manually. Cloud Run was simpler because I only needed to provide the container image and deployment settings while Google managed the infrastructure, networking, and scaling. Cloud Functions required the least operational effort because I deployed source code directly from main.py without building or managing a container myself. The platform automatically built and deployed the function and handled scaling and runtime management.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-89248238899.us-central1.run.app
LAST DEPLOYED BY: gabriel.lapointe@eurecom.fr
LAST DEPLOYED AT: 2026-10-29T14:15:58.094016Z

SERVICE: echo-function
REGION: us-central1
URL: https://echo-function-89248238899.us-central1.run.app
LAST DEPLOYED BY: service-89248238899@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-29T14:41:27.365098Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows that both the Cloud Run application and the Cloud Function appear as Cloud Run services. This indicates that Cloud Functions Gen 2 is implemented on top of Cloud Run. As a result, both platforms share a similar execution environment and scaling model, which explains why their cold-start measurements are relatively close compared with the VM deployment. In both cases Google creates and manages the underlying container infrastructure automatically. The main difference is the deployment workflow. Cloud Run deploys a container image that I built and stored in Artifact Registry, while Cloud Functions deploys source code directly. Google then builds, packages, and runs the function on my behalf. Operationally they are managed similarly, but the developer experience and packaging model differ.
<!--/answer-->

```

<img width="4032" height="633" alt="image 1" src="https://github.com/user-attachments/assets/2a40ca3b-8408-40a3-8eae-e6586eab4c1c" />

I will get first the image size: 

```markdown
gcloud artifacts docker images list \
us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt
#output
Listing items under project project-54f8e752-5266-4cfd-9ff, location us-central1, repository eurecomgpt.

IMAGE: us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot
DIGEST: sha256:7fb2e52d7bcf48b1b620a22fd69c9c6b91a9b60c0078180660655036e531c500
CREATE_TIME: 2026-10-28T10:25:01
UPDATE_TIME: 2026-10-28T10:25:01
SIZE: 48984737

IMAGE: us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot
DIGEST: sha256:bd5ff13133d341d398fd2ce375688f23acafd0494f0118fb53ec995699f57d19
CREATE_TIME: 2026-10-28T10:25:02
UPDATE_TIME: 2026-10-28T10:25:02
SIZE: None

IMAGE: us-central1-docker.pkg.dev/project-54f8e752-5266-4cfd-9ff/eurecomgpt/echo-bot
DIGEST: sha256:d64b843a7fdad38b637aa157f1330b6aff37e5d81de564639334db9dcdba6c34
CREATE_TIME: 2026-10-28T10:25:01
UPDATE_TIME: 2026-10-28T10:25:01
SIZE: 1650
```

I wil now get the cloud run services output by running: 

```markdown
gcloud run services list --region=us-central1
#output
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-89248238899.us-central1.run.app
LAST DEPLOYED BY: gabriel.lapointe@eurecom.fr
LAST DEPLOYED AT: 2026-10-29T14:15:58.094016Z

✔
SERVICE: echo-function
REGION: us-central1
URL: https://echo-function-89248238899.us-central1.run.app
LAST DEPLOYED BY: service-89248238899@gcf-admin-
```

Then, I will get: 

```markdown
gcloud run services list --region=us-central1
#output
✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-89248238899.us-central1.run.app
LAST DEPLOYED BY: gabriel.lapointe@eurecom.fr
LAST DEPLOYED AT: 2026-10-29T14:15:58.094016Z

✔
SERVICE: echo-function
REGION: us-central1
URL: https://echo-function-89248238899.us-central1.run.app
LAST DEPLOYED BY: service-89248238899@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-29T14:41:27.365098Z
```

## Sources

1. Docker multi-stage builds: [https://docs.docker.com/build/building/multi-stage/](https://docs.docker.com/build/building/multi-stage/)
2. Why gunicorn for Flask in a container: [https://flask.palletsprojects.com/en/latest/deploying/gunicorn/](https://flask.palletsprojects.com/en/latest/deploying/gunicorn/)
3. **Cloud Run** (container PaaS): [https://cloud.google.com/run/docs/overview/what-is-cloud-run](https://cloud.google.com/run/docs/overview/what-is-cloud-run)
4. **Cloud Functions (2nd gen)** (FaaS): [https://cloud.google.com/functions/docs/concepts/version-comparison](https://cloud.google.com/functions/docs/concepts/version-comparison)
5. **Artifact Registry** (image storage): [https://cloud.google.com/artifact-registry/docs/overview](https://cloud.google.com/artifact-registry/docs/overview)
6. **Cold starts** — what they are and why min-instances matters: [https://cloud.google.com/run/docs/tips/general#using_minimum_instances_to_reduce_cold_starts](https://cloud.google.com/run/docs/tips/general#using_minimum_instances_to_reduce_cold_starts)
7. **Compute Engine** VMs & the free `e2-micro`: [https://cloud.google.com/free/docs/free-cloud-features#compute](https://cloud.google.com/free/docs/free-cloud-features#compute)
8. https://flask.palletsprojects.com/en/stable/quickstart/
