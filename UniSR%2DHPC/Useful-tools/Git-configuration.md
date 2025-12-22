
# Purpose & Overview
When working on a remote cluster, you need a secure way to communicate with Git hosting services (e.g. GitHub). 
    
By default, Git often uses **HTTPS**, which requires you to enter your username and a Personal Access Token (PAT) frequently. By switching to **SSH (Secure Shell)**, you use a cryptographic key pair for authentication:
1.  **Private Key:** Stays securely on the cluster (never share this!).
2.  **Public Key:** Uploaded to your Git provider.
    
Once configured, the cluster will automatically "handshake" with the Git server, allowing you to push and pull code seamlessly without manual logins.

# 1. Initial Git Identity Configuration
Before using Git, you should set your identity. This information is attached to your commits so others know who authored the code. Run these commands with your info:
 
```bash
# Set your name
git config --global user.name "Your Name" 
# Set your email (should match your Git provider email)
git config --global user.email "your_email@example.com"
```

# 2. Generate a New SSH Key Pair

If you don't have a key, generate a new one using the `ed25519` algorithm (the current standard for security and performance): 

    ssh-keygen -t ed25519 -C "your_email@example.com"
    
*   `your_email@example.com` should coincide with the email of your GitHub account!
*   When prompted to **"Enter a file in which to save the key,"** press **Enter** to use the default location.
    
*   When prompted for a **passphrase**, you can either enter one for extra security or press **Enter** twice for no passphrase (convenient for automated cluster work).
    

# 3. Register the Public Key with Your Git Provider

You must provide your **Public Key** to your Git service so it recognizes your cluster account.
1.  **Display the public key:**
    
        cat ~/.ssh/id_ed25519.pub
        
    
2.  **Copy the entire output** (starting with `ssh-ed25519` and ending with your email).
    
3.  **Paste it into your provider's settings:**
    *   **GitHub:** Settings -> SSH and GPG keys -> New SSH Key.
        
    *   **GitLab:** Edit Profile -> SSH Keys -> Add new key.
        
    *   **Bitbucket:** Personal settings -> SSH keys -> Add key.
        


# 4. Test Your Connection

Run the following command to ensure the handshake works:

Bash

    # For GitHub
    ssh -T git@github.com
    
    # For GitLab
    ssh -T git@gitlab.com
    

> **Note:** If this is your first time connecting, you will see a message: _"The authenticity of host ... can't be established."_ Type **yes** and hit **Enter**. You should receive a "Hi [Username]! You've successfully authenticated" message.



# 5. (Optional) Clone a GitHub repo

Remember to clone a repo using it's SSH url, you can find it right next to your usual HTTPS link on the repo you want to clone.
