---
layout: post
title: How to publish gradle plugin to private git repo
---

**Problem**: make custom build for gradle plugin and use it in project.

You made changes in some gradle plugin, sent PR to maintainer, but there is no new build right now. You can use git repository as maven repository and publish custom build to it.

Add plugins `java-gradle-plugin` and `maven-publish` to plugin's `build.gradle`.
`maven-publish` provides the ability to publish artifacts to an Apache Maven repository. `java-gradle-plugin` assists for publishing gradle plugins.

```Groovy
plugins {
	...
	id 'maven-publish'
	id 'java-gradle-plugin'
}
```

Add publishing section and set path where plugin should be published.

```Groovy
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            url "file:///path/to/local/directory"
        }
    }
}
```

Add `gradlePlugin` section for defining plugin, set id and class.

```bash
gradlePlugin {
    plugins {
        MyGradlePlugin {
            id = 'com.github.blindpirate.gogradle'
            implementationClass = 'com.github.blindpirate.gogradle.GolangPlugin'
        }
    }
}
```

Now you can run `./gradlew tasks` and see new tasks was added:
```bash
...
publishPluginMavenPublicationToMavenRepository - Publishes Maven publication 'pluginMaven' to Maven repository 'maven'.
...
publishMyGradlePluginPluginMarkerMavenPublicationToMavenRepository - Publishes Maven publication 'MyGradlePluginPluginMarkerMaven' to Maven repository 'maven'.
...
```

The first task publish jar-file with implementation and second publish pom definition. Run it and smt like this should be in local directory.

```bash
𝝺 tree
.
└── com
    └── github
        └── blindpirate
            └── gogradle
                ├── 0.10.3-butik
                │   ├── gogradle-0.10.3-butik.jar
                │   ├── gogradle-0.10.3-butik.jar.md5
                │   ├── gogradle-0.10.3-butik.jar.sha1
                │   ├── gogradle-0.10.3-butik.pom
                │   ├── gogradle-0.10.3-butik.pom.md5
                │   └── gogradle-0.10.3-butik.pom.sha1
                ├── com.github.blindpirate.gogradle.gradle.plugin
                │   ├── 0.10.3-butik
                │   │   ├── com.github.blindpirate.gogradle.gradle.plugin-0.10.3-butik.pom
                │   │   ├── com.github.blindpirate.gogradle.gradle.plugin-0.10.3-butik.pom.md5
                │   │   └── com.github.blindpirate.gogradle.gradle.plugin-0.10.3-butik.pom.sha1
                │   ├── maven-metadata.xml
                │   ├── maven-metadata.xml.md5
                │   └── maven-metadata.xml.sha1
                ├── maven-metadata.xml
                ├── maven-metadata.xml.md5
                └── maven-metadata.xml.sha1

7 directories, 15 files

```
Next, add it to git and push to server.

Add `settings.gradle` to project than should use new plugin:
```bash
pluginManagement {
    repositories {
        maven {
            url "http://raw.github.com/ORGANIZATION/REPO/BRANCH"
        }
    }
}
```
set `url` to format `http://raw.github.com/ORGANIZATION/REPO/BRANCH` for GitHub or `https://bitbucket.com/USER/REPO/raw/BRANCH` for BitBucket.

Now you can use project with custom plugin build. 