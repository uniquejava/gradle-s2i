# Gradle S2I OpenShift builder

https://docs.openshift.com/online/pro/using_images/s2i_images/java.html

https://stackoverflow.com/questions/42297387/docker-build-with-build-arg-with-multiple-arguments

## Build from scratch

```
$ oc new-project gradle-s2i-builder
$ oc new-build \
    --name gradle-s2i \
    --build-arg GRADLE_VERSION=7.2 \
    --build-arg OLD_S2I_PATH=/usr/local/s2i \
    --context-dir docker \
    -i openshift/java:openjdk-11-el7 \
    --strategy docker \
    https://github.com/uniquejava/gradle-s2i

# 然后可以去Builds -> Builds 里查看build的日志

$ oc delete all -l build=gradle-s2i

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
$ oc delete all -l app=springboot-gradle-s2i
```

## OpenShift 自带的 ImageStream

如何确定`oc new-build -i` 的值有哪些？

Administrator: Builds -> Images Streams

![How to get image stream name](docs/assets/image%20stream%20name.png)

The argument "java:openjdk-11-rhel7" could apply to the following Docker images, OpenShift image streams, or templates:

- Image stream "java" (tag "latest") in project "openshift"
  Use --image-stream="openshift/java:latest" to specify this image or template

- Image stream "java" (tag "openjdk-11-el7") in project "openshift"
  Use --image-stream="openshift/java:openjdk-11-el7" to specify this image or template

- Image stream "java" (tag "openjdk-11-ubi8") in project "openshift"
  Use --image-stream="openshift/java:openjdk-11-ubi8" to specify this image or template

- Image stream "java" (tag "openjdk-8-el7") in project "openshift"
  Use --image-stream="openshift/java:openjdk-8-el7" to specify this image or template

- Image stream "java" (tag "openjdk-8-ubi8") in project "openshift"
  Use --image-stream="openshift/java:openjdk-8-ubi8" to specify this image or template
