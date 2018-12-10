# Eureka Microservices Example

This repository is one of a collection of microservices and Eureka components. This is a Eureka service registry which work with the following repositories.
* [eureka-microservices-zuul](https://github.com/bernardpark/eureka-microservices-zuul) - Zuul implementation
* [eureka-microservices-guestbook](https://github.com/bernardpark/eureka-microservices-guestbook) - Sample "guestbook" microservice using a Redis backend


## Getting Started

Clone this repository to an environment of your choice.

```bash
git clone https://github.com/bernardpark/eureka-microservices-demo.git
git clone https://github.com/bernardpark/eureka-microservices-zuul.git
git clone https://github.com/bernardpark/eureka-microservices-guestbook.git
```

## Running Locally

### Prerequisites
Ensure that you have the following

1. JDK 1.8.x
1. Maven
1. Redis 4.0.x

### Start services
You can run this eureka implementation by running the following commands.

```bash
redis-server
mvn clean package -f eureka-microservices-demo/pom.xml
mvn clean package -f eureka-microservices-zuul/pom.xml
mvn clean package -f eureka-microservices-guestbook/pom.xml
java -jar /eureka-0.0.1-SNAPSHOT.jar
java -jar /zuul-0.0.1-SNAPSHOT.jar
java -jar /guestbook-0.0.1-SNAPSHOT.jar
```

### Verify Guestbook Service
See the API endpoints for this service [here](https://github.com/bernardpark/eureka-microservices-guestbook)
Try this curl command to make sure your service is running and bound to your Redis backend.

```bash
curl localhost:8080/add \
  -d '{"firstName":"Hello","lastName":"World","email":"hi@there.com","areYouLate":true}' \
  -H "Content-Type: application/json" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -X POST
```

You should recieve an HTTP 200 response.

### Verify Service Registry
In your browser, navigate to `localhost:8761` to see your service registry and registered services. You should see `GUESTBOOK` as a registered service.

### Verify Zuul Gateway
The Zuul instance should uses the `/api` prefix and maps the `/guestbook/` path to the guestbook microservice. In your browser, visit `localhost:8762/api/guestbook/find` to see if you can access the guestbook service through Zuul. If successful, you should see a json response of the guestbook entry you added earlier.


## Running in PCF

### Prerequisites
Ensure that you have the following

1. JDK 1.8.x
1. Maven
1. CF CLI 6.40.x

### Push Services
After targeting your PCF Foundation, run these commands to push your services while replacing the variables.

```bash
# Push Services
cd eureka-microservices-demo; mvn clean package; cf push; cd ..
cd eureka-microservices-zuul; mvn clean package; cf push; cd ..
cd eureka-microservices-guestbook; mvn clean package; cf push; cd ..

# Create and Bind Services
cf create-service $REDIS_MARKETPLACE_SERVICE $REDIS_PLAN pcf-ms-redis
cf bind-service pcf-ms-guestbook pcf-ms-redis
cf restage pcf-ms-guestbook

# Add Whitelist Network Policies
cf add-network-policy pcf-ms-guestbook --destination-app pcf-ms-demo --protocol tcp --port 8761
cf add-network-policy pcf-ms-zuul --destination-app pcf-ms-demo --protocol tcp --port 8761
cf add-network-policy pcf-ms-zuul --destination-app pcf-ms-guestbook --protocol tcp --port 8080
```

### Verify Guestbook Service
See the API endpoints for this service [here](https://github.com/bernardpark/eureka-microservices-guestbook)
Try this curl command to make sure your service is running and bound to your Redis backend. Again, make sure you change the variables.

```bash
curl https://pcf-ms-guestbook.$YOUR_APP_DOMAIN/add \
  -k \
  -d '{"firstName":"Hello","lastName":"World","email":"hi@there.com","areYouLate":true}' \
  -H "Content-Type: application/json" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -X POST
```

You should recieve an HTTP 200 response.

### Verify Service Registry
In your browser, navigate to `https://pcf-ms-demo.$YOUR_APP_DOMAIN` to see your service registry and registered services. You should see `GUESTBOOK` as a registered service.

### Verify Zuul Gateway
The Zuul instance should uses the `/api` prefix and maps the `/guestbook/` path to the guestbook microservice. In your browser, visit `https://pcf-ms-zuul/api/guestbook/find` to see if you can access the guestbook service through Zuul. If successful, you should see a json response of the guestbook entry you added earlier.


## Authors
* **Bernard Park** - [Github](https://github.com/bernardpark)
