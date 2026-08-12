# ==============================
# Stage 1: Build Spring Boot app
# ==============================
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

# Copy Maven configuration first
COPY pom.xml .

# Download dependencies
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build application
RUN mvn package -DskipTests


# ==============================
# Stage 2: Run Spring Boot app
# ==============================
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Copy generated JAR
COPY --from=build /app/target/exam-portal-backend-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]