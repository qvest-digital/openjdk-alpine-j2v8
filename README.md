# openjdk-alpine-j2v8

Openjdk-alpine-j2v8 is an extension to the [openjdk](https://github.com/docker-library/openjdk) Alpine flavored Docker image, allowing to run [J2V8](https://github.com/eclipsesource/J2V8) Java applications on Alpine systems. This is achieved by recompiling the J2V8 bindings with musl in Alpine.

This image comes both in a JRE and JDK variant, with respective tags. For details look at the [Tags](https://hub.docker.com/r/tarent/openjdk-alpine-j2v8/tags/) section.

## Why?

The official maven artifact for J2V8 in linux flavor is compiled against glibc. This is problematic, since Alpine runs not on glibc but musl. And of course, the two are not necessarily compatible. While it should be possible to get the V8 binding running on Alpine via compatibility packages as for example [andyshinn/alpine-pkg-glibc](https://github.com/sgerrand/alpine-pkg-glibc), this does not work with OpenJDK (at least [openjdk:8-jdk-alpine](https://github.com/docker-library/openjdk)). The problem is that said image pulls OpenJDK from the Alpine package manager, installing several musl libraries (e.g. libstdc++) on the way. This leads the system to mix both glib and musl dependencies while trying to load the V8 binding. [And chaos ensues...](https://github.com/sgerrand/alpine-pkg-glibc/issues/48)

It is worth mentioning that there is [frolvlad/alpine-oraclejdk8](https://github.com/frol/docker-alpine-oraclejdk8) as an alternative. However, this uses OracleJDK instead of OpenJDK. While the legal situation for the [former might be problematic](http://blog.takipi.com/running-java-on-docker-youre-breaking-the-law/) depending on your scenario, its definitely bigger than [openjdk:8-jdk-alpine](https://github.com/docker-library/openjdk/blob/master/8-jdk/alpine/Dockerfile) and [openjdk:8-jre-alpine](https://github.com/docker-library/openjdk/blob/master/8-jre/alpine/Dockerfile) in any case.

## How?

This image at its core recompiles the J2V8 binding, and stuffs it into the JDK lib folder. This works, because at its core J2V8 tries to load the bindings lib from the [platform library path first](https://github.com/eclipsesource/J2V8/blob/c97922e95c3ba7d6e544f81bcf6bf14b788e03a0/src/main/java/com/eclipsesource/v8/LibraryLoader.java#L53). If nothing can be found, the packaged lib is extracted from the jar and then loaded.

Another point why this works is that both with glib and musl compiled bindings have the same name, as for J2V8 its just "linux".

## Next?

As it is unknown if J2V8 will implement a linux-musl flavor (probably not), it is hard to say how relevant this image will stay in the future. However, as it stands it has a huge advantage: It allows to run any regular Java application using V8, meaning that you don't have to make any code modifications to transition from a glibc development environment (or ci platform for that matter) to a slim Alpine JDK image.
