---
title: "Loan Collection System"
layout: post
date: 2018-06-16
tag: 
    - EMI
    - BHIM UPI
    - Bharat QR
    - Rule Engine
    - Micro Services Architecture
image: https://www.npci.org.in/sites/all/themes/npcl/images/product-overview/bharat_qr-logo.png
headerImage: true
projects: true
hidden: false # don't count this post in blog pagination
description: "This project was a loan collection system designed to facilitate small business owners to take full control of their loan repayments."
category: project
author: pranjal
externalLink: false
---

## Summary:
This project was developed to facilitate small business owners to take full control of their loan repayments and at the same time reducing the default rate and hence benefiting the lender.

---

### Tech stack:
* Spring Boot, Spring AOP
* Hibernate
* MySQL
* Maven -- Parent POM and Child POM
* Swagger
* ZXing
* OAuth2 (Authentication by bearer schema)
* AWS EC2, RDS, SNS
* Drools Rule Engine

---

### Further reading:
**Spring Boot** was the choice of framework because of its ease of use. Additionally, using *Spring Boot* means a huge amount of code could be reused from earlier projects. **Spring AOP** was used in places where it was necessary to find the time taken by each function to execute, so as to work on optimizing the code in future. Also, when a request was received, it was first authenticated (using decryption) and then verified (using checksum); all of which was done using Spring AOP.

**Hibernate** was the choice of ORM because of its vendor independance and easy integration with Spring environment. **MySQL** databasae was used which was deployed on an **AWS RDS**.

Since this project had a lot of resusable code, it was a good idea to divide the project in a parent module and a child module. **Maven** helped in making parent and child modules which had different POMs. The parent module had basic configurations in place for *Resource Server* and *Authentication Server*. Also, **Swagger** was configured in the parent module. Child module was able to pick and choose which features/configurations it wanted to use from the parent module and was free extend the code or simply write it from scratch.

Swagger 1.3 was used instead of Swagger 2 because, at the time of development, Swagger 2 did not allow for token based authentication using **OAuth2** bearer schema.

OAuth2 authentication was used for clients (typically Android app and Web app) to authenticate against the service. Clients sent **JWT** in *Bearer schema* to request access to a part of the system. However, other web services were authenticated using *Basic Authentication* and public/private keys.

In addition to this, **ZXing** was used to generate *UPI QR* and *Bharat QR* depending on the client request. Client could select the type of QR (static/dynamic and UPI/BQR) at the time of registration. Also, for generating QR Codes a <u>15% error correction at *Level M*</u> was used as a provision to embed organization logo in the QR. Additionally **Drools** was used to validate *BHARAT QR*.

Stay tuned for a blog post on **How to embed a logo in a QR code**.
