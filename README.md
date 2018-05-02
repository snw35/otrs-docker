# otrs-docker
Docker compose files for lightweight and efficient OTRS 6.

This repository contains the docker-compose files that will automate the deployment of my Alpine-based OTRS container images:

 * [otrs-fcgi - ](https://github.com/snw35/otrs-fcgi) [![Build Status](https://travis-ci.org/snw35/otrs-fcgi.svg?branch=master)](https://travis-ci.org/snw35/otrs-fcgi)
 * [otrs-daemon -](https://github.com/snw35/otrs-daemon) [![Build Status](https://travis-ci.org/snw35/otrs-daemon.svg?branch=master)](https://travis-ci.org/snw35/otrs-daemon)
 * [otrs-nginx - ](https://github.com/snw35/otrs-nginx) [![Build Status](https://travis-ci.org/snw35/otrs-nginx.svg?branch=master)](https://travis-ci.org/snw35/otrs-nginx)

These images follow the [official docker library image guidelines](https://github.com/docker-library/official-images) as closely as possible. They are designed with maintainability, simplicity, and security in mind.

There are two docker-compose files available:

 * fcgi-mariadb - Nginx with fast-cgi and MariaDB
 * fcgi-postgres - Nginx with fast-cgi and Postgres


## How to use

 1. Install docker and docker-compose if needed.
 1. Create required volumes:

    ```
    docker volume create otrs-db
    docker volume create otrs-config
    ```

 1. Create a file called dbpassword.txt and place a database password of your choosing inside.
 1. Download the appropriate docker-compose.yml file into the same directory.
 1. Start OTRS:

    ```
    docker-compose up -d
    ```

 1. Navigate to http://localhost/otrs/installer.pl

 1. Proceed through the installation. For database settings, use 'otrs-db' as host, 'postgres' as user if using posgres or 'root' if using mariadb, and the password you placed in the file.

    NOTE: The warning banner stating that the OTRS daemon is not running may persist the first time you bring the environment up due to the daemon attempting to start before the database exists. If this happens, simply go through the installation, then restart the environment with "docker-compose down" and "docker-compose up -d". The warning should disappear after 5 minutes.

## Considerations for production use

If you have tried the above and are thinking of deploying these images in production, then some additional considerations:

 * Enable and use Docker's [user namespace remapping](https://docs.docker.com/engine/security/userns-remap) feature for additional security.
 * Email attachments - OTRS stores these inside the database by default. This is fine for small deployments, but data sizes can grow quickly on medium or larger ones. Consider creating a new volume for attachments, mounting it into the otrs-fcgi container, and configuring OTRS to use that location to store attachment files instead.
 * Backups. You must, at minimum, back up the otrs-config volume contents and take a copy of the OTRS database from the postgres container.


## Custom skins

There are two custom skins pre-configured: one for the Agent interface, and one for the Customer interface. They are created empty and ready for you to place your own custom skin files inside. To use them:

 1. Create your skin and make sure it is in the correct folder layout.
 1. Copy it into the otrs-config volume through the running container:

    ```
    docker cp ./skin-folder/* otrs-fcgi:/data/custom-agent-skin/
    or
    docker cp ./skin-folder/* otrs-fcgi:/data/custom-customer-skin/
    ```

 1. Go into OTRS' sysconfig and make sure the 'custom-agent-skin' or 'custom-customer-skin' is enabled.
 1. Activate the skin in your preferences, and if it works, set it as the default in sysconfig.

## Upgrading from OTRS 5

Upgrading between major OTRS versions is not always straightforward and usually requires several manual steps. If you are currently running non-containerized OTRS 5 and are thinking of migrating to containerized OTRS 6, the best thing to do would be to manually upgrade your existing install to OTRS 6 first, and then migrate that to a containerized setup (described below).

## Migrating OTRS 6 from local install

If you have OTRS 6 running locally, then migrating to a containerized setup is straightforward:
 1. Back up your existing database and installation directory.
 1. Start the postgres or mariadb docker-compose deployment as appropriate.
 1. Restore your database into the database container using the appropriate method (pg_restore, mysql client, etc).
 1. Modify your config.pm to point to the database container and copy it into the otrs-config volume:

    ```
    docker cp ./Config.pm otrs-fcgi:/data/Config.pm
    ```

 1. Try the web interface and see your configuration comes up. If not, restart the environment and try again:

    ```
    docker-compose down
    docker-compose up -d
    ```
