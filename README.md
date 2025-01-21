// Backend - Node.js with Express
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());

// Database Connection
mongoose.connect('mongodb://localhost/peritia', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const UserSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  role: { type: String, enum: ['Admin', 'Supervisor', 'Perito', 'Derivador', 'Contable'], required: true },
});

const User = mongoose.model('User', UserSchema);

// Routes
app.post('/register', async (req, res) => {
  const { email, password, role } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);

  try {
    const newUser = new User({ email, password: hashedPassword, role });
    await newUser.save();
    res.status(201).send('User registered successfully');
  } catch (err) {
    res.status(400).send('Error registering user');
  }
});

app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user) return res.status(404).send('User not found');

  const validPassword = await bcrypt.compare(password, user.password);
  if (!validPassword) return res.status(401).send('Invalid credentials');

  const token = jwt.sign({ id: user._id, role: user.role }, 'secret', { expiresIn: '1h' });
  res.json({ token });
});

app.get('/cases', (req, res) => {
  res.send('List of cases');
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));

// Frontend - React (Basic Setup)
// App.js
import React from 'react';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <h1>Peritia - Gestión de Casos</h1>
      </header>
      <main>
        <p>Bienvenido a Peritia</p>
        <button>Acceder</button>
      </main>
    </div>
  );
}

export default App;

// App.css
body {
  font-family: Arial, sans-serif;
  background-color: #f0f0f0;
  margin: 0;
}

.App {
  text-align: center;
}

.App-header {
  background-color: #20232a;
  padding: 20px;
  color: white;
}

button {
  padding: 10px 20px;
  background-color: #d32f2f;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
