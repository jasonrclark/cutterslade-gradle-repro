# Repro for apiHelper problem with ca.cutterslade.gradle:gradle-dependency-analyze:1.6.0

This repro contains a minimal example of a problem I'm having with releases of gradle-dependency-analyze after 1.5.3.

We have a internal plugin where one of its tasks forces resolution of configurations during its processing. After updating to 1.6.0 I was seeing failures like the following on certain combinations of commands:

```
> Task :analyzeClassesDependencies FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':analyzeClassesDependencies'.
> Cannot change dependencies of dependency configuration ':apiHelper' after it has been resolved.
```

After some experimentation, I found that these two tasks pulled into this repo invoke the problem on >= 1.6.0 but don't on 1.5.3.

```
./gradlew forceConfigurationResolution analyzeClassesDependencies
```

I suspect that this is an unintended side-effect in the recent gradle-dependency-analyze since it worked before, and I don't know why analyzing things would need to modify the dependency collections.

Relevent section of the stack trace. Full trace in [this gist](https://gist.github.com/jasonrclark/77bad41315b35b6e3a4bf2ef760e9214)

```
Caused by: org.gradle.api.InvalidUserDataException: Cannot change dependencies of dependency configuration ':apiHelper' after it has been resolved.
        at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration.preventIllegalMutation(DefaultConfiguration.java:1260)
        at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration.validateMutation(DefaultConfiguration.java:1229)
        at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration.extendsFrom(DefaultConfiguration.java:382)
        at ca.cutterslade.gradle.analyze.ProjectDependencyResolver.configureApiHelperConfiguration(ProjectDependencyResolver.groovy:280)
        at ca.cutterslade.gradle.analyze.ProjectDependencyResolver.<init>(ProjectDependencyResolver.groovy:59)
        at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at java.base/jdk.internal.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at ca.cutterslade.gradle.analyze.AnalyzeDependenciesTask.action(AnalyzeDependenciesTask.groovy:52)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:104)
        at org.gradle.api.internal.project.taskfactory.StandardTaskAction.doExecute(StandardTaskAction.java:58)
        at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:51)
        at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:29)
        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter$2.run(ExecuteActionsTaskExecuter.java:494)
        at org.gradle.internal.operations.DefaultBuildOperationRunner$1.execute(DefaultBuildOperationRunner.java:29)
```
