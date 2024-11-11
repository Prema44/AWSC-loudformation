# Outputs

The outputs section declares output values that we can
1. Import into other stacks (to create cross stack reference)
2. When using nested stacks, we can see how outpts of a nested stack can be used in Root stack

Max 60 outputs in cfn template

## Syntax

```
Outputs:
    Logical-ID:
        Description: Information about the value
        Value: Value to return
        Export:
            Name: Name for exporting the value
```

Eg:

Outputs:
    InstanceId:
        Description: Instance Id
        Value: 
            Ref: MyEc2Instance
        Export:
            Name:
                Fn::Sub: "${AWS:StackName}-InstanceId"
    MyAZ:
        Description: AZ of instance


### Fn::Sub

Syntax:

```
Fn::Sub:
- String
- Var1: Val1
  Var2: Val2
```

Eliminate varaible map if only template parameter, resource logicial id or resource attribute is to be used.
For template parameter and resource logical id, Sub works same as Ref 
For resource attribute, Sub works same as Fn::GetAtt. 
Eg: Fn::Sub: "${MyInstance.PublicIP}" == Fn::GetAtt: [ MyInstance, PublicIP]

Eg: Using Variable map
```
WWWBucket:
    Type: AWS::S3::Bucket
    Properties:
        BucketName:
            Fn::Sub:
            - 'www.${Domain}'
            - Domain: 
                Ref: RootDomainName
```

### Fn:GetAtt

Syntax

```
Fn::GetAtt:
- LogicalId of Resource
- Attribute name of resource
```

### Points to Remember

1. Export Name should be unique within the region for AWS Account
   Hence, we use ${AWS::StackName} as that is constrained to be unique 

2. Use *Fn::ImportValue* for importing values from another stack


### Fn::ImportValue

Syntax

```
Fn::ImportValue:
    shared_value_to_import
```

Eg:

```
Fn::ImportValue:
    Fn::Sub: ${NetworkStackName}-SecurityGroupId
```

### Fn::Join

Syntax

```
Fn::Join:
    - delimiter
    - set of values
```

Eg:

```
Fn::Join:
    - "-"
    - - "stackname"
      - Ref: EnvironmentName
```
