---
title: "Domain-Driven Design - Book Takeaways"
date: 2019-03-12T21:53:48-04:00
draft: false
type: "post"
---

## Background

Domain-Driven Design: <i>Tackling Complexity in the Heart of Software</i> is a book authored by Eric Evans that was originally published in 2003. This book introduces the "DDD" software development approach and offers an extensive set of techniques and ideologies to build software for complex business domains.

## Why I Read This Book

The concept (philosophy?) of "DDD" is often quoted and evangelized in the software industry and I wanted to understand what all the hype was about. I heard that this book was responsible for "coining" the term so it seemed like a great place to start!

Warning: Domain-Driven Design was NOT an easy read but there were many gems buried within the academic language and verbosity.

## Takeaways

### <i>Effective domain modelling can be boiled down to a few general steps.</i>

* Capture a knowledge rich model
* Bind model and implementation
* Cultivate a language built on the model
* Distill the model (drop unhelpful concepts)
* Brainstorm & experiment to improve the model

### <i>Domain modelling is a continuous, iterative process.</i>

An effective business is constantly evolving - the same goes for your domain model! Models and implementations that are initially sufficient will inevitably become out-dated as a product evolves.

### <i>Work together with the domain experts to establish a domain model and develop a ubiquitous language while you're at it.</i>

Speaking with the same language as your domain experts is vital to establishing a knowledge rich domain model. Often times implementation details become apparent when discussed in a language that both developers and domain experts can understand.

### <i>Keep business rules in the domain layer. Use layered architecture for good separation of concerns.</i>

Create boundaries around your domain model! The benefits of separating concerns don't need to be restated.

### <i>Express the model directly in your code - i.e. use objects to represent model entities and domain concepts.</i>

Use conceptual objects that can help express domain models:

<u>Domain Services</u>

Domain logic that span many entities, co-located with domain model. Defined in terms of what it can do for its client. Domain services relate to concepts not natural to a entity or a value object. Ideally these are stateless!

<u>Entities</u>

Objects that have an identity that is meaningful to the model - i.e. a User. Entities can exist in many forms (for example, across bounded contexts in a microservice architecture).

<u>Value Objects</u>

Immutable data objects that don't have an identity (ID) and are not typically stored in a database - can be created on the fly). Value objects can be composed of information from entities but should be used when you care only about the attributes. Avoid bi-directional associations in value objects and consider optimizing with flyweight patterns.

<u>Aggregates</u>

Cluster of associated objects to be treated as a unit. Only the root entity should be passed around or referenced outside of the aggregate. Root entity can provide other entities but only as value objects. Deleting an aggregate must remove everything within the aggregate boundary.

<u>Factories</u>

Responsible for complex assembly operations that don't fit the responsibilities of the created object. Can be used to create entire aggregates. Atomic & consistent.

<u>Repositories</u>

Provide a simple model for managing persistence objects. Decouple application and domain design from persistence technology (DBs). Leaves transaction control to the client. Factories create new objects, repositories find old ones!

### <i>Be aware of events that challenge your domain model and be ready to make updates.</i>

After all, you don't know what you don't know. The important part is that domain modelling is an iterative process and the model should be constantly refined and refactored!

### <i>If a developer must consider the implementation of a component in order to use it, encapsulation is lost.</i>

This is a warning sign that means your interfaces are not intention-revealing. Ensure that you have a supple design by having well defined interfaces, side effect free functions, assertions, etc.

### <i>Relate design patterns to the model if they make sense - but keep in mind the difference between a domain pattern and a design pattern.</i>

Design patterns such as Strategy/Policy, Composite, Command, State etc might be applicable as domain patterns and can be helpful when designing a domain model. Design patterns like flyweight, although useful, provide a solution to a technical problem and does not fit as a domain pattern.

### <i>Don't feel the need to stick to a single model. Total unification of a domain model is not feasible or cost effective. Separate models can be developed for separate contexts.</i>

A context can be a department in your business or even a separate scrum team. Trying to stick to a single domain model doesn't make sense but it is still possible to apply domain driven design on a smaller scale using models separated by <b>bounded contexts</b>. Setting up a translation layer with facades, adapters, etc between bounded contexts is an effective way to avoid unwanted diffusion across domains - a concept Eric Evans calls an <b>anticorruption layer</b>.

### <i>If you want to succeed in domain design design, find developers who have an interest in the domain and pair them with domain experts.</i>

Restating this point because it is important. Effective domain modelling is not just in the code - it's also in the minds of the development team! Building domain knowledge and context within your team is crucial for identifying domain model improvements and inconsistencies.

### <i>How much effort you put into domain driven design can depend on the skill of your team members.</i>

Building an effective domain model is challenging and often times must require team members to understand both the design concepts and the domain. Assess the skills of your team members before trying domain driven design or the cost may outweigh any potential gain.
