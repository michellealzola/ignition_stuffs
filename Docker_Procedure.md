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


