---
layout: default
---

## DockerFile

Lambda docker file - Make sure requirements.txt and revelant packages locally

```
FROM amazonlinux:2017.03
RUN yum -y install git \
    python37 \
    zip \
    python37-pip \
    && yum clean all
RUN python3 -m pip install --upgrade pip 
ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
ADD requirements.txt /
RUN python3 -m pip install -r requirements.txt
ADD function.py $VIRTUAL_ENV/lib/python3.6/site-packages
RUN cd $VIRTUAL_ENV/lib/python3.6/site-packages && zip -r9 function.zip .
```

Local lambda testing without AWS account

```
docker run --rm -v "$PWD":/var/task:ro,delegated lambci/lambda:python3.8 lambda_function.lambda_handler
```

Copy the file from container to host
```
docker run --name temp-container-name ec842c07d3d0 /bin/true
docker cp temp-container-name:/opt/venv/lib/python3.6/site-packages/function.zip .
docker rm temp-container-name
```





[back](./)
