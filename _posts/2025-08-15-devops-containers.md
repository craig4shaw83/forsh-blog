---
title: "Creating Self-Hosted Azure DevOps Agents with Azure Container App Jobs and Managed Identity"
date: 2025-08-15
---

Using container app jobs for self-hosted Azure DevOps agents allows for more control over what is running on your DevOps agents. Both VMSS and the newer managed DevOps pools give you the option to run agents on your own virtual network which is excellent for securing network traffic but if you also need to have control what is running on them then configuring the agents with docker in a container app job is a good option. You also have the added security of Defender for containers integration to ensure you can keep your images secure.
