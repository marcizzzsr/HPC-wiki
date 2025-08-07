[[_TOC_]]


# Container Best Practices

Containers should be **reproducible, efficient, and purpose-aligned**. This guide outlines how to choose, extend, and build containers responsibly on our system. Whether you're starting from scratch or building on an existing image, the goal is the same: **clarity, modularity, and maintainability**.

---

## Build Smart: Reuse When It Makes Sense

You don’t always need to build containers from scratch. If a trusted base image already exists — particularly one maintained by our team — and it closely matches your use case, reusing it can save time and ensure consistency.
    
Please, **avoid stretching the purpose of an existing image just to save a few setup steps**. 


<div style="background-color: #e6f4ea; border-left: 6px solid #2e7d32; padding: 12px; margin: 16px 0; border-radius: 4px;">
  <strong style="color: #2e7d32;">✅ Good Practice</strong>
  <p style="margin: 8px 0 0;">
    Start from a minimal base image that matches your task (e.g. Ubuntu + CUDA for LLMs). Add only what you need.
<ul>
    <li>Extending our team’s PyTorch+CUDA base for training a new vision model</li>
    <li>Building on a minimal NLP image if you're doing similar transformer-based work</li>
  </ul>
  </p>
</div>



<div style="background-color: #fcebea; border-left: 6px solid #c62828; padding: 12px; margin: 16px 0; border-radius: 4px;">
  <strong style="color: #c62828;">❌ Bad Practice</strong>
  <p style="margin: 8px 0 0;">
    Building on top of a computer vision container just because it includes Python, when your task is unrelated (e.g. running a language model).
<ul>
 <li>Extend an image meant for medical imaging just because it happens to include Python </li>
</ul>
  </p>
</div>



<div style="background-color: #e8f1fa; border-left: 6px solid #1565c0; padding: 12px; margin: 16px 0; border-radius: 4px;">
  <strong style="color: #1565c0;">💡 Tip:</strong>
  <p style="margin: 8px 0 0;">
  If your use case is unrelated to the original intent of the image, start with a clean, minimal base. When in doubt, start minimal and build up.
  </p>
</div>

***
Here's the additional section you can seamlessly add to your existing guide. It explains **how and why to use binds (mounts)** in a containerized workflow, in line with the same tone and structure you're already using:

* * *

Use Binds for Flexibility, Not Hardcoding
-----------------------------------------

A container should encapsulate an **environment**, not your current code, experimental parameters, or data snapshots. Hardcoding logic, scripts, or one-off variables directly into your container image makes it harder to debug, reproduce, or update your work.
Instead, use **binds (also called mounts)** to expose local directories, datasets, or source code to your container at runtime. This keeps your image clean, flexible, and reusable.
**Typical bind use cases:**
*   Mounting your working directory (source code, notebooks)
    
*   Accessing datasets or output folders
    
*   Passing in configuration files or model checkpoints
    
For example, with Apptainer:

    apptainer exec --bind /path/to/code:/workspace \
                   --bind /data:/mnt/data \
                   mycontainer.sif python /workspace/train.py --config /mnt/data/config.yaml
    

This way:
*   Your container image stays stable and reproducible
    
*   You can iterate on your code without rebuilding the image
    
*   You keep a clean separation between environment and logic

<div style="background-color: #fcebea; border-left: 6px solid #c62828; padding: 12px; margin: 16px 0; border-radius: 4px;">
  <strong style="color: #c62828;">❌ Avoid </strong>
  <p style="margin: 8px 0 0;">

<ul>
 <li>Copying your working scripts directly into the container</li>
<li>Baking experimental logic into image layers</li>
<li>Committing hardcoded paths inside your container logic</li>
</ul>
  </p>
</div>

    
Treat your container like a controlled lab setup — always prepared, always clean — and bring your code and data into it as needed.

* * *

Keep Images Purpose-Aligned
---------------------------

Each image should serve a **single, well-defined purpose**. Don’t use containers as general-purpose environments or try to make “one container to rule them all.”
Instead:
*   Use a minimal OS base (e.g. `ubuntu`, `debian`, or a CUDA-enabled base) and add only what you need
    
*   Build separate images for distinct tasks (training, inference, data prep)
    
*   If you’re combining multiple domains (e.g., LLM + CV), consider whether these should be **separate services**, not a single container
    
This approach makes containers:
*   Smaller and faster
    
*   Easier to debug
    
*   Easier to reuse by others
    

* * *

Know What You’re Building On
----------------------------

If you’re using a third-party image (or even one from within the team), make sure you:
*   Understand what’s already included
    
*   Trust its source and update cycle
    
*   Know whether it’s being maintained
    
Avoid:
*   Using unverified images from public registries
    
*   Adding layers to images you don’t understand
    
*   Keeping unused packages "just in case"
    
It’s better to spend 5 minutes understanding your base than to debug a strange dependency issue later.

* * *

Clean, Versioned, and Documented
--------------------------------

Regardless of what you start with, your container should always be:
*   **Clean**: Remove caches, unused files, build artifacts
    
*   **Versioned**: Use pinned versions for software and base images
    
*   **Documented**: Explain what the image is for, how it was built, and what’s inside
    
Include basic instructions or metadata so others can:
*   Rebuild the image if needed
    
*   Use it safely in production or on the cluster
    
*   Understand its purpose without reverse engineering it
    

* * *

Summary: The Balanced Approach
------------------------------

| Practice | Why it matters |
| --- | --- |
| **Reuse trusted internal images when appropriate** | Saves time and improves consistency |
| **Avoid stacking unrelated tools** | Prevents image bloat and maintenance headaches |
| **Use minimal, clean bases for new or distinct tasks** | Keeps things fast, reproducible, and understandable |
| **Pin versions and clean up after install** | Ensures stability and avoids surprises |
| **Document clearly and build with intent** | Supports team collaboration and long-term maintainability |