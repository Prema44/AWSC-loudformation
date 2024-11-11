# Mappings

Mappings section matches a key to a corresponding set of named values

## Syntax
```
Mappings:
    Mapping01:
        Key01:
            Name: Value01
        Key02:
            Name: Value02
        Key03:
            Name: Value03
```

To fetch values from a map, use an intrinsic function ***Fn::FindInMap***

### Fn::FindInMap

The intrinsic function *Fn::FindInMap* returns the value corresponding to keys in a two-level map that is declared in mappings sections

Parameters of Fn::FindInMap:
- Map Name
- First Level Key
- Second Level Key


