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

# 4. Building a Simple Game with Server-Side Logic

We'll implement a rock-paper-scissors game with win tracking.

### Step 1: Set up the game logic

Add this to your `server.js`:

```javascript
// ... previous code ...

// Game state
let gameHistory = [];

// Game logic
function playRPS(playerChoice) {
  const choices = ['rock', 'paper', 'scissors'];
  const computerChoice = choices[Math.floor(Math.random() * choices.length)];
  
  let result;
  if (playerChoice === computerChoice) {
    result = 'tie';
  } else if (
    (playerChoice === 'rock' && computerChoice === 'scissors') ||
    (playerChoice === 'paper' && computerChoice === 'rock') ||
    (playerChoice === 'scissors' && computerChoice === 'paper')
  ) {
    result = 'win';
  } else {
    result = 'lose';
  }
  
  gameHistory.push({ playerChoice, computerChoice, result });
  return { playerChoice, computerChoice, result };
}

// Game routes
app.use(express.json()); // For parsing application/json

app.post('/api/play', (req, res) => {
  const { choice } = req.body;
  if (!['rock', 'paper', 'scissors'].includes(choice)) {
    return res.status(400).send('Invalid choice');
  }
  const result = playRPS(choice);
  res.json(result);
});

app.get('/api/history', (req, res) => {
  res.json(gameHistory);
});

// ... rest of the code ...
```

### Step 2: Create a simple frontend

Create a new file `public/game.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rock Paper Scissors</title>
</head>
<body>
    <h1>Rock Paper Scissors</h1>
    <button onclick="play('rock')">Rock</button>
    <button onclick="play('paper')">Paper</button>
    <button onclick="play('scissors')">Scissors</button>
    <div id="result"></div>
    <h2>Game History</h2>
    <ul id="history"></ul>

    <script>
        async function play(choice) {
            const response = await fetch('/api/play', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ choice }),
            });
            const result = await response.json();
            document.getElementById('result').innerText = `You chose ${result.playerChoice}, computer chose ${result.computerChoice}. You ${result.result}!`;
            updateHistory();
        }

        async function updateHistory() {
            const response = await fetch('/api/history');
            const history = await response.json();
            const historyList = document.getElementById('history');
            historyList.innerHTML = '';
            history.forEach(game => {
                const li = document.createElement('li');
                li.innerText = `Player: ${game.playerChoice}, Computer: ${game.computerChoice}, Result: ${game.result}`;
                historyList.appendChild(li);
            });
        }

        updateHistory();
    </script>
</body>
</html>
```

# 5. [Advanced] Responding to new Gmail messages

## Prerequisites

- Node.js and npm installed
- A Google account
- Basic knowledge of JavaScript and Node.js

## Step 1: Set Up Google Cloud Project

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project.
3. Enable the Gmail API: 
   - Go to "APIs & Services" > "Library"
   - Search for "Gmail API" and enable it
4. Create credentials:
   - Go to "APIs & Services" > "Credentials"
   - Click "Create Credentials" > "OAuth client ID"
   - Choose "Web application"
   - Set the Authorized redirect URI to `http://localhost:3000/oauth2callback`
   - Download the client configuration file

## Step 2: Set Up Node.js Project

Create a new directory and initialize the project:

```bash
mkdir gmail-integration
cd gmail-integration
npm init -y
npm install express googleapis google-auth-library dotenv
```

Create a `.env` file in the project root and add your Google Cloud credentials:

```sh
CLIENT_ID=your_client_id
CLIENT_SECRET=your_client_secret
```

## Step 3: Create the Server

Create a file named `server.js` and add the following code:

```javascript
require('dotenv').config();
const express = require('express');
const { google } = require('googleapis');
const { OAuth2Client } = require('google-auth-library');

const app = express();
const port = 3000;

const CLIENT_ID = process.env.CLIENT_ID;
const CLIENT_SECRET = process.env.CLIENT_SECRET;
const REDIRECT_URI = 'http://localhost:3000/oauth2callback';

const oauth2Client = new OAuth2Client(CLIENT_ID, CLIENT_SECRET, REDIRECT_URI);

app.get('/', (req, res) => {
  res.send('Gmail Integration App');
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

## Step 4: Implement OAuth 2.0 Authentication

Add the following routes to `server.js`:

```javascript
app.get('/auth', (req, res) => {
  const authUrl = oauth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: ['https://www.googleapis.com/auth/gmail.readonly']
  });
  res.redirect(authUrl);
});

app.get('/oauth2callback', async (req, res) => {
  const { code } = req.query;
  const { tokens } = await oauth2Client.getToken(code);
  oauth2Client.setCredentials(tokens);
  res.send('Authentication successful! You can close this window.');
});
```

## Step 5: Implement Email Fetching

Add the following function and route to `server.js`:

```javascript
async function getNewEmails() {
  const gmail = google.gmail({ version: 'v1', auth: oauth2Client });
  const res = await gmail.users.messages.list({
    userId: 'me',
    q: 'is:unread'
  });

  const messages = res.data.messages;
  if (messages && messages.length > 0) {
    for (let message of messages) {
      const email = await gmail.users.messages.get({
        userId: 'me',
        id: message.id
      });
      console.log('New email:', email.data.snippet);
    }
  } else {
    console.log('No new emails');
  }
}

app.get('/check-emails', async (req, res) => {
  await getNewEmails();
  res.send('Checked for new emails. See console for results.');
});
```

## Step 6: Run the Application

1. Start the server:
   ```
   node server.js
   ```
2. Open a browser and go to `http://localhost:3000/auth` to authenticate.
3. After authentication, visit `http://localhost:3000/check-emails` to fetch new emails.

## Extended Functionality: Email Notification

Let's add a simple email notification system that checks for new emails every minute and logs them to the console.

Add the following code to `server.js`:

```javascript
function startEmailCheck() {
  setInterval(async () => {
    console.log('Checking for new emails...');
    await getNewEmails();
  }, 60000); // Check every minute
}

app.get('/start-notification', (req, res) => {
  startEmailCheck();
  res.send('Email notification system started. Check console for updates.');
});
```

Now, when you visit `http://localhost:3000/start-notification`, the server will start checking for new emails every minute and log them to the console.

## Error Handling

Add basic error handling to your `server.js`:

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});
```

## Security Considerations

- Never commit your `.env` file or share your client ID and secret.
- In a production environment, store tokens securely (e.g., in a database).
- Use HTTPS in production to encrypt data in transit.

That's it! You now have a basic Gmail integration that can listen for new emails in your inbox. Remember to handle token expiration and refreshing in a production application.

### Gmail API Integration Overview

1. Set up a Google Cloud Project
2. Enable the Gmail API
3. Create credentials (OAuth 2.0 client ID)
4. Install the Google API client library: `npm install googleapis`

### Basic Concept

```javascript
const { google } = require('googleapis');
const gmail = google.gmail({ version: 'v1', auth: oauth2Client });

async function checkNewEmails() {
  const res = await gmail.users.messages.list({
    userId: 'me',
    q: 'is:unread',
  });

  const messages = res.data.messages;
  if (messages && messages.length) {
    // Process new messages
    for (let message of messages) {
      const email = await gmail.users.messages.get({
        userId: 'me',
        id: message.id,
      });
      // Process the email content
      // You can reply to the email here
    }
  }
}

// You would typically run this function periodically
// setInterval(checkNewEmails, 60000); // Check every minute
```

### Security Considerations
- Store API credentials securely (use environment variables)
- Implement proper OAuth 2.0 flow for user authentication
- Be mindful of API usage limits

### Hands-on Discussion:
1. What are some potential use cases for automatically responding to emails?
2. What security concerns should we consider when integrating with email services?
3. How could we implement error handling and logging for this kind of integration?

### Hands-on Exercise:
1. Implement a win streak counter
2. Add a route to reset the game history
3. Display the win/loss/tie statistics on the frontend