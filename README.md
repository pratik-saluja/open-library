# Open Library

This repository is responsible to maintain public libraries files only. The purpose of the repository to have compiled SDK without source code.

### [What is Library?](https://developer.android.com/studio/projects/android-library)
What is an Android library? It helps to include everything which is needed to build an app, such as source code, resource files, and an Android manifest. It doesn't compile into an APK that runs on a device.


### Types of Library:
1. **Public Library**: Public libraries are accessible by anyone on the internet without the need of any authentication process. 
2. **Private Library**: Private libraries are only accessible by the individuals who are granted and authorized to access them. A good example can be an employee of an organization.

### How to create a private library?

Before jumping off directly on Android Studio let's cross-check and make sure the following tools are installed on the system:

- Gradle v7
- JDK 11

Run this command on the terminal exactly where the gradlew is present:
```
./gradlew --version
```
This will dump the Gradle version installed on your system as default and the Java version being utilized by Gradle as default. 

![gradle-info](/assets/Screenshot%202021-11-25%20at%2011.53.52%20AM.png)

### Let's start the process!

#### build.gradle - Project Level

In order to access maven in the project, maven needs to be added in Gradle.

```
buildscript {
    repositories {
        ...
        mavenCentral()
    }
    ...
}

...
...

allprojects {
    repositories {
        ...
        mavenCentral()
        maven { url "https://jitpack.io" }
    }
}

...
```

Add the following at the end
```
ext {
    libGroupId = 'com.github.pratik-saluja'
    libArtifactId = 'open-library'
    libVersionCode = 1
    libVersionName = '1.0'
}
```

The variable defines the metadata for the library:

- libGroupId - This is the package name of the lib.
- libArtifactId - By which name the library will be identified?
- libVersionCode - Do we need any version code for identification. This is not added in POM. It's generally for developers' reference.
- libVersionName - What is the version name?

The above information is going to be added in the library POM file

#### What is POM?

It maintains the versioning history and information regarding the library. In simple it's the first file being hit by the Project to gain lib info at glance.

#### Add Maven in build.gradle - Module Level

Add plugin at the top:

```
plugins {
    ...
    id 'maven-publish'
}
```

#### How are we going to build the metadata for the library?

```
project.afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from(components.release)
                groupId = rootProject.libGroupId
                artifactId = rootProject.libArtifactId
                version = rootProject.libVersionName
            }
        }
    }
}
```

#### Why hardcode version in default config? Let's load it from vars

```
defaultConfig {
        minSdk 21
        targetSdk 30
        versionCode rootProject.libVersionCode
        versionName rootProject.libVersionName
    }
```

#### Now the final call! Anything added in the Gradle must go under the Gradle task. But how? Let's create our Gradle tasks

```
task publishLibToMavenLocal(
        dependsOn: [':app:publishReleasePublicationToMavenLocal'],
        group: 'publish pratik open-libraries'
)
```

![task](/assets/Screenshot%202021-11-30%20at%206.12.00%20PM.png)

This would add an entry `publishLibToMavenLocal` under the Gradle task list.

![entry](/assets/Screenshot%202021-11-30%20at%206.12.19%20PM.png)

Now test the implementation by running the newly created task.

Output will be generated in `C:\Users\{username}\.m2`

#### Setup JitPack

Sign In to https://jitpack.io/ 

#### Release the Module

Enter the ```tag``` and ```version number``` and ```publish the release```.

![version](/assets/Screenshot%202021-11-30%20at%205.46.20%20PM.png)
![tag](/assets/Screenshot%202021-11-30%20at%205.46.29%20PM.png)

#### Find and Check in JitPack

Open https://jitpack.io/ and enter the repo url and hit Look up

![jitpack](/assets/Screenshot%202021-11-30%20at%205.47.54%20PM.png)

Notice the color. It can be either Red or Green. Red indicates build and upload error. Green indicates the library is successfully uploaded and published.

#### Access the library

[![](https://jitpack.io/v/pratik-saluja/open-library.svg)](https://jitpack.io/#pratik-saluja/open-library)

```
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```

```
	dependencies {
	        implementation 'com.github.pratik-saluja:open-library:1.1'
	}
```

#### :bulb: Change Java Version on JitPack Build Process

Create ```jitpack.yml``` at root directory of the project.

```
before_install:
   - sdk install java 11.0.10-open
   - sdk use java 11.0.10-open

jdk:
  - openjdk11
```

#### :bulb: Add GitHub Checks

Add Workflow by creating task in ```.github/workflows/master.yml```

```
name: Pull Request & Master CI

# Controls when the workflow will run
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  test:
    name: Run Gradle
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v1
      - name: Setup JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Make Gradle executable
        run: chmod +x ./gradlew
```

#### :bulb: Related useful links:
- https://jitpack.io/docs/BUILDING/
- https://medium.com/@ome450901/publish-an-android-library-by-jitpack-a0342684cbd0
- https://github.com/jitpack/android-example
- https://stackoverflow.com/questions/69021225/resource-linking-fails-on-lstar

