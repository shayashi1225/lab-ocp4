# OpenShift 4 with AWS Broker Workshop

This workshop is for OpenShift version 4.1 and above. It provides an intriduction to OpenShift and 
developing an application which connects to an instance of MySQL via the Relational Database Service (RDS) of AWS. 

The workshop exercises can be found [here](https://github.com/sjbylo/lab-ocp4/tree/master/workshop/content/exercises).

This workshop uses the [AWS Broker](https://github.com/awslabs/aws-servicebroker/) to provision an RDS instance in AWS.  
The Broker needs to be installed and 
working on the OpenShift cluster for two of the exercises to work.  The instructions to install the
AWS Broker are [here](https://github.com/awslabs/aws-servicebroker/blob/master/docs/getting-started-openshift.md).
Note that, even if the AWS Broker is not available the other exercises still provide considerable learning of OpenShift. 

Please note this issue when installing the AWS Broker on OpenShift 4.1 https://github.com/awslabs/aws-servicebroker/issues/139 

# Homeroom

This workshop uses Homeroom. If you are interested in Homeroom, a good place to start learning about it is by running it's [workshop on how to create content for Homeroom](https://github.com/openshift-homeroom/lab-workshop-content).

The dashboard for the workshop environment should look like:

![](images/screenshot.png)

## How to launch the workshop on OpenShift

To launch the workshop, first clone a copy of this Git repository to your own computer.

```
git clone https://github.com/sjbylo/lab-ocp4.git
```

Then within the Git repository directory, run:

```
git submodule update --init --recursive
```

This will checkout a copy of a Git submodule which contains scripts to help you deploy the workshop.

You now have two choices to deploy the workshop. You can deploy it for just yourself, or you can deploy it for a workshop where you have multiple users.

To deploy it just for yourself, run:

```
.workshop/scripts/deploy-personal.sh
```

To deploy it for a workshop with multiple users, run:

```
.workshop/scripts/deploy-spawner.sh
```

Note that you will need to be a cluster admin in the OpenShift cluster to deploy a workshop for multiple users.

Once the deployment has completed, because this workshop is not currently setup to be automatically built into an image and hosted on an image registry, you will need to manually build the workshop image.

To build the workshop image run:

```
.workshop/scripts/build-workshop.sh
```

If you make changes to the workshop content, you can run this build command again. In the case of deploying it just for yourself, the workshop deployment will be automatically re-deployed. If you have deployed it for multiple users, any users currently using the workshop environment, will need to select "Restart Workshop" from the menu of the workshop dashboard.

Once deployed, visit the URL route for the workshop. You will be prompted to login to the OpenShift cluster. Use your own username and password. In the case of deploying the workshop for yourself, you will need to have been a project admin for the project it is deployed in.

You can learn more about the scripts used to perform the deployment by looking at the README and files contained in the `.workshop/scripts` directory.

If you need to ever update the deployment scripts to the latest version run:

```
git submodule update --recursive --remote
```

This will update the commit reference for the Git submodule, so if you want to keep it for the future, you will need to commit the change back to the repository.

To delete a personal workshop instance when done, run:

```
.workshop/scripts/delete-personal.sh
```

To delete a multi user workshop instance when done, run:

```
.workshop/scripts/delete-spawner.sh
```

Also delete the build configuration for the workshop image by running:

```
.workshop/scripts/delete-workshop.sh
```
