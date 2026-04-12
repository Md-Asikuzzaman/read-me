
# 🐬 MySQL `.sql` Import Guide (Ubuntu Terminal)

A simple, reusable step-by-step guide to import a `.sql` file into MySQL using the terminal. Perfect for saving in GitHub and reusing in future projects.

---

## 📦 Prerequisites

Make sure you have:

* MySQL installed
* Terminal access (Ubuntu/Linux)
* A `.sql` file (e.g., `abc.sql`)

---

## 🚀 Step-by-Step Guide

### 1️⃣ Open Terminal

```bash
Ctrl + Alt + T
```

---

### 2️⃣ Login to MySQL

```bash
mysql -u root -p
```

* Enter your MySQL password when prompted

---

### 3️⃣ Create a Database (if not exists)

```sql
CREATE DATABASE my_database;
```

> Replace `my_database` with your desired database name

---

### 4️⃣ Use the Database

```sql
USE my_database;
```

---

### 5️⃣ Exit MySQL

```sql
exit;
```

---

### 6️⃣ Import `.sql` File

```bash
mysql -u root -p my_database < /path/to/your/file.sql
```

#### ✅ Example:

```bash
mysql -u root -p my_database < ~/Downloads/abc.sql
```

* Enter password again when prompted

---

### 7️⃣ Verify Import

```bash
mysql -u root -p
```

Then run:

```sql
USE my_database;
SHOW TABLES;
```

If tables appear → ✅ Import successful

---

## ⚡ One-Line Shortcut (Optional)

Create database + import in one go:

```bash
mysql -u root -p -e "CREATE DATABASE my_database;"
mysql -u root -p my_database < ~/Downloads/abc.sql
```

---

## 🛠️ Troubleshooting

### ❌ Access denied

* Check username/password

### ❌ Unknown database

* Make sure database is created first

### ❌ File not found

Check path:

```bash
ls ~/Downloads
```

---

## 💡 Pro Tips

* Use absolute file paths to avoid errors
* Keep `.sql` files organized (e.g., `/database/dumps/`)
* Use version control (Git) for schema files

---

## 📌 Summary

```bash
# Create DB
mysql -u root -p -e "CREATE DATABASE my_database;"

# Import SQL file
mysql -u root -p my_database < file.sql
```

---

## 🧠 Use Case

* Laravel / Node.js project setup
* Database migration restore
* Sharing DB structure via GitHub

---

## 📁 Suggested Repo Structure

```
project-root/
│
├── database/
│   └── dumps/
│       └── abc.sql
│
└── README.md
```

---

## 🙌 Done!

You can now quickly import any `.sql` file using terminal like a pro 🚀

---

> Feel free to ⭐ this repo or reuse this guide in your projects
