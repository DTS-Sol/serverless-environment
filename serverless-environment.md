# Serverless Environment with the Serverless Framework

Serverless Computing is a suited solution for those big applications that need to hanlde big amounts of users and/or requests every time. It allows to decouple our application in small modules (aka _microservices_), in which every one of them are in charge of a single task, thus reducing the complexity of mantaining one big app to mantain small services.

A core benefit of doing this is that in this way we can duplicate the services that are being requested a lot by users and then balance those request between the replicas of that service. What we gain with this:

  * Duplicate only the service in use and not the whole application.
  * Reduce costs by paying only for the duplicated services that are most requested.
  * Deploying time is reduced in every update due to re-deploying only the service we need to update instead of the entire app.

It may be with serverless containers (AWS Fargate) or serverless functions (AWS Lambda), whatever suits the best for you and depending on your needs, but a migration to a serverless architecture is a good invest to consider to in the long term.

---

In a previous chapter we talked about building serverless applications with the AWS Cloud, which is the cloud of our choice here in DTS Solutions. We focused in the service called __Lambda__ and you could see all the components involved in lifting up an environment to get running a serverless application.

It could be an overhead to manually set up every piece of your app, including IAM policies, API Gateway methods and resources, Lambda functions, uploading your code, etc. And while you could use the AWS CLI for set up your environment, you would still need to explicitely create every resource, not forgetting it is too verbose.

## Introducing the Serverless Framework

The Serverless Framework is a technology that allows us to easily build, deploy and scale serverless applications to different cloud vendors. It is an open source project written in NodeJS.

In this blog we will be focusing on installing it on our machine and deploying our first serverless application to our AWS account.

__Prerequisites__

For this tutorial we need:

  * NodeJS installed (LTS version recommended)
  * An AWS account
  * Have set up the AWS CLI
  * Set up our AWS Access Key and Secret Access Key on our computer

We learnt to set up our AWS credentials and the AWS CLI in the previous chapters to interact with AWS resources, so you will be probably more than ready with this. Though we are not going to use the AWS CLI for creating any resource, it is still necessary to let the serverless framework create them for us.

### Installation

The installation part is very easy, you just need to enter one single command:

```bash
$ npm install -g serverless
```

Once the installation finishes, we can then verify it by typing the next command:

```bash
$ serverless --version
```

Or just:

```bash
$ sls --version
```

For the short version. If all went right, We might expect an output like this:

![sls version](https://i.imgur.com/5cjqUPk.png)

Now we are ready to create our serverless application!

### Creating a Serverless Framework Project

To start building our app, we first need to initialize a Serverless Framework Project, for this we need to specify the name of our project and the name of the template containing the SDK we will be using within the app.

But before doing this, lets create the directory that will contain our project:

```bash
$ mkdir serverless-app
$ cd serverless-app/
```

Now, for this tutorial we are going to use the NodeJS template and the name is going to be _serverless-app_:

```bash
$ sls create --name serverless-app --template aws-nodejs
```

We will have an output like this:

![Serverless project created](https://i.imgur.com/ABZLaUS.png)

Now if we check the directory, we will notice that two files were created, a _handler.js_ and a _serverless.yml_ file:

  * The _serverless.yml_ file contains all the information in concerning to our serverless project, it also contains all the needed resources for our app to work, same ones that it will be deploying to our AWS account.
  * The _handle.js_ file is just a javascript file which has the code that we will be deploying. It could be a more complex file structure but for terms of simplicity we are going to use this file.

### Building the serverless application

Now we have set up our components, lets modify the _serverlss.yml_ and _handle.js_ files to match our desired app.

The _serverless.yml_ file will look like this:

```yaml
service: serverless-app

frameworkVersion: '2'

provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256
  stage: dev

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          method: GET
          path: /hello
```

With this we are saying that we want our serverless app to have an specific configuration for the Lambda functions, and that we have one function called _hello_ which has a _handler.hello_ handler function and it will be called through a GET request on the resource named _hello_.

For our _handler.js_ file we want to keep it simple:

```javascript
module.exports.hello = async event => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Hello World from Serverless Framework!!',
    }),
  };
};
```

### Deploying and testing

We are now ready to deploy our app, this is done by typing this command:

```bash
$ sls deploy -v
```

With the -v flag we ask for a verbose output to see the details.

If we see the output:

![sls deploy](https://i.imgur.com/yEzkx0J.png)

We can appreciate that in fact, the Serverless Framework uses AWS CloudFormation behind the scenes to deploy the resources to our AWS Cloud, thus making the deploy process very easy for us by just defining a yml file containing all of our serverless components without the verbosity of CloudFormation templates.

Now, if the process succeded, we will be prompt with the endpoint for the AWS Lambda function we just deployed:

![Lambda endpoint](https://i.imgur.com/beHnI8t.png)

And now if we make a request to that endpoint through a web browser or a tool like _curl_:

![GET Request](https://i.imgur.com/vlvTSfP.png)

We get our desired response.

And there you have it, we lifted up all the necessary AWS resources by just defining our yml file, without the need of typing by hand every command to launch any resource, just one single command to deploy our entire app.

And if we wanted to add an IAM Policy, we could just add a section to our yml file:

```yaml
service: serverless-app

frameworkVersion: '2'

provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256
  stage: dev
  iamRoleStatements: # This section
    - Effect: Allow
      Action:
        - sts:AssumeRole
      Resource:
        - '*'

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          method: GET
          path: /hello
```
So every resource needed for our serverless application can be specified here, you just need to re-deploy.

And if we wanted to remove the stack launched by the Serverless Framework:

```bash
$ sls remove -v
```

With this all the resources will be cleaned up preventing unexpected billing and it keeps our resources organized and under control

What are your thoughts on the Serverless Framework and on serverless applications in general? Let us know in the comments!
