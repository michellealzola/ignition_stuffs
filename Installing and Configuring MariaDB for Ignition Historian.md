# Procedure

## Installing and Configuring MariaDB for Ignition Historian

**Ignition Version:** 8.1.47
**Database:** MariaDB 10.11 LTS
**Operating System:** Windows
**Use Case:** Ignition Tag Historian (Local Gateway)

---

## 1. Purpose

This procedure describes the complete process for installing MariaDB and configuring it as a **Tag Historian database** for an Ignition Gateway running **Ignition 8.1.47**.

The goal is to:

* Install a stable, long-term supported database
* Configure secure database access for Ignition
* Connect Ignition Historian to the database
* Verify that historical data is successfully stored and displayed in trends

---

## 2. Prerequisites

Before starting, ensure the following:

* Ignition Gateway **8.1.47** is installed and running
* Administrator access to the Windows machine
* Local access to the Ignition Gateway Web Interface
* A supported Java runtime (installed automatically with Ignition)

---

## 3. MariaDB Version Selection

### 3.1 Why MariaDB 10.11 LTS

MariaDB **10.11.x (LTS)** was selected because:

* It is a **Long-Term Support** release
* It is stable and well-tested for long-running workloads
* It is suitable for **write-intensive historian data**
* It aligns with typical Ignition production deployments

> Rolling or short-term MariaDB versions (e.g., 11.x non-LTS or 12.x Rolling) are **not recommended** for historian use.

---

## 4. MariaDB Installation

### 4.1 Installer Selection

Download the following package:

* **MariaDB Server 10.11.x**
* Operating System: **Windows**
* Architecture: **x86_64**
* Package Type: **MSI Installer**

---

### 4.2 Installation Steps

1. Run the MSI installer as Administrator
2. Accept the license agreement
3. Choose **Typical** installation
4. Set a **root password**

   * This password is for database administration only
   * It is **not** used by Ignition
5. Enable **Install as a service**
6. Set the service to **Start automatically**
7. When prompted for the TCP port:

   * If port **3306** is already in use, select an alternate port (e.g., **3307**)
   * This procedure uses **port 3307**
8. Complete the installation

---

### 4.3 Verify MariaDB Service

Confirm that MariaDB is running:

* Open `services.msc`
* Verify **MariaDB** service status is **Running**
* Startup type should be **Automatic**

---

## 5. Database and User Setup

### 5.1 Open MariaDB Command Line

Because MariaDB is not added to the Windows PATH by default, run:

```bat
"C:\Program Files\MariaDB 10.11\bin\mysql.exe" -u root -p -P 3307
```

Log in using the root password created during installation.

---

### 5.2 Create the Historian Database

```sql
CREATE DATABASE historian
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_general_ci;
```

Verify creation:

```sql
SHOW DATABASES;
```

---

### 5.3 Create a Dedicated Ignition User

Ignition should **never** use the `root` account.

```sql
CREATE USER 'ignition'@'localhost'
IDENTIFIED BY 'StrongPasswordHere';
```

Grant required permissions:

```sql
GRANT ALL PRIVILEGES ON historian.* TO 'ignition'@'localhost';
FLUSH PRIVILEGES;
```

Exit MariaDB:

```sql
EXIT;
```

---

## 6. MariaDB JDBC Driver Installation

### 6.1 Why This Step Is Required

Ignition is a **Java-based platform** and requires a **JDBC driver** to communicate with MariaDB.
MariaDB Connector/C, ODBC, or other connectors will **not work**.

---

### 6.2 Download JDBC Driver

Download:

* **MariaDB Connector/J (Java 8+)**
* Version: **3.x (GA release)**
* File type: `.jar`
* OS: **Platform Independent**

Expected file name format:

```
mariadb-java-client-3.x.x.jar
```

---

### 6.3 Install the Driver

Copy the `.jar` file to:

```
C:\Program Files\Inductive Automation\Ignition\lib\core\gateway\
```

---

### 6.4 Restart Ignition Gateway

Ignition only loads JDBC drivers at startup.

Restart the gateway using **one** of the following:

* `stop-ignition.bat` → `start-ignition.bat` (Run as Administrator)
* OR restart **Ignition Gateway** from `services.msc`

---

## 7. Database Connection Configuration in Ignition

### 7.1 Open Gateway Web Interface

```
http://localhost:8088
```

Navigate to:

```
Config → Databases → Connections
```

---

### 7.2 Edit or Create Database Connection

An existing historian connection may be edited.

Set the following parameters:

| Field         | Value                    |
| ------------- | ------------------------ |
| Database Type | MariaDB                  |
| Host          | localhost                |
| Port          | 3307                     |
| Database      | historian                |
| Username      | ignition                 |
| Password      | (ignition user password) |
| Translator    | MYSQL                    |

> The MYSQL translator is correct for MariaDB due to protocol compatibility.

Save the connection.

---

### 7.3 Verify Connection Status

The connection status should change to:

```
Connected
```

If faulted, review the **Database Connection Status** page for error details.

---

## 8. Historian Provider Configuration

Navigate to:

```
Config → Tags → History
```

Confirm:

* A **Datasource History Provider** exists
* Provider name (e.g., `LACTHistorian`)
* Status is **Running**

Ignition automatically creates required historian tables (`sqlth_*`) when the provider starts.

---

## 9. Verification and Testing

### 9.1 Create a Test Tag

In Ignition Designer:

1. Create a **Memory Tag**
2. Data type: Float
3. Enable **History**
4. Assign history provider (`LACTHistorian`)

---

### 9.2 Generate Data

Change the tag value multiple times over several seconds.

---

### 9.3 View Trends

Open **Tag History** for the tag.

**Successful result:**

* Historical values appear on the trend
* Confirms end-to-end historian functionality

---

## 10. Result

At completion of this procedure:

* MariaDB 10.11 LTS is installed and running
* A dedicated historian database and user are configured
* MariaDB JDBC driver is loaded into Ignition 8.1.47
* Ignition Historian is connected and operational
* Trends are successfully displaying historical data

---

## 11. Notes and Best Practices

* Avoid using database root credentials in Ignition
* Use LTS database releases for historian systems
* Document custom ports (e.g., 3307) clearly
* Restart Ignition whenever JDBC drivers are added or changed
* Periodically review historian storage and retention settings


