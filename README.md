## **Docker Compose for Web Application and Database Deployment**
This project demonstrates how to use **Docker Compose** to deploy a **Flask web application** with a **MySQL database**. It includes database setup, networking, logging, and cleanup steps.  

---

### **Project Structure**
```
project-root/
│── docker-compose.yml
│── web/
│   │── Dockerfile
│   │── app.py
│   │── requirements.txt
│── db/
│   │── init.sql
```

---

## **1. Docker Compose Setup (`docker-compose.yml`)**
Defines two services:  
✅ **web** - Flask application  
✅ **db** - MySQL database  

```yaml
version: '3.8'

services:
  web:
    build: ./web
    ports:
      - "5000:5000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=mysql+pymysql://user:password@db:3306/mydatabase
    volumes:
      - ./web:/app

  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydatabase
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  db_data:
```

---

## **2. Web Application Setup**

### **Dockerfile (`web/Dockerfile`)**
This file defines how to build the Flask application.

```dockerfile
# Base image
FROM python:3.9

# Set working directory
WORKDIR /app

# Copy application files
COPY . .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose port
EXPOSE 5000

# Start the application
CMD ["python", "app.py"]
```

---

### **Flask Web App (`web/app.py`)**
A simple **Flask** application that connects to MySQL.

```python
from flask import Flask, jsonify
import pymysql
import os

app = Flask(__name__)

# Database config
DB_HOST = "db"
DB_USER = "user"
DB_PASSWORD = "password"
DB_NAME = "mydatabase"

def get_db_connection():
    return pymysql.connect(
        host=DB_HOST,
        user=DB_USER,
        password=DB_PASSWORD,
        database=DB_NAME
    )

@app.route("/")
def home():
    return "Flask App Running with Docker!"

@app.route("/users")
def get_users():
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users;")
    users = cursor.fetchall()
    conn.close()
    return jsonify(users)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

---

### **Database Initialization (`db/init.sql`)**
This script creates a **users** table and inserts sample data.

```sql
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

INSERT INTO users (name) VALUES ('Alice'), ('Bob');
```

---

### **Install Dependencies (`web/requirements.txt`)**
This file lists the required Python packages.

```
flask
pymysql
```

---

## **3. Deployment Steps**
### **Start the Containers**
Run the following command to **build and start** the application:

```sh
docker-compose up -d
```

✔️ **This will:**  
- Build the Flask application  
- Start the MySQL database  
- Load the `init.sql` script  

---

## **4. Log Checking**
Check logs of the web and database containers:

```sh
docker-compose logs web
docker-compose logs db
```

---

## **5. Basic Networking Test**
Check if the **web** container can communicate with the **database**:

```sh
docker exec -it <web-container-id> ping db
```

Test MySQL connection manually:

```sh
docker exec -it <web-container-id> python
```

```python
import pymysql
conn = pymysql.connect(host="db", user="user", password="password", database="mydatabase")
print("Connected successfully!")
```

---

## **6. Application Testing**
### **Check if Flask app is running**
Open the browser and go to:

```
http://localhost:5000
```

### **Verify database connection**
Test if the app retrieves data from MySQL:

```
http://localhost:5000/users
```

Expected output:

```json
[
    [1, "Alice"],
    [2, "Bob"]
]
```

---

## **7. Cleanup**
When done, shut down the containers:

```sh
docker-compose down
```

To remove **volumes** (database data):

```sh
docker-compose down --volumes
```

---

## **Next Steps**
✅ Add **.env file** for credentials  
✅ Use **Nginx** as a reverse proxy  
✅ Deploy with **Docker Swarm or Kubernetes**  

---

### **Author**
📌 **Project by [DAMILARE SAMSON]**  
📌 **GitHub: [https://github.com/Daresamson/-Docker-Compose/edit/main/README.md]**  

---

