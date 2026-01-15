Integration tests with docker-compose | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

When developing an application that uses a database, it's important to test your application against a real database. As good as your unit tests may be, some issues can only be caught by running proper integration tests.

If you use Atlas to manage your database schema, it only makes sense to use Atlas to prepare your database for integration tests as well. One way to achieve this is by using [docker-compose](https://docs.docker.com/compose/) to create a test environment for your tests to run against. Docker-compose allows you to easily spin up a test database and run your migrations against it, so you can verify that the application works as expected with the updated schema.

On a high-level, the process of setting up integration tests with docker-compose looks like this:

*   The database container is provisioned using docker-compose.
*   When the database is ready, Atlas is run to apply all existing migrations on this database prior to the tests running.
*   Integration tests are provided a connection string to the migrated, local database.

## Example[​](#example "Direct link to Example")


Suppose your project has the following directory structure:
```codeBlockLines_AdAo
.├── docker-compose.yaml└── migrations    ├── 20221207103204_init.sql    └── atlas.sum1 directory, 3 files
```
The `docker-compose.yaml` file looks like this:
```codeBlockLines_AdAo
version: "3.9"services:  mysql:    image: mysql:8.0.29    platform: linux/amd64    healthcheck:      test: mysqladmin ping -ppass    environment:      MYSQL_DATABASE: test      MYSQL_ROOT_PASSWORD: pass    ports:      - "3306:3306"    networks:      - db  migrate:    image: arigaio/atlas:latest    command: >      migrate apply      --url mysql://root:pass@mysql:3306/test    networks:      - db    depends_on:      mysql:        condition: service_healthy    volumes:      - ./migrations/:/migrationsnetworks:  db:
```
When you run `docker-compose up`, this is what happens:

*   The `mysql` service is started.
*   As it takes a while for the database to start, Docker will probe the container's healthcheck until it returns a success status.
*   Once the database is ready, the `migrate` service is started. Which runs the `atlas migrate apply` command to apply all migrations on the database.

After the migrations are applied, the database is ready to be used by your integration tests.

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In conclusion, using `docker-compose` to set up your integration tests allows you to easily spin up a test database and apply your migrations to it. This ensures that your application is tested against an up-to-date database schema, and allows you to catch any issues that may arise when running against a real database.

Have questions? Feedback? Find our team [on our Discord server](https://discord.gg/zZ6sWVg6NT).

*   [Example](#example)
*   [Wrapping up](#wrapping-up)