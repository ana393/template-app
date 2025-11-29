# template-app

This is an API  reusable  artifact for mule project. It contains project structure standarts and CICD automation to CH2.0

## Getting started
To make it easy for you to get started with a new project, here's a list of recommended next steps.

## CI/CD Immediate Next Steps:
 - Create a project in the parent root (ex. Mulesoft->template-app)
 - Create the branch "development"  and protect it (/template-app/-/settings/repository#branch-rules)
        1. Allowed to merge: Maintainers
        2. Allowed to push and merge: Maintainers
 - Create a feature-v1 branch from development branch
 - Clone the project 
 - Place this template and start working. !Important adjust the project name in POM file
 - Once done with daily changes, push to remote for collaboration. Case done with the implementation PR to develompent branch.
 - Observe the pipilene from the projects itself.

## Description
> Description of the API.  Include all applicable information, including considerations for different regions.

## Flow descriptions
> List all the flow xml files with brief descriptions.

- Healthcheck endpoint is available at the path below.
```/```

------

## Test and Deploy
| Environment   | Status        | 
| :-------------: |:-------------:| 
| Development   |![Build Status](https://gitlab.com/Mulesoft/template-app)| 
| Production   |![Build Status](https://gitlab.com/Mulesoft/template-app)| 
