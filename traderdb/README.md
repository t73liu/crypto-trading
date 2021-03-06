## Trader DB

This directory contains migrations for a PostgreSQL database called `traderdb`.
This database contains financial information (e.g. OHLC, quarterly reports) and
user information (e.g. watchlist, positions).

### Development

Download [Docker](https://www.docker.com/) and run the following commands in order
to bring up a PostgreSQL database.

```shell
# Pull the PostgreSQL docker image
docker pull postgres:12-alpine

# Run the PostgreSQL docker
docker run --detach \
 --name traderdb \
 --env POSTGRES_PASSWORD=test \
 --volume $HOME/pgdata/:/var/lib/postgresql/data \
 --publish 5432:5432 \
 postgres:12-alpine
 
# Create PostgreSQL DB
docker exec -it traderdb createdb --username=postgres traderdb

# Access PSQL CLI
docker exec -it traderdb psql --username=postgres traderdb
```

Use the corresponding Go scripts in order to populate the DB:
- `stocks`: `${TRADING_BOT_REPO}/jobs/populate-stocks/main.go`
- `stock_candles`: `${TRADING_BOT_REPO}/jobs/populate-db/stock_candles.go`
- `users`: `${TRADING_BOT_REPO}/adhoc/populate-db/users.go`

### Migrations

Migrations are done with the [migrate](https://github.com/golang-migrate/migrate)
CLI. Usage instructions can be found [here](https://github.com/golang-migrate/migrate/tree/master/cmd/migrate).

```shell
# Example migrating local DB with user = "postgres" and password = "test"
cd ${TRADING_BOT_REPO}
migrate -path traderdb/migrations \
 -database "postgres://postgres:test@localhost:5432/traderdb?sslmode=disable" \
 up 1

# Fix failed migration manually and force correct migration version number (e.g. 4)
migrate -path traderdb/migrations \
 -database "postgres://postgres:test@localhost:5432/traderdb?sslmode=disable" \
 force 4
```

### Backup

The database should be routinely backed up.

```shell
# Generates SQL commands to recreate the DB
docker exec -it traderdb pg_dump -U postgres traderdb > backup.sql

# Create and restore the DB from SQL dump
docker exec -it createdb --template=template0 --username=postgres traderdb
docker exec -it psql --single-transaction traderdb < backup.sql
```
