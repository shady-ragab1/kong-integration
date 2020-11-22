#Kong community edition with spring boot service
- OpenApi specs available under /docs
- Swagger interface available under /docs.html

###create network
- docker network create kong-micro


###run postgres DB container
- docker run -d --name kong-database --network=kong-micro -p 5432:5432 -e POSTGRES_USER=kong -e POSTGRES_DB=kong -e POSTGRES_PASSWORD=kong postgres:9.6


###prepare DB
- docker run --rm --network=kong-micro -e KONG_DATABASE=postgres -e KONG_PG_HOST=kong-database -e KONG_PG_PASSWORD=kong kong:latest kong migrations bootstrap


###run Kong
- docker run -d --name kong --network=kong-micro -e KONG_DATABASE=postgres -e KONG_PG_HOST=kong-database -e KONG_PG_PASSWORD=kong -e KONG_CASSANDRA_CONTACT_POINTS=kong-database -e KONG_PROXY_ACCESS_LOG=/dev/stdout -e KONG_ADMIN_ACCESS_LOG=/dev/stdout -e KONG_PROXY_ERROR_LOG=/dev/stderr -e KONG_ADMIN_ERROR_LOG=/dev/stderr -e KONG_ADMIN_LISTEN="0.0.0.0:8001, 0.0.0.0:8444 ssl" -p 8000:8000 -p 8443:8443 -p 127.0.0.1:8001:8001 -p 127.0.0.1:8444:8444 kong:latest


###create image : navigate to root folder and run the command(image name could be changed from pom.xml file)
- mvn spring-boot:build-image


###run container 
- docker run -d --name temperature --network=kong-micro -p 8080:8080 temperature


###run this command just to find the service IP in kong-micro network to be used in the next command
- docker network inspect kong-micro


###add service to kong (IP:172.19.0.4 from previous step)
- curl -i -X POST --url http://localhost:8001/services/ --data 'name=temperature' --data 'url=http://172.19.0.4:8080/v1/temperature'


###add route
- curl -i -X POST — url http://localhost:8001/services/temperature/routes/ --data 'name=temperature' --data 'paths[]=/temperature-v1'


###test service
- curl http://localhost:8000/temperature-v1/convert-to-fahrenheit/2000
- curl http://localhost:8000/temperature-v1/convert-to-celsius?fahrenheit=300
