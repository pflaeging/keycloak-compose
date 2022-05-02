# KeyCloak and Docker-Compose

Quick startpoint to run keycloak in a containerized deployment. Normally kubernetes would be a better environment, but YMMV.

## install

- docker
- docker-compose

## start

### first time

Generate a certificate:

~~~shell
keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore keycloak.keystore
~~~

Check back the passwords and configs in [./docker-compose.yaml](./docker-compose.yaml).
Pay special attention to:

- ports => on which port should it be accessible (maybe better 443 than 8443 and 80 then 8080)
- look for matching password and users:
  - DB_PASSWORD == POSTGRES_PASSWORD
  - DB_USER == POSTGRES_USER

You have to backup to persistent storages:

- ./pg-data (the postgres database)
- ./keycloak.keystore (your TLS certificate keystore)

### run

- Start your workload with `docker-compose up -d`
- check the running system with `docker-compose ps`
- check the logs with `docker-compose logs` (`-f` for follow mode)
- stop with `docker-compose down`
- restart with `docker-compose restart`

---
Peter Pflägging <<peter@pflaeging.net>>
