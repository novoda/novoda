# Novoda team-props scaffolding

At Novoda we use the `team-props` scaffolding system to hold a variety of settings and convenience build code.
It is a simple and expandable system that could also be used for holding application secrets, signing configurations, etc.

This is a template of such scaffolding, ready to be copy-pasted into a new project — or an existing one. It currently uses
the [Novoda Gradle Static Analysis plugin](https://github.com/novoda/gradle-static-analysis-plugin), which is geared
towards Java projects. It supports running Checkstyle, Findbugs and PMD.

It currently does _not_ support Kotlin static analysis such as KtLint and Detekt, nor Android Lint.

## How to use this folder

 1. Copy this folder to the root of your Android project's repository
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
            classpath 'com.novoda:gradle-static-analysis-plugin:0.5.2'
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
 4. Add this statement at the bottom of the root `build.gradle` file:
    ```gradle
    apply from: teamPropsFile('android-code-quality.gradle')
    ```
 5. Add this closure to the root `build.gradle` file:
    ```gradle
    ext {
        checkstyleVersion = '8.8'
        findbugsVersion = '3.0.1'
        pmdVersion = '6.0.1'
    }
    ```
    Don't forget to check if there's newer versions of the tools; these are the most recent at the time of writing.
 6. Configure the static analysis settings from the `team-props/static-analysis.gradle` file
 7. Add the following Lint configuration to the `build.gradle` of all your Android projects:
    ```gradle
    //...
    android {
        //...
        lintOptions {
            lintConfig teamPropsFile('static-analysis/lint-config.xml')
            abortOnError true
            warningsAsErrors true
        }
    }
    ```

Now all the checks are integrated in your `check` task.

Need a more complicated example, that includes the [Novoda Gradle Build Properties plugin](https://github.com/novoda/gradle-build-properties-plugin), application secrets, git pre-commit hooks, Android Lint, and support for Kotlin tools such as KtLint and Detekt, please check out the [Squanchy-android](https://github.com/squanchy-dev/squanchy-android/) open source project, which is a testing ground for the evolution of this scaffolding.
