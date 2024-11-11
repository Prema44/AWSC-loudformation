# Nested stacks

1. The _AWS::Cloudformation::Stack_ resource types nests a stack as a resource in top-level template

2. We can add the output value of a nested stack within the root stack. We use the _Fn::GetAtt_ function with nested stacks logical name and the name of the output value in the nested stack.

_Syntax_
```
VpcId: 
    Fn::GetAtt: NestedStackLogicalName.Outputs.NestedStackOutputName
```


