## Parameters

## Syntax

```
Parameters:
    Logical ID:
        Type: DataType
        ParameterProperty: value
```

### Parameter Type

- String
- Number

    Although while taking input through console its treated as number, but while referencing in template , its interpreted as string. Eg 88, is "88" while referring
- List\<Number\>

    Similar to Number, 
    Eg: 88,8080. While refering in template, it becomes ["88","8080"]
- CommaDelimitedList

    Eg: test,dev,prod will becomes ["test", "dev", "prod"] while referring in template
- AWSSpecific
    Except AWS:EC2::Image:Id, rest most of the types while give a dropdown of exisiting resources of that type.
    - AWS::EC2::Instance::Id
    - AWS::EC2::VPC::Id
    - List\<AWS::EC2::Subnet::Id\>
- SSMParameterType
    - AWS::SSM::Parameter::Name
        
        Only checks if the parameter is present or not. No value is retrieved
    - AWS::SSM::Parameter::Value\<String\>

        Gets the value of the parameter name your provide in the console if present and the value should also satisfy the constraint of being a string i.e the parameter type must be string
    - AWS::SSM::Parameter::Value\<List\<String\>\> OR       AWS::SSM::Parameter::Value\<CommeDelimitedList\>

        Gets the value of the parameter name your provide in the console if present and the value should also satisfy the constraint of being a list of string i.e the parameter type must be StringList
    - AWS::SSM::Parameter::Value\<AWS::EC2::Image::Id\>

        Similar to above type but now the value must satisfy constraint of being an EC2 AMI Id

Eg:
```
Parameters:
    MyInstanceType:
        Type: String
        AllowedValues: 
        - t2.micro
        - t2.medium
        - t2.nano
        Default: t2.micro
        Description: Instance Type for Dev instance
```

### Parameter Properties

- AllowedPattern

    Regex constraint of the parameter values
- AllowedValues

    List of allowed values for the parameter
- ConstraintDescription

    without constraint description, Incase the template user doesnt follows and provides invalid parameter value, the cloudformation shows message Invalid input provided, value must satisfy constraint [A-Za-z0-9+]. this message is not user friendly rather we would like to provide a message such as "Values must have uppercase,lowercase and a digit". For this we use ConstraintDescription
- Default

    Default value to be used incase nothing provided by the template user
- Description

    Description for parameter
- MaxLength

    Max length of character for string literal
- MaxValue

    Max Value for integer/float
- MinLength

    Min length of characters for string literal
- MinValue

    Min value for integer/float
- NoEcho
    
    Masks the value in console. Dont use this parameter value in outputs, metadata section of template or metadata attribute section of resources. Masking doesnt work in these sections

## Resolve the SecureString SSM Parameter or Secrets Manager secret in template

Some of the resources attributes support resolution of secure-string ssm parameter store or secrets manager secret. Not all resources support this resolution

- Secure String Parameter resolution

    {{resolve:ssm-secure:parameter-name:version}}

- Secrets Manager secret

    {{resolve::secretsmanager:secret-id:SecretString:json-key:version-stage:version-id}}

    If version-id and version-stage both are not provided, then AWSCURRENT version stage label is used for secret.
    If json-key is not provided, whole secret value is retrieved
    For secrets in another account, specify arn for secret-id