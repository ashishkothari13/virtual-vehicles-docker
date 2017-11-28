# Continuous Integration and Delivery of Microservices

_Continuous Integration and Delivery of Microservices-based REST API with RestExpress, Java EE, and MongoDB, using Jenkins CI, Docker Machine, and Docker Compose._

__PROJECT CODE UPDATED 11-09-2016 to v4.3.0__

## Introduction

In the below series of posts, we learned how to use Jenkins CI, Maven, Docker, Docker Compose, and Docker Machine to take a set of Java-based microservices from source control on GitHub, to a fully tested set of integrated Docker containers running within an Oracle VirtualBox VM. We performed integration tests, using a scripted set of synthetic transactions, to make sure the microservices were functioning as expected, within their containers.

[![ELK Stack 3D Diagram](https://programmaticponderings.files.wordpress.com/2015/08/elk-stack-3d-diagram-1.png?w=620)](https://programmaticponderings.files.wordpress.com/2015/08/elk-stack-3d-diagram-1.png)

## Reference Blog Posts

- [Containerized Microservice Log Aggregation and Visualization using ELK Stack and Logspout](http://wp.me/p1RD28-1wl)
- [Continuous Integration and Delivery of Microservices using Jenkins CI, Docker Machine, and Docker Compose](http://wp.me/p1RD28-1uZ)
- [Building a Microservices-based REST API with RestExpress, Java EE, and MongoDB: Part 3](http://wp.me/p1RD28-1sc)

## Build Test Environment Project

```bash
# check for latest versions of required apps
docker -v && docker-compose -v && \
docker-machine -v && VBoxManage --version

# pull this GitHub project
git clone https://github.com/garystafford/virtual-vehicles-docker.git && \
cd virtual-vehicles-docker

# clean up any previous machine failures
docker-machine stop test || echo "nothing to stop" && \
docker-machine rm test   || echo "nothing to remove"

# use docker-machine to create and configure 'test' environment
docker-machine --debug create --driver virtualbox test
eval "$(docker-machine env test)"

# pull build artifacts from virtual-vehicles-demo project
# build (4) Dockerfiles and docker-compose.yml from templates
sh pull_and_build.sh

# use docker-compose to pull and build new images and containers
# this will take up to 20 minutes or more to pull images
docker-compose -p test up -d

# list machines, images, and containers
# check all containers are running
docker-machine ls && \
docker images && \
docker ps -a

# wait for containers to fully start before tests fire up
sleep 20

# add local dns name to hosts file for demo (mac-friendly)
sudo -- sh -c -e "echo '$(docker-machine ip test)   api.virtual-vehicles.com' >> /etc/hosts";

# test the services
sh tests_color.sh api.virtual-vehicles.com

# alternate, test the services using IP address:
# sh tests_color.sh $(docker-machine ip test)

# delete all images and containers
docker rmi -f $(docker images -a -q) && \
docker rm -f $(docker ps -a -q)

# alternate, complete tear down: stop and remove 'test' environment when complete
docker-machine stop test && \
docker-machine rm test
eval "$(docker-machine env -u)"

# other useful commands
# clean up orphaned volumes
docker volume rm $(docker volume ls -qf dangling=true)

# clean up orphaned networks
docker network rm $(docker network ls -q)

# delete all project images, but not base images
docker rmi -f $(docker images | grep 'test_' | awk '{print $3}')

# delete all <none> images
docker rmi -f $(docker images | grep '^<none>' | awk '{print $3}')
```

### Test Results

[![Integration Tests](https://programmaticponderings.files.wordpress.com/2015/08/integration-tests1.png?w=620)](https://programmaticponderings.files.wordpress.com/2015/08/integration-tests1.png)

## Browse the Project

- NGINX: <http://api.virtual-vehicles.com>
- NGINX: <http://api.virtual-vehicles.com/nginx_status>
- Kibana: <http://api.virtual-vehicles.com:8200>
- Elasticsearch: <http://api.virtual-vehicles.com:9200>
- Elasticsearch: <http://api.virtual-vehicles.com:9200/_status?pretty>
- Logspout: <http://api.virtual-vehicles.com:8000/logs>
- Graphite: <http://api.virtual-vehicles.com:8500>

## Environment Architecture

[![ELK Ports](https://programmaticponderings.files.wordpress.com/2015/07/elk-ports.png?w=620)](https://programmaticponderings.files.wordpress.com/2015/07/elk-ports.png)
