# Create new Jenkins Slave on Openshift

## Install Jenkins

    oc new-project my-jenkins --display-name "Jenkins"
    oc new-app jenkins-persistent --param ENABLE_OAUTH=true --param MEMORY_LIMIT=2Gi --param VOLUME_CAPACITY=4Gi -n my-jenkins

## Create custom slave

    oc new-build  -D $'FROM docker.io/openshift/jenkins-agent-maven-35-centos7:v3.11\n
      USER root\nRUN yum -y install skopeo && yum clean all\n
      USER 1001' --name=jenkins-agent-appdev -n my-jenkins

## Configure Jenkins to use your slave

In Jenkins select `Manage Jenkins`, then click on `Configure System` and finally scroll down to the Cloud section. Click `Add Pod Template` and select `Kubernetes Pod Template` to add another pod template to Jenkins.

Make sure you get the following settings right:

* Labels: This is the name that you use in your pipeline to specify this image. Suggestion: `maven-appdev`.

* Docker-Image: The fully qualified name of your Docker image. Use the OpenShift internal service name (and port).

* Memory limit: Use 2Gi for the container memory limit.

From the Jenkins home screen, select `Manage Jenkins → Configure System`.

Select `Cloud → Kubernetes → Add Pod Template → Kubernetes Pod Template`:

* Name: `maven-appdev`

* Namespace: 

* Labels: `maven-appdev`

* Usage: `Use this node as much as possible`

* The name of the pod template to inherit from: 

* Containers: `Add Container / Container Template`

* Name: `jnlp`

* Docker image: `docker-registry.default.svc:5000/my-jenkins/jenkins-agent-appdev:latest`

* Always pull image: Unchecked

* Working directory: `/tmp`

* Command to run: 

* Arguments to pass to the command: `${computer.jnlpmac} ${computer.name}`

* Allocate pseudo-TTY: Unchecked

Click `Advanced…` to open the advanced container template settings.

* Limit Memory: `2Gi`

* Click `Advanced…` at the very bottom of the pod template definition (just above `Delete Template`).

Click `Save` at the bottom of the screen.

## Create a pipeline to test it

Create a new Jenkins job of type Pipeline and use this test pipeline

    node('maven-appdev') {
        stage('Test skopeo') {
            sh("skopeo --version")
            sh("oc whoami")
        }
    }