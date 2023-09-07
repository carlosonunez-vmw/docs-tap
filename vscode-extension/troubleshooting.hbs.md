# Troubleshooting Tanzu Developer Tools for VS Code

This topic tells you what to do when you encounter issues with
VMware Tanzu Developer Tools for Visual Studio Code (VS Code).

## <a id='cannot-view-workloads'></a> Unable to view workloads on the panel when connected to GKE cluster

{{> 'partials/ide-extensions/ki-cannot-view-workloads' }}

## <a id='cancel-action-warning'></a> Warning notification when canceling an action

{{> 'partials/ide-extensions/ki-cancel-action-warning' }}

## <a id='lu-not-working-wl-types'></a> Live update might not work when using server or worker Workload types

{{> 'partials/ide-extensions/ki-lu-not-working-wl-types' }}

## <a id='lu-not-wrkng-classversion'></a> Live Update fails with `UnsupportedClassVersionError`

### Symptom

After live-update has synchronized changes you made locally to the running workload, the workload pods
start failing with an error message similar to the following:

```console
Caused by: org.springframework.beans.factory.CannotLoadBeanClassException: Error loading class
[com.example.springboot.HelloController] for bean with name 'helloController' defined in file
[/workspace/BOOT-INF/classes/com/example/springboot/HelloController.class]: problem with class file
or dependent class; nested exception is
java.lang.UnsupportedClassVersionError: com/example/springboot/HelloController has been compiled by
a more recent version of the Java Runtime (class file version 61.0), this version of the Java Runtime
only recognizes class file versions up to 55.0
```

### Cause

The classes produced locally on your machine are compiled to target a later Java virtual machine (JVM).
The error message mentions `class file version 61.0`, which corresponds to Java 17.
The buildpack, however, is set up to run the application with an earlier JVM.
The error message mentions `class file versions up to 55.0`, which corresponds to Java 11.

The root cause of this is a misconfiguration of the Java compiler that VS Code uses. The cause might
be a suspected issue with the VS Code Java tooling, which sometimes fails to properly configure the
compiler source and target compatibility-level from information in the Maven POM.

For example, in the `tanzu-java-web-app` sample application the POM contains the following:

```java
<properties>
        <java.version>11</java.version>
        ...
</properties>
```

This correctly specifies that the app must be compiled for Java 11 compatibility.
However, the VS Code Java tooling sometimes fails to take this information into account.

### Solution

Force the VS Code Java tooling to re-read and synchronize information from the POM:

1. Right-click the `pom.xml` file.
2. Click **Reload Projects**.

This causes the internal compiler level to be set correctly based on the information from `pom.xml`.
For example, Java 11 in `tanzu-java-web-app`.

## <a id="live-update-timeout"></a> Timeout error when Live Updating

{{> 'partials/ide-extensions/ki-timeout-err-live-updating' }}
