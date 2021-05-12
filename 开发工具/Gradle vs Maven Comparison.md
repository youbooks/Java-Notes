# Gradle vs Maven Comparison

> 转载：[Gradle vs Maven Comparison](https://gradle.org/maven-vs-gradle/)

The following is a summary of the major differences between Gradle and Apache Maven: flexibility, performance, user experience, and dependency management. It is not meant to be exhaustive, but you can check the [Gradle feature list](https://gradle.org/features) and [Gradle vs Maven performance comparison](https://gradle.org/gradle-vs-maven-performance/) to learn more.

![](https://gradle.org/images/gradle-vs-maven.gif)

This GIF shows a side-by-side clean build of the [Apache Commons Lang library](https://github.com/gradle/performance-comparisons/tree/commons-lang) using Maven and Gradle (without build cache). You can view the [build scan for this build here](https://scans.gradle.com/s/of466wbcmynxm).

## 1. Flexibility

Google chose Gradle as the [official build tool for Android](https://developer.android.com/studio/build/index.html); not because build scripts are code, but because Gradle is modeled in a way that is extensible in the most fundamental ways. Gradle's model also allows it to be [used for native development with C/C++](https://github.com/gradle/gradle-native) and can be expanded to cover any ecosystem. For example, Gradle is designed with embedding in mind using its [Tooling API](https://docs.gradle.org/current/userguide/embedding.html).

Both Gradle and Maven provide convention over configuration. However, Maven provides a very rigid model that makes customization tedious and sometimes impossible. While this can make it easier to understand any given Maven build, as long as you don’t have any special requirements, it also makes it unsuitable for many automation problems. Gradle, on the other hand, is built with an empowered and responsible user in mind.

## 2. Performance

Improving build time is one of the most direct ways to *ship faster*. Both Gradle and Maven employ some form of parallel project building and parallel dependency resolution. The biggest differences are Gradle's mechanisms for work avoidance and incrementality. The top 3 features that make Gradle much faster than Maven are:

- [Incrementality](https://blog.gradle.org/introducing-incremental-build-support) — Gradle avoids work by tracking input and output of tasks and only running what is necessary, and only processing [files that changed](https://blog.gradle.org/incremental-compiler-avoidance) when possible.
- [Build Cache](https://blog.gradle.org/introducing-gradle-build-cache) — Reuses the build outputs of any other Gradle build with the same inputs, including between machines.
- [Gradle Daemon](https://docs.gradle.org/current/userguide/gradle_daemon.html) — A long-lived process that keeps build information "hot" in memory.

These and more [performance features](https://gradle.org/features/#performance) make Gradle at least twice as fast for nearly every scenario (100x faster for large builds using the build cache) in this [Gradle vs Maven performance comparison](https://gradle.org/gradle-vs-maven-performance/).

**Note:** Both Gradle and Maven users can take advantage of the Build Cache technology available in Gradle Enterprise. Gradle users typically experience an additional build time reduction of ~50%, while Maven users often experience reductions of ~90%. [Watch this video](https://tv.gradle.com/maven-build-cache-demo) to learn more about the Gradle Enterprise Maven Build Cache technology and business case.

![](https://image.ldbmcs.com/2021-05-12-BZLk6T.png)

## 3. User Experience

Maven's longer tenure means that its support through IDEs is better for many users. Gradle's IDE support continues to improve quickly, however. For example, Gradle now has a [Kotlin-based DSL](https://github.com/gradle/kotlin-dsl) that provides a much better IDE experience. The Gradle team is working with IDE-makers to make editing support much better — [stay tuned](https://twitter.com/gradle) for updates.

![](https://image.ldbmcs.com/2021-05-12-A2TxrO.jpg)

Although IDEs are important, a large number of users prefer to execute build operations through a command-line interface. Gradle provides a modern CLI that has discoverability features like `gradle tasks`, as well as improved logging and [command-line completion](https://github.com/gradle/gradle-completion).

Finally, Gradle provides an interactive web-based UI for debugging and optimizing builds: [build scans](https://gradle.com/build-scans). These can also be hosted on-premise to allow an organization to collect build history and do trend analysis, compare builds for debugging, or optimize build times.

![2021-05-12-X7ZPTa](https://image.ldbmcs.com/2021-05-12-X7ZPTa.jpg)

## 4. Dependency Management

Both build systems provide built-in capability to resolve dependencies from configurable repositories. Both are able to cache dependencies locally and download them in parallel.

As a library consumer, Maven allows one to override a dependency, but only by version. Gradle provides customizable [dependency selection](https://docs.gradle.org/current/userguide/dependency_management.html#component_selection_rules) and [substitution rules](https://docs.gradle.org/current/userguide/dependency_management.html#sec:module_substitution) that can be declared once and handle unwanted dependencies project-wide. This substitution mechanism enables Gradle to build multiple source projects together to create [composite builds](https://docs.gradle.org/current/userguide/composite_builds.html).

Maven has few, built-in dependency scopes, which forces awkward module architectures in common scenarios like using test fixtures or code generation. There is no separation between unit and integration tests, for example. Gradle allows [custom dependency scopes](https://docs.gradle.org/current/userguide/dependency_management.html#sub:configurations), which provides better-modeled and faster builds.

Maven dependency conflict resolution works with a shortest path, which is impacted by declaration ordering. Gradle does full conflict resolution, selecting the highest version of a dependency found in the graph. In addition, with Gradle you can declare versions as *strictly* which allows them to take precedence over transitive versions, allowing to [downgrade a dependency](https://docs.gradle.org/current/userguide/dependency_downgrade_and_exclude.html#sec:enforcing_dependency_version).

As a library producer, Gradle allows producers to [declare `api` and `implementation` dependencies](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_separation) to prevent unwanted libraries from leaking into the classpaths of consumers. Maven allows publishers to provide metadata through [optional dependencies](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html), but as documentation only. Gradle fully supports [feature variants and optional dependencies](https://docs.gradle.org/current/userguide/feature_variants.html).

## 5. Next Steps

We recommend you look more in-depth at [Gradle's features](https://gradle.org/features) or start with these resources.