FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean compile exec:java -q

FROM nginx:alpine

COPY --from=builder /app/output /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
