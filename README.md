# Understanding_JENKINS
This Repository contains my understanding of Jenkins.

Here’s a complete documentation for using credentials with `withCredentials()` in a Jenkins pipeline, formatted for a `README.md` file suitable for your GitHub repository.

```markdown
# Jenkins Credentials Documentation

This document provides a comprehensive overview of using credentials in Jenkins pipelines, specifically focusing on the `withCredentials()` function.

## Overview

The `withCredentials()` function in Jenkins is used to wrap a block of steps that require access to sensitive information, such as usernames, passwords, API keys, and other secret data. This function helps ensure that sensitive data is securely handled during the execution of Jenkins pipelines.

## Supported Credential Types

### 1. Username with Password

Use this type for services that require both a username and a password.

```groovy
withCredentials([usernamePassword(credentialsId: 'my-credentials-id', 
                                  usernameVariable: 'USERNAME', 
                                  passwordVariable: 'PASSWORD')]) {
    // Use $USERNAME and $PASSWORD
}
```
- **Arguments**:
  - `credentialsId`: The ID of the credentials stored in Jenkins.
  - `usernameVariable`: The name of the environment variable that will hold the username.
  - `passwordVariable`: The name of the environment variable that will hold the password.

---

### 2. Secret Text

Use this type for storing sensitive strings like API keys or tokens.

```groovy
withCredentials([string(credentialsId: 'my-secret-key', 
                         variable: 'SECRET_KEY')]) {
    // Use $SECRET_KEY
}
```
- **Arguments**:
  - `credentialsId`: The ID of the secret text credential.
  - `variable`: The name of the environment variable that will hold the secret text.

---

### 3. SSH Username with Private Key

Use this type for accessing servers via SSH.

```groovy
withCredentials([sshUserPrivateKey(credentialsId: 'my-ssh-credentials', 
                                     keyVariable: 'SSH_KEY', 
                                     usernameVariable: 'SSH_USER', 
                                     passphraseVariable: 'SSH_PASSPHRASE')]) {
    // Use $SSH_USER and $SSH_KEY
}
```
- **Arguments**:
  - `credentialsId`: The ID of the SSH credentials.
  - `keyVariable`: The name of the environment variable that will hold the private key.
  - `usernameVariable`: The name of the environment variable that will hold the username.
  - `passphraseVariable` (optional): The name of the environment variable for the passphrase for the private key.

---

### 4. AWS Credentials

Use this type for AWS services, which include both Access Key ID and Secret Access Key.

```groovy
withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                  credentialsId: 'aws-credentials']]) {
    // AWS environment variables will be set
}
```
- **Arguments**:
  - `credentialsId`: The ID of the AWS credentials.
  - You can specify `variable` to change the environment variable names if needed.

---

### 5. Kubernetes Service Account

Use this type for accessing Kubernetes clusters securely.

```groovy
withCredentials([kubeconfigFile(credentialsId: 'kubeconfig-credentials', 
                                  variable: 'KUBECONFIG')]) {
    // Use $KUBECONFIG
}
```
- **Arguments**:
  - `credentialsId`: The ID of the Kubernetes config file credential.
  - `variable`: The name of the environment variable that will hold the kubeconfig file path.

---

### 6. Certificate

Use this type for SSL certificates used in communications.

```groovy
withCredentials([certificate(credentialsId: 'my-cert-id', 
                              variable: 'CERTIFICATE')]) {
    // Use $CERTIFICATE
}
```
- **Arguments**:
  - `credentialsId`: The ID of the certificate credential.
  - `variable`: The name of the environment variable that will hold the certificate.

---

### Example of Using Multiple Credentials

You can also use multiple credentials within a single stage:

```groovy
pipeline {
    agent any

    stages {
        stage('Use Credentials') {
            steps {
                script {
                    // Username with Password
                    withCredentials([usernamePassword(credentialsId: 'my-credentials-id', 
                                                      usernameVariable: 'USERNAME', 
                                                      passwordVariable: 'PASSWORD')]) {
                        echo "Logging in as $USERNAME"
                    }

                    // Secret Text
                    withCredentials([string(credentialsId: 'my-secret-key', 
                                             variable: 'SECRET_KEY')]) {
                        echo "Using secret key: $SECRET_KEY"
                    }

                    // SSH Username with Private Key
                    withCredentials([sshUserPrivateKey(credentialsId: 'my-ssh-credentials', 
                                                         keyVariable: 'SSH_KEY', 
                                                         usernameVariable: 'SSH_USER')]) {
                        sh "ssh -i $SSH_KEY $SSH_USER@my.server.com 'whoami'"
                    }

                    // AWS Credentials
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                                      credentialsId: 'aws-credentials']]) {
                        sh 'aws s3 ls'
                    }
                }
            }
        }
    }
}







In Jenkins, nested and parallel stages are powerful features that enhance the pipeline's ability to manage complex workflows efficiently. Here's an explanation of each, along with real-time scenarios and basic examples.

### Nested Stages

**Description**: 
Nested stages allow you to organize your pipeline into a hierarchy, where one stage can contain other stages. This is useful for structuring complex processes and grouping related tasks.

**Use Case Scenario**: 
Consider a software development project where you have a "Build" stage that consists of several steps: compiling the code, running unit tests, and packaging the application. Each of these steps can be defined as a nested stage within the main "Build" stage.

pipeline {
    agent any

    stages {
        stage('Build') {
            stages {
                stage('Compile') {
                    steps {
                        echo 'Compiling the code...'
                        // Command to compile the code
                    }
                }
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests...'
                        // Command to run unit tests
                    }
                }
                stage('Package') {
                    steps {
                        echo 'Packaging the application...'
                        // Command to package the application
                    }
                }
            }
        }
    }
}


### Parallel Stages

**Description**: 
Parallel stages allow multiple tasks to run simultaneously, which can significantly reduce build time. This is particularly useful when you have independent processes that do not rely on each other.

**Use Case Scenario**: 
Imagine a scenario where you want to run integration tests on multiple environments (e.g., staging and production) at the same time after a successful build. Instead of running the tests sequentially, you can run them in parallel.

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                // Command to build the application
            }
        }
        stage('Test') {
            parallel {
                stage('Integration Tests - Staging') {
                    steps {
                        echo 'Running integration tests on Staging...'
                        // Command to run tests on Staging
                    }
                }
                stage('Integration Tests - Production') {
                    steps {
                        echo 'Running integration tests on Production...'
                        // Command to run tests on Production
                    }
                }
            }
        }
    }
}


### Combined Nested and Parallel Stages

You can also combine both nested and parallel stages to create even more complex workflows.

pipeline {
    agent any

    stages {
        stage('Build') {
            stages {
                stage('Compile') {
                    steps {
                        echo 'Compiling the code...'
                    }
                }
                stage('Test') {
                    parallel {
                        stage('Unit Tests') {
                            steps {
                                echo 'Running unit tests...'
                            }
                        }
                        stage('Integration Tests') {
                            steps {
                                echo 'Running integration tests...'
                            }
                        }
                    }
                }
            }
        }
    }
}


In Jenkins, parameters are used to pass information into jobs or pipelines at runtime. They allow you to customize builds based on input values, enabling dynamic behavior in your CI/CD processes. Here’s an overview of the different types of parameters in Jenkins and how they can be used:

### Types of Parameters

1. **String Parameter**
   - **Description**: Accepts a single line of text input.
   - **Use Case**: Useful for passing simple values, like a version number or a specific configuration.
   - **Example**: A user can enter the version number of the software to build.

   ```groovy
   properties([
       parameters([
           string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version number')
       ])
   ])
   ```

2. **Boolean Parameter**
   - **Description**: A checkbox that accepts true or false values.
   - **Use Case**: Useful for enabling or disabling features in the build process.
   - **Example**: A parameter to decide whether to run tests or not.

   ```groovy
   properties([
       parameters([
           booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
       ])
   ])
   ```

3. **Choice Parameter**
   - **Description**: Provides a dropdown list of options for the user to choose from.
   - **Use Case**: Useful for selecting from a predefined set of values.
   - **Example**: Choosing the target environment (development, staging, production).

   ```groovy
   properties([
       parameters([
           choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Select environment')
       ])
   ])
   ```

4. **File Parameter**
   - **Description**: Allows users to upload a file to be used during the build.
   - **Use Case**: Useful for scenarios where a specific file is needed, like a configuration file or a data file.
   - **Example**: Uploading a custom configuration file for deployment.

   ```groovy
   properties([
       parameters([
           file(name: 'CONFIG_FILE', description: 'Upload configuration file')
       ])
   ])
   ```

5. **Password Parameter**
   - **Description**: A text input that masks the input to keep it secure.
   - **Use Case**: Useful for sensitive data, such as API keys or passwords.
   - **Example**: Entering a deployment password.

   ```groovy
   properties([
       parameters([
           password(name: 'DEPLOY_PASSWORD', description: 'Deployment password')
       ])
   ])
   ```

6. **Run Parameter**
   - **Description**: Allows you to select a specific build to trigger another build with its parameters.
   - **Use Case**: Useful for chaining builds where parameters from one build are needed in another.
   - **Example**: Triggering a downstream job with parameters from an upstream job.

   ```groovy
   properties([
       parameters([
           run(name: 'UPSTREAM_BUILD', description: 'Select upstream build')
       ])
   ])
   ```

### Using Parameters in a Jenkins Pipeline

You can access the parameters in your pipeline using the `params` object. Here's how you can use them in a Jenkinsfile:

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version number')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    }

    stages {
        stage('Build') {
            steps {
                echo "Building version: ${params.VERSION}"
                if (params.RUN_TESTS) {
                    echo 'Running tests...'
                } else {
                    echo 'Skipping tests...'
                }
            }
        }
    }
}
```

