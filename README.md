# Gradle S2I OpenShift builder

https://docs.openshift.com/online/pro/using_images/s2i_images/java.html

redhat-openjdk18-openshift (RHEL 7, JDK 8)

ubi8-openjdk-11 (RHEL UBI 8, JDK 11)

ubi8-openjdk-8 (RHEL UBI 8, JDK 8)

openjdk-11-rhel7 (RHEL 7, JDK 11)

## Build from scratch

```
$ export GRADLE_VERSION=7.2
$ export OPENJDK_IMAGE_STREAM_VERSION=1.10
$ oc new-project gradle-s2i-builder
$ oc new-build \
    --name gradle-s2i \
    --build-arg GRADLE_VERSION=$GRADLE_VERSION,OLD_S2I_PATH=/usr/local/s2i \
    --context-dir docker \
    -i openjdk-11-rhel7:$OPENJDK_IMAGE_STREAM_VERSION \
    --strategy docker \
    https://github.com/uniquejava/gradle-s2i

$ oc new-app \
    --name hello-gradle \
    --build-env SCRIPT_DEBUG=true \
    --context-dir complete \
    -i gradle-s2i \
    https://github.com/spring-guides/gs-spring-boot

$ oc new-app \
    --name hello-maven \
    --build-env BUILDER=maven \
    --context-dir complete \
    -i gradle-s2i \
    https://github.com/spring-guides/gs-spring-boot

```

## Build from template

```
$ oc new-project hello-gradle
$ oc create -f openshift/springboot-gradle-s2i-template-1.yaml
$ oc new-app springboot-gradle-s2i \
    -p IMAGE_STREAM_NAMESPACE hello-gradle
```
