# We seledt the base image from a locally available image or from https://hub.docker.com/
# FROM java:8   #  this is preaty bloated switching to anapsix/alpine-java
FROM anapsix/alpine-javaThen 

# We define the user we will us in this instance to prevent using root that is bad even
# in a container, this can be a security risk.
ENV APPLICATION_USER backend
ENV CLIENT byu
ENV SERVER_TYPE development
ENV SOURCE_DIR .
ENV BACKEND_APPLICATION_DIRECTORY /code/BYU
ENV BUILD_LIB ./build/libs
ENV APPLICATION_JAR byu-movie-api-application.jar



# add the user, create the /app folder and give permissions to our user.
RUN aduser -D -g  '' ${APPLICATION_USER}
RUN mkdir ${BACKEND_APPLICATION_DIRECTORY}
RUN chown -R ${APPLICATION_USER} ${BACKEND_APPLICATION_DIRECTORY}
# both ADD and VOLUME are on the same directory, as the container running in
# production won't have a volume with the project contents mounted, so they have
# to be added separly
ADD ${SOURCE_DIR} ${BACKEND_APPLICATION_DIRECTORY}
VOLUME ${BACKEND_APPLICATION_DIRECTORY}

# Marking this container to use the specified  $APPLICATION_USER
USER ${APPLICATION_USER}
EXPOSE 8080




ENV GRADLE_OPTS -Dorg.gradle.native-false
CMD ["./gradlew","bootRun"]

# Install maven
RUN apt-get update
RUN apt-get install -y maven

WORKDIR ${BACKEND_APPLICATION_DIRECTORY}

# Prepare by downloading dependencies
ADD pom.xml ${BACKEND_APPLICATION_DIRECTORY}/pom.xml 
RUN ["mvn", "dependency:resolve"]
RUN ["mvn", "verify"]

# Adding source, compile and package into a fat jar
ADD src /code/src
RUN ["mvn", "package"]

COPY ${BUILD_LIB}/${APPLICATION_JAR}

# We launch java to execute the jar, with good defauls intended for containers.
CMD ["java", "-server", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-XX:InitialRAMFraction=2", "-XX:MinRAMFraction=2", "-XX:MaxRAMFraction=2", "-XX:+UseG1GC", "-XX:MaxGCPauseMillis=100", "-XX:+UseStringDeduplication", "-jar", "my-application.jar"]
