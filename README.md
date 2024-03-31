# csd548_ex2
1. **Provide the YAML that runs a Pod with Nginx 1.23.3-alpine**

Yaml name is nginx.yaml

1. apiVersion: v1
2. kind: Pod
3. metadata:
4.   name: nginx-pod
5. spec:
6.   containers:
7.   - name: nginx
8.     image: nginx:1.21.6-alpine
9.     ports:
10.     - containerPort: 80
11.       name: http
12.       protocol: TCP

**as well as the kubectl commands needed to:**

- 1. **Install the manifest on Kubernetes and start the Pod.**

Firstly run minikube start Then

kubectl apply -f nginx.yaml

- 1. **Forward port 80 locally, so that it answers calls through a browser (or curl or wget).**

kubectl port-forward nginx-pod 8080:80

- 1. **See the logs of the running container.**

kubectl logs nginx-pod

&nbsp;     /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration

/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/

/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh

10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf

10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf

/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh

/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh

/docker-entrypoint.sh: Configuration complete; ready for start up

2024/03/17 19:55:35 \[notice\] 1#1: using the "epoll" event method

2024/03/17 19:55:35 \[notice\] 1#1: nginx/1.21.6

2024/03/17 19:55:35 \[notice\] 1#1: built by gcc 10.3.1 20211027 (Alpine 10.3.1_git20211027)

2024/03/17 19:55:35 \[notice\] 1#1: OS: Linux 5.10.102.1-microsoft-standard-WSL2

2024/03/17 19:55:35 \[notice\] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576

2024/03/17 19:55:35 \[notice\] 1#1: start worker processes

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 32

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 33

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 34

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 35

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 36

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 37

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 38

2024/03/17 19:55:35 \[notice\] 1#1: start worker process 39

127.0.0.1 - - \[17/Mar/2024:19:55:52 +0000\] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" "-"

2024/03/17 19:55:52 \[error\] 32#32: \*1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "<http://localhost:8080/>"

127.0.0.1 - - \[17/Mar/2024:19:55:52 +0000\] "GET /favicon.ico HTTP/1.1" 404 555 "<http://localhost:8080/>" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" "-"

- 1. **Open a shell session inside the running container and change the first sentence of the default page to "Welcome to MY nginx!". Close the session.**

kubectl exec -it nginx-pod – sh

vi /usr/share/nginx/html/index.html

exit

- 1. **From your computer terminal (outside the container), download the default page locally and upload another one in its place.**

kubectl cp nginx-pod:/usr/share/nginx/html/index.html ./index.html

cp ./hy548/html/public/index.html nginx-pod:

/usr/share/nginx/html/index.html

- 1. **Stop the Pod and remove the manifest from Kubernetes.**

kubectl delete -f nginx.yaml

**2\. The code that produces the course's website is available on GitHub (<https://github.com/chazapis/hy548>). Provide the YAML that creates a Job using Ubuntu 20.04, which when started will run a script (defined in a ConfigMap) that will download the repository (and submodules), hugo (the tool that builds the website), and build the website.**

We need the extended hugo version, otherwise it will not work

Yaml name is ubuntu.yaml

apiVersion: v1

kind: ConfigMap

metadata:

&nbsp; name: website-build-script

data:

&nbsp; build-script.sh: |

&nbsp;   #!/bin/bash

&nbsp;   apt-get update && apt-get install -y git wget tar

&nbsp;   wget -qO- <https://github.com/gohugoio/hugo/releases/download/v0.94.0/hugo_extended_0.94.0_Linux-64bit.tar.gz> | tar xvz

&nbsp;   mv hugo /usr/local/bin/

&nbsp;   git clone --recursive <https://github.com/chazapis/hy548.git>

&nbsp;   cd ./hy548

&nbsp;   git submodule update --init --recursive

&nbsp;   cd ./html

&nbsp;   hugo -D

\---

apiVersion: batch/v1

kind: Job

metadata:

&nbsp; name: ubuntu

spec:

&nbsp; template:

&nbsp;   spec:

&nbsp;     volumes:

&nbsp;     - name: script-volume

&nbsp;       configMap:

&nbsp;         name: website-build-script

&nbsp;     restartPolicy: Never

&nbsp;     containers:

&nbsp;     - name: website-builder

&nbsp;       image: ubuntu:20.04

&nbsp;       volumeMounts:

&nbsp;       - name: script-volume

&nbsp;         mountPath: /scripts

&nbsp;       command: \["/bin/bash", "/scripts/build-script.sh"\]

Then run

kubectl apply -f ubuntu.yaml

**Which command can you use to confirm that the Job completed successfully?**

kubectl logs -f job/ubuntu

And we can see the logs

At the end we see that website was build

Start building sites …

hugo v0.94.0-63B23660+extended linux/amd64 BuildDate=2022-03-10T09:46:36Z VendorInfo=gohugoio

&nbsp;                  | EL | EN

\-------------------+----+-----

&nbsp; Pages            |  3 |  3

&nbsp; Paginator pages  |  0 |  0

&nbsp; Non-page files   |  0 |  0

&nbsp; Static files     | 60 | 60

&nbsp; Processed images |  0 |  0

&nbsp; Aliases          |  1 |  0

&nbsp; Sitemaps         |  0 |  0

&nbsp; Cleaned          |  0 |  0

Total in 90 ms

**3\. Following the previous two exercises, provide a single YAML that will run the Pod with Nginx, the above Job with the script, and a CronJob that will refresh the content every night at 2:15 (only if changes have been made to git). The Nginx Pod should serve the web pages produced by the Jobs instead of the default page. Briefly describe how data is communicated between containers.**

Yaml name is ubuntu-host.yaml

So we use the previous 2 yaml files but we add the PersistentVolumeClaim, in order to communicate the data from Job to Pod. We copy the html/public to /websites (which is mounted to PersistentVolume.) Then we mount the same volume to Pod too, in /usr/share/nginx/html. And in this way we have the data from Job inside the Pod

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: nginx-pod

spec:

&nbsp; volumes:

&nbsp; - name: website-content

&nbsp;   persistentVolumeClaim:

&nbsp;     claimName: ubuntu-pvc

&nbsp; containers:

&nbsp; - name: nginx

&nbsp;   image: nginx:1.21.6-alpine

&nbsp;   ports:

&nbsp;   - containerPort: 80

&nbsp;     name: http

&nbsp;     protocol: TCP  

&nbsp;   volumeMounts:

&nbsp;   - name: website-content

&nbsp;     mountPath: /usr/share/nginx/html

\---

apiVersion: v1

kind: ConfigMap

metadata:

&nbsp; name: website-build-script

data:

&nbsp; build-script.sh: |

&nbsp;   #!/bin/bash

&nbsp;   apt-get update && apt-get install -y git wget tar

&nbsp;   wget -qO- <https://github.com/gohugoio/hugo/releases/download/v0.94.0/hugo_extended_0.94.0_Linux-64bit.tar.gz> | tar xvz

&nbsp;   mv hugo /usr/local/bin/

&nbsp;   git clone --recursive <https://github.com/chazapis/hy548.git>

&nbsp;   cd ./hy548

&nbsp;   git submodule update --init --recursive

&nbsp;   cd ./html

&nbsp;   hugo -D

&nbsp;   cp -r ./public/\* /website

\---

apiVersion: batch/v1

kind: Job

metadata:

&nbsp; name: ubuntu

spec:

&nbsp; template:

&nbsp;   spec:

&nbsp;     volumes:

&nbsp;     - name: script-volume

&nbsp;       configMap:

&nbsp;         name: website-build-script

&nbsp;     - name: website-content

&nbsp;       persistentVolumeClaim:

&nbsp;         claimName: ubuntu-pvc

&nbsp;     restartPolicy: OnFailure

&nbsp;     containers:

&nbsp;     - name: website-builder

&nbsp;       image: ubuntu:20.04

&nbsp;       command: \["/bin/bash", "/scripts/build-script.sh"\]

&nbsp;       volumeMounts:

&nbsp;       - name: script-volume

&nbsp;         mountPath: /scripts

&nbsp;       - name: website-content

&nbsp;         mountPath: /website

\---

apiVersion: batch/v1

kind: CronJob

metadata:

&nbsp; name: ubuntu

spec:

&nbsp; schedule: "15 2 \* \* \*"

&nbsp; jobTemplate:

&nbsp;   spec:

&nbsp;     template:

&nbsp;       spec:

&nbsp;         volumes:

&nbsp;         - name: script-volume

&nbsp;           configMap:

&nbsp;             name: website-build-script

&nbsp;         - name: website-content

&nbsp;           persistentVolumeClaim:

&nbsp;             claimName: ubuntu-pvc

&nbsp;         restartPolicy: OnFailure

&nbsp;         containers:

&nbsp;         - name: website-builder

&nbsp;           image: ubuntu:20.04

&nbsp;           volumeMounts:

&nbsp;           - name: script-volume

&nbsp;             mountPath: /scripts

&nbsp;           - name: website-content

&nbsp;             mountPath: /website

&nbsp;           command: \["/bin/bash", "/scripts/build-script.sh"\]

\---

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

&nbsp; name: ubuntu-pvc

spec:

&nbsp; storageClassName: ""

&nbsp; volumeName: ubuntu-pv

&nbsp; accessModes: \[ReadWriteOnce\]

&nbsp; resources:

&nbsp;   requests:

&nbsp;     storage: 1Gi

\---

apiVersion: v1

kind: PersistentVolume

metadata:

&nbsp; name: ubuntu-pv

spec:

&nbsp; accessModes: \[ReadWriteOnce\]

&nbsp; capacity:

&nbsp;   storage: 1Gi

&nbsp; hostPath:

&nbsp;   path: /website

**4\. Following on from the previous exercise, embed the Nginx Pods in a Deployment (keeping the Job and Cronjob in the YAML) and use an init container to start the Pods when the web page is finished building. Also add a Service to the manifest. Provide the overall YAML.**

Yaml name is ubuntu-dev.yaml

To port forward, use kubectl port-forward service/nginx-service 8080:80

apiVersion: v1

kind: ConfigMap

metadata:

&nbsp; name: website-build-script

data:

&nbsp; build-script.sh: |

&nbsp;   #!/bin/bash

&nbsp;   apt-get update && apt-get install -y git wget tar

&nbsp;   wget -qO- <https://github.com/gohugoio/hugo/releases/download/v0.94.0/hugo_extended_0.94.0_Linux-64bit.tar.gz> | tar xvz

&nbsp;   mv hugo /usr/local/bin/

&nbsp;   git clone --recursive <https://github.com/chazapis/hy548.git>

&nbsp;   cd ./hy548

&nbsp;   git submodule update --init --recursive

&nbsp;   cd ./html

&nbsp;   hugo -D

&nbsp;   cp -r ./public/\* /website

\---

apiVersion: batch/v1

kind: Job

metadata:

&nbsp; name: ubuntu

spec:

&nbsp; template:

&nbsp;   spec:

&nbsp;     volumes:

&nbsp;     - name: script-volume

&nbsp;       configMap:

&nbsp;         name: website-build-script

&nbsp;     - name: website-content

&nbsp;       persistentVolumeClaim:

&nbsp;         claimName: ubuntu-pvc

&nbsp;     restartPolicy: OnFailure

&nbsp;     containers:

&nbsp;     - name: website-builder

&nbsp;       image: ubuntu:20.04

&nbsp;       command: \["/bin/bash", "/scripts/build-script.sh"\]

&nbsp;       volumeMounts:

&nbsp;       - name: script-volume

&nbsp;         mountPath: /scripts

&nbsp;       - name: website-content

&nbsp;         mountPath: /website

\---

apiVersion: batch/v1

kind: CronJob

metadata:

&nbsp; name: ubuntu

spec:

&nbsp; schedule: "15 2 \* \* \*"

&nbsp; jobTemplate:

&nbsp;   spec:

&nbsp;     template:

&nbsp;       spec:

&nbsp;         volumes:

&nbsp;         - name: script-volume

&nbsp;           configMap:

&nbsp;             name: website-build-script

&nbsp;         - name: website-content

&nbsp;           persistentVolumeClaim:

&nbsp;             claimName: ubuntu-pvc

&nbsp;         restartPolicy: OnFailure

&nbsp;         containers:

&nbsp;         - name: website-builder

&nbsp;           image: ubuntu:20.04

&nbsp;           volumeMounts:

&nbsp;           - name: script-volume

&nbsp;             mountPath: /scripts

&nbsp;           - name: website-content

&nbsp;             mountPath: /website

&nbsp;           command: \["/bin/bash", "/scripts/build-script.sh"\]

\---

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

&nbsp; name: ubuntu-pvc

spec:

&nbsp; storageClassName: ""

&nbsp; volumeName: ubuntu-pv

&nbsp; accessModes: \[ReadWriteOnce\]

&nbsp; resources:

&nbsp;   requests:

&nbsp;     storage: 1Gi

\---

apiVersion: v1

kind: PersistentVolume

metadata:

&nbsp; name: ubuntu-pv

spec:

&nbsp; accessModes: \[ReadWriteOnce\]

&nbsp; capacity:

&nbsp;   storage: 1Gi

&nbsp; hostPath:

&nbsp;   path: /website

\---

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: nginx-deployment

spec:

&nbsp; replicas: 1

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: nginx

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: nginx

&nbsp;   spec:

&nbsp;     volumes:

&nbsp;     - name: website-content

&nbsp;       persistentVolumeClaim:

&nbsp;         claimName: ubuntu-pvc

&nbsp;     initContainers:

&nbsp;     - name: wait-for-website

&nbsp;       image: busybox:latest

&nbsp;       command: \["sh", "-c", "#until \[ -f /usr/share/nginx/html/index.html \]; do sleep 1; done"\]

&nbsp;       volumeMounts:

&nbsp;       - name: website-content

&nbsp;         mountPath: /usr/share/nginx/html

&nbsp;     containers:

&nbsp;     - name: nginx

&nbsp;       image: nginx:1.21.6-alpine

&nbsp;       ports:

&nbsp;       - containerPort: 80

&nbsp;         name: http

&nbsp;         protocol: TCP  

&nbsp;       volumeMounts:

&nbsp;       - name: website-content

&nbsp;         mountPath: /usr/share/nginx/html

\---

apiVersion: v1

kind: Service

metadata:

&nbsp; name: nginx-service

spec:

&nbsp; selector:

&nbsp;   app: nginx

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80

&nbsp;     targetPort: 80
