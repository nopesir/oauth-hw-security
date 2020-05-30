#### Stage 1: Build the application
FROM openjdk:8-jdk-alpine as build

# Retrieve the secrets from docker-compose-build
# Google
ARG client_id_g
ARG client_secret_g

# Facebook
ARG client_id_f
ARG client_secret_f

# GitHub
ARG client_id_git
ARG client_secret_git

# Install envsubst on Alpine sh
RUN apk add gettext

# Set the current working directory inside the image
WORKDIR /app

# Copy maven executable to the image
COPY mvnw .
COPY .mvn .mvn

# Copy the pom.xml file
COPY pom.xml .

# Enable permissions on Maven executable
RUN chmod u+x ./mvnw

# Build all the dependencies in preparation to go offline. 
# This is a separate step, the dependencies will be cached unless 
# the pom.xml file has changed.
RUN ./mvnw dependency:go-offline -B

# Copy the project source
COPY src src


RUN envsubst '\$client_id_g \$client_id_f \$client_id_git \$client_secret_g \$client_secret_f \$client_secret_git' < src/main/resources/application.yml.template > src/main/resources/application.yml

RUN cat src/main/resources/application.yml
# Package the application
RUN ./mvnw package -DskipTests

# Clear the source code already built (the secrets too)
RUN rm -R src

# Clear unused files
RUN rm -R .mvn
RUN rm mvnw

RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

#### Stage 2: A minimal docker image with command to run the app 
FROM openjdk:8-jre-alpine

ARG DEPENDENCY=/app/target/dependency

# Copy project dependencies from the build stage
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

ENTRYPOINT ["java","-cp","app:app/lib/*","com.example.springsocial.SpringSocialApplication"]