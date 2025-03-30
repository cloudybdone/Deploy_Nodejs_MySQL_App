# Deploy Nodejs_MySQL Applicaton with Systemd Service


## Overview

This project demonstrates how to set up a Node.js application with a MySQL database, deploy it as a systemd service, and ensure automatic restarts.

## Part 1: Application Setup

### 1. Create Project Directory and Initialize

```bash
mkdir node-mysql-app
cd node-mysql-app
npm init -y
```

### 2. Install Dependencies

```bash
npm install express mysql2
```

### 3. Create Node.js Application (app.js)

```javascript
const express = require('express');
const mysql = require('mysql2/promise');

const app = express();
const port = 3000;

// Database configuration
const dbConfig = {
  host: 'localhost',
  user: 'poridhi_user',
  password: 'tcl@12345',
  database: 'practice_app'
};

// Middleware to parse JSON
app.use(express.json());

// Health endpoint
app.get('/health', async (req, res) => {
  try {
    const connection = await mysql.createConnection(dbConfig);
    await connection.ping();
    await connection.end();
    res.json({ status: 'healthy', database: 'connected' });
  } catch (error) {
    res.status(500).json({ status: 'unhealthy', database: 'disconnected', error: error.message });
  }
});

// Users endpoint
app.get('/users', async (req, res) => {
  try {
    const connection = await mysql.createConnection(dbConfig);
    const [rows] = await connection.execute('SELECT * FROM users');
    await connection.end();
    res.json(rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```
![Image1](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-09-38.png)
![Image2](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-10-34.png)
![Image3](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-11-04.png)

## Part 2: Database Setup

### 1. Install MySQL (Ubuntu/Debian Example)

```bash
sudo apt update
sudo apt install mysql-server
```

### 2. Secure MySQL Installation

```bash
sudo mysql_secure_installation
```

### 3. Create Database, User, Table, and Sample Data

```bash
sudo mysql -u root -p
```
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-13-12.png)

```sql
CREATE DATABASE practice_app;
CREATE USER 'poridhi_user'@'localhost' IDENTIFIED BY 'tcl@12345';
GRANT ALL PRIVILEGES ON practice_app.* TO 'poridhi_user'@'localhost';
FLUSH PRIVILEGES;
USE practice_app;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL
);

INSERT INTO users (name, email) VALUES
  ('John Doe', 'john@example.com'),
  ('Jane Smith', 'jane@example.com'),
  ('Bob Johnson', 'bob@example.com');

EXIT;
```
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-14-12.png)
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-16-00.png)



## Part 3: systemd Configuration

### 1. Create Dedicated System User

```bash
sudo adduser --system --group --no-create-home nodeapp
```

### 2. Set Up Application Directory

```bash
sudo mkdir -p /opt/node-mysql-app
sudo cp -r ./* /opt/node-mysql-app/
sudo chown -R nodeapp:nodeapp /opt/node-mysql-app
sudo chmod -R 755 /opt/node-mysql-app
```

### 3. Create systemd Service File

```bash
sudo nano /etc/systemd/system/node-mysql-app.service
```

```ini
[Unit]
Description=Node.js MySQL Application
After=network.target mysql.service

[Service]
ExecStart=/usr/bin/node /opt/node-mysql-app/app.js
WorkingDirectory=/opt/node-mysql-app
Restart=always
RestartSec=5s
User=nodeapp
Group=nodeapp
Environment=NODE_ENV=production
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-16-42.png)

### 4. Enable and Reload systemd

```bash
sudo systemctl daemon-reload
sudo systemctl enable node-mysql-app.service
```

## Part 4: Testing

### 1. Start Service and Verify

```bash
sudo systemctl start node-mysql-app.service
sudo systemctl status node-mysql-app.service
```
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-17-23.png)

### 2. Test Endpoints

```bash
curl http://localhost:3000/health
curl http://localhost:3000/users
```
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-18-58.png)
![Description of Image](https://github.com/cloudybdone/Deploy_Nodejs_MySQL_App/blob/main/Screenshot%20from%202025-03-30%2010-19-11.png)


### 3. Test Crash Recovery

```bash
# Find process ID
ps aux | grep node
# Kill the process
sudo kill -9 <PID>
# Check if it restarts
sudo systemctl status node-mysql-app.service
```

### 4. Reboot and Verify

```bash
sudo reboot
# After reboot
sudo systemctl status node-mysql-app.service
curl http://localhost:3000/health
```

