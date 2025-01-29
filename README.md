# Batagor
This repository containes Dockerfiles to create Google Cloud Workstation images

# Building Noble

```
cd batagor/ws/noble
docker build -t noble --platform linux/amd64 .
docker tag noble gcr.io/batagor/ws-noble:v1
docker push gcr.io/batagor/ws-noble:v1
```

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) 