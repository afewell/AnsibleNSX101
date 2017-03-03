# Using Ansible with VMware NSX
This repo provides an introductory overview of the [VMware NSX Modules for Ansible] (https://github.com/vmware/nsxansible).
__Please Note:__ This training course is in the early stages of development so most information is not yet populated. I anticipate the first complete draft of all materials to be complete within a few weeks, so please check back!

## Lab Guide
- [Course Introduction]()
- [Lab 1 - Lab Setup](Lab1-LabPrep/)
- [Lab 2 - NSX Deploy & Register](Lab2-NSXDeploy/)
- [Lab 3 - Control & Data Plane Setup](Lab3-Control%and%Data%Plane%Setup/)
- [Lab 4 - Virtual Network Configuration](Lab4-Virtual%20Network%20Configuration/)

This training module will focus primarily on hands-on examples and will provide the reader with step-by-step instructions to walk through installation and basic usage the [VMware NSX Modules for Ansible](https://github.com/vmware/nsxansible). Because this is primarily a hands-on course, users should ensure they have access to a lab environment with licensed access to the required software, as detailed in the [Lab Setup](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep). This course does not provide access or licensing for any software.

## Course Introduction

## Prerequisites
1. Before taking this training, you should have a good technical understanding of VMware NSX. The course will cover using Ansible to perform installation, setup and basic operational configuration of NSX and assumes users already understands the concepts behind these operations. 
2. Light to moderate vSphere experience is recommended. While the lab excercises themselves only require very light vCenter configuration, the base topology for the lab environment will need vCenter to be installed from scratch and configured with basic settings. Users building their own labs should be comfortable enough with the software to perform these tasks independently as detailed instructions are not provided for setting up the base lab environment.
3. While you should have basic familiarity with Linux & Bash, advanced Linux Knowledge is not required. Each lab will provide step-by-step instructions for those less comfortable in the bash shell.
4. Basic Familiarity with Ansible is nice-to-have but not required. If you do not have Ansible experience, the step-by-step instructions should enable you to get through the labs, however this is not a course on Ansible and does not provide sufficient instruction for operating Ansible in production environments. You should pursue Ansible-specific training if you plan on using these modules beyond this course.
5. Students should have access to a lab environment capable of supporting the software used in the course, as detailed in the [Lab Setup](../Lab1-LabPrep/) section.
6. Students need to provide their own licensed access to any software used in this course. This course provides instruction for both Open Source and Commercial Software and provides no licensing or access to any software.

## Getting Started
**To Get Started, go to the [Lab Setup ](../Lab1-LabPrep/) section, and then proceed through the labs in order**.

Once you complete the lab setup, there are no specific dependencies between labs, so feel free to jump into any specific lab - but beware that some technical explanations may be provided in earlier labs that are not repeated in later labs so in general I recommend that you go through the labs in order.

I hope in time to add more slides and videos to provide more detailed explanations about the technologies used in the labs, but these arent ready yet. Please check back here in the future for updates.

## [Click here to proceed to the next lab](../Lab1-LabPrep/)