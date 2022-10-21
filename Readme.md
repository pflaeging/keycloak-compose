# KeyCloak and Docker-Compose

Quick startpoint to run keycloak in a containerized deployment. Normally kubernetes would be a better environment, but YMMV.

This setup is tested with KeyCloak 19.0.1 (quarkus edition) and PostgreSQL 14 

## Install

- docker
- docker-compose

Example AlmaLinux (RHEL Clone):

~~~shell
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf remove podman buildah
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl enable docker.service --now
~~~


## Preface

we have 3 virtual machines in our testcase:

- DB test server (machine1)
- KeyCloak cluster member (machine2 and machine3)

This repo helps you to deploy a KeyCloak cluster with infinispan based on docker compose.
We will use machine1 as DB server with postgres. You should use a redundant installation (either as docker, on k8s or as vm's). The other 2 machines implement a 2 node KeyCloak cluster with infinispan (with jdbc-ping-tcp).
You have to implement a load-balancer before the 2 nodes. It's possible to work with reencrypt (port 18443 default) or you can offload the TLS on the load-balancer (http on port 18080 default).

## Start

Generate a certificate and distribute it to all members:

~~~shell
keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore keycloak.keystore
~~~

Check back the passwords and configs in [./docker-compose.yaml](./docker-compose.yaml).
Pay special attention to:

- ports => on which port should it be accessible
- look for matching password and users:
  - KC_DB_PASSWORD (docker-compose.yaml) == POSTGRES_PASSWORD (docker-compose-postgres.yaml)
  - KC_DB_USERNAME (docker-compose.yaml) == POSTGRES_USER (docker-compose-postgres.yaml)

Now you can start on 3 machines (for my test I'm not using redundant DB's. You should use them in production)

- machine1 (DB-test machine)

    ~~~shell
    docker compose -f docker-compose-postgres.yml up
    ~~~

- machine2 (KeyCloak cluster member 1)

    ~~~shell
    # replace my.database.server.net in docker-compose.yml with your DB-machine (machine1 in our example)
    export MYIP=192.168.33.11 # insert your IP address for the first machine
    docker compose up
    ~~~

- machine3  (KeyCloak cluster member 1)

    ~~~shell
    # replace my.database.server.net in docker-compose.yml with your DB-machine (machine1 in our example)
    export MYIP=192.168.99.33 # insert your IP address for the first machine
    docker compose up
    ~~~

You have to backup to persistent storages:

- `./pg-data` (the postgres database) on the DB-machine
- `./keycloak.keystore` (your TLS certificate keystore)


## HaProxy config

Here's a haproxy container config to test with:

1. Generate a cert-file for haproxy:

    ~~~shell
    openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout haproxy.key -out haproxy.crt -subj "/CN=localhost"
    cat haproxy.key haproxy.crt > haproxy.pem
    ~~~

1. Adapt your config in `haproxy.cfg` (mainly the keycloaks you want to connect to under `backend keycloak_backend`)

1. Start aproriate docker-compose on your gateway machine:

    ~~~shell
    docker compose -f docker-compose-haproxy.yml up
    ~~~

There's a small example for testing haproxy -> keycloak only in `docker-compose-combi.yml`.

---
Peter Pflägging <<peter@pflaeging.net>>
