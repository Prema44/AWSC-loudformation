# Cloudformation Helper Scripts

Following are the cloudformation python helper scripts that are provided in EC2 for us to install and configure services and applications that we create as part of stack

- cfn-init
- cfn-hup
- cfn-signal
- cfn-get-metadata

These cfn-helper scripts are invoked or called from userdata section of EC2.

## Metadata - AWS::Cloudformation::Init

**AWS::Cloudformation::Init** key contains the metadata for the EC2 resource. 
*_cfn-init_* helper script looks for the metadata in this AWS::Cldouformation::Init key

1. The metadata is organized into config keys
2. config keys can be grouped into configsets
3. You can specify a configset while invoking cfn-init
4. If no configset is provided to cfn-init, it looks for config key named *_config_*

cfn-init helper script processes the configuration sections in the below order:
1. packages
2. groups
3. users
4. sources
5. files
6. commands
7. services

If you want a different order, separate out these sections into different config keys and use these config keys to create a configset in the order you want to call these config keys

### Syntax

```
Resources:
    MyEC2Instance:
        Type: "AWS::EC2::Instance"
        Metadata:
            AWS::Cloudformation::Init:
                config:
                    packages:
                    groups:
                    users:
                    sources:
                    files:
                    commands:
                    services:
```

**Invoking Single Configset**

The below section calls two configsets which call two config keys in different order

```
AWS::Cloudformation::Init:
    configsets:
        ascending:
            - config1
            - config2
        descending:
            - config2
            - config1
    config1:
        commands:
            test:
                command: "echo $CFNTEST > test.txt"
                env:
                    CFNTEST: "I am from config1"
                cwd: "~"
    config2:
        commands:
            test:
                command: "echo $CFNTEST > test.txt"
                env:
                    CFNTEST: "I am from config2"
                cwd: "~"
```

Calling/Invoking the configset using cfn-init in the userdata section

-c argument takes a comma-separated list of configsets to call in order. For single configset we provides only one configset to this argument

1. Ascending order
```
Userdata:
    Fn::Base64: |
        cfn-init -c ascending
```
***Output***: test.txt contains "I am from config2"

2. Descending order
```
Userdata:
    Fn::Base64: |
        cfn-init -c descending
```
***Output***: test.txt contains "I am from config1"

**Multiple configsets**

You can call mulitple configsets in order where each configset can contain configkeys as well as reference to another configsets.

The below example shows 3 configsets.
- configset _test1_ contains config key _1_
- configset _test2_ contains config key _2_ and reference to configset _test1_
- configset _default_ contains reference to configset _test2_

Eg:
```
AWS::Cloudformation::Init:
    configsets:
        test1:
            - "1"
        test2:
            - ConfigSet: "test1"
            - "2"
        default:
            - ConfigSet: "test2"
    1:
        commands:
            test:
                command: "echo $MAGIC > test.txt"
                env:
                    MAGIC: "I am from the env"
                cwd: "~"
    2:
        commands:
            test:
                command: "echo $MAGIC" > test.txt
                env:
                    MAGIC: "I am from test2" 
                cwd: "~"
```

If you specify configset _test1_
cfn-init -c test1: only config key _1_ is ran

If you specify configset _test2_
cfn-init -c test2: config key _1_ is executed and then config key _2_

I fyou specify configset _default_ or no configsets at all,
cfn-init -c default: behaviour is same as configset _test2_ 

### config keys

1. **packages**
To download and install pre-packaged applications 

_Syntax_:

```
packages:
    rpm:
        epel: "http://download.fedora.com/pub/epel/5/i386/epel-release"
    yum:
        httpd: []
        java: "1.8.0"
        openssh: [0.1.12]

```

1. You specify the supported package manager
2. within that you sepcify the name of the package and the version of the package
Version of the package can be on string type, list type or and empty string/empty list.
Empty string / empty list means u want to install the latest version of that package.

   For rpm: version of the package is the url of the file on local or on the internet and is of string type
   For yum: version can be empty list/string for latest version or a string/list. Reason for list of string is because some package managers like yum allows you to install multiple versions of a single package to install. Not all package manager may support this.

2. **groups**
To create a group on linux

_Syntax_:
```
groups:
    docker: {}
    pingdirectory: 
        gid: 45
```

3. **users**
Users to create on linux machine

_Syntax_

```
users:
    jenkins:
        uid: 1004
        groups: 
            - "jenkins"
            - "wheel"
        homeDire: /var/lib/jenkins
```

4. **sources**

You can use this to download an archive file and extract it to a directory on the instance.
Supported formats:
- tar (.tar)
- tar+gzip (.tar.gz)
- zip

Source can be 

- GitHub
Eg:
Github allows you to download a specific version(branch) of your repo directory in either tarball/zipball format
https://github.com/\<your_directory\>/(\<tarball/zipball\>)/\<version\>
sources:
    /etc/my-repo: "https://github.com/my-user/my-repo/tarball/main"

- S3Bucket

Eg: 
sources:
    /tmp/ssam: "https://s3.amazonaws.com/psd2-encrypted=bucket/ssam.zip"

*_Note_*:
Use AWS::Cloudformation::Authentication resource to specify credentials.
[Refer this section](#awscloudformationauthentication)

5. **files**

Use this key to create files on ec2 instance. The content of this file can be mentioned inline of the template or can be pulled from URL

_Syntax_:
```
files:
    content:
    source:
    encoding:
    group:
    owner:
    mode:
    authentication:
```

where:
- content: is a string or a JSON string. You can use Fn::GetAtt, Ref functions, etc and these.
    If you create a symlink, you can specify the target file as the content
- source: a URL to download content from
- encoding: plain|base64
- group: group owernship of file
- owner: author or owner of the file
- mode: sx digit to specify ownership or symlink i.e xxxxxx
    - for _ownership_, use _000644_
    - for symlink, use _120xxx_ where xxx is the permission on the target file and the symlink
- authenticaion: The name of the authentication to use which overrides any default authentication.
    You can use this to specify the method to use mentioned/defined in the _AWS::Cloudformation::Authentication_

*_Exmaples_*

files:
    /tmp/setup.sh:
        content:
            Fn::Sub: |
                echo "Hello from script file"
        owner: pingdirectory
        group: pingdirectory
        mode: 000755

Symlink eg:

files:
    /tmp/file2.txt:
        content: /tmp/file1.txt
        mode: 120644

6. **commands**
commands are executed in alphabetical order

_Syntax_:

```
commands:
    mycmd1:
        command: "java -jar myapp.jar"
        env:
            JAVA_HOME: "/usr/lib/jvm/amazon-corretto-8"
        cwd: "~"
        test: "test ! -e ~/test.txt"
        IgnoreErrors: "false"

    mycmd2:

```
where 
    command: is the command to be run
    env: dictionary of environment variables
    cwd: current working directory for running a command
    test: the command in _command_ key is ran only if the command in _test_ key is ran succcesfully
    IgnoreErrors: do you want cfn-init to continue if _command_ throws and error

7. **services**

You can use this key for enabling or disabling the systemd/sysVinit services when instance is launched
Also for eg, if a httpd file content is updated in metadata in _files_ key of _AWS::Cloudformation::Init_ section, this updated file needs httpd service to be restarted. 

_Syntax_

```
services:
    systemd:
        _<service_name>_:
            ensureRunning:
            enabled:
            files:
            sources:
            packages:
            commands:
```

where
    ensureRunning: allows to make sure if service should be running after cfn-init finishes or not
    enabled: enabled service on boot or not
    files: . A list of files. If cfn-init installs or updates the files in metadata key, svc will be restarted
    sources: A list of directories. If cfn-init expands the archive in this directory, svc will be restarted.
    packages: A map of package manager and a list of pacakges. Ff cfn-init installs or updates the packages, svc will be restarted
    commands: A list of command names. If cfn-init runs the commands, svc will be restarted


## **AWS::Cloudformation::Authentication**

1. Use this resource to specify authentication credentials for files or sources thats u specify in AWS::Cloudformation::Init.

2. To include this credentials in files/sources, use 
    - _uris_ as the property is the source is a URI
    - _buckets_ as the property if the source is S3 bucket

3. For files, CF looks in following order for authn cred
    1. For _authentication_ property inside the _AWS::Cloudformation::Init_ key specified for a file directly
    2. For _uris_ or _bucket_ property specified in _AWS::Cloudformation::Authentication_ resource

4. For sources, CF looks for _uris_ or _bucket_ property inside the _AWS::Cloudformation::Authentication_ resource


_Syntax_:

```
Metadata:
    AWS::Cloudformation::Authentication:
        Name_For_authentication:
            type:
            accessKeyId: 
            secretKey:
            username:
            password:
            roleName:
            buckets:
                - 
            uris:
                -
```
where
_type_: *_basic_* (username and password) | *_S3_*
    If you specify _basic_, use _username_ , _password_, and _uris_
    If you specify _S3_, specify _accessKeyId_, _secretKey_ and _buckets_(optional)
_accessKeyId_: for s3 authentication only i.e _type_ is _S3_
_secretKey_: For s3 authentication only
_username_: for basic authentication only
_password_: for basic authentication only

_buckets_: a comma demilited list of s3 buckets to be associated with S3 credentials. only 
    when  _type_ is _S3_

_uris_: a comma delimiited list of uris to associate the basic authn creds with

*_Example_*


AWS::Cloudformation::Authentication:
    rolebased:
        type: S3
        roleName:
            Ref: MyRole
        buckets:
            - psd2-config-bucket
AWS::Cloudformation::Init:
    config:
        files:
            /tmp/setup.sh: 
                source: https://s3.amazonaws.com/mybucket/setup.sh
                authentication: rolebased


**NOTE**
Please refer this section for more detailed example and continuation of cfn-helper scripts. [Click Here](./sample_cfn-helper-scripts-demo.md)
