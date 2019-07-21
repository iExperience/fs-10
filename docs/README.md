# iXperience - Full Stack Day 10

Welcome to iXperience Full Stack 2019!

[[toc]]

## Authentication  

### Update your api

Install jsonwebtoken library / dependency to create and verify JWT tokens

```bash

npm install jsonwebtoken

```

Create the following folders and files

config/config.js
middleware/jwt-verify.js
middleware/cross-origin.js

In your config.js file add your jwt secret
file: config/config.js

```js

module.exports = {
    secret: 'whoknowsthesecret'
};

```

In your jwt-verify.js file create a middleware jwt function which checks a token before allowing access to certain routes / end points
file: middleware/jwt-verify.js

```js

let jwt = require('jsonwebtoken');
const config = require('../config/config');

let checkToken = (req, res, next) => {
  let token = req.headers['x-access-token'] || req.headers['authorization']; // Express headers are auto converted to lowercase
  if (token) {
    if (token.startsWith('Bearer ')) {
      // Remove Bearer from string
      token = token.slice(7, token.length);
    }
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

In your cross-origin.js file create a function which allows http queries from any ip address / host name with the following http headers
file: middleware/cross-origin.js

```js

// Cross-Origin Middleware
const crossOrigin = (req, res, next) => {
    res.header("Access-Control-Allow-Origin", "*"); // allow all http clients to make http requests to our api
    // allow requests with these types of http header options (important for jwt token authentication)
    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept, Authorization"); 
    next();
};

module.exports = crossOrigin;

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

            let token = jwt.sign({email: userInput.email},
                config.secret,
                { expiresIn: '24h' // expires in 24 hours
                }
              );
              // return the JWT token for the future API calls
              resolve({
                success: true,
                message: 'Authentication successful!',
                data: token
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
const jwtVerify = require('./src/middleware/jwt-verify');  // jwt token middleware 

....

//Middleware execute:
app.use(logger);
app.use(crossOrigin);
app.use(express.json());
app.use(express.urlencoded({extended : false}));

//update to following app routes to
//App routes
app.use("/api/auth", authRoutes);
app.use("/api/users", middleware.checkToken, userRoutes);

```

To register users successfully using the JWT token as a response, update your register function in the services/auth-service.js file 

```js

...
register(user) {
    let email = user.email;
    // return promise (asynchronous function method)
    // https://developers.google.com/web/fundamentals/primers/promises
    return new Promise((resolve, reject) => { 
        userServer.createUser(user).then(userReturned => {
            let token = jwt.sign({email: email},
                config.secret,
                { expiresIn: '24h' // expires in 24 hours
                }
              );
              // return the JWT token for the future API calls
              resolve({
                success: true,
                message: 'Authentication successful!',
                data: token
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

### Create Ionic App

Create ionic app in file directory

```bash

ionic start jwt-front-end

```

Create register page, listings page, listing detail page, auth service, listings service, listing model and user model. 

```bash

ionic generate page pages/register
ionic generate page pages/users
ionic generate page pages/update-user
ionic generate service services/user
ionic generate service services/auth
ionic generate class models/user
ionic generate class models/http-response

```

Add HttpModule to app.module.ts

```ts

import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [BrowserModule, IonicModule.forRoot(), AppRoutingModule, HttpClientModule],

```

In home.page.html replace the view with the following html code

```html

<ion-header>
  <ion-toolbar class="background-header">
    <ion-title>Login Page</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="background">
  <div>
    <ion-card class="background-card">
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
        <div class="login-buttons">
          <ion-button size="medium" (click)="login()">Login</ion-button>
          <ion-button size="medium" (click)="register()">Register</ion-button>
          <ion-button size="medium" (click)="populateLogin()">Populate Login</ion-button>
        </div>
      </ion-card-content>
    </ion-card>
    <ion-card class="background-card">
      <ion-card-header>
        <ion-card-title>Refresh Token to Empty and Go to Users Page</ion-card-title>
      </ion-card-header>
      <div class="user-button">
        <ion-button size="medium" (click)="navToUsers()">Go to Users Page</ion-button>
      </div>
    </ion-card>
  </div>
</ion-content>

```

Add the following css code to home.page.scss

```css

.user-button {
    margin-left: 1%;
    margin-bottom: 10px;
  }

  ion-content.background{
    --background: url(https://www.everysteph.com/wp-content/uploads/2019/01/things-to-do-in-lisbon-featured-1440x1000.jpg) 0 0/100% 100% no-repeat;
  }

  ion-card.background-card{
    background-color: beige;
  }

  .background-header{
    --background: #909688;
  }

  .login-buttons {
    margin-top: 10px;
  }

  ```

 Ensure that your home.page.ts file looks like the section below

  ```ts

import { Component } from '@angular/core';
import { AuthenticationService } from '../../services/auth.service';
import { NavController } from '@ionic/angular';

@Component({
selector: 'app-home',
templateUrl: 'home.page.html',
styleUrls: ['home.page.scss'],
})
export class HomePage {

  email: string;
  password: string;

  constructor(private authenticationService: AuthenticationService, private navController: NavController) {}

  login() {
    this.authenticationService.login(this.email, this.password);
  }

  navToUsers() {
    this.authenticationService.setTokenToEmpty();
    this.navController.navigateForward('users');
  }

  register() {
    this.navController.navigateForward('register');
  }

  populateLogin() {
    this.email = "jason.energetic@gmail.com";
    this.password = "jenergetic";
  }

}

```

Ensure that your services/authentication.service.ts file looks like the section below

```ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http'; // module to perform http request
import { NavController } from '@ionic/angular';
import { HttpResponse } from '../models/http-response'; // class which specifies what our response from the api should look like

@Injectable({
  providedIn: 'root'
})
export class AuthenticationService {

  private token: string; // declare our token variable that will hold our token while using the app

  constructor(private http: HttpClient, private navController: NavController) { }

  login(email, password) {
    this.http.post('http://localhost:5000/api/auth/login', {email: email, password: password}).subscribe((response: HttpResponse) => {
      if (response.success) { // successful http request, same format as HttpResponse model / class
        this.token = response.data; // set our token to the data in the response object
        this.navController.navigateForward('users'); // navigate to the users page
      }
      else {
        alert(response.message); // display an alert if response has an error 
      }
      console.log(response);
    });
  }

  register(user) {
    this.http.post('http://localhost:5000/api/auth/register', user).subscribe((response: HttpResponse) => {
      if (response.success) { // successful http request, same format as HttpResponse model / class
        this.token = response.data; // set our token to the data in the response object
        this.navController.navigateForward('users'); // navigate to the users page
      }
      else {
        alert(response.message); // display an alert if response has an error 
      }
      console.log(response);
    });
  }

  getToken(): string { // get our token from another service or component in our app
    return this.token
  }

  setTokenToEmpty() {
    this.token = ""; // reset the token to an empty string
  }

}

```

Ensure that your services user.service.ts has the following code to ensure that can access the user data from the api.

```ts

import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http'; // module to perform http request and set http headers
import { AuthenticationService } from './auth.service'; // authentication service which holds our token
import { User } from '../models/user'; // user model
import { NavController } from '@ionic/angular';
import { HttpResponse } from '../models/http-response'; // class which specifies what our response from the api should look like

@Injectable({
  providedIn: 'root'
})
export class UserService {

  private userToUpdate: User; // user the our update user page will update
  private callBack: Function; // callback function (deleteUser) that we can call from any page which imports this service 

  httpOptions: { // declare our http options - used to create our http headers which will store our token
    headers: HttpHeaders
  };

  constructor(private http: HttpClient, private authenticationService: AuthenticationService, private navController: NavController) {
    
  }

  getUsers(): any { 
    this.httpOptions = {
      headers: new HttpHeaders({
        'Authorization': 'Bearer ' + this.authenticationService.getToken() // get our token from the authentication service and add it the http headers
      })
    };
    return this.http.get('http://localhost:5000/api/users', this.httpOptions); // this returns an observable object which allows which ever function that calls get users to receive the http request for get users 
  }

  setUserToUpdate(user: User) {
    this.userToUpdate = user; // set the user that we should update in our update user page
  }

  getUserToUpdate(): User {
    return this.userToUpdate; // get the user that we should update in our update user page
  }

  setCallBack(callback: Function) {
    this.callBack = callback; // set our callback function (deleteUser)
  }

  invokeCallBack(user) {
    this.callBack(user); // call our callback function (deleteUser) from another page or service
  }

  deleteUser(user: User) {
    this.httpOptions = {
      headers: new HttpHeaders({
        'Authorization': 'Bearer ' + this.authenticationService.getToken() // get our token from the authentication service and add it the http headers
      })
    };
    this.http.post('http://localhost:5000/api/users/delete/' + user.id, {userId: user.id}, this.httpOptions).subscribe((response: HttpResponse) => {
      if (response.success) { // successful http request, same format as HttpResponse model / class
        this.navController.navigateForward('users'); // navigate to the users page
      }
      else {
        alert(response.message); // display an alert if response has an error 
      }
      console.log(response);
    });
  }

  updateUser(user: User) {
    this.httpOptions = {
      headers: new HttpHeaders({
        'Authorization': 'Bearer ' + this.authenticationService.getToken() // get our token from the authentication service and add it the http headers
      })
    };
    this.http.post('http://localhost:5000/api/users/update', user ,this.httpOptions).subscribe((response: HttpResponse) => {
      if (response.success) { // successful http request, same format as HttpResponse model / class
        this.navController.navigateForward('users'); // navigate to the users page
      }
      else {
        alert(response.message); // display an alert if response has an error 
      }
      console.log(response);
    });
  }

}

```

users.page.ts

```ts

import { Component, OnInit } from '@angular/core';
import { UserService } from '../../services/user.service'; // user service which has our user http requests
import { User } from '../../models/user'; // user model
import { HttpResponse } from '../../models/http-response'; // class to tell our http request what the response object should look like
import { NavController } from '@ionic/angular';

@Component({
  selector: 'app-users',
  templateUrl: './users.page.html',
  styleUrls: ['./users.page.scss'],
})
export class UsersPage implements OnInit {

  users: User[]; // local variable - array of user objects

  deleteUser = (userToDelete) => { // call back to update local list of users on the frontend / client. Otherwise our database would update and our list on this page would still show the user
    this.users = this.users.filter(user => user.id !== userToDelete.id);
  }

  constructor(private userService: UserService, private navController: NavController) { 
    this.userService.setCallBack(this.deleteUser); // send call back function (deleteUser) to userService to allow us to call in from another page
  }

  ngOnInit() {
    this.userService.getUsers().subscribe((response: HttpResponse) => { // get users from api when page loads
      console.log(response);
      if (response.success) { 
        this.users = response.data; // if we get data successfully from our api we set our user local variable / object to the data response from the api
      }
      else {
        alert(response.message); // display error from our http request to the api
      }
    });;
  }

  navToUserUpdate(user: User) {
    this.userService.setUserToUpdate(user); // set user to update in UserService - we will read this in our update user page
    this.navController.navigateForward('update-user') // navigate to our user update page
  }

}

```

users.page.html

```html

<ion-header>
  <ion-toolbar class="background-header">
    <ion-title>
      Users
    </ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="background">
  <div>
    <ion-card *ngFor="let user of users" (click)="navToUserUpdate(user)" class="background-card">
      <ion-card-header>
        <ion-grid>
          <ion-row>
            <ion-col push-md align-self: center>
              <ion-card-subtitle>{{user.name}}</ion-card-subtitle>
              <ion-card-title>{{user.surname}}</ion-card-title>
              <ion-card-title>{{user.email}}</ion-card-title>
            </ion-col>
            <ion-col align-self: center>
              <ion-card-title>{{user.role}}</ion-card-title>
            </ion-col>
          </ion-row>
        </ion-grid>

      </ion-card-header>
      <ion-card-content>
        Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's
        standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make
        a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting,
        remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing
        Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions
        of Lorem Ipsum.
      </ion-card-content>
    </ion-card>
  </div>
</ion-content>

```
users.page.scss

```css

.welcome-card ion-img {
  max-height: 35vh;
  overflow: hidden;
}


.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 50%;
}

.blackText {
  color: black;
}

ion-col {
  text-align: center;
}

ion-content.background{
  --background: url(https://www.everysteph.com/wp-content/uploads/2019/01/things-to-do-in-lisbon-featured-1440x1000.jpg) 0 0/100% 100% no-repeat;
}

ion-card.background-card{
  background-color: beige;
}

.background-header{
  --background: #909688;
}

```

update-user.page.ts

```ts

import { Component, OnInit } from '@angular/core';
import { UserService } from '../../services/user.service'; // user service which has our user http requests
import { User } from '../../models/user'; // user model

@Component({
  selector: 'app-update-user',
  templateUrl: './update-user.page.html',
  styleUrls: ['./update-user.page.scss'],
})
export class UpdateUserPage implements OnInit {

  constructor(private userService: UserService) { }

  user: User = new User();;

  ngOnInit() { // function runs when page loads
    this.user = this.userService.getUserToUpdate(); // get user from UserService that we want to update
  }

  update() {
    this.userService.updateUser(this.user); // call function from UserService to perform api http request to update user 
  }

  delete() {
    this.userService.invokeCallBack(this.user); // run callback function in our users page to delete user from the list (note: frontend only) 
    this.userService.deleteUser(this.user); // call function from UserService to perform api http request to delete user 
  }

}

```

update-user.page.html

```html

<ion-header>
  <ion-toolbar class="background-header">
    <ion-title>Update User</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="background">
  <ion-card class="background-card">
      <ion-card-header>
        <ion-card-title>User Information</ion-card-title>
      </ion-card-header>
      <ion-card-content>
        <ion-item>
          <ion-input type="text" placeholder="First Name" [(ngModel)] = "user.name" ></ion-input>
        </ion-item>
        <ion-item>
          <ion-input type="text" placeholder="Last Name" [(ngModel)] = "user.surname"></ion-input>
        </ion-item>
        <ion-item>
            <ion-input type="text" placeholder="Choose an Email" [(ngModel)] = "user.email"></ion-input>
          </ion-item>
          <ion-item>
            <ion-input type="text" placeholder="Choose a Password" type="password" [(ngModel)] = "user.password"></ion-input>
          </ion-item>
        <div>
              <ion-button size="medium" (click)="update()">Update</ion-button>
              <ion-button size="medium" (click)="delete()">Delete</ion-button>
        </div>
    </ion-card-content>

    </ion-card>

</ion-content>

```
update-user.page.scss

```css

ion-content.background{
    --background: url(https://www.everysteph.com/wp-content/uploads/2019/01/things-to-do-in-lisbon-featured-1440x1000.jpg) 0 0/100% 100% no-repeat;
}

ion-card.background-card{
    background-color: beige;
}

.background-header{
    --background: #909688;
}

```