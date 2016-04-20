[![Build Status](https://travis-ci.org/mmisw/orr-ont.svg?branch=master)](https://travis-ci.org/mmisw/orr-ont)
[![Coverage Status](https://coveralls.io/repos/github/mmisw/orr-ont/badge.svg?branch=master)](https://coveralls.io/github/mmisw/orr-ont?branch=master)



`orr-ont` is a new version of the `Ont` service.
See [wiki](https://github.com/mmisw/orr-ont/wiki).


# Build and deployment

**NOTE**: 
These notes are WiP while the complete build/deployment workflow itself is refined.


## Build `orr-ont` package

    sbt package
    
## Build and push `mmisw/orr-ont` image

    docker build -t mmisw/orr-ont --no-cache .
    docker push mmisw/orr-ont
    

## Deployment

### Docker images:

    docker pull mongo
    docker pull franzinc/agraph
    docker pull mmisw/httpd
    docker pull mmisw/orr-ont
    
  
### Preparations

    BASE_DIR=/home/carueda/orr-ont-base-directory
    MONGO_DATA=${BASE_DIR}/mongo-data
    
> my mac:
>
>    ```
>    BASE_DIR=/Users/carueda/orr-ont-base-directory
>    MONGO_DATA=${BASE_DIR}/mongo-dbpath
>    ```
>
    
    mkdir -p ${BASE_DIR}
    mkdir -p ${MONGO_DATA}
    
    scp somewhere:template.orront.conf ./orront.conf
    vim ./orront.conf   # edit if needed
    

    
### Run containers

#### Mongo

    docker run --name mongo -d \
           -p 27017:27017 \
           -v {MONGO_DATA}:/data/db \
           mongo
           
> Note (MacOS): Due to VirtualBox bug, -v not supported so no way to have 
> a local share for the mongo data.
>
>    ```
>    docker run --name mongo -d \
>           -p 27017:27017 \
>           mongo
>    ```
           
    
#### AllegroGraph

    docker run --name agraph -d \
           -m 1g -p 10000-10035:10000-10035 franzinc/agraph

    
#### orr-ont

    docker run --name orr-ont -d \
           --link mongo \
           --link agraph \
           -v `pwd`/orront.conf:/etc/orront.conf \
           -v ${BASE_DIR}:/opt/orr-ont-base-directory \
           -p 9090:8080 \
           mmisw/orr-ont

#### HTTP proxy

    docker run --name httpd -d \
           -p 80:80 \
           --link mongo \
           --link agraph \
           --link orr-ont \
           mmisw/httpd
               
>
> [nginx-proxy](https://github.com/jwilder/nginx-proxy) 
> looks very interesting ... but it currently does not support paths
> (see eg., [this](https://github.com/jwilder/nginx-proxy/pull/254)),
> and it's not immediately clear how to "intercept" requests/responses 
> to add headers (CORS), etc.
> 
> Basically, nginx-proxy would be run as this:
>
>    ```
>    docker run --name nginx-proxy -d \
>           -p 80:80 \
>           -v /var/run/docker.sock:/tmp/docker.sock:ro \
>           jwilder/nginx-proxy
>    ```
>
> AllegroGraph would be started with, for example,
> `-e VIRTUAL_HOST=sparql.somehost.net -e VIRTUAL_PORT=10035`,
> and orr-ont with
> `-e VIRTUAL_HOST=somehost.net -e VIRTUAL_PORT=8080`;
> then one could open AG at http://sparql.somehost.net 
> and the orr-ont at http://somehost.net/orr-ont
>


### Use

Open http://localhost/orr-ont in your browser.


> my mac:
>
>    ```
>    open http://`docker-machine ip`/orr-ont
>    ```
>
