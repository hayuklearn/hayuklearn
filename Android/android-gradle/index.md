[TOC]

# Gradle

## check for error

```shell
./gradlew compileDebugSources --stacktrace -info
```

## check for dependencies

```shell
./gradlew dependencies app:dependencies >> dependencies.log

./gradlew -q dependencies :products:demo:compose:app-demo-compose:dependencies
```