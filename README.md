# authenticion-sysytem-frontend
// Step 1: Set up the project environment
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const mongoose = require('mongoose');
const app = express();

// Step 2: Design the database schema
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  password: String,
  accessLevel: { type: String, default: 'pending' }
});

const User = mongoose.model('User', userSchema);

// Step 3: Create the user signup functionality
app.post('/signup', async (req, res) => {
  const { name, email, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = new User({ name, email, password: hashedPassword });
  await newUser.save();
  res.status(201).json({ message: 'User created' });
});

// Step 4: Create the admin dashboard
app.get('/admin', async (req, res) => {
  const users = await User.find();
  res.status(200).json({ users });
});

app.put('/admin/:userId', async (req, res) => {
  const { name, email, password, accessLevel } = req.body;
  const userId = req.params.userId;
  const user = await User.findById(userId);
  if (!user) return res.status(404).json({ message: 'User not found' });
  user.name = name || user.name;
  user.email = email || user.email;
  if (password) {
    const hashedPassword = await bcrypt.hash(password, 10);
    user.password = hashedPassword;
  }
  user.accessLevel = accessLevel || user.accessLevel;
  await user.save();
  res.status(200).json({ message:
