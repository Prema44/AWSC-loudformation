# EC2 Userdata

Use *Fn::Base64* intrinsic function with userdata. This returns the base64 encoded representation of input string.

Yaml Pipe symbol (|): Any indented text that follows the pipe symbol is treated as multi-line scaler string value i.e. it preservers the new lines

Eg:
```
Userdata:
    Fn::Base64: |
    #!/bin/bash
    yum upgrade -y
    yum install httpd
    service httpd restart
```

### Cons

***Userdata scripts*** and the ***cloud-init directives*** run only on the boot-cycle during first-launch of ec2 instance

Note: We can still make some configurations to run userdata and cloud-init directives on every boot.

### Points to remember

1. Userdata doesnt provides any mechanism whether the userdata execution was successful. Cloudformation only cares about creation of resource and execution of userdata and its status.

2. We can use cfn-helper scripts and the CreationPolicy/DeletionPolicy/UpdatePolicy  together to make sure that the cloudformation shows the resource created only when the execution of scripts is successful