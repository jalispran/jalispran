---
title: "Bill Payments System"
layout: post
date: 2017-11-03
tag: 
    - Bharat Bill Payments System
    - Java Reflection
    - BASIC Authentication
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
**Bharat Bill Payments System** is *NPCI*'s centralised Bill Payment System For Everyday Bills. This project integrates with *BBPS* enabled and *non-BBPS* billers to provide a consistent and provides users (Individuals/Banks/Aggregators) with the ability to make bill payments.

**Spring Boot** was the choice of framework because of its ease of use, a huge community support and excellent documentation. **Maven** was used as a build tool. The resulting *jar* after *Maven* build was deployed on **AWS EC2** instance which was maintained in the same *security group* as an **AWS RDS** instance.

Since there are huge number of billers and each biller

For one of the use cases, name of some java classes were stored in the database and **Java Reflection** was used to call methods of that calss. These classes represented the different flows the client could take in its lifecycle within the system. The database used was a **MySQL** database with **Hibernate** used as an *ORM*. This database was deployed on **AWS RDS**. Also, the databse maintained audit trail of each and every operation performed by the service. This was achieved using **Spring AOP**.

Stay tuned for a blog post about **Basics of Spring AOP**.