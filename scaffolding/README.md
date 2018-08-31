# Novoda team-props scaffolding

At Novoda we use the `team-props` scaffolding system to hold a variety of settings and convenience build code.
It is a simple and expandable system that could also be used for holding application secrets, signing configurations, etc.

This is a template of such scaffolding, ready to be copy-pasted into a new project — or an existing one. It currently uses
the [Novoda Gradle Static Analysis plugin](https://github.com/novoda/gradle-static-analysis-plugin), which is geared
towards Java projects. It supports running Checkstyle, Findbugs, PMD, Androint Lint and Detekt for Kotlin.

## How to use this folder

 1. Copy the `team-props` folder to the root of your Android project's repository
 2. Add this method to the root `build.gradle` file:
    ```gradle
    def teamPropsFile(propsFile) {
        def teamPropsDir = file('team-props')
        return new File(teamPropsDir, propsFile)
    }
    ```
 3. Make sure you import the Novoda Gradle Static Analysis plugin as a dependency in the `buildscript` of the root `build.gradle` file:
    ```gradle
    buildscript {
        repositories {
            // ...
            jcenter()
        }

        // ...

        dependencies {
            classpath 'com.novoda:gradle-static-analysis-plugin:0.6'
            // ...
        }
    }
    ```
 4. Make sure you have a `subprojects` closure in the root `build.gradle` file, and it contains this statement:
    ```gradle
    subprojects {
        // ...
        apply from: teamPropsFile('static-analysis.gradle')
    }
    ```
 5. Add this statement at the bottom of the root `build.gradle` file:
    ```gradle
    apply from: teamPropsFile('scaffolding.gradle')
    ```
 6. Configure the `prb` task from the `team-props/ci.gradle` file according to your needs and trigger it from Jenkins.
 7. Add this closure to the root `build.gradle` file:
    ```gradle
    ext {
        checkstyleVersion = '8.12'
        findbugsVersion = '3.0.1'
        pmdVersion = '6.6.0'
        detektVersion = '1.0.0.RC8'
        ktLint='0.27.0'
    }
    ```
    You can then use these values in the `staticAnalysis` closure and/or in your build files to configure the version of the tools to use.
    Don't forget to check if there's newer versions of the tools; these are the most recent at the time of writing.
 8. Configure the static analysis settings from the `team-props/static-analysis.gradle` file. Please refer to the [Static Analysis plugin documentation](https://github.com/novoda/gradle-static-analysis-plugin/blob/master/README.md#simple-usage) for more details.
 
Now all the checks are integrated in your `check` task.

Need a more complicated example, that includes the [Novoda Gradle Build Properties plugin](https://github.com/novoda/gradle-build-properties-plugin), application secrets, git pre-commit hooks, Android Lint, and support for Kotlin tools such as KtLint and Detekt, please check out the [Squanchy-android](https://github.com/squanchy-dev/squanchy-android/) open source project, which is a testing ground for the evolution of this scaffolding.
