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
import { AuthenticationService } from '../services/authentication.service';
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
0
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




### Create Table

Run the following query to create a table called listing. The table has a primary key: id and fields userId, title, description, location and pricePerNight.

```sql

CREATE TABLE `fs_bnb`.`listing` (
  `id` INT(6) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `userId` INT(6) UNSIGNED NOT NULL,
  `title` VARCHAR(45) NOT NULL,
  `description` VARCHAR(45) NULL,
  `location` VARCHAR(45) NULL,
  `pricePerNight` DECIMAL(10) NOT NULL
)

```

### Insert Data into Table

Run the following query to insert data in the listing table. The insert query contains data for the fields userId, title, description, location and pricePerNight.

```sql

INSERT INTO fs_bnb.listing (userId, title, description, location, pricePerNight)
VALUES (2, "Hout Bay Town House", "Town House", "Hout Bay", "120")

```