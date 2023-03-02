---
title: Distroless Container Images
---

Usually source code is compiled and added as a new layer on some existing container base image. Some
programming languages require some interpreter to run like a Python interpreter or a virtual machine
running Java bytecode.

It is convenient to use a base image populated with normal *nix tooling and maybe even based on a known
Linux distribution such as Ubuntu. This allows for easy debugging by executing commands inside of the
running container image. However this also expands the surface of attack both with respect to the number
of tools and service that might contain vulnerabilites but also the tools aviailable should someone be
able to execute arbitrary commands within the running conatiner.

At the same time the more utilities and libraries that exists in the images the bigger the image
becomes. The size in itself is not a problem as such however size do matter when it comes to startup
times and also the amount of storage required both on the Kubernetes worker nodes as well as in the
container registry.

To reduce both attack surface and size it is recommended that production images are built based on
distroless base images - if at all possible. Google provides distroless base images for a number of
interpreted and compiled languages see [distroless](https://github.com/GoogleContainerTools/distroless).
