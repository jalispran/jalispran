---
title: "Bill Payments System"
layout: post
date: 2017-09-19
tag: 
    - Bharat Bill Payments System
    - Java Reflection
    - BASIC Authentication
    - Micro Services Architecture
image: https://5.imimg.com/data5/PO/YE/CD/SELLER-25093353/bharat-bill-payment-services-500x500.jpg
headerImage: true
projects: true
hidden: false # don't count this post in blog pagination
description: "This was a integration with Indian Govt.'s Bharat Bill Payments Plaltform."
category: project
author: pranjal
externalLink: false
---

## Summary:
This was one of the most ambitious project that I had worked on. It included vast amount of technologies.

---

### Tech stack:
* Spring Boot
* HIbernate
* MySQL
* Maven
* Java Reflection
* AWS EC2, RDS

---

### Further reading:
**Bharat Bill Payments System** is *NPCI*'s centralised Bill Payment System For Everyday Bills. This project integrates with *BBPS* enabled and *non-BBPS* billers to provide a uniform and consistent interface for all the users (Individuals/Banks/Aggregators) where they can make bill payments.

**Spring Boot** was the choice of framework because of its ease of use, a huge community support and excellent documentation. **Maven** was used as a build tool. The resulting *jar* after *Maven* build was deployed on **AWS EC2** instance which was maintained in the same *security group* as an **AWS RDS** instance.

Since there were huge number of billers and each bill is a function of a set of multiple disjoint variables, managing the bill payment code was a nightmare. Hence, **Java Reflection** to the rescue. All the variables for each biller were stored in a **MySQL** database in a standard format and *Reflection* was used to get values of these variables, from user input, at runtime. This made the code easy to understand and maintain and moreover, it introduced uniformity in an otherwise chaotic code.