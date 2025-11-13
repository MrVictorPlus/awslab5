# Лабораторная работа №5  
## Облачные базы данных: Amazon RDS и DynamoDB  


## 📘 Описание лабораторной работы
В данной лабораторной работе изучаются сервисы облачной платформы **Amazon Web Services (AWS)**, предназначенные для работы с базами данных:

- **Amazon RDS** — реляционные базы данных (MySQL, PostgreSQL, MariaDB и др.);  
- **Amazon DynamoDB** — нереляционные (NoSQL) базы данных.  

Целью работы является изучение принципов создания, настройки и подключения к экземплярам реляционных баз данных Amazon RDS, а также знакомство с концепцией **Read Replica** — реплик для чтения, применяемых для повышения производительности и отказоустойчивости.

---

## 🎯 Постановка задачи
1. Создать инфраструктуру AWS: VPC, подсети и группы безопасности.  
2. Развернуть экземпляр **Amazon RDS (MySQL)**.  
3. Настроить **Read Replica** и проверить синхронизацию данных.  
4. Подключиться к базе данных с **EC2** и выполнить CRUD-операции.   
5. Подготовить отчёт с пояснениями и скриншотами.

---

## ⚙️ Цель и основные этапы работы

**Цель:**  
Научиться работать с реляционными и нереляционными базами данных в облаке AWS, освоить создание, настройку, подключение и репликацию баз данных.

**Основные этапы:**
1. Подготовка VPC, подсетей и групп безопасности;  
2. Создание Subnet Group для RDS;  
3. Развёртывание MySQL в Amazon RDS;  
4. Подключение EC2 к RDS и выполнение SQL-запросов;  
5. Создание Read Replica и проверка репликации;  
6. Подключение CRUD-приложения к master и replica базам.

---

## 🧠 Теоретическая часть

**Amazon RDS (Relational Database Service)** — управляемый сервис баз данных, упрощающий настройку, эксплуатацию и масштабирование реляционных баз данных в облаке.  

**Read Replica** — копия базы данных, доступная только для чтения. Используется для распределения нагрузки на чтение и резервирования данных.  

**Subnet Group** — набор подсетей внутри VPC, где AWS размещает экземпляры RDS. Используется для указания, в каких зонах доступности (AZ) можно запускать базу данных.  

---

## 🧩 Практическая часть

### 🔹 Шаг 1. Подготовка среды (VPC, Subnets, Security Groups)

1. В консоли AWS создана VPC с именем `project-vpc`.  
2. Добавлены **2 публичные** и **2 приватные** подсети в разных **Availability Zones (AZ)**.  
3. Созданы группы безопасности:
   - **web-security-group**:  
     - Входящий трафик: HTTP (80) от всех; SSH (22) от моего IP.  
   - **db-mysql-security-group**:  
     - Входящий трафик MySQL (3306) только от `web-security-group`.  
4. В `web-security-group` добавлено исходящее правило на порт 3306 для связи с базой данных.

![](/img/1.jpg)
![](/img/2.jpg)
![](/img/3.jpg)
![](/img/4.jpg)
![](/img/5.jpg)

---

### 🔹 Шаг 2. Создание Subnet Group

**Subnet Group** — это логическая группа подсетей, в которых RDS может размещать свои экземпляры.  
Необходима, чтобы база данных могла быть доступна из нескольких зон (AZ) внутри одной VPC.

1. В разделе **Amazon RDS → Subnet Groups** создана группа:
   - Название: `project-rds-subnet-group`
   - VPC: `project-vpc`
   - Подсети: две приватные из разных AZ.

![](/img/6.jpg)
![](/img/7.jpg)

---

### 🔹 Шаг 3. Развёртывание экземпляра Amazon RDS (MySQL)

**Параметры создания базы данных:**
- Engine: **MySQL 8.0.42**
- Template: **Free tier**
- Availability: **Single-AZ**
- DB identifier: `project-rds-mysql-prod`
- Master username: `admin`
- Password: `<указан при создании>`
- Instance class: `db.t3.micro`
- Storage: 20 GB (тип gp3, autoscaling до 100 GB)
- Subnet Group: `project-rds-subnet-group`
- Public access: No
- Security Group: `db-mysql-security-group`
- Initial DB name: `project_db`
- Backups: включены
- Encryption и Auto minor upgrade: отключены

![](/img/8.jpg)
![](/img/9.jpg)
![](/img/10.jpg)
![](/img/11.jpg)
![](/img/12.jpg)
![](/img/13.jpg)
![](/img/14.jpg)
![](/img/15.jpg)
![](/img/16.jpg)


После создания статус изменился на **Available**, и был получен **Endpoint**

---

### 🔹 Шаг 4. Создание EC2 для подключения к базе данных

1. Создан экземпляр **EC2** типа `t3.micro` (Amazon Linux 2023).  
2. Подключён к **публичной подсети**.  
3. Применена группа безопасности `web-security-group`.

**Установка MySQL клиента:**
```bash
#!/bin/bash
dnf update -y
dnf install -y mariadb105
```
![](/img/17.jpg)
![](/img/18.jpg)
![](/img/19.jpg)
![](/img/20.jpg)

**Подключение к RDS:**

![](/img/21.jpg)
![](/img/22.jpg)
![](/img/23.jpg)

**Создание таблиц:**

![](/img/24.jpg)

**Добавление данных:**

![](/img/25.jpg)

**Выборка с JOIN:**

![](/img/26.jpg)

---

### 🔹 Шаг 5. Создание Read Replica

1. В RDS выбрана база `project-rds-mysql-prod`.
2. **Actions → Create read replica**.
3. Параметры:

   * Identifier: `project-rds-mysql-read-replica`
   * Class: `db.t3.micro`
   * Storage: `gp3`
   * Public access: `No`
   * Security group: `db-mysql-security-group`

Создание Read Replica

![](/img/27.jpg)
![](/img/28.jpg)
![](/img/29.jpg)
![](/img/30.jpg)
![](/img/31.jpg)

После создания реплика стала **Available**, у неё свой endpoint

**Проверка чтения:**

![](/img/32.jpg)
![](/img/33.jpg)

✅ Данные идентичны основной БД.

**Попытка записи:**

![](/img/34.jpg)

→ Ошибка: *The MySQL server is running with the --read-only option.*

**Добавление записи в master:**

![](/img/35.jpg)

Через короткое время запись появилась и на реплике

---

### 🔹 Шаг 6. Подключение приложения (CRUD)

Для демонстрации создано простое Node.js-приложение, работающее с RDS.

![](/img/36.jpg)
![](/img/37.jpg)
![](/img/38.jpg)

**.env:**

```js
Master (для записи)
DB_MASTER_HOST=<MASTER_RDS_ENDPOINT>
DB_MASTER_USER=admin
DB_MASTER_PASSWORD=<MASTER_PASSWORD>
DB_MASTER_DATABASE=project_db

# Read Replica (для чтения)
DB_READ_HOST=<READ_REPLICA_ENDPOINT>
DB_READ_USER=admin
DB_READ_PASSWORD=<MASTER_PASSWORD>
DB_READ_DATABASE=project_db
```

**server.js:**

```js
import express from "express";
import mysql from "mysql2/promise";
import path from "path";
import { fileURLToPath } from "url";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());
const PORT = 3000;

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
app.use(express.static(path.join(__dirname, "public")));

// Master (запись)
const masterPool = mysql.createPool({
  host: process.env.DB_MASTER_HOST,
  user: process.env.DB_MASTER_USER,
  password: process.env.DB_MASTER_PASSWORD,
  database: process.env.DB_MASTER_DATABASE,
  waitForConnections: true,
  connectionLimit: 5,
  queueLimit: 0
});

// Read Replica (чтение)
const readPool = mysql.createPool({
  host: process.env.DB_READ_HOST,
  user: process.env.DB_READ_USER,
  password: process.env.DB_READ_PASSWORD,
  database: process.env.DB_READ_DATABASE,
  waitForConnections: true,
  connectionLimit: 5,
  queueLimit: 0
});

// --- CRUD API ---

// Categories
app.get("/categories", async (req, res) => {
  try {
    const [rows] = await readPool.query("SELECT * FROM categories");
    res.json(rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.post("/categories", async (req, res) => {
  try {
    const { name } = req.body;
    const [result] = await masterPool.query("INSERT INTO categories (name) VALUES (?)", [name]);
    res.json({ id: result.insertId, name });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.put("/categories/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const { name } = req.body;
    await masterPool.query("UPDATE categories SET name=? WHERE id=?", [name, id]);
    res.json({ id, name });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.delete("/categories/:id", async (req, res) => {
  try {
    const { id } = req.params;
    await masterPool.query("DELETE FROM categories WHERE id=?", [id]);
    res.json({ message: "Category deleted" });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Todos
app.get("/todos", async (req, res) => {
  try {
    const [rows] = await readPool.query(`
      SELECT todos.id, todos.title, todos.status, categories.name as category
      FROM todos
      JOIN categories ON todos.category_id = categories.id
    `);
    res.json(rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.post("/todos", async (req, res) => {
  try {
    const { title, category_id, status } = req.body;
    const [result] = await masterPool.query(
      "INSERT INTO todos (title, category_id, status) VALUES (?, ?, ?)",
      [title, category_id, status]
    );
    res.json({ id: result.insertId, title, category_id, status });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.put("/todos/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const { title, category_id, status } = req.body;
    await masterPool.query(
      "UPDATE todos SET title=?, category_id=?, status=? WHERE id=?",
      [title, category_id, status, id]
    );
    res.json({ id, title, category_id, status });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.delete("/todos/:id", async (req, res) => {
  try {
    const { id } = req.params;
    await masterPool.query("DELETE FROM todos WHERE id=?", [id]);
    res.json({ message: "Todo deleted" });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// --- запуск сервера ---
app.listen(PORT, "0.0.0.0", () => {
  console.log(`Server started on port ${PORT}`);
});
```

**public/index.html:**

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>RDS CRUD App</title>
</head>
<body>
  <h1>Приложение CRUD для RDS</h1>

  <h2>Категории</h2>
  <ul id="categories"></ul>

  <input type="text" id="categoryName" placeholder="Название">
  <button onclick="addCategory()">Добавить</button>

  <h2>Задачи</h2>
  <ul id="todos"></ul>

  <input type="text" id="todoTitle" placeholder="Название задачи">
  <input type="number" id="todoCategory" placeholder="ID категории">
  <input type="text" id="todoStatus" placeholder="Статус">
  <button onclick="addTodo()">Добавить задачу</button>

  <script>
    async function fetchCategories() {
      const res = await fetch('/categories');
      const data = await res.json();
      const ul = document.getElementById('categories');
      ul.innerHTML = '';
      data.forEach(cat => {
        const li = document.createElement('li');
        li.textContent = ${cat.id}: ${cat.name};
        li.innerHTML +=  <button onclick="deleteCategory(${cat.id})">Удалить</button>;
        ul.appendChild(li);
      });
    }

    async function addCategory() {
      const name = document.getElementById('categoryName').value;
      await fetch('/categories', { 
        method: 'POST', 
        headers: {'Content-Type':'application/json'}, 
        body: JSON.stringify({name})
      });
      fetchCategories();
    }

    async function deleteCategory(id) {
      await fetch(/categories/${id}, { method: 'DELETE' });
      fetchCategories();
    }

    async function fetchTodos() {
      const res = await fetch('/todos');
      const data = await res.json();
      const ul = document.getElementById('todos');
      ul.innerHTML = '';
      data.forEach(todo => {
        const li = document.createElement('li');
        li.textContent = ${todo.id}: ${todo.title} [${todo.category}] - ${todo.status};
        li.innerHTML +=  <button onclick="deleteTodo(${todo.id})">Удалить</button>;
        ul.appendChild(li);
      });
    }

    async function addTodo() {
      const title = document.getElementById('todoTitle').value;
      const category_id = document.getElementById('todoCategory').value;
      const status = document.getElementById('todoStatus').value;
      await fetch('/todos', {
        method: 'POST',
        headers: {'Content-Type':'application/json'},
        body: JSON.stringify({title, category_id, status})
      });
      fetchTodos();
    }

    async function deleteTodo(id) {
      await fetch(/todos/${id}, { method: 'DELETE' });
      fetchTodos();
    }

    fetchCategories();
    fetchTodos();
  </script>
</body>
</html>
```

**CRUD-приложение**

![](/img/39.jpg)
![](/img/40.jpg)

---

## 🧠 Контрольные вопросы

**1. Что такое Subnet Group? И зачем необходимо создавать Subnet Group для базы данных?**
Subnet Group — это логическая группа подсетей (subnets) в пределах VPC, которые используются Amazon RDS для размещения экземпляров базы данных.
Она нужна для того, чтобы база данных могла быть развернута в разных зонах доступности (Availability Zones), обеспечивая отказоустойчивость и доступность.
Без Subnet Group RDS не сможет корректно выбрать подсети для размещения экземпляра, и база данных просто не создастся.

**2. Какие данные вы видите? Объясните почему.**
При подключении к Read Replica видны те же данные, что и в основном экземпляре (master), но с небольшой задержкой.
Это происходит потому, что реплика получает обновления от основного экземпляра асинхронно, и данные синхронизируются не мгновенно, а через короткий промежуток времени.

**3. Получилось ли выполнить запись на Read Replica? Почему?**
Нет, выполнить запись на Read Replica не получится.
Реплика предназначена только для операций чтения (SELECT).
Она не принимает запросы на изменение данных (INSERT, UPDATE, DELETE), так как все изменения происходят только на master и затем передаются на реплику.

**4. Отобразилась ли новая запись на реплике? Объясните почему.**
Да, новая запись появилась на реплике спустя короткое время после добавления в master.
Это объясняется тем, что Amazon RDS использует асинхронную репликацию — данные сначала записываются в основной экземпляр, а затем копируются на реплику с небольшой задержкой.


---

## 📚 Список использованных источников

* [Moodle](https://elearning.usm.md/mod/assign/view.php?id=320200)
* [ChatGPT](https://chatgpt.com/)
* [AWS](https://aws.amazon.com/ru/)

---

## ✅ Вывод

В ходе лабораторной работы была развернута облачная реляционная база данных Amazon RDS с использованием MySQL, настроена группа подсетей (Subnet Group) и группа безопасности для безопасного подключения. Создана и протестирована реплика для чтения (Read Replica), продемонстрирована асинхронная репликация данных между основным экземпляром и репликой.

Также выполнено подключение к базе данных с виртуальной машины EC2, созданы таблицы со связью «один ко многим» и проведены CRUD-операции.

В результате были закреплены навыки работы с облачными базами данных AWS, настройкой сетевой инфраструктуры, репликацией и взаимодействием приложений с реляционными и нереляционными хранилищами данных.


