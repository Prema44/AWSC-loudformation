# Condition

Conditions section contains statements that define circumstances under which entities (resources,outputs) can be created or configured 

We can create a condition and thne associate it with a output or resource so that Cloudformation
only creates the resource or output if the condition is true

We can associate the condition with a property so that cloudformation set the property to a specific value if the condition is true, else to a different value that we specify

## Syntax

```
Conditions:
    Logical-ID:
        Intrinsic Function
```

Eg:
```
Conditions:
    CreateEIPForProd:
        Fn::Equals:
        - Ref: EnvironmentName
        - prod
```

### Points to Remember

1. Within each condition we can reference other condition
2. We can associate condition at three places:
    - Resources
    - Resource Properties
    - Outputs

### Intrinsic Functions for Conditions

- Fn::And: [condition1,condition2,...] min 2 condition and max 10 
- Fn::Or: [condition1, condition2,...] min2 condition and max 10

    Eg: 
    ```
    Fn::Or:
    - Fn::Equals
        - Ref: Environment
        - prod
    - Condition: SomeOtherCondition
    ```
- Fn::Not: [condition]
- Fn::Equals: [first value, second value]
- Fn::If: [condition, value_if_true, value_if_false]




