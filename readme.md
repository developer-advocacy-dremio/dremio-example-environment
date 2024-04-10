## Lakehouse On Your Laptop

Some Blogs Covering Similar Exiercises:
- [Postgres --> Iceberg/Dremio --> Superset Dashboard](https://www.dremio.com/blog/from-postgres-to-dashboards-with-dremio-and-apache-iceberg/)
- [SQLServer --> Iceberg/Dremio --> Superset Dashboard](https://www.dremio.com/blog/from-sqlserver-to-dashboards-with-dremio-and-apache-iceberg/)
- [MongoDB --> Iceberg/Dremio --> Superset Dashboard](https://www.dremio.com/blog/from-mongodb-to-dashboards-with-dremio-and-apache-iceberg/)
- [AWS Glue --> Dremio --> Superset Dashboard](https://www.dremio.com/blog/bi-dashboards-with-apache-iceberg-using-aws-glue-and-apache-superset/)
- [Running Graph QUeries on Iceberg Tables with Dremio & Puppygraph](https://www.dremio.com/blog/run-graph-queries-on-apache-iceberg-tables-with-dremio-puppygraph/)

## Setting Up Your Environment

You can use the directions below to setup a full lakehouse environment (dremio/nessie/minio) with reflections enabled, if you just want to spin up Dremio alone you can use the following command:

```
docker run -p 9047:9047 -p 31010:31010 -p 32010:32010 -p 45678:45678 -e DREMIO_JAVA_SERVER_EXTRA_OPTS=-Dpaths.dist=file:///opt/dremio/data/dist -e DREMIO_JAVA_EXTRA_OPTS=-Ddebug.addDefaultUser=true -e SERVER_GC_OPTS=-XX:+UseG1GC --name dremio_latest dremio/dremio-oss:latest
```

#### With Nessie/Minio/Dremio

**Pre-Reqs:** Git, Docker & Docker-Compose installed

### Step 1 - The docker-compose.yml

Fork & Clone this Repo to your laptop: `git clone <repo_url>` 

The docker-compose.yml will define all the pieces you need in your lakehouse which will include:

- Nessie: Catalog with Git-Like functionality for Apache Iceberg tables

- Minio: S3 Compliant Object Storage software to act as our data lake storage.

- Dremio: Data Lakehouse platform to provide an easy to use and fast point of access for the Apache Iceberg tables stored on Nessie/Minio and other sources we connect.

Open up a terminal in the same folder as this `docker-compose.yml` file and run the command

```shell
# latest versions of docker-desktop
docker compose up

# older versions
docker-compose up
```

This will create all the containers specified in our `docker-compose.yml` if you ever need to shut them down in another terminal in the same folder just run:

```shell
docker compose down
## or
docker-compose down
```

### Step 2 - Setting up our storage bucket

- Open up an internet browser

- Visit minio at `http://localhost:9001`

- login with the username: `admin` and the password: `password` (these were specified in the `docker-compose.yml`)

- Create a bucket, let's call it `warehouse`

### Step 3 - Connect Dremio to Data Lake

- Open up a new internet browser tab

- Visit Dremio at `http://localhost:9047`

- Fill out the form to create your account

- Then on the dashboard choose to connect a new source

- Select Nessie as your new source

There are two sections we need to fill out, the **general** and **storage** sections:

##### General (Connecting to Nessie Server)
- Set the name of the source to “nessie”
- Set the endpoint URL to “http://catalog:19120/api/v2”
Set the authentication to “none”

##### Storage Settings 
##### (So Dremio can read and write data files for Iceberg tables)

- For your access key, set “admin”
- For your secret key, set “password”
- Set root path to “/warehouse”
    Set the following connection properties:
    - `fs.s3a.path.style.access` to `true`
    - `fs.s3a.endpoint` to `storage:9000`
    - `dremio.s3.compat` to `true`
- Uncheck “encrypt connection” (since our local Nessie instance is running on http)

### Testing it Out

- Head to the SQL Runner on Dremio

- Run the following SQL statements

```
CREATE TABLE nessie.names (name varchar);
INSERT INTO nessie.names VALUES ('Gnarly the Narwhal');
SELECT * FROM nessie.names;
```

- Go explore you storage on minio, you should see all the Apache Iceberg data & metadata stored in your warehouse bucket.