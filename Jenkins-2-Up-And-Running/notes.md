# Table of content

- [Chapter 1: Introducing Jenkins 2](#chapter-1-introducing-jenkins-2)
    + [Pipeline](#pipeline)
    + [Organization](#organization)
    + [Summary](#summary)
- [Chapter 2: The Foundations](#chapter-2-the-foundations)
  * [Syntax: Scripted Pipelines Versus Declarative Pipelines](#syntax-scripted-pipelines-versus-declarative-pipelines)
    + [Choosing Between Scripted and Declarative Syntax](#choosing-between-scripted-and-declarative-syntax)
  * [Systems: Masters, Nodes, Agents, and Executors](#systems-masters-nodes-agents-and-executors)
    + [Master](#master)
    + [Node](#node)
    + [Agent](#agent)
      - [Directives Versus Steps](#directives-versus-steps)
    + [Executor](#executor)
  * [Creating Nodes](#creating-nodes)
    + [A quick note about node labels](#a-quick-note-about-node-labels)
  * [Structure: Working with the Jenkins DSL](#structure-working-with-the-jenkins-dsl)
    + [node](#node)
      - [Leveraging Multiple Labels on a Node](#leveraging-multiple-labels-on-a-node)
    + [stage](#stage)
    + [steps](#steps)
    + [Understanding step syntax](#understanding-step-syntax)
  * [Supporting Environment: Developing a Pipeline Script](#supporting-environment-developing-a-pipeline-script)
    + [Starting a Pipeline Project](#starting-a-pipeline-project)
    + [Working with the Snippet Generator](#working-with-the-snippet-generator)
  * [Running a Pipeline](#running-a-pipeline)
  * [Replay](#replay)
- [Chapter 3. Pipeline Execution Flow](#chapter-3-pipeline-execution-flow)
  * [Triggering Jobs](#triggering-jobs)

# Chapter 1: Introducing Jenkins 2

### Pipeline

As the name implies, the Pipeline type of project is intended for creating pipelines. This is done by writing the code in the Jenkins DSL. This is the main type of project we’ll be talking about throughout the book.
As already noted, pipelines can either be written in a **“scripted” syntax style or a “declarative” syntax style**. Pipelines created in this type of project can also be made easily into Jenkinsfiles.

### Organization

Certain source control platforms provide a mechanism for grouping repositories into “organizations.” Jenkins integrations allow you to store Jenkins pipeline scripts as Jenkinsfiles in the repositories within an organization and execute based on those. Currently GitHub and Bitbucket organizations are supported, with others planned for the future. For simplicity in this book, we’ll talk mainly about GitHub Organization projects as our example.

### Summary

Jenkins 2 also provides several new project types. The Folder type allows for grouping projects together under a shared namespace and shared environment. The Multibranch Pipeline type provides easy automated job creation per branch and continuous integration, all triggered by Jenkinsfiles residing in the branches. And the organization project type extends the multibranch functionality across all projects in an organization structure on GitHub or Bitbucket.

# Chapter 2: The Foundations

Four basic areas:
* The two styles of syntax that can be used for creating pipelines
* The systems used to run the pipeline processes
* The basic structure of a pipeline
* The support environment (and tooling) that Jenkins provides for pipeline development and execution

## Syntax: Scripted Pipelines Versus Declarative Pipelines

**Scripted syntax** refers to the initial way that pipelines-as-code have been done in Jenkins. It is an imperative style, meaning it is based on defining the logic and the program flow in the pipeline script itself. It is also more dependent on the Groovy language and Groovy constructs - especially for things like error checking and dealing with exceptions.

**Declarative syntax** is a newer option in Jenkins. Pipelines coded in the declarative style are arranged in clear sections that describe (or “declare”) the states and outcomes we want in the major areas of the pipeline, rather than focusing on the logic to accomplish it.

```
// Scripted Pipeline
node('worker_node1') {
  stage('Source') { // Get code
    // get code from our Git repository
    git 'git@diyvb2:/home/git/repositories/workshop.git'
  }
  stage('Compile') { // Compile and do unit testing
    // run Gradle to execute compile and unit testing
    sh "gradle clean compileJava test"
  }
}


// Declarative Pipeline
pipeline {
  agent {label 'worker_node1'}
  stages {
    stage('Source') { // Get code
      steps {
        // get code from our Git repository
        git 'git@diyvb2:/home/git/repositories/workshop.git'
      }
    }
    stage('Compile') { // Compile and do unit testing
      steps {
        // run Gradle to execute compile and unit testing
        sh "gradle clean compileJava test"
      }
    }
  }
}
```

### Choosing Between Scripted and Declarative Syntax

In short, the declarative model should be easier to learn and maintain for new pipeline users or those wanting more ready-made functionality like the traditional Jenkins model. This comes at the price of less flexibility to do anything not supported by the structure.
The scripted model offers more flexibility. It provides the “power-user” option, allowing users to do more things with less imposed structure.

## Systems: Masters, Nodes, Agents, and Executors

In traditional Jenkins, there were only two categories: **masters** and **slaves**.

### Master

A Jenkins **master** is the primary controlling system for a Jenkins instance. It has complete access to all Jenkins configuration and options and the full list of jobs. It is the default location for executing jobs if another system is not specified.
However, it is not intended for running any heavyweight tasks. **Jobs requiring any substantial processing should be run on a system other than the master.**

### Node

**Node** is the generic term that is used in Jenkins 2 to mean any system that can run Jenkins jobs. This covers both masters and agents, and is sometimes used in place of those terms. Furthermore, a node might be a container, such as one for Docker.

### Agent

An **agent** is the same as what earlier versions of Jenkins referred to as a **slave**. Traditionally in Jenkins, this refers to any nonmaster system. The idea is that these systems are managed by the master system and allocated as needed, or as specified, to handle processing the individual jobs.

#### Directives Versus Steps

**node** is associated with a Scripted Pipeline.

```
// Scripted Pipeline
node('worker') {
  stage('Source') { // Get code
    // Get code from our Git repository
```

**agent**, on the other hand, is a directive in a Declarative Pipeline.
```
// Declarative Pipeline
pipeline {
  agent {label:'worker'}
  stages {
    stage('Source') { // Get code
```

Just use **node** for Scripted Pipelines and **agent** for Declarative Pipelines.

### Executor

**executor** is just a slot in which to run a job on a node/agent. A node can have zero or more executors. **The number of executors defines how many concurrent jobs can be run on that node.** When the master funnels jobs to a particular node, there must be an available executor slot in order for the job to be processed immediately. Otherwise, it will wait until an executor becomes available.

## Creating Nodes

In traditional versions of Jenkins, jobs would run either on the master instance or on slave instances. As noted previously, in Jenkins 2 terminology these kinds of instances are both referred to by the generic term **node**.

Manage Jenkins -> Manage Nodes -> Select New Node and fill in the forms (choose Permanent Agent) -> Enter parameters like showed in picture.

![creating-node.PNG](pictures/creating-node.PNG)

### A quick note about node labels
Labels can be used for both system and user purposes. For example, labels can be used to:
* Identify a specific node (via a unique label).
* Group classes of nodes together (by giving them the same label).
* Identify some characteristic of a node that is useful to know for processing (via a meaningful label, such as “Windows” or “West Coast”).

The last bullet is a recommended practice.

## Structure: Working with the Jenkins DSL

In this section, we’ll cover some basic terms and the structure and functionality of a Jenkins DSL pipeline. We’ll be talking about this in terms of a Scripted Pipeline (meaning without the enhancements that the declarative functionality adds). In Chapter 7, we’ll explain the differences and look at the changes that creating a Declarative Pipeline entails.

Here’s a very simple pipeline expressed in the Jenkins DSL:
```
node ('worker1') {
  stage('Source') { // for display purposes
    // Get some code from our Git repository
    git 'https://github.com/brentlaster/gradle-greetings.git'
  }
}
```
Let’s break this down and explain what each part is doing.

### node

As mentioned in “Node”, we can think of this as the new term for a master or agent. Nodes are defined through the Manage Jenkins → Manage Nodes interface and can be set up just like slaves. Each node then has a Jenkins agent installed on it to execute jobs. (Note that in this case we are assuming we have a node already set up on the Jenkins instance labeled ``worker1``).

This line tells Jenkins on which node it should run this part of the pipeline. It binds the code to the particular Jenkins agent program running on that node. A particular one is specified by passing a defined name as a parameter (label). This must be a node or system that has already been defined and that your Jenkins system is aware of. You can omit supplying a label here, but if you omit a label, then you need to be aware of how this will be handled:
* If master has been configured as the default node for execution, Jenkins will run the job on master (master can be configured to not run any jobs).
* Otherwise, an empty node label (or ``agent any`` in declarative syntax) will tell Jenkins to run on the first executor that becomes available on any node.

When this part of the pipeline is executed, it connects to the node, creates a workspace (working directory) for the code to execute in, and schedules the code to run when an executor is available. 

#### Leveraging Multiple Labels on a Node

In the configuration for a node, you can assign multiple labels in the Labels entry box. You can specify multiple labels using standard logic operands such as
``||`` for “or” and ``&&`` for “and.” So, in this case, you could add the label Linux to all of the nodes and an additional label to indicate where each is located—i.e., east or west:
```
node("linux && east")
```

### stage

Within a node definition, a stage closure allows us to group together individual settings, DSL commands, and logic. A stage is required to have a name, which provides a mechanism for describing what the stage does.

### steps

Inside the stage, we have the actual Jenkins DSL commands. These are referred to as steps in Jenkins terminology. **A step is the lowest level of functionality defined by the DSL.** These are not Groovy commands, but can be used with Groovy commands. In the case of our example, we have this initial step to get our source: 
```
git 'https://github.com/brentlaster/gradle-greetings.git'
```

### Understanding step syntax

Steps in the Jenkins DSL always expect mapped (named) parameters. To illustrate this, here’s another version of the git step definition:
```
git branch: 'test',
    url: 'https://github.com/brentlaster/gradle-greetings.git'
```

Groovy also allows skipping the parentheses for parameters. Without these shortcuts, the longer version of our step would be:
```
git([branch: 'test', url: 'http://github.com/brentlaster/gradle-greetings.git'])
```

Another trick is this: if there is a single required parameter, and only one value is passed, the parameter name can be omitted. This is how we arrive at our short version of the step as:
```
git 'https://github.com/brentlaster/gradle-greetings.git'
```

If a named parameter is not required, then the default parameter is the script object. An example here is with the bat step, which is used to run batch or shell processing on Windows system. Writing this with the full syntax would look like this:
```
bat([script: 'echo hi'])
```
Taking into account the shortcuts that are offered, this can simply be written as:
```
bat 'echo hi'
```

![relationship-between-node-and-stages.PNG](pictures/relationship-between-node-and-stages.PNG)

## Supporting Environment: Developing a Pipeline Script

A pipeline script in Jenkins can either be created within a **Jenkins job of type Pipeline** or as an **external file named Jenkinsfile**. If created as a Jenkinsfile, then it can be stored with the source.

### Starting a Pipeline Project

When you select Pipeline as the type of project to create, you’re presented with a familiar web-based form for a new Jenkins project. The main tab we are interested in for our new Pipeline project is the Pipeline tab. Switching to that presents a text entry screen where we can enter the code for our pipeline script.

![pipeline-tab.PNG](pictures/pipeline-tab.PNG)

### Working with the Snippet Generator

To simplify finding the correct semantics and syntax for steps, Jenkins 2 includes a pipeline syntax help wizard, also known as the Snippet Generator.

The Snippet Generator provides a way to search through the available DSL steps and find out the syntax and semantics of ones you are interested in. Additionally, it provides online help to explain what the step is intended to do. But perhaps the most useful option it provides is a web form with areas to enter values for the parameters you want to use.

Let’s work through a simple example to see how this works. Suppose we want to create the earlier step to retrieve our Git code. Figure 2-11 shows our starting point.

![snippet-generator-1.PNG](pictures/snippet-generator-1.PNG)

We know we want to use Git, but we’re not sure of the syntax, so we click the Pipeline Syntax link at the bottom of the Pipeline tab. This takes us to the opening screen for the Snippet Generator.

![snippet-generator-2.PNG](pictures/snippet-generator-2.PNG)


## Running a Pipeline

Nothing of interest.

## Replay

Coding pipelines is more involved than web form interaction with Jenkins. There may be times where something fails and you want to retry it in a temporary way without modifying your code. Replay allows you to modify your code after a run, and then run it again with the modifications. A new build record of that run is kept, but the original code remains unchanged.

![Replay.PNG](pictures/Replay.PNG)

Now Jenkins presents us with an edit window just like the one for the Pipeline tab of a Pipeline project. In this window, we can make any changes to our program that we want and then select Run to try out the changes. (Here, we’re changing bat back to sh).

![Replay-2.PNG](pictures/Replay-2.PNG)

Jenkins will attempt to run the edited code in the Replay window. In this case it will succeed, creating run #3. However, if we click Configure in the menu on the left and go back and look at our code in the Pipeline tab, we’ll see that it still shows bat. The Replay functionality allowed us to try out a change, but we still need to go back and update our code in the pipeline job to make the change.

# Chapter 3. Pipeline Execution Flow

## Triggering Jobs

To specify triggering events for pipeline code, there are three different approaches:
* If working in the Jenkins application itself in a pipeline job, the trigger(s) can be specified in the traditional way within the project’s General configuration section in the web interface.
* If creating a Scripted Pipeline, a ``properties`` block can be specified (usually before the start of the pipeline) that defines the triggers in code. (Note that this properties section will be merged with any properties defined in the web interface, with the web properties taking precedence).
* If creating a Declarative Pipeline, there is a special ``triggers`` directive that can be used to define the types of things that should trigger the pipeline.

## Build After Other Projects Are Built

As the name implies, selecting this option allows you to start your project building after one or more other projects. You can choose the ending status you want the builds of the other projects to have (stable, unstable, or failed). For a Scripted Pipeline, the syntax for building your pipeline after another job, Job1, is successful would be like the following:
```
properties([
  pipelineTriggers([
    upstream(
      threshold: hudson.model.Result.SUCCESS,
      upstreamProjects: 'Job1'
    )
  ])
])
```

If you need to specify a branch for a job (as for a multibranch job), add a slash after the job name and then the branch name (as in ``'Job1/master'``).

## Build Periodically

This option provides a cron type of functionality to start jobs at certain time intervals. While this is an option for builds, **this is not optimal for continuous integration**, where the builds are based on detecting updates in source management.

Here’s an example of the syntax in a Scripted Pipeline. In this case, the job runs at 9 a.m., Monday–Friday:
```
properties([pipelineTriggers([cron('0 9 * * 1-5')])])
```

And here’s an example of the syntax in a Declarative Pipeline:
```
triggers { cron(0 9 * * 1-5)
```

## GitHub Hook Trigger for GitSCM Polling

A GitHub project configured as the source location in a Jenkins project can have a push hook (on the GitHub side) to trigger a build for the Jenkins project. When this is in place, a push to the repository causes the hook to fire and trigger Jenkins, which then invokes the Jenkins SCM polling functionality.

```
properties([pipelineTriggers([githubPush()])])
```

## Poll SCM

This is the standard polling functionality that periodically scans the source control system for updates. If any updates are found, then the job processes the changes. **This can be a very expensive operation.** The syntax for Scripted Pipelines is as follows (polling every 30 minutes):
```
properties([pipelineTriggers([pollSCM('*/30 * * * *')])])
```
The corresponding syntax for Declarative Pipelines would be this:
```
triggers { pollSCM(*/30 * * * *) }
```

## Quiet Period

The value specified here serves as a “wait time” or offset between when the build is triggered (an update is detected) and when Jenkins acts on it. This can be useful for staggering jobs that frequently have changes at the same time.

## Trigger Builds Remotely

This allows for triggering builds by accessing a specific URL for the given job on the Jenkins system. This is useful for triggering builds via a hook or a script. An authorization token is required. 

## User Input

A key aspect of some Jenkins jobs is the ability to change their behavior based on user input. Jenkins offers a wide variety of parameters for gathering specific kinds of input. Jenkins pipelines provide constructs for this as well.

The DSL step ``input`` is the way we get user input through a pipeline. The step accepts the same kinds of parameters as a regular Jenkins job for a Scripted Pipeline. For a Declarative Pipeline, there is a special ``parameters`` directive that supports a subset of those parameters.

### input

As the name suggests, the input step allows your pipeline to stop and wait for a user response. Here’s a simple example:
```
input 'Continue to next stage?'
```

This step can also optionally take parameters to gather additional information. Within the Jenkins application, the default form is to print a message and offer the user a choice of "Proceed" or "Abort".

![jenkins-input-dialog-box.PNG](pictures/jenkins-input-dialog-box.PNG)

**!NOTE.** As defined earlier in the book, an executor is a slot on a node for processing code. Using the input step in a node block ties up the executor for the node until the input step is done.

The input step can have several parameters. These include:
* *Message* (message) - The message to be displayed to the user, as demonstrated in the previous example. Can also be empty, as indicated by input ''.
* *Custom ID* (id) - An ID that can be used to identify your input step to automated or external processing, such as when you want to respond via a REST API call. A unique identifier will be generated if you don’t supply one. As an example, you could add the custom ID, ctns-prompt (for "Continue to next stage" prompt) to our input step definition. The input step would then look as follows:
```
input id: 'ctns-prompt', message: 'Continue to the next stage?'
```
Given this step, when you run the job, a POST to this URL could be used to respond. The URL format would be: ``http://[jenkins-base-url]/job/[job_name]/[build_id]/input/Ctns-prompt/proceedEmpty``  to tell Jenkins to proceed without any input, or: ``http://[jenkins-base-url]/job/[job_name]/[build_id]/input/Ctns-prompt/abort`` to tell Jenkins to abort. (Notice that the parameter name is capitalized in the URL.)

* *OK button caption* (ok) - A different label you can use instead of “Proceed.” For example: 
```
input message: '<message text>', ok: 'Yes'
```

### Parameters

With the ``input`` statement, you have the option to add any of the standard Jenkins parameter types. For each parameter type, the different “subparameters” (arguments) that it can take are also listed. If the purpose of the subparameter is self-evident from its name (e.g., name, default value, description), the argument name will be listed without additional explanation.

#### Boolean

This is the basic true/false parameter. The subparameters for a Boolean are Name, Default Value, and Description.
An example of the syntax would be:
```
def answer = input message: '<message>',
parameters: [booleanParam(defaultValue: true,
description: 'Prerelease setting', name: 'prerelease')]
```

![boolean-parameter.PNG](pictures/boolean-parameter.PNG)


#### Choice

This parameter allows the user to select from a list of choices. The subparameters for a Choice are Name, Choices, and Description. Here, Choices refers to a list of choices you enter to present to the user. An example of the syntax would be:

```
def choice = input message: '<message>',
parameters: [choice(choices: "choice1\nchoice2\nchoice3\nchoice4\n",
description: 'Choose an option', name: 'Options')]
```

Notice the syntax here for the list of choices—a single string with each choice separated by a newline character. There are other ways to instantiate a set of choices, but this is the simplest.

#### Credentials

This parameter allows the user to select a type and set of credentials to use. The available subparameters include Name, Credential Type, Required, Default Value, and Description.

The options for Credential Type include Any, Username with password, Docker Host Certificate Authentication, SSH Username with private key, Secret file, Secret text, and Certificate.

#### File

This parameter allows for choosing a file to use with the pipeline. The subparameters include File Location and Description. The syntax is:
```
def selectedFile = input message: '<message>',
parameters: [file(description: 'Choose file to upload', name: 'local')]
```

#### List Subversion tags

More in the book.

#### Multiline String

More in the book.

#### Password

This parameter allows the user to enter a password. For passwords, the text the user enters is hidden while they type it. The available subparameters are Name, Default Value, and Description. Here’s an example:
```
def pw = input message: '<message>',
parameters: [password(defaultValue: '',
description: 'Enter your password.', name: 'passwd')]
```
When run, the user is presented with a field to enter the password, with the text being hidden as they type.

#### Run

More in the book.

#### String

More in the book

## Return Values from Multiple Input Parameters

If there were instead no parameters, such as having only a Proceed or Abort option, then the return value would be null. And when you have multiple parameters, a map
is returned where you can extract each parameter’s return value via the parameter’s name. An example follows.

Suppose we wanted to add a traditional login screen to our pipeline. We would use two parameters—one String parameter for the login name and one Password parameter for the password. We can do that in the same input statement and then extract the return values for each from the returned map.

The following example code shows how to define the input statement along with some print statements that show different ways to access the individual return values (don’t forget that you can use the Snippet Generator for generating the input statement as well):

```
def loginInfo = input message: 'Login', parameters: [string(defaultValue: '', description: 'Enter Userid:', name: 'userid'), password(defaultValue: '', description: 'Enter Password:', name: 'passwd')]
echo "Username = " + loginInfo['userid']
echo "Password = ${loginInfo['passwd']}"
echo loginInfo.userid + " " + loginInfo.passwd
```

## Parameters and Declarative Pipelines

### Using the parameters section

Within the Declarative Pipeline structure, there is a section/directive for declaring parameters. This is within the agent block of the main pipeline closure. Use of the parameters directive is covered in detail with Declarative Pipelines in Chapter 7, but here’s a simple example of the syntax (see “parameters” on page 234 for more details):

```
pipeline {
    agent any
    parameters {
        string(name: 'USERID', defaultValue: '',
        description: 'Enter your userid')
    }
    stages {
        stage('Login') {
            steps {
                echo "Active user is now ${params.USERID}"
            }
        }
    }
}
```

If you are working in the Jenkins application itself, creating parameters like this in the code will also instantiate the “This build is parameterized” part of the job. **This approach is the recommended approach for Declarative Pipelines.**

### Using the Jenkins application to parameterize the build

If you have created a job in the Jenkins application (rather than using a Jenkinsfile automatically), a second approach for adding parameters is to simply use the traditional method for parameterizing a job. That is, in the General configuration section, select the checkbox for “This project is parameterized” and then define your parameters as normal in the job’s web interface (Figure 3-8).

![Generating-parameters-in-jenkins-job.PNG](pictures/Generating-parameters-in-jenkins-job.PNG)

You can then simply reference the job parameters via params.<name of parameter> without having the input line in the code, as shown here:

```
pipeline {
    agent any
    stages {
        stage('Login') {
            steps {
                echo "Active user is now ${params.USERID}"
            }
        }
    }
}   
```

### Using a script block

While Declarative Pipelines are continuing to evolve and add more functionality, there may still be instances where you need to do something in one that the declarative style doesn’t support or renders very difficult to implement. For those cases, the declarative syntax supports a ``script`` block. 

A script block allows you to use nondeclarative syntax within the bounds of the block. This includes defining variables, which is not something you can do in a Declarative Pipeline outside of a script block. This also means that you cannot reference variables that are defined inside a script block outside of that block. Jenkins flags those with a "no such property" error.

```
stage ('Input') {
    steps {
        script {
            def resp = input message: '<message>',
            parameters: [string(defaultValue: '',
            description: 'Enter response 1',
            name: 'RESPONSE1'), string(defaultValue: '',
            description: 'Enter response 2', name: 'RESPONSE2')]
            echo "${resp.RESPONSE1}"
        }
        echo "${resp.RESPONSE2}"
    }
}
```

Here we have two parameters defined as part of an input step inside of a stage in a Declarative Pipeline. Since the first echo is in the script block where the variable resp is also defined, it will print out the response that is entered for that parameter as expected.

Notice, though, that the second echo is outside of the scope where the resp variable is defined. Groovy/Jenkins will throw an error when it gets to this one.

Because of this, it is advisable to try to limit accessing input to a small section of your code if you have to use a script block. However, there is one other workaround if you need to use the value outside the scope of the script block. **You can put the return value into an environment variable and then access the environment variable wherever you need the value.**

```
stage ('Input') {
    steps {
        script {
            env.RESP1 = input message: '<message>', parameters: [
                string(defaultValue: '', description: 'Enter response 1',
                name: 'RESPONSE1')]
            env.RESP2 = input message: '<message>', parameters: [
                string(defaultValue: '', description: 'Enter response 2',
                name: 'RESPONSE2')]
            echo "${env.RESP1}"
        }
        echo "${env.RESP2}"
    }
}
```

We are putting the results of the input steps into the environment variable namespace (env). **Because these are environment variables, the values are set in the environment and therefore available for the pipeline to use wherever it needs.** 

### Using external code

One other option available to you is putting scripted statements (like the calls to input) in an external shared library or an external Groovy file that you load and execute. For example, we could code our input processing in a file named ``vars/getUser.groovy`` in a shared library structure, like this:

```
#!/usr/bin/env groovy
def call(String prompt1 = 'Please enter your data', String prompt2 = 'Please enter your data') {
    def resp = input message: '<message>', parameters: [string(defaultValue: '', description: prompt1, name: 'RESPONSE1'), string(defaultValue: '', description: prompt2, name: 'RESPONSE2')]
    echo "${resp.RESPONSE1}"
    echo "${resp.RESPONSE2}"
    // do something with the input
}
```

If our library were named ``Utilities``, then we could import it and call the ``getUser`` function as shown here:

```
@Library('Utilities')_
pipeline {
    agent any
    stages {
        stage ('Input') {
            steps {
                getUser 'Enter response 1','Enter response 2'
            }
        }
    }
}
```

One of the challenges with using an input statement is what happens if you don’t get input in an expected amount of time. While waiting for input, the node is effectively stopped, waiting on a response. To prevent this from going on too long, **you should consider wrapping the input call with another type of flow control construct: the ``time out`` statement.**

## Flow Control Options

One of the benefits of writing your pipeline-as-code in Jenkins is that you have more options for controlling the flow through the pipeline. 

### timeout

The timeout step allows you to limit the amount of time your script spends waiting for an action to happen. The syntax is fairly simple. Here’s an example:
```
timeout(time:60, unit: 'SECONDS') {
    // processing to be timed out inside this block
}
```

A best practice is to wrap any step that can pause the pipeline (such as an input step) with a timeout. This is so that your pipeline continues to execute (if desired) even if something goes wrong and the expected input doesn’t occur within the time limit. Here’s an example:
```
node {
    def response
    stage('input') {
        timeout(time:10, unit:'SECONDS') {
            response = input message: 'User', parameters: [string(defaultValue: 'user1', description: 'Enter Userid:', name: 'userid')]
        }
        echo "Username = " + response
    }
}
```

### retry

The retry closure wraps code in a step that retries the process n times if an exception occurs in the code. n here refers to a value you pass in to the retry step. The syntax is just:

```
retry(<n>) { // processing }
```

If the retry limit is reached and an exception occurs, then the processing is aborted (unless that exception is handled, such as with a try-catch block).

### sleep

This is the basic delay step. It accepts a value and delays that amount of time before continuing processing. The default time unit is seconds, so sleep 5 waits for 5 seconds before continuing processing. If you want to specify a different unit, you just add the unit name parameter, as in:
```
sleep time: 5, unit: 'MINUTES'
```

### waitUntil

More in the book.

## Dealing with Concurrency

For the most part, having concurrency in your pipeline builds is a good thing. Typically, concurrency refers to parallelism—being able to run similar parts of your jobs concurrently on different nodes. This can be especially useful in cases such as running tests, as long as you limit duplicate access to resources appropriately.

### Locking Resources with the lock Step

If you have the **Lockable Resources plugin** installed, there is a DSL ``lock`` step available to restrict multiple builds from trying to use the same resource at the same time.

"Resource" here is a loose word. It could mean a node, an agent, a set of them, or just a name to use for the locking.

The DSL lock step is a blocking step. It locks the specified resource until the steps within its closure are completed. In its simplest case, you just supply the resource name as the default argument. For example:
```
lock('worker_node1') {
    // steps to do on worker_node1
}
```

Alternatively, you can supply a label name to select a set of resources that have a certain label and a quantity to specify the number of resources that match that label to lock (reserve):
```
lock(label: 'docker-node', quantity: 3) {
// steps
}
```

You can think of this as, "How many of this resource do I have to have available to proceed?" If you specify a label but no quantity, then all resources with that label are locked.
Finally, there is an ``inversePrecedence`` optional parameter. If this parameter is set to true, then the most recent build will get the resource when it becomes available. Otherwise, builds are awarded the resource in the same order that they requested it.

As a quick example, consider a Declarative Pipeline where we want to use a certain agent to do the build on, no matter how many instances of the pipeline we are running. (Perhaps it is the only agent with the specific tools or setup we want at the moment.) Our code might look like this with the lock step:

```
stage('Build') {
    // Run the gradle build
    steps {
        lock('worker_node1') {
            sh 'gradle clean build -x test'
        }
    }
}
```

If we start multiple builds running of the same project or if we have multiple projects with this same lock code for the resource, then one build/project will get the resource first and other builds/projects will have to wait. For the first build or project that gets the resource, the console log might show something like this:

```
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] lock
00:00:02.858 Trying to acquire lock on [worker_node1]
00:00:02.864 Resource [worker_node1] did not exist. Created.
00:00:02.864 Lock acquired on [worker_node1]
[Pipeline] {
[Pipeline] tool
[Pipeline] sh
00:00:02.925 [gradle-demo-simple-pipe] Running shell script
00:00:03.213 + /usr/share/gradle/bin/gradle clean build -x test
00:00:06.671 Starting a Gradle Daemon
...
00:00:16.887
00:00:16.887 BUILD SUCCESSFUL
00:00:16.887
00:00:16.887 Total time: 13.16 secs
[Pipeline] }
00:00:17.187 Lock released on resource [worker_node1]
[Pipeline] // lock
```

And for the other builds/jobs trying to acquire the same lock, console output might look like this:

```
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] lock
00:00:03.262 Trying to acquire lock on [worker_node1]
00:00:03.262 Found 0 available resource(s). Waiting for correct
amount: 1.
00:00:03.262 [worker_node1] is locked, waiting...
```

### Controlling Concurrent Builds with Milestones

One of the scenarios that you might have to deal with at some point in Jenkins is builds of the same pipeline running concurrently that can have contention for resources. The runs could be reaching key points at different times and stepping on each other, or one run could be modifying required resources that leave things in a bad state when the other run makes it to that point. In short, there’s no guarantee that after one run has modified a resource, another run won’t come along and modify it while the earlier run is still in progress.

To prevent the case where builds could run out of order (in terms of the order they were started) and step on each other, Jenkins pipelines can use the ``milestone`` step. When a milestone step is put in the pipeline, it prevents an older build from moving past the ``milestone``, if the newer build has already gotten there.

The following example shows a milestone step placed in a script after a Gradle build:

```
  sh "'${gradleLoc}/bin/gradle' clean build"
}
milestone label: 'After build', ordinal: 1
stage("NotifyOnFailure") {
```

Suppose we have two runs of this build happening concurrently. If build #11 gets to the milestone step first during its processing, then when build #10 arrives, it will be canceled. This prevents build #10 from overwriting or modifying any resources already in use or modified by build #11.

The rules for milestone processing can be summed up as:
* Builds pass the milestones in order by build number.
* Older builds abort if a newer build has already passed the milestone.
* When a build passes a milestone, Jenkins aborts older builds that have passed the previous milestone, but not this milestone.
* If an older build passes a milestone, newer builds that haven’t passed the milestone won’t abort it.

### Restricting Concurrency in Multibranch Pipelines

The pipeline DSL includes a way to restrict Multibranch Pipelines to only building one branch at a time. When this is in place (in the Jenkinsfiles of the branches), requested builds for branches other than the one currently building will be queued.

In a scripted syntax, the property can be set this way:
```
properties([disableConcurrentBuilds()])
```
In declarative syntax, it would look like this:
```
options {
  disableConcurrentBuilds()
}
```

### Running Tasks in Parallel

More in the book.

#### stash and unstash

More in the book.

### Alternative parallel syntax for Declarative Pipelines

More in the book.

#### parallel and failFast

More in the book.

## Conditional Execution

Jenkins pipelines can provide similar functionality. In the case of a Scripted Pipeline, it’s as simple as using the Groovy/Java language conditionals in your pipeline code.

```
node ('worker_node1') {
    def responses = null
        stage('selection') {
            responses = input message: 'Enter branch and select build type', parameters:[string(defaultValue: '', description: '', name: 'BRANCH_NAME'), choice(choices: 'DEBUG\nRELEASE\nTEST', description: '', name: 'BUILD_TYPE')]
        }
        stage('process') {
        if ((responses.BRANCH_NAME == 'master') && (responses.BUILD_TYPE == 'RELEASE')) {
            echo "Kicking off production build\n"
        }
    }
}
```

Declarative Pipelines in Jenkins provide their own implementation for executing code based on conditionals. In general, it takes the form of a ``when`` that tests one or more ``expression`` blocks to see whether they are true. If so, then the remaining code in a stage is executed. If not, then the code is not executed.
```
pipeline {
    agent any
    parameters {
        string(defaultValue: '', description: '', name : 'BRANCH_NAME')
        choice ( choices: 'DEBUG\nRELEASE\nTEST', description: '', name : 'BUILD_TYPE')
    }
    stages {
        stage('process') {
            when {
                allOf {
                    expression {params.BRANCH_NAME == "master"}
                    expression {params.BUILD_TYPE == 'RELEASE'}
                }
            }
            steps {
                echo "Kicking off production build\n"
            }
        }
    }
}
```

Notice the use of the parameters section to formally define the parameters in use in the Declarative Pipeline. Also, you can see how the when and allOf blocks combine like the if-&& construct in the Scripted Pipeline.

## Post-Processing

Post-build actions where users can add actions that always occur after a build is finished, regardless of whether it completed successfully, failed, or was aborted.

### Scripted Pipelines Post-Processing

Scripted Pipelines do not have built-in support for post-build processing. Use ``try-catch-finally`` mechanism.

However, the Jenkins DSL includes another step that acts as a shortcut for the ``try-catch-finally`` functionality: ``catchError``.

#### try-catch-finally

What we want to have is a way to always do certain actions regardless of the final state of the build. We can accomplish that by catching any exceptions with a ``try-catch`` and using the finally clause to then do our processing based on the build’s state.

```
def err = null
try {
    // pipeline code
    node ('node-name') {
        stage ('first stage') {
        ...
        } // end of last stage
    }
}
catch (err) {
    currentBuild.result = "FAILURE"
}
finally {
    (currentBuild.result != "ABORTED"){
        // Send email notifications for builds that failed
        // or are unstable
    }
}
```

The try-catch could also be within the node block if we preferred. That would, however, not catch issues thrown while trying to get the node, which might also not be able to send the notification. Finally, if we wanted to propagate the error, we could throw it again in our finally block.

#### catchError

The Jenkins pipeline syntax also provides a more advanced way of handling exceptions. The ``catchError`` block provides a way to detect the exception and change the overall build status, but still continue the processing.

With the ``catchError`` construct, if an exception is thrown by a block of code, the build is marked as a failure. But the code in the pipeline continues to be executed from the statement following the catchError block.

An example of using this is shown here:
```
node ('node-name') {
    catchError {
        stage ('first stage') {
            ...
        } // end of last stage
    }
    // step to send email notifications
}
```

This is essentially equivalent to the following code:
```
node ('node-name') {
    try {
        stage ('first stage') {
            ...
        } // end of last stage
    } catch (err) {
        echo "Caught: ${err}"
        currentBuild.result = 'FAILURE'
    }
    // step to send email notifications
}
```

The advantages are the simpler syntax and the build result automatically being marked as failed if an exception occurs.

### Declarative Pipelines and Post-Processing

Declarative Pipelines have a dedicated section for post-build processing. Not surprisingly, the section is called ``post``. A ``post`` section can be at the end of a stage or at the end of a pipeline — or both. The declarative syntax provides several predefined “build conditions” that can be checked and, if true, then initiate further action: 
* ``always`` -  Always executes the steps in the block
* ``changed`` - Executes the steps in the block if the current build’s status is different from the previous build’s status
* ``success`` - Executes the steps in the block if the current build was successful
* ``failure`` - Executes the steps in the block if the current build failed
* ``unstable`` - Executes the steps in the block if the current build’s status is unstable

So, for example, we can declare that if the failure condition is true, we want to send an email about the failure. The syntax here is fairly simple. Here’s an outline for a simple post structure at the end of a build:
```
        }
    } // end stages
    post {
        always {
            echo "Build stage complete"
        }
        failure {
            echo "Build failed"
            mail body: 'build failed', subject: 'Build failed!', to: 'devops@company.com'
        }
        success {
            echo "Build succeeded"
            mail body: 'build succeeded', subject: 'Build Succeeded', to: 'devops@company.com'
        }
    }
} // end pipeline
```

Notice that the post section for the entire build comes after all of the stages in the pipeline.

# Chapter 4. Notifications and Reports

One of the core uses of Jenkins is implementing automation. In addition to repeatable processing that is triggered by some event, we also rely on being automatically notified when processes have completed, and of their overall status. Additionally, many plugins and steps produce useful reports as part of their processing.

## Notifications

For most cases, this will happen in the “post-processing” parts of a pipeline.

## Email

### Jenkins Location


# Chapter 5. Access and Security

# Chapter 6. Extending Your Pipeline

# Chapter 7. Declarative Pipelines

Declerative pipelines provide:
* There is a well-defined, enforced structure. (You can think of this like the sections on the pages of a Jenkins web form.)
* Defining a pipeline section is more about declaring the high-level steps/goals than defining the logic to accomplish it. (This is similar to filling in the fields in a Jenkins web form.)
* Familiar Jenkins processing constructs are provided and don’t have to be emulated with programming. (For example, you have a way to do post-build processing and send notifications, as opposed to having to use try-catch-finally Groovy programming to handle this.)
* All of the above enable better validation and error checking. (Errors are identified and presented in the context of the expected structure and keywords, not just Groovy tracebacks.)

Declarative Pipelines are easiest for someone new to using the pipeline functionality. This is because they more closely resemble what was done and available in the web forms, and they have clearer, more contextual validation and error checking.

Scripted Pipelines provide more flexibility and the ability to mix in programming constructs to execute logical flows, decision handling, assignments, etc. that are not available in Declarative Pipelines. For more experienced users or advanced applications, Scripted Pipelines can be the best option.

## Motivation

### Not Intuitive

As we’ve discussed, moving from a web interface (with specific forms, help buttons, and UI elements that guide you in setting up jobs) to creating scripts, is not intuitive.

### Getting Groovy

While it’s not a requirement to be able to program in Groovy to create DSL scripts, sometimes it can feel that way to users. For missing functionality, Groovy constructs may be the only alternative. Verification such as syntax checking is done at the Groovy level. Also, errors are surfaced as Groovy errors (tracebacks) and not as DSL specific ones.

### Additional Assembly Required

Building on a point raised earlier, additional code can be required to get the familiar Jenkins constructs we had in the web forms version. For example, the simple task of sending email after a failed build has to be handled with something like a try-catchfinally construct, instead of the familiar built-in post-build functionality.

## The Structure

### Block

A block here is really just any set of code that has a beginning and end. In Groovy, this translates to a closure (a section of code where the beginning and end are bracketed with { and }). It looks like this:
```
pipeline {
  // code in declarative syntax
}
```

### Section

Sections in a Declarative Pipeline are a way to collect items that need to be executed at particular points during the overall flow of the pipeline. Currently, there are three areas we refer to as sections:
* stages - This section wraps all of the individual stage definitions (directives) that define the main body and logic for the pipeline.
* steps - This section wraps a set of DSL steps within a stage definition. It serves to separate the collection of steps from other items within a stage, such as environment definitions.
* posts - This section wraps around steps and conditions to be done or checked at the end of a pipeline run or at the end of a stage.

```
pipeline {
    agent any
    stages {
        stage('name1') {
            steps {
                ...
            }
            post {
                ...
            }
        }
        stage('name2') {
            steps {
                ...
            }
        }
    }
    post {
        ...
    }
}
```

### Directives

A directive can be thought of as a statement or block of code that does any of the following in a pipeline:
* Defines values - An example of this is the agent directive, which allows us to specify a node or container to run an entire pipeline or a stage in. If we wanted to run our pipeline on a node named worker, we could use ``agent ('worker')``.
* Configures behavior - An example of this is the triggers directive that lets us configure how often Jenkins checks for source updates or triggers our pipeline. If we wanted it to retrigger our pipeline at 7 a.m. every weekday, we could use triggers { cron ('0 7 0 0 1-5') }.
* Specifies actions to be done - An example of this is the stage directive, which is expected to have a steps section containing DSL steps to be executed.

### Steps

The label ``steps`` itself is a section title with in a stage of the pipeline. However, within the ``steps`` section, we can have any valid DSL statement, such as git, sh, echo, etc. You can think of a step here as corresponding to one of these statements.

### Conditionals

Conditionals supply a condition or criteria under which an action should occur. These are optional. There are two cases you may encounter/use:
* When: Strictly speaking, this is a directive. It resides within a stage definition and defines criteria for whether or not a stage should be executed. For example:
```
stage ('build') {
  when {
    branch 'foo'
  }
  <steps>
}
```
* Conditions blocks in the post section that define the criteria for doing postprocessing. The criteria (conditions) here refer to the status of the build, such as success or failure.

## The Building Blocks

Those with **dotted lines around them are optional in that part of the structure**. Those with **solid lines are required in that part of the structure**. Note that there are some directives that can occur at both the pipeline and stage level. They may be required in one area and optional in another.

![Overview-of-declerative-pipeline-structure.PNG](pictures/Overview-of-declerative-pipeline-structure.PNG)

### pipeline

The ``pipeline`` block is required in a Jenkins Declarative Pipeline. It is the outermost section and signals that this is a Pipeline project.
```
pipeline {
  // pipeline code
}
```

### agent

The ``agent`` directive specifies where the entire pipeline or a specific stage runs. This is similar to how the node directive is used in Scripted Pipelines. In fact, you can reasonably think of an agent as a node, except that the master node is not an agent.

**An ``agent`` directive near the top of the pipeline block is required as a "default" place for execution.** However, individual agent directives can optionally be specified at the beginning of individual stages to indicate where the code in those stages should be run.

What the agent directive actually does is indicate which (if any) nodes to use in the execution of the pipeline or stage. It does this by mapping the argument supplied to it to the label(s) specified for the nodes in your Jenkins system.

The possible options are summarized in the following sections:
* ``agent any`` - This syntax tells Jenkins that the pipeline or stage can run on any agent that is defined, without regard to what label it has.
* ``agent none`` - When used at the top level, this indicates that we are not specifying an agent globally for the pipeline. The implication is that an agent will be specified, if needed, for individual stages.
* ``agent { label "<label>"}`` - This indicates that the pipeline or stage can run on any agent that has the label <label>.
    
#### Labels and custom workspaces

A recent addition to the label syntax for agents allows us to specify a custom workspace for a pipeline or stage. Given an agent definition, we can include the custom Workspace directive to specify where the workspace that the agent uses should live. The syntax looks like this:
```
agent {
  label {
    label "<labelname>"
    customWorkspace "<desired directory>"
  }
}
```

#### Agents and Docker

The final agent options we’ll look at are Docker containers. There are two shorthand ways to get a Docker image — specifying an existing image or creating an image from a Dockerfile — in the ``agent`` declaration. Alternatively, the longer version of the declaration can be used to specify additional elements, such as a node to use for the container, and arguments for the container.

First, we’ll look at the formats for using an existing Docker image:
* ``agent { docker '<image>' }`` - This short syntax tells Jenkins to pull the given image from Docker Hub and run the pipeline or stage in a container based on the image, on a dynamically provisioned node.
* ``agent { docker { <elements> } }`` - This long syntax allows for defining more specifics about the Docker agent. There are three additional elements that you can add in the declaration (within the { } block):
  * ``image '<image>'`` - Like the short form, this tells Jenkins to pull the given image and use it to run the pipeline code.
  * ``label '<label>'`` -  If this element is present in the declaration, it tells Jenkins to instantiate the container and "host" it on a node matching <label>. 
  * ``args '<string>'`` - If this element is present in the declaration, it tells Jenkins to pass these arguments to the Docker container; the syntax here should be the same as you would normally pass to a Docker container.

Here’s an example declaration using the long form:
```
agent {
  docker {
    image "image-name"
    label "worker-node"
    args "-v /dir:dir"
  }
}
```

The syntax for using a Dockerfile as the basis for the container is similar. Again, there are short and long forms:
* ``agent { dockerfile true }`` - This short syntax is intended to be used when you have a source code repository, that you retrieve, that has a Dockerfile in its root (note that dockerfile here is a literal). In that case, this will tell Jenkins to build a Docker image using that Dockerfile, instantiate a container, and then run the pipeline (or the stage’s code if run in a stage) in that container.
* ``agent { dockerfile { <elements> } }`` -  This long syntax allows for defining more specifics about the Docker agent you are trying to create from a Dockerfile. There are three additional elements that you can add in the declaration (within the { } block): 
  * ``filename '<path to dockerfile>'`` -  This allows for specifying an alternate path to a Dockerfile, including a different name. Jenkins will try to build an image from the Dockerfile, instantiate a container, and use it to run the pipeline code.
  * ``label '<label>'`` - If this element is present in the declaration, it tells Jenkins to instantiate the container and “host” it on a node matching <label>.
  * ``args '<string>'`` - If this element is present in the agent Dockerfile declaration, it tells Jenkins to pass these arguments to the Docker container; the syntax here should be the same as you would normally pass to a Docker container.
    
An example of specifying a Docker agent via a Dockerfile using the long form is shown here:
```
agent {
  dockerfile {
    filename "<subdir/dockerfile name>"
    label "<agent label>"
    args "-v /dir:dir"
  }
}
```

##### Using the same node for Docker and non-Docker stages
There is one other aspect associated with using Docker agents. Suppose you define a particular non-Docker agent at the top of your pipeline:
```
pipeline {
  agent {label 'linux'}
```
Later, in a particular stage, you want to run the code in a Docker container - but you also want to use the same node and workspace that you defined for the pipeline. To enable this, the pipeline has a directive you can use with the Docker specification: ``reuseNode``. It would look something like the following in practice:
```
stage 'abc' {
  agent {
    docker {
      image 'ubuntu:16.6'
      reuseNode true
```

This tells Jenkins to reuse the same node and workspace that were defined for the original pipeline agent to “host” the resulting Docker container. Next, we’ll look at how to configure environment values for a pipeline.

### environment

This is an optional directive for your Declarative Pipeline. As the name implies, this directive allows you to specify names and values for environment variables that are then accessible within the scope of your pipeline. Like agent, you can have an instance of ``environment`` in the main pipeline definition and/or in individual stages.

An environment definition in the top-level pipeline block will make the variable accessible to all steps in the pipeline. An environment definition within a stage will make the variable accessible to only the scope of the stage. Here is an example of defining an environment variable in this way:
```
environment {
  TIMEZONE = "eastern"
}
```

Environment variable definitions can also incorporate variables that are already defined. The syntax for this is just to include the existing variable in the definition string in ${<variable>}:

```
environment {
  TIMEZONE = "eastern"
  TIMEZONE_DS = "${TIMEZONE}_daylight_savings"
}
```

#### Credentials and environment variables

In the environment block, you can assign a global variable to a particular credentials ID. Then you can use that variable throughout your pipeline in place of the ID. This can simplify things if you need to specify the ID in multiple places. The syntax is to assign the variable name to the string credentials('<credentials-id>'). For example:

```
environment {
  ADMIN_USER = credentials('admin-user')
}
```

### tools

Jenkins users are familiar with using the Global Tool Configuration screen to configure versions, paths, and installers for tools. Once configured there, the tools directive allows us to specify which of these we want to have autoinstalled and made available in the path on the agent we’ve chosen.

For example, suppose we had the configuration shown in Figure 7-4.

![Global-gradle-tool.PNG](pictures/Global-gradle-tool.PNG)

Then, in our tools block, we could refer to Gradle via:
```
tools {
  gradle "gradle3.2"
}
```

The lefthand part of this declaration is a specific string defined in the pipeline model. As of this writing, the valid tool types you can specify in declarative syntax are:
* ant
* git
* gradle
* jdk
* jgit
* jgitapache (JGit with Apache HTTP client)
* maven

Attempts to use other types that are not yet valid will result in an "Invalid tool type" error when running your pipeline.

Once this is set up, the tool is autoinstalled and put on the path. We can then simply use the string gradle in place of the GRADLE_HOME path in our pipeline steps and Jenkins will map it back to this Gradle installation on our system. For example:
```
steps {
  sh 'gradle clean compile'
}
```

Also, it’s worth noting that the tools directive can use the value of a parameter if you need to input a particular version to use. Here’s an example:
```
pipeline {
  agent any
  parameters {
    string(name: 'gradleTool', defaultValue: 'gradle3',
    description: 'Gradle Version')
  }
  tools {
    gradle "${params.gradleTool}"
  }
```

Just keep in mind that there is currently a limitation with the declarative syntax such that Jenkins doesn’t recognize that a build requires a parameter the first time the pipeline is run.


### options

This directive can be used to specify properties and values for predefined options that should apply across the pipeline. These would be the type of things that we would set on the General tab of a project in the Jenkins web forms (other than parameters, which have their own section). You can think of it as a place to set Jenkins-defined job options.

A simple example is the option to discard builds.
```
options {
  buildDiscarder(logRotator(numToKeepStr:’3’))
}
```
As well, there can be specific options for the declarative structure. Here’s an example of one:
```
options {
  skipDefaultCheckout()
}
```

#### Options summary

The following list below enumerates the available options and, briefly, their meaning and usage:
* ``buildDiscarder`` - Keep the console output and artifacts for the specified number of executions of the pipeline.
* ``disableConcurrentBuilds`` - Prevent Jenkins from starting concurrent executions of the same pipeline. The use case could be for preventing simultaneous access to shared resources or preventing a faster concurrent execution from overtaking a slower one.
```
options { disableConcurrentBuilds() }
```
* ``retry`` - If the pipeline execution fails, retry the entire pipeline the specified number of times.
```
options { retry(2) }
```
* ``skipDefaultCheckout`` - this removes an implied checkout scm statement, thus skipping the automatic source code checkout from a pipeline defined in a Jenkinsfile.
* ``skipStagesAfterUnstable`` - If a stage of the pipeline renders the pipeline unstable, don’t process the remaining stages.
```
options { skipStagesAfterUnstable()}
```
* ``timeout`` - Sets a timeout value for an execution of the pipeline. If this timeout value is passed, Jenkins will abort the pipeline.
```
options { timeout(time: 15, unit: 'MINUTES') }
```
* ``timestamps`` - Add timestamps to the console output. This option requires the Timestamper plugin. Note that this option applies globally to the whole pipeline execution.
```
options { timestamps() }
```

### triggers

This directive allows you to specify what kinds of triggers should initiate builds in your pipeline. Note that these do not apply to Multibranch Pipeline or GitHub organization or Bitbucket team/project jobs that are marked by Jenkinsfiles and triggered otherwise — such as by a webhook that notifies Jenkins when a change is made.

There are four different (SCM-neutral) triggers currently available: 
* ``cron`` - Refers to executing the pipeline at a specified regular interval, and ``pollSCM`` is for checking for source code updates (polling the source control management system) at a specified regular interval. If a source change is detected, the pipeline will be executed.
* ``upstream`` - Takes a comma-separated string of Jenkins jobs and a condition to check. When a job in the string finishes and the result matches the treshold, the current pipeline will be retriggered. For example:
```
triggers {
  upstream(upstreamProjects: 'jobA,jobB', threshold: hudson.model.Result.SUCCESS)
}
```
* ``githubPush`` - Refers to the same kind of behavior as the “GitHub hook trigger for GitSCM polling” setting in the Build Triggers section of a project in the Jenkins application. That is, if a webhook is set up on the GitHub side for events related to the GitHub repository, then when the payload is sent to Jenkins, it will trigger SCM polling for that repo from the Jenkins job to pick up any changes. The syntax should be simply:
```
triggers { githubPush() }
```

**Note.** There is ``bitbucketPush`` similar to ``githubPush``.

### parameters

This directive allows us to specify project parameters for a Declarative Pipeline. The input values for these parameters can come from a user or an API call. You can think of these parameters as being the same sort that you would specify in the web form with the “This build is parameterized” option.

The valid parameter types, with a description and example of each, are listed here:
* ``booleanParam`` - This is the basic true/false parameter. The subparameters for a ``booleanParam`` are ``name``, ``defaultValue``, and ``description``.
```
parameters { booleanParam(defaultValue: false, description: 'test run?', name: 'testRun')}
```
* ``choice`` - This parameter allows selection from a list of choices. The subparameters for a choice are ``name``, ``choices``, and ``description``. Here, choices refers to a list of choices you enter, **separated by newlines**, to present to the user. The first one in the list will be the default.
```
parameters{ choice(choices: 'Windows-1\nLinux-2', description: 'Which platform?', name: 'platform')}
```
* ``file`` - This parameter allows for choosing a file to use with the pipeline. The subparameters include ``fileLocation`` and ``description``. The selected file location specifies where to put the file that is selected and uploaded. The location will be relative to the workspace.
```
parameters{ file(fileLocation: '', description: 'Select the file to upload')}
```
* ``text`` - This parameter allows the user to input multiple lines of text. The subparameters include ``name``, ``defaultValue``, and ``description``.
```
parameters{ text(defaultValue: 'No message', description: 'Enter your message', name: 'userMsg')
```
* ``password`` - This parameter allows the user to enter a password. For passwords, the text entered is hidden. The available subparameters are name, defaultValue, and description.
```
parameters{ password(defaultValue: "userpass1", description: 'User password?', name: 'userPW')}
```
* ``run`` - This parameter allows the user to select a particular run from a job. This might be used, for example, in a testing environment. The subparameters available include ``name``, ``project``, ``description``, and ``filter``.
* ``string`` - This parameter allows for entering a string. (This is not hidden like a password parameter is.) The subparameters include ``description``, ``defaultValue``, and ``name``.

#### Using parameters in a pipeline

Once you define a parameter in the parameters block, you can reference it in your pipeline via the ``params`` namespace, as in ``params.<parameter_name>``. Here’s a simple example using a ``string`` parameter in a Declarative Pipeline:
```
pipeline {
  agent any
  parameters{
    string(defaultValue: "maintainer", description: 'Enter user role:', name: 'userRole')
  }
  stages {
    stage('listVals') {
      steps {
        echo "User's role = ${params.userRole}"
      }
    }
  }
}
```

### libraries

One of the newer directives introduced in Jenkins for Declarative Pipelines is the ``libraries`` directive. This directive allows Declarative Pipelines to import shared libraries so that code contained in them can then be called and used.

In addition to providing a way to share and include common code, **shared libraries can also be valuable for Declarative Pipeline use by encapsulating code that is not declarative, and couldn’t normally be directly used in a pipeline**.

The syntax here is pretty straightforward, as shown in the following example. Note that the @ sign here provides a way of specifying (after it) which version of a shared library we want. In the first lib statement here, we are asking for the latest version from the master branch for this library:
```
pipeline {
  agent any
  libraries {
    lib("mylib@master")
    lib("alib")
  }
  stages {
    ...
```

## stages

Whether in a Scripted Pipeline or a Declarative Pipeline, Jenkins wants our code steps to be contained in one or more stages. In a Declarative Pipeline, the collection of individual stages is wrapped by the ``stages`` section. **``stages`` is a required section, and you must have at least one stage within it.** A section of a pipeline demonstrating this syntax is shown here:
```
pipeline {
  agent any
  stages {
    stage('name1') {
      steps {
        ...
      }
```

### stage

Within the ``stages`` section are the individual ``stage``s. Each ``stage`` has at least a name and one or more DSL ``steps``. Within a ``stage``, you may also have local environment, tools, and agent directives. **If there are also corresponding global directives that define values with the same names, then the value defined in the directive in the stage will override the global one.**

### steps

The ``steps`` block is required and indicates the actual work that will happen in the stage. It has the form:
```
steps {
  <individual steps - i.e., DSL statements>
}
```

#### Conditional execution of a stage

In any stage, you can have conditional execution. That is, you can have Jenkins decide whether or not to execute the steps in the stage based on one or more conditions evaluating to true.

There are several different conditions that you can work with. The choices are:
* ``branch "<name>"`` - Only proceed if the branch name is <name> or matches the (Ant-style) pattern.

```
stage('debug_build') {
  when {
    branch 'test'
  }
  ...
}
```

* ``environment name: <name>, value: <value>`` - Only proceed if the specified environment variable <name> has the specified environment variable <value>.

```
stage('debug_build') {
  when {
    environment name: "BUILD_CONFIG", value: "DEBUG"
  }
  ...
}
```

* ``expression <valid Groovy expression>`` - Only proceed if the specified Groovy expression evaluates to true (meaning not false and not null).

```
stage('debug_build') {
  when {
    expression {
      echo "Checking for debug build parameter..."
      expression { return params.DEBUG_BUILD }
    }
    ...
}
```

#### Conditional execution with and, or, not.

In addition to using these conditions one at a time only when they are true, we can also use logical operators to check multiple conditions, or the inverse of one. The keywords for the three logical operators are:
* ``allOf`` - When used in a ``when`` statement for conditional stage execution, the ``allOf`` keyword functions like an “and.” In order for the stage to proceed with its processing, "all of" the conditions included must be true.
```
when {
  allOf {
    environment name: "BUILD_CONFIG", value: "DEBUG"
    branch 'test'
  }
}
```

* ``anyOf`` - When used in a when statement for conditional stage execution, the anyOf keyword functions like an "or." In order for the stage to proceed with its processing, "any of" the conditions included must be true.

```
when {
  anyOf {
    environment name: "BUILD_CONFIG", value: "DEBUG"
    branch 'test'
  }
}
```

* ``not`` - When used in a when statement for conditional stage execution, the not keyword functions just as the name implies. In order for the stage to proceed with its processing, the specified conditions must not be true.

```
when {
  not {
    branch 'prod'
  }
}
```

## post

post is another section available for use in the pipeline or in a stage. It is optional in both places. If present, it gets executed at the end of a pipeline or stage if the conditions are met.

The conditions in the post block are based on the build status. The syntax is as follows:
```
post {
  <condition name> {
    <valid DSL statements>
  }
  <condition name> {
    <valid DSL statements
  }
...
```
The available conditions are:
* ``always`` - Always execute the steps in the block.
* ``changed`` - If the current build’s status is different from the previous build’s status, then execute the steps in the block.
* ``success`` - If the current build was successful, then execute the steps in the block.
* ``failure`` - If the current build failed, then execute the steps in the block.
* ``unstable`` - If the current build’s status was unstable, then execute the steps in the block.

Here’s an example of using these notifications in a stage:
```
stage('Build') {
  steps {
    gradle 'clean build'
    ...
  }
  post {
    always {
      echo "Build stage complete"
    }
    failure{
      echo "Build failed"
      mail body: <some text>, subject: 'Build failed!',
      to: 'devops@mycompany.com'
    }
    success {
      echo "Build succeeded"
      archiveArtifacts '**/*'
    }
  }
}
```

## Dealing with Nondeclarative Code

The Declarative Pipeline syntax is great for simplifying the way we define pipelines. However, if you need to do something that can’t be expressed declaratively, it can be challenging to figure out how to accomplish that within the declarative structure. Let’s take, for example, cases where you may need to do a simple assignment operation, or multiple ones. Here are some sample assignments needed to use Artifactory with Gradle in Scripted Pipeline code:
```
  def server = Artifactory.server 'my-server-id'
  def rtGradle = Artifactory.newGradleBuild()
  rtGradle.tool = 'gradle tool name'
```

Attempting to put these in a steps section in a stage and run them yields a failed build with error messages like these:
```
org.codehaus.groovy.control.MultipleCompilationErrorsException:
startup failed:
        WorkflowScript: 15: Expected a step @ line 15, column 16.
        def server = Artifactory.server 'my-server-id'
        ^
        WorkflowScript: 17: Expected a step @ line 17, column 1.
        def rtGradle = Artifactory.newGradleBuild()
        ^
        WorkflowScript: 19: Expected a step @ line 19, column 1.
        rtGradle.tool = 'gradle3'
        ^
    3 errors
```

The problem here is that these assignment statements are trying to directly modify values via the DSL and are not declarative. While these statements are legal to use in Scripted Pipelines, they are not in Declarative Pipelines.

## Solutions

### Check Your Plugins

If you are trying to port scripted code that works with a plugin, check to see if there is an updated version of the plugin that supports the declarative syntax.

### Create a Shared Library

Earlier in this chapter, we discussed the libraries directive for importing shared libraries into Declarative Pipelines. Rather than having to try to embed the code directly in the pipeline, you can put it in a shared library, then load the shared library and call the function declaratively through that.

### Place Code Outside of the Pipeline Block

Terrible option.

### The script Statement

The ``script`` DSL statement is a special statement intended just for use in Declarative Pipelines; it allows you to define a block/closure that can house any nondeclarative code.

Turning back to our example assignment statements, wrapping them in a script statement would look like this:
```
stage('stage1') {
    <declarative code>
    script {
        def server = Artifactory.server 'my-server-id'
        def rtGradle = Artifactory.newGradleBuild()
        rtGradle.tool = 'gradle tool name'
    }
    <declarative code>
```

## Using parallel in a Stage

With regard to using ``parallel`` in Declarative Pipelines, **you can use it in a stage if it’s the only step in that stage.**

```
stage ('Unit Test') {
    steps {
        parallel(
            set1 : {
                ...

stage('Unit Test') {
    parallel{
        stage ('set1') {
            agent { label 'worker_node2' }
            steps {
```

## Script Checking and Error Reporting

As mentioned at the beginning of the chapter, one of the other nice features of Declarative Pipelines is that the formal structure allows for better script checking and more precise error reporting.


# Chapter 8. Understanding Project Types

# Chapter 9. The Blue Ocean Interface

# Chapter 10. Conversions

# Chapter 11. Integration with the OS (Shells, Workspaces, Environments, and Files)

# Chapter 12. Integrating Analysis Tools

# Chapter 13. Integrating Artifact Management

# Chapter 14. Integrating Containers

# Chapter 15. Other Interfaces

# Chapter 16. Troubleshooting
