# Using Ansible with VMware NSX
This repo provides an introductory overview of the [VMware NSX Modules for Ansible] (https://github.com/vmware/nsxansible).

__Please Note:__ This training course is a spare-time project for me, and I could really use your help to find any typos/errors or input on where I could improve or update the materials. If you see anything that needs improvement, please submit an issue, I would be grateful! If you would like to contribute to the project all comers are welcome!

## Lab Guide
- [Course Introduction](https://github.com/afewell/AnsibleNSX101#course-introduction)
  - [Where to start](https://github.com/afewell/AnsibleNSX101#getting-started)
- [Lab 1 - Lab Setup](Lab1-LabPrep/)
- [Lab 2 - NSX Deploy & Register](Lab2-NSXDeploy/)
- [Lab 3 - Control & Data Plane Setup](https://github.com/afewell/AnsibleNSX101/tree/master/Lab3-Control%20and%20Data%20Plane%20Setup)
- [Lab 4 - Virtual Network Configuration](Lab4-Virtual%20Network%20Configuration/)

This training module will focus primarily on hands-on examples and will provide the reader with step-by-step instructions to walk through installation and basic usage the [VMware NSX Modules for Ansible](https://github.com/vmware/nsxansible). Because this is primarily a hands-on course, users should ensure they have access to a lab environment with licensed access to the required software, as detailed in the [Lab Setup](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep). This course does not provide access or licensing for any software.

Please Note: This course is open to community contribution. All material on this site should be considered as the effort of the individual contributor and does not represent any opinion, assurance or warranty from the contributors or their respective employers. The original course material was provided by Arthur Fewell, who is a vmware employee but whose contributions are his own and do not officially represent vmware. This course is produced independently from vmware and does not represent any official position, opinion, assurance or warranty from VMware. 

## Course Introduction
## Prerequisites
1. Before taking this training, you should have a good technical understanding of VMware NSX. The course will cover using Ansible to perform installation, setup and basic operational configuration of NSX and assumes users already understands the concepts behind these operations. 
2. Light to moderate vSphere experience is recommended. While the lab excercises themselves only require very light vCenter configuration, the base topology for the lab environment will need vCenter to be installed from scratch and configured with basic settings. Users building their own labs should be comfortable enough with the software to perform these tasks independently as detailed instructions are not provided for setting up the base lab environment.
3. While you should have basic familiarity with Linux & Bash, advanced Linux Knowledge is not required. Each lab will provide step-by-step instructions for those less comfortable in the bash shell.
4. Basic Familiarity with Ansible is nice-to-have but not required. If you do not have Ansible experience, the step-by-step instructions should enable you to get through the labs, however this is not a course on Ansible and does not provide sufficient instruction for operating Ansible in production environments. You should pursue Ansible-specific training if you plan on using these modules beyond this course.
5. Students should have access to a lab environment capable of supporting the software used in the course, as detailed in the [Lab Setup](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep) section.
6. Students need to provide their own licensed access to any software used in this course. This course provides instruction for both Open Source and Commercial Software and provides no licensing or access to any software.

## Getting Started
- **VMware Employees:** Two vApps have been prepared in Onecloud for this course with the base lab topology pre-built for you
  - If you would like to install and prepare the Ansible control server, [proceed to lab 1d](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep/Lab1d-AnsibleInstall#vmware-employees-using-onecloud).
  - If you would like a pod with Ansible pre-installed so you can go straight to work on the playbooks, [proceed to lab 2](https://github.com/afewell/AnsibleNSX101/tree/master/Lab2-NSXDeploy#prerequisites). 

__All other participants:__ 

To Get Started, go to the [Lab Setup ](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep) section, and then proceed through the labs in order.

Once you complete the lab setup, there are no specific dependencies between labs, so feel free to jump into any specific lab - but beware that some technical explanations may be provided in earlier labs that are not repeated in later labs so in general I recommend that you go through the labs in order.

I hope to continue adding more slides and videos to provide more detailed explanations about the technologies used in the labs, so please check back here in the future for updates.

## [Click here to proceed to Lab 1](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep)