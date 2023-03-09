# KeyCloak and Docker-Compose example part 2

## Start

- Generate a certificate for keycloak and for haproxy and distribute it to all members:

    ~~~shell
    keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore keycloak.keystore
    openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout haproxy.key -out haproxy.crt -subj "/CN=localhost"
    cat haproxy.key haproxy.crt > haproxy.pem
    ~~~

- edit `.env`
- adapt the `haproxy.cfg` file 
- Before the first run create the keycloak DB in the patroni cluster (look at [./Create-DB.sh](./Create-DB.sh))
- Now you can start the keycloak compose on all 3 machines with `docker compose up -d`

---
Peter Pflägging <<peter@pflaeging.net>>
