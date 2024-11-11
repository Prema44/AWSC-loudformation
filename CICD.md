# Continous Integration and Continous Delivery

### Stages in Release process

1. Source: 
    - Check in source code
    - Peer review of new code

2. Build:
    - Compile Code and build the artifacts (war files)
    - Unit Tests

3. Test:
    - Integration testig with other systems
    - Performance load testing
    - UI tests
    - Security tests

4. Production:
    - Deployment in production
    - monitor the code to quickly detect the errors

Source                  Build               Test                Production

<-- Continous integration -->
<-- Continous Delivery ------------------------\<manual approval\>------->
<-- Continous Deployment ------------------------------------------------>

The only difference between continous delivery and deployment is that we no manual approval is required to deploy the code in environment. Everything is automated

### Continous Intgration

- Automatically kickoff a new release when new code is checked-in
- Build and test code in a consistent and repeatable environment
- Continually have an artifact ready for deployment


### AWS Code Services for CI CD

Source              Build               Test                 Deployment         Monitor

AWSCodeCommit     AWSCodeBuild      AWSCodeBuild +          AWSCodeDeploy       AWS XRay +
                                        3rd party                             Cloudwatch      tools                                            