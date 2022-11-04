
### - Manual installation 

- Create a virtual environment and install the application dependencies:

```console
$ poetry shell && poetry install
```

- Configure your environment variables:

```dotenv
BASE_PATH = /tmp/fastgeoapi
DATABASE_PATH = /tmp/fastgeoapi/database.json
GITHUB_ACCESS_TOKEN=`cat ~/.github_access_token`
GITHUB_USERNAME=<your_github_username>
GITHUB_URL=<your_github_url where the authorization code lives>
CLIENT_ID=<your_github_client_id>
CLIENT_SECRET=<your_github_client_secret>
SECRET_KEY=<your_secret_key>
ALGORITHM=<your_algorithm e.g HS256>
ENVIRONMENT=<your_environment e.g. production|development>
```

- Run the application from the entry point:

```console
$ python3 main.py
```

- Open [localhost:8000/docs](localhost:8000/docs) for API Documentation

### - Build the application image and start the container -

```console
$ Configure environment variables - the .env file
$ docker-compose -f docker-compose.dev.yml up -d

```

### - Pull the official image from Docker Hub and start the container 

```console
$ docker pull geobeyond/policy-manager-api
$ docker run geobeyond/policy-manager-api
```
