# Lab A03 – Git and Infrastructure as Code Notes

## Group 5 Members
- Ilyas Zazai 
- Vijay Walter

## Purpose of this Lab

This lab is about practicing Git, GitHub, branching, collaboration, and Infrastructure as Code using Pulumi.

The main idea is that cloud infrastructure should not only be created manually in the Azure Portal. Instead, infrastructure can be described in code, stored in GitHub, reviewed by team members, and deployed in a repeatable way.

## Repository Structure

The project starts from the professor's GitHub repository. One group member forks the professor's repository to create a group repository.

The workflow is:

```text
Professor starter repository
        ↓ fork
Group GitHub repository
        ↓ clone
Local developer machine
```

The professor's repository is the clean starting point. The group repository is the version that the team works on and submits.

## Branching Workflow

The `main` branch should stay clean and stable. Lab work is completed in a separate branch.

For Part One, the working branch is:

```text
lab-a03
```

The team should not push lab work directly to `main`. Instead, the team works in `lab-a03`, commits changes, pushes the branch to GitHub, and later merges it into `main` after the work is complete and tested.

The expected workflow is:

```text
main
 └── lab-a03
```

After Part One is complete, `lab-a03` can be merged back into `main`.

For Part Two, a new branch should be created from the updated `main` branch:

```text
main
 └── hybrid-h03
```

## What Part One Does

Part One uses Pulumi to deploy the weather app to Azure.

The infrastructure code creates and manages cloud resources such as:

* Azure Resource Group
* Azure Container Registry
* Docker image build and push
* Azure Container Instance
* Public application endpoint
* Environment variables for the container

The deployment flow is:

```text
Application code
   ↓
Docker image
   ↓
Azure Container Registry
   ↓
Azure Container Instance
   ↓
Public weather app URL
```

## Why Pulumi Is Used

Pulumi is an Infrastructure as Code tool. It allows developers and DevOps engineers to define infrastructure using programming languages such as TypeScript.

Instead of creating cloud resources manually, Pulumi reads the infrastructure code and creates the resources automatically.

This makes the infrastructure:

* repeatable
* version-controlled
* easier to review
* easier to update
* easier to destroy and recreate
* better for team collaboration

## Git Collaboration Notes

When multiple team members work on the same branch, each person should pull the latest changes before pushing.

Useful commands:

```bash
git status
git pull --rebase origin lab-a03
git add .
git commit -m "Add Lab A03 Git and IaC notes"
git push origin lab-a03
```

If two people change different files, Git usually combines the changes automatically.

If two people edit the same file and same lines, Git may create a merge conflict. In that case, the team must manually decide which version to keep.

## Important Security Note

In Part One, the weather API key may be placed directly in the container environment variable because the lab intentionally starts with an insecure approach.

In real DevOps work, secrets should not be hardcoded in source code.

In Part Two, the API key should be moved to Pulumi secret configuration so it can be encrypted and managed more safely.

## Summary

This lab teaches a real DevOps workflow:

```text
GitHub repository
   ↓
branch-based collaboration
   ↓
Infrastructure as Code with Pulumi
   ↓
Docker image build
   ↓
Azure deployment
   ↓
tested public application
```

The most important lesson is that infrastructure changes should be written as code, committed to Git, reviewed through branches, and deployed in a controlled way.
