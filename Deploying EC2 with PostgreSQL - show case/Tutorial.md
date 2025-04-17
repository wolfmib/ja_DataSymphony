## 🚀 EC2 PostgreSQL + Python Flask Setup

### ✅ PostgreSQL on Amazon Linux 2023

**Check PostgreSQL service:**
```bash
sudo systemctl status postgresql-15
```

### ✅ PostgreSQL Setup (psql prompt)

1. 🔧 **Create a user:**
```sql
CREATE USER hello_user WITH PASSWORD 'hello_123!';
```

2. 📦 **Create the database:**
```sql
CREATE DATABASE hello_one OWNER hello_user;
```

3. 🛡️ **Grant DB privileges:**
```sql
GRANT ALL PRIVILEGES ON DATABASE hello_one TO hello_user;
```

4. 🚪 **Connect to the DB:**
```sql
\c hello_one
```

5. 🧱 **Create a sample table:**
```sql
CREATE TABLE test_data (
    id SERIAL PRIMARY KEY,
    name TEXT,
    score INT,
    created_at TIMESTAMP DEFAULT NOW(),
    is_active BOOLEAN
);
```

6. 🔄 **Insert sample data:**
```sql
INSERT INTO test_data (name, score, is_active)
SELECT 'User_' || g, (RANDOM() * 100)::INT, (g % 2 = 0)
FROM generate_series(1, 20) AS g;
```

7. 🧪 **Check sample rows:**
```sql
SELECT * FROM test_data LIMIT 10;
```

---

### ✅ Fix Table Ownership (if needed)

If table was created by `postgres`, grant rights to `hello_user`:
```sql
GRANT ALL PRIVILEGES ON TABLE test_data TO hello_user;
```

---

### 🐍 Python Flask Server (EC2 API Endpoint)

1. 📦 **Install dependencies:**
```bash
pip3 install flask psycopg2-binary
```

2. 🧠 **Create `server.py`:**
```python
from flask import Flask, request, jsonify
import psycopg2

app = Flask(__name__)

DB_CONFIG = {
    "dbname": "hello_one",
    "user": "hello_user",
    "password": "hello_123!",
    "host": "localhost",
    "port": 5432
}

API_TOKEN = "api_token_123!"

@app.route("/get_hello_db", methods=["GET"])
def get_hello_db():
    token = request.args.get("token")
    if token != API_TOKEN:
        return jsonify({"error": "Unauthorized"}), 401

    try:
        conn = psycopg2.connect(**DB_CONFIG)
        cur = conn.cursor()
        cur.execute("SELECT * FROM test_data ORDER BY id DESC LIMIT 10")
        rows = cur.fetchall()
        colnames = [desc[0] for desc in cur.description]
        result = [dict(zip(colnames, row)) for row in rows]
        cur.close()
        conn.close()
        return jsonify(result)
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

3. ▶️ **Run the server:**
```bash
python3 server.py
```

4. 🌐 **Open port 5000 in EC2 Security Group:**
- Type: Custom TCP Rule
- Port Range: 5000
- Source: 0.0.0.0/0 *(or specific IP)*

5. 🧪 **Test your endpoint:**
```bash
curl "http://<YOUR_EC2_PUBLIC_IP>:5000/get_hello_db?token=api_token_123!"
```

---

### 🧠 Optional Enhancements
- Use `X-API-Key` in headers instead of query param
- Add HTTPS via Nginx + certbot
- Dockerize for production
- Add logging / audit table to track access

---

🧨 **Demo proof date**: April 16, 2025  
📹 Demo video hosted on: Google Drive

