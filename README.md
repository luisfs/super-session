# Super Session
The best of autentication with token and session on back-end in same module. Made with love to NodeJS.

## Installation

```bash
$ npm install --save super-session
```

## Guide
* [Quick usage](#quick-usage)
* [Why use](#why-use)
* [Configure](#configure)
* [Decode](#decode)
* [Create session](#create-session)
* [Delete user sessions](#delete-user-sessions)
* [Logout](#logout)
* [Tests](#tests)
* [Related projects](#related-projects)
* [License](#license)

## Quick usage

### Back-end
```javascript
const express = require('express');
const { Router } = require('express');
const superSession = require('super-session').superSession;

const app = express();
const router = Router();

// Set to use super session
this.app.use(superSession.decode());

// Create the options object
const superSessionOptions = {
    // Connection is optional, without connection the session will be saved on cache
    connection: {
        dbUrl: 'your mongo connection', // Necessary
        dbName: 'your db name like test or production' // Necessary
    },
    secret: 'your secret', // Default is get miliseconds since 1970
    tokenHeaderName: 'x-access-token',
    duration: 15,
    mult: true,
    reqAttribute: 'session',
    collectionName: 'xsessions'
};

// Configure the super session
superSession.configure(superSessionOptions, () => {
    createRoutes();
});

function createRoutes() {
    // The routes of your app

    // Get users
    router.get('/users', function (req, res) {
        // If user is logged, return users data
        if (req.session) {
            // Now you can access all user data
            return res.json([{ name: 'Thor', email: 'thor@asgard.com' }]);
        }
        res.status(401).json({}); // User is not logged
    });

    // Login
    router.get('/users/login', function (req, res) {
        // The role to user make login, check password...
        // ...

        // If login it's ok, create the user session
        // Is necessary use a unique identifier to create the session, like _id or email
        // Any unique identifier

        // The session data, put anything
        const sessionData = { 
            _id: '5c0fa99d2c75fb077adbb8ec', 
            name: 'Thor', 
            email: 'thor@asgard.com', 
            permissions: ['list-users', 'all'] 
        };

        // Creating the session using the user._id
        superSession.createSession(sessionData._id, sessionData)
            .then((token) => {
                return res.json({ userToken: token, userData: sessionData, logged: true });
            });
    });

    // Logout
    router.get('/users/logout', function (req, res) {
        // If user is logged, make logout
        if (req.user) {
            req.session.logout().then(() => {
                return res.json({ logged: false });
            });
        }
        else {
            res.status(401).json({}); // User is not logged
        }
    });
}
```

### Front-end
We will need request to the back-end and get the response, like this. Just example using angular HTTP client
```javascript
http.post(`${API}/users/login`, { user: 'thor@asgard.com', password: 'loki' })
    .subscribe((res) => {
        storage.set('userToken', res.userToken); // Save the user token on storage
    });

// And last we will get the token on storage and put on headers to each request
// To do this, use the interceptor
// The header name would be equal the option tokenHeaderName on back-end
// Example
// ...
intercept(req, next) {
    return Observable.create((obs) => {
        const token = storage.get('userToken');
        if (token) {
            const cloneReq = req.clone({
                setHeaders: {
                    'x-access-token': token
                }
            });
            return obs.next(next.handle(cloneReq));
        }
        return obs.next(next.handle(req));
    });
}
``` 

## Why use
It's fast cause don't send the user data to front and save a session on cache. It's Safe cause save the session on your database.
And have few options to use and control multiples sessions or a unique session by user.

## Configure
Configure the super session, will need configure in express file
```javascript
// Set to use super session
this.app.use(superSession.decode());

// Create the options object
const superSessionOptions = {
    connection: {
        dbUrl: 'your mongo connection',
        dbName: 'your db name like test or production'
    },
    secret: 'your secret',
    tokenHeaderName: 'x-access-token',
    duration: 15,
    mult: true,
    reqAttribute: 'session',
    collectionName: 'xsessions'
};

// Configure the super session
superSession.configure(superSessionOptions, () => {
    // Continue your server configuration
    // ...
});
```

## Options
Avaliable options bellow to call configure function
```javacript
{
    // Connection is optional, without connection the session will be saved on cache
    "connection": {
        "dbUrl": "your mongo connection", // Necessary
        "dbName": "your db name like test or production" // Necessary
    },

    // Optional, default is get miliseconds since 1970
    "secret": "your secret",

    // Optional, default is access-token
    "tokenHeaderName": "x-access-token",

    // Optional, default is 14
    "duration": 15,

    // Optional, default is false
    // When true, the user can log in many computers and browsers and all sessions will be active
    "mult": true, 

    // Opitional, default is session. Can be change to any word.
    // If change to user, the session data will be in req.user
    "reqAttribute": "session",

    // Optional, default is sessions. The collection name that store the sessions
    "collectionName": "xsessions"
}
```

## Decode
Set to use the super session decode on express app
```javascript
// The server need this to decode the token of user
this.app.use(superSession.decode());
```

## Create session
Is necessary use a unique identifier to create the session, like _id or email. Any unique identifier
```javascript
// The session data, put anything
const sessionData = { _id: '5c0fa99d2c75fb077adbb8ec', name: 'Thor', email: 'thor@asgard.com', permissions: ['list-users', 'all'] };

superSession.createSession(sessionData._id, sessionData)
    .then((token) => {
        console.log('userToken', token);
    });
```

## Delete user sessions
Use the session id to delete all sessions of a user. We was used the user id, then send the userId
```javascript
superSession.deleteUserSessions('5c0fa99d2c75fb077adbb8ec')
    .then(() => {
        console.log('Delete all sessions of user 5c0fa99d2c75fb077adbb8ec');
    });
```

## Logout
To user make logout, just check exist the session and call req.session.logout() (It's a promise)
```javascript
router.get('/users/logout', function (req, res) {
    // If user is logged, make logout
    if (req.user) {
        req.session.logout().then(() => {
            return res.json({ logged: false });
        });
    }
    else {
        res.status(401).json({}); // User is not logged
    }
});
``` 

## Tests
To run the test suite, first install the dependencies, then run npm run test:

```bash
$ npm install
$ npm run test
```

## Related projects
[express-session](https://github.com/expressjs/session)

## License
[MIT](LICENSE)
