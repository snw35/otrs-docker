# otrs-docker
Docker compose files for a fully-functional OTRS deployment.

This repository contains the docker-compose files that will deploy my OTRS containers as a working setup. The fastest method of deployment is via the docker-compose.yml file at the top level of the repository, which will pull the pre-built images from Dockerhub.

## Deployment

### Short Version

```
docker volume create otrs-db
docker volume create otrs-config
vi ./pgpassword.txt
(enter a database password)
(download the docker-compose.yml file from this repo)
docker-compose up -d
```

Navigate to http://localhost/otrs/installer.pl

For database settings, use 'otrs-db' as host, 'postgres' as user, and the password you placed in the file.


### Long Version

To deploy these containers via docker compose, you will first have to create the required volumes. This is because these volumes are marked as external in docker compose, which is required for them to persist beyond restarts or shutdowns of the environment (docker-compose down commands, host restart, etc). These will hold your database and configuration files.

```
docker volume create otrs-db
docker volume create otrs-config
```

You then need to create a file called pgpassword.txt and enter a password to use for the database:

```
vi ./pgpassword.txt
```

You can then download the docker-compose.yml file into the same directory and start it up:

```
(place docker-compose.yml file in same directory)
docker-compose up -d
```

You should now have a working OTRS system at its default state. Navigate to http://localhost/otrs/installer.pl to set your installation up. Use "otrs-db" as the database host, "postgres" as the user, and the password that you placed in the pgpassword.txt file as the database password.

NOTE: The warning about the OTRS daemon not running should disappear after 5 minutes or so. The message may persist the first time you bring the environment up due to the daemon attempting to start before the database exists. If this happens, simply restart it with "docker-compose down" and "docker-compose up -d".

## Custom skins

Coming soon.


## Next Steps

From here, you can configure an SSL proxy to forward traffic to the otrs-nginx host, edit the docker-compose file to use your own nginx container instead (perhaps with SSL / letsencrypt etc).
