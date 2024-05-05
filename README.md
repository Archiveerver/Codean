// Import necessary modules
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const jwt = require('jsonwebtoken');

// Connect to MongoDB database
mongoose.connect('mongodb://localhost/anvil', { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));
db.once('open', () => {
    console.log('Connected to MongoDB database');
});

// Define User schema
const UserSchema = new mongoose.Schema({
    username: String,
    password: String
});

// Create User model
const User = mongoose.model('User', UserSchema);

// Initialize Express app
const app = express();
app.use(bodyParser.json());

// Secret key for JWT
const secretKey = 'your_secret_key';

// Create endpoint for user login
app.post('/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        const user = await User.findOne({ username });

        // Check if user exists and password is correct
        if (!user || user.password !== password) {
            return res.status(401).json({ error: 'Invalid username or password' });
        }

        // Generate JWT token
        const token = jwt.sign({ username: user.username }, secretKey);
        res.status(200).json({ token });
    } catch (error) {
        res.status(500).json({ error: 'Internal server error' });
    }
});

// Middleware function to verify JWT token
function verifyToken(req, res, next) {
    const token = req.headers.authorization;
    if (!token) {
        return res.status(401).json({ error: 'Unauthorized' });
    }

    jwt.verify(token, secretKey, (err, decoded) => {
        if (err) {
            return res.status(403).json({ error: 'Forbidden' });
        }
        req.user = decoded;
        next();
    });
}

// Example protected route
app.get('/protected', verifyToken, (req, res) => {
    res.status(200).json({ message: 'You are authorized' });
});

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
