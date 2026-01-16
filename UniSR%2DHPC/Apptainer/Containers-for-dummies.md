# Containers for Dummies: A Beginner's Guide

Welcome! If you're new to containers, this guide is for you. Whether you're a developer, researcher, or just curious about how software runs these days, we'll break down the concepts in a simple, tool-agnostic way.

[[_TOC_]]


---

## What Is a Container?

A **container** is a lightweight, portable way to package software and everything it needs to run:
- Code
- Libraries
- Tools
- Settings

You can think of it like a **"mini-computer" inside your computer**. It runs exactly the same way anywhere — on your laptop, on a cluster, or in the cloud.

> Imagine sending someone a zip file that contains your code *and* a tiny environment to run it — that’s a container.

---

## Why Should You Care?

Using containers:
- Ensures your app works the same on different machines
- Makes your code easier to share and reproduce
- Simplifies setup — no more "it works on my machine"
- Is essential for modern HPC, cloud, and scientific workflows

---

## Key Concepts

| Term | Meaning |
|------|---------|
| **Image** | A frozen package of a container (like a template) |
| **Container** | A running instance of an image |
| **Recipe** | Build instructions for an image (e.g., `Dockerfile` or Apptainer `.def`) |
| **Registry** | A place where images are stored and shared |
| **Build** | The process of creating an image |
| **Run** | Starting a container from an image |

---

## Image vs Container: Two Helpful Metaphors

Pick the metaphor that matches your background:

### Metaphor A — Virtual machines

- The **image** is like an **ISO file** (a frozen template of an operating system/environment).
- The **container** is like the **virtual machine** you start from that ISO.

### Metaphor B — Object-oriented programming

- The **image** is like a **class**.
- The **container** is like an **object (instance)** created from that class.

In both metaphors: you can create many containers (instances) from the same image (template), and they behave consistently.

## Where does an image come from?

An image is created from a “recipe” (build instructions):

- In Docker, that recipe is a `Dockerfile`.
- In Apptainer, it’s a **definition file** (usually a `.def` file).

Think of the recipe as the **source code of the class**: it defines what the image will contain and how it behaves (installed tools/libraries, environment variables, default run behavior).

## How Containers Fit Into Your Workflow

1. **Build an image**  
   Package your code, environment, and dependencies into an image.

2. **Run a container**  
   Start a container from the image. It runs in isolation, like a sandbox!

3. **Do your work**  
   Run scripts, software, experiments, etc., all inside the container.

4. **Share or reuse**  
   The image can be shared or reused on another machine.

---

What Goes in a Container?
----------------------------

*   Operating system base (e.g., Ubuntu, Alpine)
    
*   Required packages or modules
    
*   Your application code
    
*   Any scripts or configs
    

> A container image is like a **snapshot** of a tiny working system.

* * *

File Access & Isolation
--------------------------

Containers can:
*   Access files on your host (if you allow it)
    
*   Have isolated environments (no interference with your system)
    
*   Run as non-root (in many systems)
    
This makes them great for HPC environments and reproducible science.

* * *

FAQ
----

- **Q: Do I need to learn Docker?**  
**A:** Not necessarily. We use Apptainer, but the core ideas are the same across all tools.
- **Q: Is it like a virtual machine?**  
**A:** Not quite. Containers are faster, lighter, and don’t simulate hardware.
- **Q: Can I use it with Python/R/MATLAB?**  
**A:** Absolutely. You can package any language or tool in a container.

* * *

Further Reading (Optional)
-----------------------------

*   [Containers 101 - RedHat](https://www.redhat.com/en/topics/containers/what-is-a-linux-container)
    
*   [Apptainer Documentation](https://apptainer.org/docs/)
    
*   [Docker Concepts (Still Useful!)](https://docs.docker.com/get-started/overview/)
    

* * *

## Next steps

If you’re using containers on the UniSR cluster and wish to understand how they integrate into our system, see:

*   **[HPC: simple introduction](/UniSR%2DHPC/HPC-simple-introduction)**
*   **[Introduction to workload managers](/UniSR%2DHPC/SLURM/Workload-Managers-for-dummies)**

**Remember:** A container is a way to say “Run this code exactly the way I intended — anywhere.”

    
