# Nodecellar Auto-scale Auto-heal Blueprint

This blueprint deploys a demo wine store application that is based on nodejs and mongodb using Cloudify.

## prerequisites

You will need a *Cloudify Manager* running in either AWS, Azure, or Openstack.

If you have not already, set up the [example Cloudify environment](https://github.com/cloudify-examples/cloudify-environment-setup). Installing that blueprint and following all of the configuration instructions will ensure you have all of the prerequisites, including keys, plugins, and secrets.


### Step 1: Install the demo application

In this step, you will run a *Cloudify CLI* command, which uploads the demo application blueprint to the manager, creates a deployment, and starts an install workflow.

When it is finished, you will be able to play with the wine store application.


#### For AWS run:

```shell
$ cfy install \
    https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/archive/4.0.1.zip \
    -b demo \
    -n aws-blueprint.yaml
```


#### For Azure run:

```shell
$ cfy install \
    https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/archive/4.0.1.zip \
    -b demo \
    -n azure-blueprint.yaml
```


#### For Openstack run:

```shell
$ cfy install \
    https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/archive/4.0.1.zip \
    -b demo \
    -n openstack-blueprint.yaml
```


You should see something like this when you execute the command:

```shell
$ cfy install \
>     https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/archive/4.0.1.zip \
>     -b demo \
>     -n aws-blueprint.yaml
Downloading https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/archive/4.0.1.zip to ...
Uploading blueprint /.../nodecellar-auto-scale-auto-heal-blueprint-4.0.1/aws-blueprint.yaml...
 aws-blueprint.yaml |##################################################| 100.0%
Blueprint uploaded. The blueprint's id is demo
Creating new deployment from blueprint demo...
Deployment created. The deployment's id is demo
Executing workflow install on deployment demo [timeout=900 seconds]
Deployment environment creation is in progress...
2017-05-01 00:00:00.000  CFY <demo> Starting 'install' workflow execution
...
...
...
2017-05-01 00:05:00.000  CFY <demo> 'install' workflow execution succeeded
```


### Step 2: Verify the demo installed and started.

Once the workflow execution is complete, we can view the application endpoint by running: <br>

```shell
$ cfy deployments outputs demo
```

You should see an output like this:

```shell
Retrieving outputs for deployment demo...
 - "endpoint":
     Description: Web application endpoint
     Value: http://10.239.0.18:8080/
```

Use the URL from the endpoint output and visit that URL in a browser. Play with the wine store application.


### Step 3: Simulate auto-scaling

Execute a benchmarking command line application to simulate an auto-scaling scenario.

```shell
$ ab -n 1000000 -c 200 http://10.239.0.18:8080/ # insert your URL instead of this one.
```

This will increase the number of requests to the application. As a result the CPU used by the node process on the nodejs_host VMs will spike above the scale up threshold. This metric is monitored by the Diamond plugin, and the Riemann auto-scale policy calls the scale workflow trigger.

Killing this command should cause the CPU to drop below the scale down threshold, and the application will scale down.

_Note: this assumes that the VM is an appropriate flavor for the benchmarking tool to sufficiently challenge the VM. This particular configuration is with a t2.micro AWS instance._


### Step 3: Simulate auto-healing

You can simulate a failed host by stopping or suspending a running nodejs_host VM. The Riemann failed host policy will recognize the lack of system cpu metrics reporting on the host and will trigger the heal workflow.


### Step 4: Unnstall the demo application

Now lets run the `uninstall` workflow. This will uninstall the application,
as well as delete all related resources. <br>

```shell
$ cfy uninstall --allow-custom-parameters -p ignore_failure=true demo
```
