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