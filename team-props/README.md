# Novoda team-props scaffolding

At Novoda we use the `team-props` scaffolding system to hold a variety of settings and convenience build code.
It is a simple and expandable system that could also be used for holding application secrets, signing configurations, etc.

This is a template of such scaffolding, ready to be copy-pasted into a new project — or an existing one. It currently uses
the Novoda Gradle Static Analysis plugin, which is geared towards Java projects. It supports running Checkstyle, Findbugs
and PMD. It currently does _not_ support Kotlin static analysis such as KtLint and Detekt, nor Android Lint.

## How to use this folder

 1. Copy this folder to the root of your Android project's repository
 2. Add this method to the root `build.gradle` file:
    ```groovy
    def teamPropsFile(propsFile) {
        def teamPropsDir = file('team-props')
        return new File(teamPropsDir, propsFile)
    }
    ```
 3. Make sure you import the Novoda Gradle Static Analysis plugin as a dependency in the `buildscript` of the root `build.gradle` file:
    ```groovy
    buildscript {
        repositories {
            // ...
            jcenter()
        }

        // ...
        
        dependencies {
            classpath 'com.novoda:gradle-static-analysis-plugin:0.4.1'
            // ...
        }
    }
    ```
 4. Make sure you have a `subprojects` closure in the root `build.gradle` file, and it contains this statement:
    ```groovy
    subprojects {
        // ...
        apply from: teamPropsFile('static-analysis.gradle')
    }
    ```
 4. Add this statement at the bottom of the root `build.gradle` file:
    ```groovy
    apply from: teamPropsFile('android-code-quality.gradle')
    ```
 5. Add this closure to the root `build.gradle` file:
    ```groovy
    ext {
        checkstyleVersion = '8.7'
        findbugsVersion = '3.0.1'
        pmdVersion = '6.0.0'
    }
    ```
    Don't forget to check if there's newer versions of the tools; these are the most recent at the time of writing.
 6. Configure the static analysis settings from the `team-props/static-analysis.gradle` file

Now all the checks are integrated in your `check` task
 
