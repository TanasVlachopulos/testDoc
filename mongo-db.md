# Mongo DB

## Run local test Mongo DB

```text
docker run -d -it --name meditter-auth-mongo \
    --restart unless-stopped \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
    -e MONGO_INITDB_ROOT_PASSWORD=secret \
    mongo
```

One-liner:

```text
docker run -d -it --name meditter-auth-mongo --restart unless-stopped -p 27017-27019:27017-27019 -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=secret -e MONGO_INITDB_DATABASE=meditter-auth-db mongo
```

