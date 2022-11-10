
### - Manual installation 

- Configure your environment variables:

```dotenv
# Base path to clone the rego repository to
BASE_PATH = /tmp

# Path to the directory where the TinyDB database is stored.
DATABASE_PATH = /tmp/database.json

ORG_NAME = geobeyond/policies       # Organization name identifies individual clients on Gitlab.
ENVIRONMENT=<your_environment e.g. production|development>

# A list of super_users that can access the admin functionality.
super_user=["<github_username|gitlab_username>"]

# Postgres database configuration to retrieve group names from.
DB_USER=geostore
PASSWORD=geostore
DATABASE=geostore
HOST=db
PORT=5432


```

- Create a virtual environment and install the application dependencies:

```console
$ poetry shell && poetry install
```



- Run the application from the entry point:

```console
$ python3 main.py
```

- Open [localhost:8000/docs](localhost:8000/docs) for API Documentation

### - Build the application image and start the container 

- Configure environment variables as mentionaed above.

- Build the image with docker-compose:

    ```console
    $ docker-compose -f docker-compose.dev.yml up -d

    ```

### -  Pull the official image from Docker Hub and start the container 

```console
$ docker run geobeyond/policy-manager-api
```
