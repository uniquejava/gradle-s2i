# Gradle S2I OpenShift builder

Fork 自：https://github.com/dsevost/gradle-s2i, 测试环境 `OpenShift 4.8.12`,

本 Fork(uniquejava) 做了如下更新：

### 1. 升级到 openjdk11 和 gradle 7

https://docs.openshift.com/online/pro/using_images/s2i_images/java.html

### 2. 解决了向 Dockerfile 传递多个参数报错的问题

https://stackoverflow.com/questions/42297387/docker-build-with-build-arg-with-multiple-arguments

### 3. 解决 spring boot 2.5+ 生成了两个 jar 包导致 oc new-app 运行失败的问题

https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#packaging-executable.and-plain-archives

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
    https://github.com/uniquejava/gs-spring-boot

$ oc delete all -l app=hello-gradle

$ oc new-app \
    --name hello-maven \
    --build-env BUILDER=maven \
    --context-dir complete \
    -i gradle-s2i \
    https://github.com/uniquejava/gs-spring-boot

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

## Cyper in Action

### 1. 制作 gradle-s2i Image Builder

```
oc new-build \
    --name gradle-s2i \
    --build-arg GRADLE_VERSION=7.2 \
    --build-arg OLD_S2I_PATH=/usr/local/s2i \
    --context-dir docker \
    --image-stream openshift/java:openjdk-11-el7 \
    --strategy docker \
    https://github.com/uniquejava/gradle-s2i
--> Found image bbf646f (7 weeks old) in image stream "openshift/java" under tag "openjdk-11-el7" for "openshift/java:openjdk-11-el7"

    Java Applications
    -----------------
    Platform for building and running plain Java applications (fat-jar and flat classpath)

    Tags: builder, java

    * A Docker build using source code from https://github.com/uniquejava/gradle-s2i will be created
      * The resulting image will be pushed to image stream tag "gradle-s2i:latest"
      * Use 'oc start-build' to trigger a new build

--> Creating resources with label build=gradle-s2i ...
    imagestream.image.openshift.io "gradle-s2i" created
    buildconfig.build.openshift.io "gradle-s2i" created
--> Success
```

### 2. 根据 gradle-s2i 创建 spring boot 项目

```
oc new-app \
    --name hello-gradle \
    --build-env SCRIPT_DEBUG=true \
    --context-dir complete \
    -i gradle-s2i \
    https://github.com/uniquejava/gs-spring-boot
warning: Cannot check if git requires authentication.
--> Found image 4706b33 (26 minutes old) in image stream "gradle-s2i-builder/gradle-s2i" under tag "latest" for "gradle-s2i"

    Gradle builder 1.0
    ------------------
    Platform for building Spring Boot applications with maven or gradle

    Tags: builder, maven-3, gradle-7.2, springboot

    * The source repository appears to match: jee
    * A source build using source code from https://github.com/uniquejava/gs-spring-boot will be created
      * The resulting image will be pushed to image stream tag "hello-gradle:latest"
      * Use 'oc start-build' to trigger a new build
      * WARNING: this source repository may require credentials.
                 Create a secret with your git credentials and use 'oc set build-secret' to assign it to the build config.

--> Creating resources ...
    imagestream.image.openshift.io "hello-gradle" created
    buildconfig.build.openshift.io "hello-gradle" created
    deployment.apps "hello-gradle" created
    service "hello-gradle" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/hello-gradle' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/hello-gradle'
    Run 'oc status' to view your app.
```

### 3. 测试 spring boot 项目

```
oc expose service hello-gradle

# 查看routes
oc status

# 或者
oc get routes -o json | jq -r '.items[0].spec.host'
```
