# xlr-hello-world

This project shows how to use Integration Server Gradle Plugin to run Digital.ai Release for testing purposes.
The configuration is the easiest to get the grasp what is the minimum configuration required to set it up.

![Release 10.2.2](./pics/release-10.2.2.png)

The configuration how to run Gradle task, can be found in `.github/workflows/build.yaml`.

If you are going to configure it on a bare VM, you have to be sure you have the next things installed:

* Java 11
* Docker
* Docker Compose 

The main piece of integration server configuration is specified in this block in `build.gradle` file:

```groovy
releaseIntegrationServer {
    servers {
        controlPlane {
            dockerImage = "xebialabs/xl-release"
            version = "10.2.2"
            httpPort = 3333
        }
    }
}
```

which is enabled by applying the plugin by 

```groovy
apply plugin: "integration.server"
```

To be able to use it you have to have it in a classpath

```groovy
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath("com.xebialabs.gradle.plugins:integration-server-gradle-plugin:10.4.0-1019.1102") // Check in your case what is the latest version now.
    }
}
```

To run the example, execute this command: `./gradlew clean startIntegrationServer` 

The brief explanation of each property for `releaseIntegrationServer`:

| Name | Description | 
| :---: | :---: |
`servers` | section describes all servers that you can run. Currently only the first one will take an effect, and the rest will be ignored.
`controlPlane` | is the name of this server. This name won't be visible anywhere, you only have to specify it if you have  to read the configuration somewhere in gradle code. Like for example if you defined your own Gradle task below, and you want  to pass there the server http port, you can do it as: `integrationServer.server.controlPlane.httpPort`  
`dockerImage` | presence of this property makes the setup of Release be a container based, which will be pulled from the specified docker repository.
`version` | here you specify the version of Release you want to spin up. 
`httpPort` | is an optional field, if you don't specify it, the random port will be used. It's a good option when you run several tests in parallel on the same computer, and you don't want them to clash by using the same port.

if you want to check logs of Release:

* run `docker ps` to find the container id 

![Docker PS](./pics/docker-ps.png)

In my example I stopped all other containers, and have only one running, in your case you can have a list, so you have 
to find the one which has the image name "xebialabs/xl-release:10.2.2".

* run `docker logs <CONTAINER ID> -f` and you'll see something like that:

![Release logs](./pics/release-logs.png)

You can pay attention to this message `You can now point your browser to 'http://localhost:5516/'`.
Take into account that the host name in this case shown for the host inside of the container. 
You have to go instead to a http://localhost:3333 as this redirection has configured in Docker compose as we saw it from
`docker ps` in PORTS column.

One of the options how you can shut down the running docker image and unmount all volumes is to run this command:
`./gradlew shutdownIntegrationServer`.

Docker based setup uses End User License Agreement, you can read more about it here: https://hub.docker.com/r/xebialabs/xl-release/.
The license of this docker container will be valid only for 7 days after the downloading.
