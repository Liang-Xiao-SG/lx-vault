---
layout: post
author: liangxiao
---
It sounds like your **OpenClaw** instance is experiencing the classic "sleepy agent" syndrome. Since you're running it on a local Mac, the most likely culprit isn't the code itself, but how macOS handles background processes and power management.

Here is a breakdown of the mechanism and why it might be ghosting you.

### 1. The Core Mechanism: The "Gateway"

OpenClaw operates using a **Gateway-centric architecture**. Think of the Gateway as the "Nervous System" of your agent.

- **Always-on Daemon:** When you run `openclaw gateway start`, it launches a Node.js process (typically on port `18789`). This process is responsible for keeping the WebSockets open to your messaging channels (like Telegram or WhatsApp).
    
- **The Lane Queue:** OpenClaw uses a **Sequential Execution** mechanism. If you send multiple messages, it places them in a "Lane Queue." It won't start Task B until Task A (the LLM reasoning or a script execution) is finished. If a model hangs or a tool takes too long, the gateway stays "busy" and won't respond to new inputs until the queue clears.
    

### 2. Why It’s Not Responding (The "Mac" Factor)

On a MacBook, the hardware is designed to be aggressive about saving energy, which is the natural enemy of a persistent background agent.

- **App Nap & Sleep:** When you close the lid or the Mac goes idle, macOS enters a "Deep Idle" state. Even if the process is technically running, the **Network Stack** often suspends long-lived WebSocket connections to save power. When you send a message, the connection is already "dead" on the Mac's side, and it only realizes it needs to reconnect once you manually wake the machine.
    
    +1
    
- **Foreground vs. Background:** If you started OpenClaw in a standard Terminal window and just left it there, macOS may deprioritize that process when the window isn't "active."
    
- **SIP and Permissions:** Occasionally, macOS's Transparency, Consent, and Control (TCC) framework "forgets" that Terminal has Full Disk Access or Accessibility permissions if the process hasn't been refreshed in a while.
    

### 3. How to Make It "Always Ready"

To transition from a "sometimes" agent to an "always-on" assistant, try these adjustments:

#### **A. Use the "Caffeinate" Command**

If you want a quick fix without changing system settings, run OpenClaw wrapped in the `caffeinate` command to prevent the system from sleeping:

Bash

```
caffeinate -is openclaw gateway start
```

_`-i` prevents system sleep; `-s` prevents sleep while on AC power._

#### **B. Deploy as a Daemon**

Instead of running it in a visible terminal tab, install it as a **launchd** service. This tells macOS the process is a persistent system utility.

Bash

```
openclaw onboard --install-daemon
```

#### **C. Tweak Energy Settings**

If you are using a MacBook in "Clamshell mode" (lid closed), ensure the following are enabled in **System Settings > Battery/Lock Screen**:

- **Wake for network access:** Enabled.
    
- **Prevent automatic sleeping on power adapter when the display is off:** Enabled.
    

---

### **Troubleshooting Checklist**

If it still feels laggy, run the "Doctor" command to see if there are internal bottlenecks:

- `openclaw doctor` — Checks your configuration and API connectivity.
    
- `openclaw gateway logs --tail 20` — Look for "Socket hang up" or "Model timeout" errors.
    

**Would you like me to help you draft a `launchd` plist file to ensure OpenClaw starts automatically every time you log in?**