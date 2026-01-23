# Procedure: Set Up Ignition (Edge or Standard) Using Docker on Windows

## 1. Prerequisites

Before starting, make sure you have:

* Windows 10 or 11 (64-bit)
* Administrator access
* Internet connection
* **PowerShell** (run as normal user unless stated)

---

## 2. Install Docker Desktop

1. Download Docker Desktop from the official Docker website
2. Run the installer
3. During installation:

   * ✅ Enable **WSL 2**
   * ✅ Use **Linux containers**
4. Restart your computer when prompted

---

## 3. Start Docker Desktop

1. Launch **Docker Desktop**
2. Wait until Docker reports **“Docker is running”**
3. Verify Docker from **PowerShell**:

```powershell
docker --version
```

Expected output:

```
Docker version XX.X.X
```

---

## 4. Pull the Ignition Image (optional but recommended)

Example for **Ignition Edge 8.3.2**:

```powershell
docker pull inductiveautomation/ignition:8.3.2
```

(This step can be skipped — Docker will pull the image automatically if it’s missing.)

---

## 5. Run Ignition in Docker (Default Ports)

### Standard Ignition ports

* **8088** → Gateway web interface
* **8043** → HTTPS

Run this command in **PowerShell**:

```powershell
docker run -d `
  --name ignition832 `
  -p 8088:8088 `
  -p 8043:8043 `
  -e ACCEPT_IGNITION_EULA=Y `
  -v ignition832-data:/usr/local/bin/ignition/data `
  inductiveautomation/ignition:8.3.2
```

### What this does

* Starts Ignition in the background
* Accepts the Ignition EULA
* Uses a **Docker-managed volume** for persistent data
* Exposes Ignition to your local machine

---

## 6. If the Default Ports Are Not Available (Important)

If you see an error like:

```
port is already allocated
```

You must map Ignition to **different host ports**.

### Example: Alternate ports

* Host `9088` → Container `8088`
* Host `9043` → Container `8043`

```powershell
docker run -d `
  --name ignition832 `
  -p 9088:8088 `
  -p 9043:8043 `
  -e ACCEPT_IGNITION_EULA=Y `
  -v ignition832-data:/usr/local/bin/ignition/data `
  inductiveautomation/ignition:8.3.2
```

You would then access Ignition at:

```
http://localhost:9088
```

---

## 7. Verify the Container Is Running

```powershell
docker ps
```

You should see:

* Container name: `ignition832`
* Status: `Up` or `Up (healthy)`

If the container exits immediately, check logs:

```powershell
docker logs ignition832
```

---

## 8. Access the Ignition Gateway

Open a browser and go to:

* Default ports:

  ```
  http://localhost:8088
  ```
* Alternate ports:

  ```
  http://localhost:9088
  ```

You should see the **Ignition Gateway Setup Wizard**.

---

## 9. Install the Ignition Designer (Required)

1. From the Gateway homepage, click:
   **“Download Designer Launcher”**
2. Install or upgrade the launcher
3. Open the Designer Launcher
4. Add a gateway:

   * Gateway URL: `http://localhost:8088` (or `9088`)
5. Launch the Designer

> Note: The Designer does **not** run in Docker. It runs locally and connects to the Gateway.

---

## 10. Stopping and Starting Ignition

### Stop Ignition only:

```powershell
docker stop ignition832
```

### Start Ignition again:

```powershell
docker start ignition832
```

### Completely shut down Docker:

* Right-click the Docker whale icon → **Quit Docker Desktop**

---

## 11. Important Notes and Best Practices

* Always use **Docker-managed volumes** on Windows
  (Avoid bind-mounting Windows folders for Ignition data)
* Docker must be running for Ignition to be available
* Projects and configuration are stored in the Docker volume and are **not lost** when containers stop
* You can run **multiple Ignition versions side-by-side** by:

  * Using different container names
  * Using different host ports

---

## Summary (Quick Reference)

* PowerShell is used for all Docker commands
* Docker Desktop must be running
* Ignition runs in a container
* Designer runs locally
* If ports are unavailable → map to different ports
* Data persists using Docker volumes



---

## 1️⃣ If 9088 is not available: how ports work (quickly)

Docker port mapping follows this rule:

```
-hostPort : containerPort
```

For Ignition:

* **Container ports NEVER change**

  * `8088` = Gateway HTTP
  * `8043` = Gateway HTTPS
* **Host ports CAN be anything that’s free**

So if:

* 8088 ❌
* 9088 ❌

You just pick another unused port, for example:

* 10088
* 18088
* 28088

---

## 2️⃣ Example: Using port **10088** instead

### PowerShell command

```powershell
docker run -d `
  --name ignition-latest `
  -p 10088:8088 `
  -p 10443:8043 `
  -e ACCEPT_IGNITION_EULA=Y `
  -v ignition-latest-data:/usr/local/bin/ignition/data `
  inductiveautomation/ignition:latest
```

### Access Ignition at:

```
http://localhost:10088
```

✅ Works exactly the same
✅ Only the URL changes

---

## 3️⃣ Using the **latest version** for personal use

When you use:

```powershell
inductiveautomation/ignition:latest
```

Docker will:

* Always pull the **most recent stable Ignition release**
* Perfect for:

  * Learning
  * Testing
  * Personal projects
  * Non-production use

### Important note

If you restart the container later, it will:

* Use the **already-downloaded version**
* NOT auto-upgrade unless you explicitly pull again

To update later:

```powershell
docker pull inductiveautomation/ignition:latest
```

---

## 4️⃣ Recommended naming for personal-use containers

To avoid confusion later:

```powershell
--name ignition-personal
-v ignition-personal-data:/usr/local/bin/ignition/data
```

This keeps it clearly separate from:

* Work projects
* Version-pinned containers (like 8.3.2)

---

## 5️⃣ How to find a free port (optional but helpful)

In PowerShell:

```powershell
netstat -ano | findstr LISTENING
```

Or just try a high port number:

* Anything above **10000** is usually safe

---

## 6️⃣ Summary (plain English)

* If **9088 is taken**, pick another port (e.g. 10088)
* The **right side (8088)** never changes
* `latest` is fine for **personal / learning use**
* Docker doesn’t auto-update unless you tell it to
* Only the browser URL changes

---

### Example URLs you might end up with

| Host Port | URL                                              |
| --------- | ------------------------------------------------ |
| 8088      | [http://localhost:8088](http://localhost:8088)   |
| 9088      | [http://localhost:9088](http://localhost:9088)   |
| 10088     | [http://localhost:10088](http://localhost:10088) |
| 18088     | [http://localhost:18088](http://localhost:18088) |

