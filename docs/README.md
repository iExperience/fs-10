# iXperience - Full Stack Day 10

Welcome to iXperience Full Stack 2019!

[[toc]]

## Authentication  

### Create Ionic App

Create ionic app in file directory

```bash

ionic start jwt-front-end

```

Create register page, listings page, listing detail page, auth service, listings service, listing model and user model. 

```bash

ionic generate page register
ionic generate page listings
ionic generate page listing-detail
ionic generate service services/listing
ionic generate service services/auth
ionic generate class models/listing
ionic generate class models/user

```

Add HttpModule to app.module.ts

```ts

import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [BrowserModule, IonicModule.forRoot(), AppRoutingModule, HttpClientModule],

```

In home.page.html replace the view with the following html code

```ts

<ion-header>
  <ion-toolbar>
    <ion-title>Login Page</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content>
    <ion-card>
      <ion-card-header>
       <ion-card-title>Login Information</ion-card-title>
      </ion-card-header>
      <ion-card-content>
        <ion-item>
          <ion-input type="text" placeholder="Email" [(ngModel)] = "email"></ion-input>
        </ion-item>
        <ion-item>
          <ion-input type="text" placeholder="Password" type="password" [(ngModel)] = "password" ></ion-input>
        </ion-item>
        <div>
          <ul>
              <li><ion-button size="medium" expand="block" (click)="login()">Login</ion-button></li>
              <li><ion-button size="medium" expand="block" (click)="register()">Register</ion-button></li>
          </ul>
        </div>
      </ion-card-content>
    </ion-card>
</ion-content>

```

Add the following css code to home.page.scss

```css

ul {
    list-style-type: none;
    margin: 0;
    padding: 0;
    overflow: hidden;
    background-color:white;
  }
  
  li {
    float: left;
  }
  
  li a {
    display: block;
    color: white;
    text-align: center;
    padding: 16px;
    text-decoration: none;
  }
  
  li a:hover {
    background-color: #111111;
  }

  ```

 Ensure that your home.page.ts file looks like the section below

  ```ts

import { Component } from '@angular/core';
import { AuthenticationService } from '../services/auth.service';
import { User } from '../models/user';

@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss'],
})
export class HomePage {

  email: string;
  password: string;

  constructor(private authenticationService: AuthenticationService) {}

  login() {
    this.authenticationService.login(this.email, this.password);
  }

}

```

Ensure that your services/authentication.service.ts file looks like the section below

```ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { User } from '../models/user';

@Injectable({
  providedIn: 'root'
})
export class AuthenticationService {

  constructor(private http: HttpClient) { }

  login(email, password) {
    this.http.post('http://localhost:5000/api/auth/login', {email: email, password: password}).subscribe((response) => {
      console.log(response);
    });
  }

}

```

### Update your api

Install jsonwebtoken library / dependency to create and verify JWT tokens

```bash

npm install jsonwebtoken

```

Create the following folders and files

config/config.js
middle/middleware.js

In your config.js file add your jwt secret
file: config/config.js

```js

module.exports = {
    secret: 'whoknowsthesecret'
};

```

In your middleware.js file create a middleware jwt function which checks a token before allowing access to certain routes / end points
file: middleware/middleware.js

```js

let jwt = require('jsonwebtoken');
const config = require('../config/config');

let checkToken = (req, res, next) => {
  let token = req.headers['x-access-token'] || req.headers['authorization']; // Express headers are auto converted to lowercase
  if (token.startsWith('Bearer ')) {
    // Remove Bearer from string
    token = token.slice(7, token.length);
  }

  if (token) {
    jwt.verify(token, config.secret, (err, decoded) => {
      if (err) {
        return res.json({
          success: false,
          message: 'Token is not valid'
        });
      } else {
        req.decoded = decoded;
        next();
      }
    });
  } else {
    return res.json({
      success: false,
      message: 'Auth token is not supplied'
    });
  }
};

module.exports = {
  checkToken: checkToken
}

```

In your services/auth-service.js file update your login function to create a JWT token when successfully finding a user in the database

```js

const UserService = require('./user-service');
const User = require("../models/user");
const userServer = new UserService();
const jwt = require('jsonwebtoken');
const config = require('../config/config');

module.exports = class AuthService {
    constructor() {}

// with services we need asynchronous functions due to the nature of JavaScript runtime environment
// Look at JavaScript concurrency model for more information

login(userInput) {
  // return promise (asynchronous function method)
  // https://developers.google.com/web/fundamentals/primers/promises
  return new Promise((resolve, reject) => {        
    User.findUserByEmailAndPassword([userInput.email, userInput.password], (err, res) => {
        if (err) {
            reject(err);
        }
        else if (res.length > 0) { // database returns a user or an array larger than length 0

            let token = jwt.sign({username: userInput.name},
                config.secret,
                { expiresIn: '24h' // expires in 24 hours
                }
              );
              // return the JWT token for the future API calls
              resolve({
                success: true,
                message: 'Authentication successful!',
                token: token
              });
        }
        else {
            reject("user does not exist");
        }
    });
});

```

Update your auth-routes file to return a JWT token in a structure which is more easily interpreted by the front-end / client
file: routes/auth-routes.js

```js

...
//login
router.post('/login', (req,res) => {
    // asynchronous function call structure 
    authServe.login(req.body).then(token => {
        res.json(token);
    }).catch(err => {
        res.json({
            success: false,
            message: err});
    });
});

```

Add your middleware JWT token verification to your index.js file

```js

...
const middleware = require('./src/middleware/middleware');

....

//update to following app routes to
//App routes
app.use("/api/auth", authRoutes);
app.use("/api/users", middleware.checkToken, userRoutes);

```

To register users successfully using the JWT token as a response, update your register function in the services/auth-service.js file 

```js

...
register(user) {
    let userName = user.name;
    // return promise (asynchronous function method)
    // https://developers.google.com/web/fundamentals/primers/promises
    return new Promise((resolve, reject) => { 
        userServer.createUser(user).then(userReturned => {
            let token = jwt.sign({username: userName},
                config.secret,
                { expiresIn: '24h' // expires in 24 hours
                }
              );
              // return the JWT token for the future API calls
              resolve({
                success: true,
                message: 'Authentication successful!',
                token: token
              });
        }).catch(err => {
            reject(err); // reject error in promise
        });
    });
}

```

Update the routes/auth-routes.js file to a structure which is more easily interpreted by the front-end / client

```js

...
//register
router.post('/register', (req,res) => {
    // asynchronous function call structure 
    authServe.register(req.body).then(user => {
        res.json(user);
    }).catch(err => {
        res.json({
            success: false,
            message: err});
    });
});

```