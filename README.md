### To run mysql container to interactive mode use the below command:
   
    docker exec -it mysql mysql -u mysql -p
    enter password: <enter password here>
    Then you can use mysql in command line

### To Run SpringBoot application + MySQL and Docker:
#### Develop the spring boot application and dependencies like below* spring-boot-starter-data-jpa

    spring-boot-starter-web 
    mysql-connector-j   
    lombok

### Add .env file along with properties

	MYSQL_ROOT_PASSWORD=rootpass
	DB_NAME=ems_db
	DB_USER=ems_user
	DB_PASSWORD=ems_pass

### Add mysql configuration in application.yaml file like below

    spring:
        datasource:
            url: jdbc:mysql://${DB_HOST:mysql}:3306/${DB_NAME:ems_db}
            username: ${DB_USER:ems_user}
            password: ${DB_PASSWORD:ems_pass}
            driver-class-name: com.mysql.cj.jdbc.Driver
        jpa:
            show-sql: true
            hibernate:
                ddl-auto: update
            properties:
                hibernate:
                    format_sql: true
            database: mysql
            database-platform: org.hibernate.dialect.MySQLDialect

### Create Dockerfile like below

    FROM maven:3.9.12-eclipse-temurin-25 as builder
    WORKDIR /app
    COPY pom.xml .
    RUN mvn dependency:go-offline
    COPY src ./src
    RUN mvn clean package -DskipTests

	FROM eclipse-temurin:25-alpine
	WORKDIR /app
	COPY --from=builder /app/target/*.jar app.jar
	EXPOSE 8080
	ENTRYPOINT [“java","-jar","app.jar"]

### Create docker-compose.yam file

    version: '3.9'
    services:
        mysql:
            image: mysql:8.4
            container_name: mysql-db
            restart: always
            environment:
                MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
                MYSQL_DATABASE: ${DB_NAME}
                MYSQL_USER: ${DB_USER}
                MYSQL_PASSWORD: ${DB_PASSWORD}
            ports:
                - "3306:3306"
            volumes:
                - mysql_data:/var/lib/mysql
            networks:
                - app-network
        postgres:
            image: postgres:18
            container_name: postgres-db
            environment:
                POSTGRES_DB: ${POSTGRES_DB}
                POSTGRES_USER: ${POSTGRES_USER}
                POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
            ports:
                - "5432:5432"
            volumes:
                - postgres_data:/var/lib/postgresql
            networks:
                app-network:
        springboot-app:
            build: .
            container_name: sb-app
            restart: always
            depends_on:
                - postgres
            environment:
                SPRING_PROFILES_ACTIVE: postgres
                DB_HOST: postgres
                DB_NAME: ${POSTGRES_DB}
                DB_USER: ${POSTGRES_USER}
                DB_PASSWORD: ${POSTGRES_PASSWORD}
            ports:
                - "8080:8080"
            networks:
                - app-network
    volumes:
        mysql_data:
        postgres_data:
    networks:
        app-network:

### To run docker-compose file like below command:

    docker compose up -d —build

### Test API’s using CURL:

	curl -X GET http://localhost:8080/api/v1/employees/all
	
	curl -X GET http://localhost:8080/api/v1/employees/1
	
	curl -X POST http://localhost:8080/api/v1/employees
		-H “Content-Type: application/json”
		-d 	‘{
				“name” : “Madhav”,
				“email” : “madhav@gmail.com”,
				“department” : “IT”,
				“salary” : 60000.00
			}’
	curl -X DELETE http://localhost:8080/api/v1/employees/1