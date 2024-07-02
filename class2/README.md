# Class 2 - Servers and APIs (via Express.js)

## Introduction (10 minutes)

A server is a computer running a program that provides services to other computers via HTTP (Hypertext Transfer Protocol). To access those services, we use URLs (Uniform Resource Locators). 

When building a web server, we often use a web Framework like Express.js, to make our lives easier.

### Key Points:
1. Express.js is a web application framework for Node.js (e.g. Javascript)
2. It's minimal, flexible, and widely used in the industry
3. Express.js simplifies the process of building web servers and APIs

# 1. Setting up the development environment

Before we begin, ensure that you following installed:
1. Node.js (latest LTS version) (test with `node -v`)
2. npm (comes with Node.js) (test with `npm -v`)
3. A text editor (e.g., Visual Studio Code, Sublime Text) (test with `code --version`)

### Setup Steps:
1. Create a new directory for your project:
   ```sh
   mkdir express-workshop
   cd express-workshop
   ```

2. Initialize a new Node.js project:
   ```sh
   npm init -y
   ```

3. Install Express.js:
   ```sh
   npm install express
   ```

Now, you're ready to start building with Express.js, open your text editor:

```sh
code .
```

# 2. Serving Static Files (25 minutes)

Let's create a basic Express.js server that serves the HTML file we made LAST WEEK.

### Step 1: Create a basic Express server

Create a file named `server.js` with the following content:

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

Run the server:
```
node server.js
```

### Step 2: Serve an HTML file

Copy your `index.html` file from last week into a new `public` directory in this express project.

Update `server.js` to serve that `index.html` file files:

```javascript
const express = require('express'); // this imports express, so we can use it via `app`
const path = require('path'); // this imports path, so we can use it via `path.join()` to make URLs and file paths
const app = express();
const port = 3000;

app.use(express.static('public'));

app.get('/', (req, res) => {
  const filePath = path.join(__dirname, 'public', 'index.html'); // this makes a path to the index.html via: ./public/index.html
  res.sendFile(filePath); // this line sends the file to the requesting client
});

// this starts the server to listen for requests on port 3000
app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

### Hands-on Exercise:
1. Create a new HTML file in the `public` directory (e.g., `about.html`)
2. Add a new route in `server.js` to serve this file
3. Test accessing both pages through the browser

# 3. Serving Data from Server Memory

We'll create an "in-memory data store" (e.g. an array variable)

### Step 1: Create an in-memory data store

Add this to your `server.js`:

```javascript
// ... previous code ...

// In-memory data store
const USERS = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
  { id: 3, name: 'Charlie', email: 'charlie@example.com' }
];

// API routes
app.get('/api/users', (req, res) => {
  res.json(USERS);
});

app.get('/api/users/:id', (req, res) => {
  const user = USERS.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).send('User not found');
  res.json(user);
});

app.post('/api/users', (req, res) => {
  if (!req.body.name || !req.body.email) return res.status(400).send('Missing required fields');
  const user = {
    id: USERS.length + 1,
    name: req.body.name,
    email: req.body.email
  }
  USERS.push(user);
  res.json(user);
});

// ... rest of the code ...
```

### Step 2: Test the API

Use a tool like cURL, Postman, or your browser to test the API endpoints:

- `http://localhost:3000/api/users`
- `http://localhost:3000/api/users/1`

testing get:
```sh
curl http://localhost:3000/api/users
```

testing get by id:
```sh
curl http://localhost:3000/api/users/1
```

testing post:
```sh
curl -X POST http://localhost:3000/api/users -H "Content-Type: application/json" -d '{"name": "NEWUSER", "email": "newuser@example.com"}'
```