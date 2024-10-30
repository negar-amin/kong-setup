## Run Kong 3.2 with Docker Containers


#### Create Docker Network
docker network create kong-net

# Export License
export KONG_LICENSE_DATA=''

### Start a Postgres Container (If running db mode)
docker run -d --name kong-database \
  --network=kong-net \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  -e "POSTGRES_PASSWORD=kongpass" \
  postgres:13.9

# Start migration
docker run --rm --network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kongpass" \
kong/kong-gateway:3.2.1.0 kong migrations bootstrap

# Start Kong
docker run -d --name kong-gateway \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_USER=kong" \
-e "KONG_LOG_LEVEL=debug" \
-e "KONG_PG_PASSWORD=kongpass" \
-e "KONG_PLUGINS=bundled" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
-e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
-e "KONG_PORTAL=on" \
-e "KONG_PORTAL_GUI_HOST=localhost:8003" \
-e "KONG_PORTAL_GUI_PROTOCOL=http" \
-e "KONG_OPENTELEMETRY_TRACING=all" \
-e "KONG_OPENTELEMETRY_TRACING_SAMPLING_RATE=1.0" \
-e "KONG_LICENSE_DATA" \
-p 8000:8000 \
-p 8443:8443 \
-p 8001:8001 \
-p 8444:8444 \
-p 8002:8002 \
-p 8445:8445 \
-p 8003:8003 \
-p 8004:8004 \
kong/kong-gateway:3.2.1.0

docker run --name jaeger -d --network=kong-net \
  -e COLLECTOR_OTLP_ENABLED=true \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  jaegertracing/all-in-one:1.36


# To Tear down
docker kill kong-gateway

docker kill kong-database

docker kill jaeger

docker container rm kong-gateway

docker container rm kong-database

docker container rm jaeger

docker network rm kong-net
