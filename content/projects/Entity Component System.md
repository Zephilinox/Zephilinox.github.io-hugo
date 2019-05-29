---
date: 2018-05-03
title: Entity Component System
weight: 2
---

After reading the blog post on [T-Machine](http://t-machine.org/index.php/2007/09/03/entity-systems-are-the-future-of-mmog-development-part-1/) I decided to create an [Entity Component System](https://github.com/Zephilinox/ECS) by using C++ and SFML. The ECS is unlike that present in Unity, as all components only contain data.

<!--more-->

Systems operate on groups of components which is how various functioality is implemented, such as movement, controls, and rendering. Each system has access to the EntityManager which in turn stores all Entities as lists of Components, using Variadic Templates to return specific subsets of entities based on the components they have.

This ECS was written a long time ago, to the point where I've struggled to find any other C++ implementation that outdates it. That said, it's incredibly basic, incredibly rough, doesn't handle all the things you'd want an ECS to handle, and isn't as performant as you'd want an ECS to be. It was an interesting project that I never fully finished.