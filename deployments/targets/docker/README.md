# Orekit Docker Build

This Dockerfile builds Orekit from source inside a Docker container, including the required Hipparchus dependency.

## What This Dockerfile Does

1. **Installs dependencies**: Git and curl in a Maven/JDK base image
2. **Builds Hipparchus**: Clones and builds the Hipparchus 4.1-SNAPSHOT dependency (required for Orekit 14.0-SNAPSHOT)
3. **Builds Orekit**: Compiles and packages Orekit

## Prerequisites

- Docker installed on your system
- Internet connection to download dependencies

## Building the Image

From the root of the Orekit repository, run:

```bash
docker build -f deployments/targets/docker/ContainerFile -t orekit:14.0-SNAPSHOT .
```

Or if you're in the docker directory:

```bash
cd deployments/targets/docker
docker build -f ContainerFile -t orekit:14.0-SNAPSHOT ../../..
```

## Build Arguments

The Dockerfile supports the following build arguments for proxy configuration:

```bash
docker build \
  --build-arg HTTP_PROXY=http://proxy.example.com:8080 \
  --build-arg HTTPS_PROXY=http://proxy.example.com:8080 \
  --build-arg MAVEN_HTTPS_PROXY=http://proxy.example.com:8080 \
  -f deployments/targets/docker/ContainerFile \
  -t orekit:14.0-SNAPSHOT \
  .
```

## Extracting the Built JAR

After the build completes, you can extract the built JAR file from the container:

```bash
# Create a temporary container
docker create --name orekit-temp orekit:14.0-SNAPSHOT

# Copy the JAR file out
docker cp orekit-temp:/orekit/target/orekit-14.0-SNAPSHOT.jar .

# Remove the temporary container
docker rm orekit-temp
```

## Running the Container

To run the container and explore the built artifacts:

```bash
docker run -it orekit:14.0-SNAPSHOT /bin/bash
```

Inside the container, the Orekit JAR will be located at:
```
/orekit/target/orekit-14.0-SNAPSHOT.jar
```

## Build Time

The build process can take several minutes depending on your system:
- Hipparchus build: ~3-5 minutes
- Orekit build: ~5-10 minutes

## Troubleshooting

### Build Failures

If the build fails, check:
1. You have sufficient disk space
2. Your internet connection is stable
3. If behind a corporate proxy, ensure proxy settings are correctly configured

### Test Failures

The Dockerfile skips tests during the Hipparchus build (`-DskipTests`) to avoid minor test failures that don't affect functionality. Orekit tests run by default during `mvn package`.

To skip Orekit tests as well, modify the Dockerfile:
```dockerfile
RUN mvn package -DskipTests
```

## Notes

- This Dockerfile is specifically for building Orekit 14.0-SNAPSHOT which depends on Hipparchus 4.1-SNAPSHOT
- For released versions of Orekit, you may not need to build Hipparchus separately
- The base image uses Eclipse Temurin JDK 17 with Maven 3.9.9
