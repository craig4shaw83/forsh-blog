---
title: "Securing Infrastructure as Code (IaC) with the Microsoft Technology Stack"
date: 2023-09-25
categories:
  - Security
  - DevOps
  - Infrastructure
tags:
  - IaC
  - Bicep
  - Azure
  - GitHub-Advanced-Security
  - Defender-for-DevOps
excerpt: "Explore how to implement secure Infrastructure as Code deployments using only Microsoft products and tools"
---

# Securing infrastructure as code (IaC) with the Microsoft technology stack

## I thought it would be interesting to see how a secure IaC deployment would work using only Microsoft products. As it turns out... pretty secure!

*Author: Craig Forshaw*

---

## The Challenge

Ok so the challenge is end-to-end infrastructure security with IaC using only the Microsoft technology stack. Do I really need any third party tooling or does Microsoft have the products to support securing the entire DevSecOps process?

---

## Bicep

Microsoft launched its own IaC declarative languages tool called [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview?tabs=bicep) on August 31, 2020. It is a domain specific language (DSL) for infrastructure deployments in Azure.

### State

Bicep is stateless in nature. The state is what is actually deployed and Bicep runs a differential comparison of the actual configuration compared to the configuration files being executed. This is an advantage from a security perspective because stateful tools like Terraform require a separate [state file](https://developer.hashicorp.com/terraform/language/state) which needs careful planning when it comes to security. Bicep reads directly from resource manager and converts its templates automatically into JSON format before deploying resources.

### Parameters

Marking string or object parameters as secure ensures these values are not saved to the deployment history and aren’t logged.

```bicep
@secure()
param demoPassword string

@secure()
param demoSecretObject object
```

### Secrets

The most important security coding practice is placing secrets in a password vault. Microsoft has Azure KeyVault, and Bicep uses the [getSecret](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions-resource#getsecret) function to return a secret from KeyVault once the pre-requisites are in place ([Key Vault secret with Bicep — Azure Resource Manager | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/key-vault-parameter?tabs=azure-cli)).

NB: Github and Azure DevOps also support secret storage for actions/pipelines.

---

## Github Copilot & Chat (Beta)

Github Copilot is an AI pair programmer that offers autocomplete-style suggestions as you code, so this tool is of huge benefit to have from a security perspective. Inline suggestions will not only fix code formatting but also make suggestions around secure code.

The recently released [Github Copilot chat beta](https://docs.github.com/en/copilot/getting-started-with-github-copilot) now takes it to the next level so that you can ask copilot to suggest recommendations for securing code vulnerabilities. Infrastructure as code vulnerabilities can be analysed within chat and recommendations given to ensure a developer can remediate issues before it goes to pull request.

![Copilot chat example](/assets/iac-security-1.png)

---

## Version Control System

Placing the config files in a VCS is a given. Luckily Microsoft has two versions to choose from, GitHub and Azure DevOps.

### GitHub Advanced Security

GitHub has a feature called advanced security which is available for Enterprise accounts as well as some features available to public repositories ([GitHub plans](https://docs.github.com/en/get-started/learning-about-github/githubs-plans)).

Security policies are available to allow for reporting of security vulnerabilities in code by adding a SECURITY.md file to the root of your repo. GitHub also has a security advisory database with known vulnerabilities and malware, and supports Entra ID SSO integration for user and role management. There are also [security hardening features](https://docs.github.com/en/organizations/keeping-your-organization-secure) available to organisations on GitHub.

### Azure DevOps

Out of the box Azure DevOps security features are restricted to permissions, access and security grouping with some pipeline secret storage available. For features such as code scanning then you would need to use the [GitHub advanced security integration feature](https://learn.microsoft.com/en-us/azure/devops/repos/security/configure-github-advanced-security-features?view=azure-devops&tabs=yaml) which has recently just become [generally available](https://devblogs.microsoft.com/devops/now-generally-available-github-advanced-security-for-azure-devops-is-ready-for-you-to-use/). This extends the GitHub advanced security features to Azure repos.

![Azure DevOps with Github Advanced Security](/assets/iac-security-2.png)

Both these version control systems ship with the standard branching pull request, peer review processes that you find in most VCS systems and are fundamental to good DevSecOps practices.

A GitHub advanced security licence includes [code scanning](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning), [secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning) and [dependency reviews](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review) that give a next level of security features that are critical in today's security landscape.

---

## Pipelines/Actions

You are now ready to deploy code using a pipeline or as GitHub calls them a GitHub action. There are a host of [security hardening for Github actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions) considerations that GitHub has published guidance for. Secrets management is the main consideration but also a lot of other important considerations need to be taken such as risk assessment and using Open ID connect when authenticating to Azure ([Configuring OpenID Connect in Azure — GitHub Docs](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)).

Before you run your pipeline or action you should run some Bicep validation checks:

- **Linting**: Linting validation will check for things like unused parameters, unused variables, interpolation, secure parameters and more. You can tell Bicep to verify your file by manually building the Bicep file through the Bicep CLI: `bicep build main.bicep`
- **Validate**: You can use the `AzureResourceManagerTemplateDeployment` task to submit a Bicep file for preflight validation.
- **What-if**: Previews the changes that will happen. The what-if operation doesn’t make any changes to existing resources. Instead, it predicts the changes if the specified Bicep file is deployed.

---

## Defender for DevOps

This action incorporates static analysis tools into a github action to be run on your IaC code repository. It has a host of open-source tools for analysis but in our case its [template analyser](https://github.com/Azure/template-analyzer) that covers analysis of Bicep configuration files.

The steps involve simply incorporating this [sample action workflow](https://github.com/microsoft/security-devops-action/blob/main/.github/workflows/sample-workflow.yml) into your actions as a pre-requisite for code deployment. With all the GitHub advanced security features enabled this can report security vulnerabilities to github under the security tab and auto-create pull requests for remediation. Additionally, this data is reported back to Defender for DevOps for visibility to the security teams for remediation and reporting.

There is an equivalent for Azure DevOps pipelines too.

![Defender for devops console](/assets/iac-security-3.png)

---

## Summary

In summary Microsoft has matured its offerings around IaC and DevSecOps with its tooling over the past few years and most recently the offerings around GitHub advanced security, GitHub Copilot and Defender for DevOps are now introducing even more advanced capabilities in this space to compete with Hashicorp, Sonar and CyberArk among others.

As more sophisticated supply chain threats emerge its important to have full end-to-end security coverage that continuously evolves over time. Microsoft is building its capability in this area which is exciting to see and will benefit its customers who need to implement DevSecOps with the latest emerging security tools to protect their environments.

---
