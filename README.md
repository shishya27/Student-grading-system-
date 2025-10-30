# Student-grading-system-
1. Install Node.js

If not installed —
🔹 Download and install from 👉 https://nodejs.org


---

📁 2. Create Project Folder

Open terminal / command prompt and run:

mkdir student-app
cd student-app


---

📦 3. Initialize Node Project

npm init -y


---

⚙️ 4. Install Required Packages

npm install express express-session bcrypt better-sqlite3


---

📂 5. Create Files and Folders

Make this structure:

student-app/
│
├── server.js
├── data.sqlite   (will be auto-created)
└── lib/
    ├── db.js
    └── grader.js


---

🧱 6. Add Code

✅ Copy your given code into these files:

lib/db.js

const Database = require('better-sqlite3');
const db = new Database('data.sqlite');
module.exports = db;

lib/grader.js

function calculateGrade(marks) {
  if (marks >= 90) return 'A+';
  if (marks >= 80) return 'A';
  if (marks >= 70) return 'B+';
  if (marks >= 60) return 'B';
  if (marks >= 50) return 'C';
  return 'F';
}
module.exports = { calculateGrade };

server.js

const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');
const db = require('./lib/db');

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(session({ secret: 'secret', resave: false, saveUninitialized: true }));

// Login route
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = db.prepare('SELECT * FROM users WHERE username = ?').get(username);
  if (user && await bcrypt.compare(password, user.password_hash)) {
    req.session.user = { id: user.id, username: user.username, role: user.role };
    return res.send(`Login successful! Welcome ${user.username} (${user.role})`);
  }
  res.send('Invalid username or password');
});

app.listen(3000, () => console.log('✅ Server running on http://localhost:3000'));


---

🗄️ 7. Create Database and Tables

Open Node REPL or a small script:

node

Then paste:

const db = require('./lib/db');
db.exec(`
CREATE TABLE IF NOT EXISTS users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username TEXT NOT NULL UNIQUE,
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL CHECK(role IN ('admin','faculty','student')),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS students (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  marks INTEGER DEFAULT 0,
  grade TEXT DEFAULT 'F'
);
`);
console.log('Tables created!');
.exit


---

👤 8. Insert Sample User

Run:

node

Then:

const bcrypt = require('bcrypt');
const db = require('./lib/db');
(async () => {
  const hash = await bcrypt.hash('password123', 10);
  db.prepare('INSERT INTO users (username, email, password_hash, role) VALUES (?,?,?,?)')
    .run('admin', 'admin@example.com', hash, 'admin');
  console.log('Sample user created!');
})();

Then type .exit


---

🚀 9. Start the Server

node server.js

You’ll see:

✅ Server running on http://localhost:3000


---

🧪 10. Test Login

Use any API tool (like Postman or curl)
Send a POST request to:

http://localhost:3000/login

with body:

username=admin
password=password123

✅ Output →

Login successful! Welcome admin (admin)
