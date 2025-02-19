# Creating a ZIP Deployment Package for a Java Function<a name="create-deployment-pkg-zip-java"></a>

This section provides examples of creating \.zip file as your deployment package\. The deployment package must have the following structure\.
+ All compiled class files and resource files at the root level\.
+ All required jars to run the code in the `/lib` directory\.
**Note**  
Lambda loads JARs in unicode alphabetical order\. If multiple JARs in the `lib` folder contain the same class, the first one is used\. You can use the following shell script to identify duplicate classes\.  

**Example test\-zip\.sh**  

  ```
  mkdir -p expanded
  unzip path/to/my/function.zip -d expanded
  find ./expanded/lib -name '*.jar' | xargs -n1 zipinfo -1 | grep '.*.class' | sort | uniq -c | sort
  ```

You can use any build and packaging tool you like to create a deployment package\. The following examples use the Gradle build and deployment tool\.

## Before You Begin<a name="create-deployment-pkg-zip-java-before-you-begin"></a>

You will need to download Gradle version 2\.0 or later\. For instructions, go to the gradle website, [https://gradle\.org/](https://gradle.org/) \.

## Example 1: Creating \.zip Using Gradle and the Maven Central Repository<a name="create-deployment-pkg-zip-java-using-central-repository"></a>

At the end of this walkthrough, you will have a project directory \(`project-dir`\) with content having the following structure:

```
project-dir/build.gradle 
project-dir/src/main/java/
```

The` /java` folder will contain your code\. For example, if your package name is `example`, and you have a `Hello.java` class in it, the structure will be:

```
project-dir/src/main/java/example/Hello.java
```

After you build the project, the resulting \.zip file \(that is, your deployment package\), will be in the `project-dir/build/distributions` subdirectory\.

1. Create a project directory \(`project-dir`\)\. 

1. In the `project-dir`, create `build.gradle` file and add the following content:

   ```
   apply plugin: 'java'
   
   repositories {
       mavenCentral()
   }
   
   dependencies {
       compile (
           'com.amazonaws:aws-lambda-java-core:1.2.0',
           'com.amazonaws:aws-lambda-java-events:2.2.6'
       )
   }
   
   task buildZip(type: Zip) {
       from compileJava
       from processResources
       into('lib') {
           from configurations.runtimeClasspath
       }
   }
   
   build.dependsOn buildZip
   ```
**Note**  
The repositories section refers to Maven Central Repository\. At build time, it fetches the dependencies \(that is, the two AWS Lambda libraries\) from Maven Central\.
The `buildZip` task describes how to create the deployment package \.zip file\.   
For example, if you unzip the resulting \.zip file you should find any of the compiled class files and resource files at the root level\. You should also find a `/lib` directory with the required jars for running the code\.
If you are following other tutorial topics in this guide, the specific tutorials might require you to add more dependencies\. Make sure to add those dependencies as required\.

1. In the `project-dir`, create the following structure:

   ```
   project-dir/src/main/java/ 
   ```

1. Under the `/java` subdirectory you add your Java files and folder structure, if any\. For example, if you Java package name is `example`, and source code is `Hello.java`, then your directory structure looks like this:

   ```
   project-dir/src/main/java/example/Hello.java
   ```

1. Run the following gradle command to build and package the project in a \.zip file\.

   ```
   project-dir> gradle build  
   ```

1. Verify the resulting `project-dir.zip` file in the `project-dir/build/distributions` subdirectory\.

1. Now you can upload the \.zip file, your deployment package to AWS Lambda to create a Lambda function and test it by manually invoking it using sample event data\. For instructions, see [Create a Lambda Function Authored in Java](get-started-step4-optional.md)\.

## Example 2: Creating \.zip Using Gradle With Local JARs<a name="create-deployment-pkg-zip-java-without-central-repository"></a>

You may choose not to use the Maven Central repository\. Instead have all the dependencies in the project folder\. In this case your project folder \(`project-dir`\) will have the following structure:

```
project-dir/jars/           (all jars go here)          
project-dir/build.gradle           
project-dir/src/main/java/  (your code goes here)
```

So if your Java code has `example` package and `Hello.java` class, the code will be in the following subdirectory:

```
project-dir/src/main/java/example/Hello.java
```

You `build.gradle` file should be as follows:

```
apply plugin: 'java'

dependencies {
    compile fileTree(dir: 'jars', include: '*.jar')
}

task buildZip(type: Zip) {
    from compileJava
    from processResources
    into('lib') {
        from configurations.runtimeClasspath
    }
}

build.dependsOn buildZip
```

Note that the dependencies specify `fileTree` which identifies `project-dir/jars` as the subdirectory that will include all the required jars\.

Now you build the package\. Run the following gradle command to build and package the project in a \.zip file\.

```
project-dir> gradle build  
```