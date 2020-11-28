# Lecture

This lecture presents a technical overview of 
1. Object Storage, 
2. data security, 
3. S3FS, 
4. Fuse, and 
5. IBM Cloud Object Storage plugin.

## Overview of IBM Cloud Object Storage

An important part of data security and persistence on Kubernetes depends on physical storage outside the container orchestration engine that Kubernetes is. You can use PersistentVolume and PersistentVolumeClaim to map data directories to external physical storage. But also, data persistence on a stateless platform like Kubernetes should require extra attention.

IBM Cloud Object Storage (COS) offers a few exceptional features that help secure data on Kubernetes. IBM Cloud Object Storage (COS) actively participates in several industry compliance programs and provides the following compliance, certifications, attestations, or reports as measure of proof:
- ISO 27001,
- PCI-DSS for Payment Card Industry (PCI) USA,
- HIPAA for Healthcare USA, (including administrative, physical, and technical safeguards required of Business Associates in 45 CFR Part 160 and Subparts A and C of Part 164),
- ISO 22301 Business Continuity Management,
- ISO 27017,
- ISO 27018,
- ISO 31000 Risk Management Principles,
- ISO 9001 Quality Management System,
- SOC1 Type 2 (SSAE 16), (System and Organization Controls 1),
- SOC2 Type 2 (SSAE 16), (System and Organization Controls 2),
- CSA STAR Level 1 (Self-Assessment),
- General Data Protection Regulation (GDPR) ready,
- Privacy shield certified.

At a high level, information on IBM Cloud Object Storage (COS) is encrypted, then dispersed across multiple geographic locations, and accessed over popular protocols like HTTP with a RESTful API.

SecureSlice distributes the data in slices across geo locations so that no full copy of data exists on any individual storage node, and automatically encrypts each segment of data before it is erasure coded and dispersed. 

The content can only be re-assembled through IBM Cloud’s `Accesser` technology at the client’s primary data center, where the data was originally received, and decrypted again by SecureSlice. 

`Data-in-place` or `data-at-rest` security is ensured when you persist database contents in IBM Cloud Object Storage. 

You also have a choice to use integration capabilities with IBM Cloud Key Management Services like IBM Key Protect (using FIPS 140-2 Level 3 certified hardware security modules (HSMs)) and Hyper Protect Crypto Services (built on FIPS 140-2 Level 4-certified hardware) for enhanced security features and compliance.

## Overview of IBM Cloud Object Storage Plugin

This lab uses the [IBM Cloud Object Storage plugin](https://github.com/IBM/ibmcloud-object-storage-plugin) to connect an encrypted Object Storage to the Kubernetes cluster via `PersistentVolume`. A MongoDB database is setup that persists its data to a highly encrypted IBM `Cloud Object Storage` through PersistentVolume. A sample Java Spring Boot application stores its data in the MongoDB database and its data gets encrypted and persisted.

`IBM Cloud Object Storage plugin` is a Kubernetes volume plugin that enables Kubernetes pods to access IBM Cloud Object Storage buckets. The plugin has two components: a dynamic provisioner and a FlexVolume driver for mounting the buckets using `s3fs-fuse` on a worker node.

[`s3fs`](https://github.com/s3fs-fuse/s3fs-fuse) allows Linux and macOS to mount an S3 bucket via FUSE.

![](../.gitbook/images/cos-plugin-architecture.png)