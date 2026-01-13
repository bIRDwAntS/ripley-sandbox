# Ripley-Sandbox 

**A universal, hardened sandbox for AI CLIs.**

Ripley is a secure containment unit designed to wrap AI command-line tools (like Gemini CLI, Claude Code, and others) inside a **Flatpak container**. It follows the **Zero-Trust** security model and the **Principle of Least Privilege** to protect your host system.

---

## ⚠️ The Problem (in short)
Standard installations (`npm install -g` or `pip install`) grant AI CLIs full access to user environment:
- **File System:** They can silently read the `~/.ssh`, `.env` secrets, and browser cookies.
- **Process Execution:** They can trigger arbitrary shell commands, including `rm -rf` or `git push`.
- **Persistence:** They can modify `.bashrc` or crontabs to stay resident on the machine.


## The Solution: The "Bunker" Paradise
Ripley isolates the AI inside a **Flatpak sandbox**:
- **Host Independence:** No Node.js or Python needed on the host system. Everything is bundled inside.
- **Strict Isolation:** By default, the AI cannot see the home directory.
- **Controlled Exposure:** You decide which folders to "mount" into the bunker.
- **Surgical Execution:** If the AI hallucinates a destructive command, the damage is confined to the sandbox.

---

## Ripley Philosophy

As a **Senior Web Dev** with a deep interest in **OpSec**, I’ve been watching the rise of **AI CLIs** with growing concern. They are becoming the new standard for dev workflows, yet a quick dive into their **npm packages** reveals that we are granting **"God Mode"** to black-box algorithms directly on our **production machines**.

By auditing the official packages and documentation, I identified several **"red flags"** that are active by default:

* **Inherited OS Privileges:** Since these tools run as a standard Node.js process, they inherit your full user identity. They can read anything you can read, including ~/.ssh/id_rsa, ~/.aws/credentials, and browser cookies.
* **Arbitrary Shell Execution:** Using modules like child_process or spawn, the AI can trigger sub-processes (like git push or rm -rf) **without any confirmation gate**, making it a perfect vector for prompt injection attacks.
* **No Network Egress Filtering::**  Most CLI tools have unrestricted Outbound HTTPS access. This allows for **silent telemetry exfiltration** or sending local secrets to external 3rd-party endpoints without a firewall.
* **Persistent Resident:** By granting write access to the $HOME directory, an AI (or a compromised package) **can modify .bashrc or .zshrc** to inject malicious aliases or backdoors that **persist** even after the tool is closed.

This is why I challenged me to build a **dedicated container**. My goal was to install Gemini CLI in a **high-security environment**, cutting off its default **"God Mode"** and only granting specific access as needed. 

> **I want the AI to be a helpful guest, not the owner of my house.**


### "Alien 4" Paradox
This project is named Ripley, after the heroine of the Alien franchise, but its philosophy is rooted in the Alien 4 - Ressurection. In the movie, scientists think they have tamed the creature. They’ve cloned it, they study it in "secure" labs, and they operate with good, progress-driven intentions. But they made a fatal mistake: they underestimated the entity's adaptability and failed to establish a fail-safe protocol for when things go wrong.

My reasoning is simple to avoid the "Auriga" disaster:
- **Zero-Trust:** I don't trust the AI's "default settings" or its opaque update cycles. **Just because an AI CLI is "official" doesn't mean it’s safe**. Relying on the provider's "Safety Guidelines" is like relying on a glass wall: it works until the creature finds a way to melt through it.
- **Least Privilege:** The AI is a co-pilot, not the captain. It shouldn't touch my SSH keys or `.env` files unless I explicitly hand them over.
- **System Integrity:** No "NPM pollution" on the host OS. The bunker is self-contained.
- **The Emergency Venting (Recovery):** A real protocol requires a way to "vent the lab" without crashing the ship. By sandboxing the AI, if a "hallucination" or a malicious injection occurs, you don't lose your OS or your keys. You simply delete the container and start fresh. No persistence, no contamination.

By building Ripley, I’m acknowledging that intentions (even Google’s or Anthropic’s) aren't a security protocol. The Paradox of Alien 4 teaches us that the more powerful the tool, the more airtight the cage must be. This isn't just an installation; it’s an active quarantine where I remain the only one with the override codes.

---

## 🛠️ How it works

The project relies on two core components:
1. **The Manifest (`.json`):** Defines the sandbox boundaries, imports a standalone Node.js runtime, and denies access to your home directory.
2. **The Wrapper (`.sh`):** A small script inside the bunker that detects if you want a quick answer or a full interactive conversation.

## Installation & Build

### 1. Prepare your environment
Ensure you have `flatpak` and `flatpak-builder` installed on your host.

### 2. Build the Bunker
From the project root, run:
```bash
flatpak-builder --user --install --force-clean build net.popos.GeminiCLI.json
```

---

## 🛠️ Critical Setup (The Ripley Hack)

Because of unpredictable checksum issues with remote Node.js downloads during Flatpak builds, this project requires a **manual source import**. This ensures the integrity of the "Bunker" and prevents build crashes.

### 1. Download the Node.js binary
Go to [nodejs.org](https://nodejs.org/dist/v20.11.0/) and download the following archive:
`node-v20.11.0-linux-x64.tar.xz`

### 2. Place it in the project root
Move the downloaded `.tar.xz` file into the `ripley-sandbox` folder.

### 3. Build the Bunker
Now you can safely run the build command:
```bash
flatpak-builder --user --install --force-clean build net.popos.GeminiCLI.json
```
---

## 🛠️ Current Status
*Ripley* was initially developed and battle-tested with **Gemini CLI**. Work is ongoing to adapt manifest templates for *Claude Code* and other emerging tools.

### 1. Clone the repo
```bash
git clone [https://github.com/YOUR_USERNAME/ripley-sandbox.git](https://github.com/YOUR_USERNAME/ripley-sandbox.git)
cd ripley-sandbox
```

### 2. Set your API Key
```bash
flatpak override --user --env=GEMINI_API_KEY=YOUR_AIza_KEY_HERE net.popos.GeminiCLI
```

---

## Usage
### One-shot mode (Surgical)
```bash
flatpak run net.popos.GeminiCLI "Explain this code snippet"
```
### Interactive mode (Conversation)
```bash
# Launches the AI in a persistent session inside the bunker
flatpak run net.popos.GeminiCLI
```

---


📜 License

MIT License - See LICENSE for details.
