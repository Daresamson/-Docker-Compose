## **Docker Compose for Web Application and Database Deployment**
This project demonstrates how to use **Docker Compose** to deploy a **Flask web application** with a **MySQL database**. It includes database setup, networking, logging, and cleanup steps.  

---

### **Project Structure**
```
Assignment/
│── docker-compose.yml
│── web/
│   │── Dockerfile
│   │── app.py
│   │── requirements.txt
│── db/
│   │── init.sql
```

---
![Screenshot (72)](https://github.com/user-attachments/assets/d56663a8-c2cc-484f-8051-d4c2e2bc86f3)



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

![Screenshot (73)](https://github.com/user-attachments/assets/1be643a1-b943-4401-8692-22e582264427)



## **2. Web Application Setup**

![Screenshot (73)](https://github.com/user-attachments/assets/9fc5f8c2-9702-426f-b5a2-8808e7cfbfa2)


### **Dockerfile (`web/Dockerfile`)**

![Screenshot (73)](https://github.com/user-attachments/assets/0adbc9a6-9acf-45dd-918e-16c719f4e204)


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
![Screenshot (75)](https://github.com/user-attachments/assets/64675ab0-cec7-4daf-85e3-e0a650bf76cd)


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

![Screenshot (74)](https://github.com/user-attachments/assets/54b6a561-7532-44e9-bfe4-d8ef3f7fab46)


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

![Screenshot (75)](https://github.com/user-attachments/assets/34f954a8-b904-4bef-bc22-d3e0567fdde1)

### **Start the Containers**

![Screenshot (78)](https://github.com/user-attachments/assets/15d92071-d56e-40c5-9d6a-1387a09c0647)

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
![Screenshot (80)](https://github.com/user-attachments/assets/c33c84e9-895d-4ff5-aa9c-86105f68ce6c)


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

![Screenshot (81)](https://github.com/user-attachments/assets/e0b228b6-c64c-477a-a736-864cbb2436e4)


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

