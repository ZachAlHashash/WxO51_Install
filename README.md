# Watsonx Orchestrate 5.1 - On-Premises Deployment Guide

## Overview
Watsonx Orchestrate is an AI-powered assistant that streamlines repetitive tasks and enhances productivity through generative AI and automation. This guide provides step-by-step instructions to deploy Watsonx Orchestrate 5.1 on an on-premises data platform.

## Deployment Options
Watsonx Orchestrate can be deployed using:
- **Managed SaaS**: IBM Cloud or AWS
- **On-Premises**: IBM Cloud Pak for Data & IBM Software Hub
## Deployment Architecture
![image](https://github.com/user-attachments/assets/485ff9e5-24e4-4163-aa46-aeffa27382bd)

## Installation Steps:
![image](https://github.com/user-attachments/assets/51225ef2-dd56-4f91-9f1d-1b43b6abc681)


Before starting the deployment, ensure the following:
Reserve OpenShift on TechZone for installation training
- **Infrastructure**: OpenShift 4.16 Cluster with 5 worker nodes, Worker node ‘flavor’ (vCPUs x 
Memory): 32x128.300gb
- **Storage**: ODF (local disks) - 2TB
- **Access to IBM TechZone**: [TechZone Reservation](https://techzone.ibm.com/my/reservations/create)

## Prepare bastion workstation
- **use the information in techzone provisioned instance to ssh to bastion workstation**
- after successful login, switch user to become a root.  run: sudo su
- mkdir cpd, cd cpd
- ![image](https://github.com/user-attachments/assets/ae015bea-07b2-4196-a3fe-69ef4ba0c708)
![image](https://github.com/user-attachments/assets/b0a154ea-06d0-4298-8cea-37bf57e2f30c)

## Install Podman
Using the bastion server SSH session as root, install the Podman container runtime engine using the Yum package manager: 
yum install -y podman

![image](https://github.com/user-attachments/assets/9bda9203-26b0-49cb-984e-2b6a276a0e0b)

## Install Software Hub ( CPD CLI)
- as root, run the following command to get the cpd-cli package
wget https://github.com/IBM/cpd-cli/releases/download/v14.1.0/cpd-cli-linux-EE-14.1.0.tgz
- untar the file using the following command
- tar -xvf <"file name>
## Modify the root account's .bashrc file to include the CPD CLI in the path
Using the same bastion server SSH session as root:

- 1- Change directory into the cpd-cli directory starting by typing cd cpd-cli<tab> and tab complete for the full directory name.
- 2- Show the path to the current directory by typing: pwd
- 3- Edit the .bashrc file: vi ~/.bashrc
- 4- Go to the bottom of the file, and add a few empty lines at the end.
- 5- Prepend the cli folder to the path: export PATH=<the  copied>:$PATH
- 6- Exit the vi editor by typing :wq to write the changes and quit
- 7- Back out of the current folder by typing cd ..
- 8- Load the path change for this current bash session by sourcing the .bashrc file: source ~/.bashrc
- 9- Verify that the CPD CLI can be found by typing cpd-cli version  


