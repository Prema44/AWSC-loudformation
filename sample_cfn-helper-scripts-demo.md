
# AWS cfn helper scripts and its example demo

Helper scripts are updated periodically. To make sure we have updated version of these script, we must install *_aws-cfn-bootstrap package_* in _userdata_

### **Calling/Invoking cfn-init**

_Syntax_: 
```
cfn-init --stack \<stack_name or stack Id\>
    --resource \<logical id of resource\>
    --region \<region_name\>
    --access-key \<aws-access-key\>
    --secret-key \<aws-secret-key\>
    --credential-file 
    --role \<role_name\>
    --configsets \<comma-delimited-list of configsets\>
    --http-proxy \<proxy_url\>
    --https-proxy \<proxy-url\>
    --verbose
```

### **cfn-signal**

Use _cfn-signal_ to signal cloudformation to indicate whether the EC2 instance has been succesfully created or updated. We should signal AWS Cloudformation that the EC2 instance has been created/updated only when all the applications and services are installed and running.

We can use *cfn-signal* in conjunction with *CreationPolicy*

_Syntax_:
```
cfn-signal --success \<true|false\>
    --exit-code \<exit-code\>
    --id \<unique-id\>
    --stack
    --resource
    --region
    --access-key 
    --secret-key
    --credential-file
    --role
    --http-proxy
    --https-proxy
    --url \<AWS cloudformation endpoint\>

```

### **CreationPolicy***

1. Associate the _CreationPolicy_ with the resource to prevent its status from reaching _CREATE_COMPLETE_ until AWS cloudfromation receives a specified _number of success signals_ or the _timeout period is exceeded_
2. To signal a resource, we can use _cfn-signal_ helper script
3. The _CreationPolicy_ is invoked only when the Cloudformation is creating the associated resource
4. The only resources supporting _CreationPolicy_ are:
    - AWS::Autoscaling::AutoscalingGroup
    - AWS::EC2::Instance
    - AWS::Cloudformation::WaitCondition

_Syntax_:
```
CreationPolicy:
    AutoscalingCreationPolicy:
        MinSuccessfulInstancesPercent: Integer
    ResourceSignal:
        Count: Integer
        Timeout: String

```

where 

MinSuccessfulInstancesPercent: 0-100, defines minimum number of instances which should send success signal before setting the status of the resource to *CREATE_COMPLETE*

Count: number of success signals to be received before CF set status to *CREATE_COMPLETE*

Timeout: PT#H#M#S where # is the number for eg PT5M means wait until 5 min to receiving the required number of signals. If required number of success signals aren't received within the timeout period or failure signals are received, CF sets the status of the reousrce to *CREATION_FAILED* and rollsback  the stack 


### **cfn-hup daemon**

*_cfn-hup_* is a daemon that detects the changes in the resource metadata and runs user-specified action when a change is detected

This allows us to make configuration updates on our running EC2 instance thorugh UpdateStack feature

_Syntax_:

```
cfn-hup --config --no-daemon --verbose

```
where
--config specifices the config directory which cfn-hup daemon should look for its _cfn-hup.conf_ file and _hooks.d_ directories


*_cfn-hup.conf_*

This file stores the name of the stack and the credentials the cfn-hup daemon targets
Format:
```
[main]
stack=<stack_name or stack id>
region=
credential-file=
role= <This parameter supersedes the credential-file if both are specified>
interval=<in minutes to check for changes to resource metadata>
verbose=<true|false  to enable verbose logging by daemon>
```

*_hooks.conf_*

The user actions that the _cfn-hup_ daemon calls periodically  are defined in the _hooks.conf_ file
Format:
```
[hookname]
triggers=
path=Resources.<logicalResourceId>(.Metadata or .PhysicalResourceId)(.<optionalMetadataPath>)
action=<any shell command>
runas=<run as user>
```

where 
hookname=A unique name for the hook
triggers= A comma-delimited list of conditions to detect. \<post.add|post.update|post.remove\>
path=
    - Resources.LogicalResourceId : monitors the last updated time of the resource, triggering on any change of the resource
    - Resource.LogicalResourceId.PhysicalResourceId: monitoring the physicalId of the resource, triggering only when the associated resource identity changes (eg new ec2 instance)
    - Resource.LogicalResourceId.Metadata.<optionalMetadataPath>: monitor the change in metadata, triggering the action

*_Points to remember_*
1. An environment variable *CFN_OLD_METADATA* is set to previous value of path and *CFN_NEW_METADATA* is set to new value of path
2. Hooks are loaded at daemon startup only, hence if any new hooks is added, daemon restart will be required.
3. A cache of previous metadata is stored at /var/lib/cfn-hup/data/metadata_db

### Example template

Resources:
    MyVMInstance:
        Type: AWS::EC2::Instance
        CreationPolicy:
            ResourceSignal:
                Count: 1 \<Default is 1\>
                Timeout: PT5M
        Metadata:
            AWS::Cloudformation::Init:
                config:
                    packages:
                        yum: 
                            java-1.8.0-openjdk.X86_64: []
                            tomcat8: []
                    groups:
                        groupOne: {}
                        groupTwo:
                            gid: "501"
                    users:
                        user1:
                            groups:
                                - groupOne
                                - groupTwp
                            uid: "501"
                            homeDire: "/tmp" 
                    sources:
                        /tmp: "https://s3.us-east-1.amazonaws.com/cfn-init/cfn/demo1.zip"
                    files:
                        /etc/cfn/cfn-hup.conf:
                            content: 
                                Fn::Sub: |
                                    [main]
                                    stack=${AWS::StackId}
                                    region=${AWS::Region}
                                    interval=7
                            mode: "000400"
                            group: "root"
                            owner: "root"
                        /etc/cfn/hooks.d/cfn-auto-reloader.conf:
                            content:
                                Fn::Sub: |
                                    [cfn-auto-reloader-hook]
                                    triggers=post.update
                                    path=Resources.MyVMInstance.Metadata.AWS::Cloudformation::Init
                                    action=/opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource MyVMInstance --region ${AWS::Region}
                            mode: "000400"
                            group: "root"
                            owner: "root"
                    commands:
                        test1:
                            command: "chmod 755 demo.war"
                            cwd: "/tmp"
                        test2:
                            command: "sudo yum erase java-1.7.0-openjdk.X86_64"
                            cwd: "~"
                        test3:
                            commands: "rm -rf demo*"
                            cwd: "/var/lib/tomcat8/webapps"
                        test4:
                            command: "cp demo.war /var/lib/tocmcat8/webapps"
                            cwd: "/tmp"
                    services:
                        sysvinit:
                            tomcat8:
                                enabled: "true"
                                ensureRunning: "true"
        Userdata:
            Fn::Base64: 
                Fn::Sub: |
                    #!/bin/bash -xe
                    ## Get latest cloudformation helper script package
                    yum install -y aws-cfn-bootstrap -y
                    
                    ##Start cfn-init to process the metadata
                    /opt/aws/bin/cfn-init -s ${AWS:StackId} -r MyVMInstance --region ${AWS::Region}

                    ## Signal the status from cfn-init
                    /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource MyVMInstance 
                    --region ${AWS::Region}

                    ##Start cfn-hup daemon so that it will keep listening to any changes in EC2 Metadata section
                    /opt/aws/bin/cfn-hup 
                