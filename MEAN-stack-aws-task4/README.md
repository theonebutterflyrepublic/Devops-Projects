MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

# Book Register MEAN Stack Project

## 1. Project Overview
The **Book Register** is a simple web application built using the **MEAN stack**:

- **MongoDB** – Document database (Stores book records (name, ISBN, author, pages)).  
- **Express** – Backend application framework (Handles API routes between client and database).  
- **AngularJS** – Frontend framework for dynamic views and client-side interaction.  
- **Node.js** – Runtime environment for executing JavaScript on the server.  

This project demonstrates a **full-stack web application** deployed on an **Ubuntu EC2 instance**

## 2. Prerequisites

- AWS account and EC2 instance (Ubuntu 20.04 LTS).  
- Node.js  
- MongoDB 
- npm installed.  
- Basic understanding of terminal/SSH commands.

## 3. Project Setup
   ### Step 0- Launch an ec2 instance on aws and ssh

   ### Step 1 – Update system and install Node.js
 using

 ```bash
 sudo apt update
 ```

```bash
 sudo apt upgrade
```

#add certificate
```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```


![images](images/001-install%20certificate.jpg)

NOTE: The Certificate command installs important system tools that help your server download and verify software securely and it is often run before setup scripts.


# run this command
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

NOTE: This command:

Downloads a setup script from NodeSource, Runs it as administrator, Adds Node.js 20 repository to Ubuntu, Prepares your system to install Node.js 20. So after running it, you can install Node.js 20

![images](images/002.jpg)

# next install nodejs to the latest version. 
run
```bash
sudo apt install -y nodejs
```
![images](images/003-installnjs.jpg)

verify installation

# run

```bash
nodejs -v
npm -v
```
![images](images/004-verify-njs.jpg)

### Step 2: Install Mongodb

NOTE: The version of MONGODB in the task 4 project is old so these are the updated version and run the command below accordingly

Install Required Tools by running the command below:
```bash
sudo apt update

sudo apt install -y curl gnupg ca-certificates

# Add MongoDB Official Key (Modern Way)

curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg

# Add MongoDB Repo (Jammy = Compatible with 24.04)

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```


![images](images/005-newmongodb.jpg)

# update package list
```bash
sudo apt update
```
# install mongodb

```bash
sudo apt install -y mongodb-org
```
![images](images/005-newmongodb.jpg)

# start,enable and check mongodb status

#run
```bash
sudo systemctl start mongod

sudo systemctl enable mongod

sudo systemctl status mongod
```
![images](images/007-enable%20mongod.jpg)

# Test It (If it opens then MongoDB is ready) and press exit :
```bash
mongosh
```

# install npm (node package manager)

```bash
sudo apt install -y npm
```
![images](images/009-installmpm.jpg)

# install body-parser,this package helps us process JSON files passed in requests to the server:

```bash
sudo npm install body-parser
```
![images](images/010-installbodyparser.jpg)

# Create folder named Books
```bash
mkdir Books && cd Books
```
# in the Books directory initialize npm project


```bash
npm init
```
![images](images/011-mkdirbnooks.jpg)

# create a server.js file in the Books directory
```bash
vi server.js

# Copy and paste the web server code below into the server.js file.

var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
![images](images/012-viserverjs.jpg)


### Step 3: Install Express and set up routes

Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straightforward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

In your Book directory run this command:
```bash
sudo npm install express mongoose
```
![images](images/013-installexpressmongoose.jpg)

# In Books folder, create a folder named apps

```bash
mkdir apps && cd apps
```
# Create a file named routes.js
```bash
nano routes.js
```
# Copy and paste the code below into routes.js

```bash
var Book = require('./models/book');

module.exports = function (app) {

  // Get all books
  app.get('/book', async function (req, res) {
    try {
      const result = await Book.find({});
      res.json(result);
    } catch (err) {
      res.status(500).json({ error: err.message });
    }
  });

  // Add new book
  app.post('/book', async function (req, res) {
    try {

      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });

      const result = await book.save();

      res.json({
        message: "Successfully added book",
        book: result
      });

    } catch (err) {
      res.status(500).json({ error: err.message });
    }
  });

  // Delete book
  app.delete('/book/:isbn', async function (req, res) {
    try {

      const result = await Book.findOneAndDelete({
        isbn: req.params.isbn
      });

      if (!result) {
        return res.status(404).json({
          message: "Book not found"
        });
      }

      res.json({
        message: "Successfully deleted the book",
        book: result
      });

    } catch (err) {
      res.status(500).json({ error: err.message });
    }
  });

  // Catch all routes
  app.all(/.*/, function (req, res) {
    res.status(404).json({ message: "Route not found" });
  });

};
```

![images](images/014-routejs.jpg)

# In the apps folder, create a folder named models
```bash
mkdir models && cd models
```
# Create a file named book.js
```bash
nano book.js
```
# Copy and paste the code below into book.js
```bsh
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```

![images](images/015-models&book.jpg)


### Step 4: Access The Routes with Angularjs

Note: AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

# Change the directory back to ‘Books’
```bash
cd ../..
```
# Create a folder named public :
```bash
mkdir public && cd public
```
# Add a file named script.js and copy and paste the code below in it :
```bash
nano script.js
```

```bsh
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```

![images](images/016-ANGULAR.jpg)

# In the public folder, create a file named index.html and paste the code below in it :
```bsh
nano index.html
```

```bsh
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>
 
        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>
 
          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```