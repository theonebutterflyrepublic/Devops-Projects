PROJECT 3
SIMPLE TO-DO APPLICATION ON MERN WEB STACK (AWS DEPLOYMENT)
INTRODUCTION

In this project, I implemented a full-stack web solution using the MERN stack and deployed it on AWS EC2 cloud infrastructure.

The MERN stack consists of:

MongoDB – A document-based NoSQL database used to store application data.

ExpressJS – A backend framework running on Node.js.

ReactJS – A frontend JavaScript library for building user interfaces.

Node.js – A JavaScript runtime environment for executing backend code.

The application allows users to:

Create a new To-Do task

View all existing tasks

Delete completed tasks

ARCHITECTURE OVERVIEW

The application follows this architecture:

User → React Frontend → Express Backend → MongoDB Database → Response to Frontend

When a user interacts with the UI:

React sends an HTTP request to Express

Express processes the request

MongoDB stores or retrieves data

The response is returned to the frontend


SIDE SELF-STUDY

During this project, I researched and understood:

Types of DBMS (Relational vs NoSQL)

RESTful APIs and CRUD operations

Backend vs Frontend frameworks

Environment variables and security best practices



STEP 0 – PREPARING PREREQUISITES
AWS EC2 Setup

I already have an AWS account
Launched EC2 instance (Ubuntu)
Instance type: t2.nano
Connected using SSH

![images](images/1-ssh.jpg)
STEP 1 – BACKEND CONFIGURATION
1. Updating Ubuntu

sudo apt update

sudo apt upgrade

2. Installing Node.js
Let’s get the location of Node.js software from Ubuntu repositories.


curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

![images](images/2-curl.jpg)


Install Node.js using


sudo apt-get install -y nodejs

![images](images/3-install-njs.jpg)

Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.

Verify the node installation with the command below:


node -v

and

npm -v

![images](images/4-v.jpg)

3. Application Code Setup
Create a new directory for your To-Do project: mkdir Todo

check to verify that the directory you created actually worked

ls

Now change your current directory to the newly created one:

cd Todo

Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.

npm init

![images](images/6-json.jpg)

4.INSTALL EXPRESSJS

To use express, install it using npm:

npm install express

Now create a file index.js with the command below

touch index.js

Run ls to confirm that your index.js file is successfully created.

Install the dotenv module by running the command below:

npm install dotenv


![images](images/7-express.jpg)

Open the index.js file with the command below:


vim index.js

Type the code below into it and save.
you can simply copy and paste the code

const express = require('express');
require('dotenv').config();
 
const app = express();
 
const port = process.env.PORT || 5000;
 
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
 
app.use((req, res, next) => {
res.send('Welcome to Express');
});
 
app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.

Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type:

node index.js

![images](images/8-vim-indexjs.jpg)
![images](images/9-nodeindex.jpg)

you should see Server running on port 5000 in your terminal. This shows that everything is working properly

Now we need to open port 5000 in EC2 Security Groups.
![images](images/10-5000.jpg)

Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:

http://PublicIP-or-PublicDNS:5000


![images](images/11-express.jpg)

5.ROUTES CONFIGURATION

There are three actions that our To-Do application needs to be able to do:

Create a new task
Display list of all tasks
Delete a completed task
Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes

mkdir routes

Tip: You can open multiple shell to connect to the same EC2

Change directory to routes folder.

cd routes

Now, create a file api.js with the command below:

touch api.js

Open the file with the command below

vim api.js or nano api.js, i personally prefer using vim. Copy below the code and paste them in the file.

const express = require ('express');
const router = express.Router();
 
router.get('/todos', (req, res, next) => {
 
});
 
router.post('/todos', (req, res, next) => {
 
});
 
router.delete('/todos/:id', (req, res, next) => {
 
})
 
module.exports = router;

![images](images/12-routes.jpg)

6.MODELS

Now comes the interesting part, since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model. A model is at the heart of JavaScript based applications, and it is what makes it interactive. We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties

Change directory back Todo folder with cd .. and install Mongoose

npm install mongoose

Create a new folder models:

mkdir models

Change directory into the newly created ‘models’ folder with:

cd models

Inside the models folder, create a file and name it todo.js with the command below:

touch todo.js

Open the file created with vi todo.js then paste the code below in the file:


const mongoose = require('mongoose');
const Schema = mongoose.Schema;
 
//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})
 
//create model for todo
const Todo = mongoose.model('todo', TodoSchema);
 
module.exports = Todo;

![images](images/14-todojs.jpg)

 we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model. In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste the code below into it then save and exit with :wq!



 const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');
 
router.get('/todos', (req, res, next) => {
 
//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});
 
router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});
 
router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})
 
module.exports = router;

![images](images/apijs.jpg)

7.MONGODB DATABASE

We need a database where we will store our data. For this we will make use of MongoDB database as a service solution (DBaaS).

So to make life easy, you will need to sign up for a shared clusters free account, which is ideal for our use case. Sign up here: Mongodb. Follow the sign up process, select AWS as the cloud provider, and choose a region near you.

Create a cluster ( build a cluster ) and follow the steps in the images below

![images](images/16-mongodb.jpg)
![images](images/17-clusterbuild.jpg)
![images](images/18-cluster2.jpg)
![images](images/19-cluster3.jpg)

I removed the default private IP entry and configured public access by allowing connections from 0.0.0.0/0 for testing purposes.

![images](images/16-mongodbnetwork.jpg)
![images](images/20-ipdelete.jpg)
![images](images/21-ipadd.jpg)

incase you didn't see the "Allow Access From Anywhere" option, add is manually by yourself just as highlighted in the image below.

![images](images/22-ipallow.jpg)

Create a MongoDB database and collection, follow the steps in the images below
![images](images/23-clusters.jpg)
![images](images/24-createdb.jpg)
![images](images/24-createdb.jpg)
![images](images/25-cdb.jpg)

In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now.

Create a file in your Todo directory and name it .env. by running the command below:

touch .env

vi .env

Add the connection string to access the database in it, just as below:


DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'

Make sure to update username, password, network-address and database according to your setup

![images](images/env.jpg)

This is how to get your connection string:
![images](images/26-connectdb.jpg)
![images](images/27.jpg)
![images](images/28.jpg)

Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database.

Simply delete existing content in the file, and update it with the entire code below by using 


vi index.js

const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();
 
const app = express();
 
const port = process.env.PORT || 5000;
 
//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));
 
//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;
 
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
 
app.use(bodyParser.json());
 
app.use('/api', routes);
 
app.use((err, req, res, next) => {
console.log(err);
next();
});
 
app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

![images](images/29-editindex.jpg)

NOTE: Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

Start your server using the command:


node index.js
You shall see a message ‘Database connected successfully'

![images](images/30-nodeindexjs.jpg)


8.TESTING OUR BACKEND CODE

So far, we have built the backend of our To-Do application. This backend handles things like saving tasks, reading tasks, updating them, and deleting them from the database. We have also successfully connected our app to a database.

However, we do not have a frontend yet. That means there is no website or user interface where a user can click buttons or type tasks.

Because of this, we still need a way to check if our backend is working correctly during development.

To do that, we use a tool called an API development client. This type of tool allows us to send requests (like GET, POST, PUT, DELETE) directly to our backend and see the responses, without needing a frontend.

In this project, we will use **Postman**

Click the link below to downnload Postman and install postman on your machine.

Postman download

Click HERE to learn how perform CRUD operartions on Postman.

Note: Make sure your set header key Content-Type as application/json

![images](images/31.0-post.jpg)

create a POST request to the API http://PublicIP-or-PublicDNS:5000/api/todos. This request sends a new task to our To-Do list so the application could store it in the database.

![images](images/31-post.jpg)

Create a GET request to your API on 

http://PublicIP-or-PublicDNS:5000/api/todos

This request retrieves all existing records from out To-do application (backend requests these records from the database and sends it us back as a response to GET request).

![images](images/32-get.jpg)

 STEP 2 - FRONTEND CREATION

Since we are done with the functionality we want from our backend and API, it is time to create a user interface for a Web client (browser) to interact with the application via API.

To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app.

In the same root directory as your backend code, which is the Todo directory, run:


npx create-react-app client


![images](images/33-react.jpg)

next Install Concurrently. It is used to run more than one command simultaneously from the same terminal window.

npm install concurrently --save-dev

Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

npm install nodemon --save-dev

![images](images/34.jpg)

In Todo folder open the package.json file. Change the highlighted part of the below 
screenshot and replace with the code below.


"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""

![images](images/35-pjson.jpg)

Change directory to ‘client’

cd client

Open the package.json file


vi package.json

Add the key value pair in the package.json file 
"proxy": "http://localhost:5000"

The purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000
 rather than always including the entire path like 
 
http://localhost:5000/api/todos

![images](images/36-localproxy.jpg)

Now, ensure you are inside the Todo directory, and simply do:

npm run dev

![images](images/37-npmrundev.jpg)

On  localhost:3000 Your app should open and start running
![images](images/38.jpg)

note:to be able to access the application from the Internet you have to open TCP port 3000 in your EC2 by adding a new Security Group rule.

From your Todo directory run:

cd client

move to the src directory

cd src

Inside your src folder create another folder called components

mkdir components

Move into the components directory with:

cd components

Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js. by running this command below:

touch Input.js ListTodo.js Todo.js

![images](images/2hh.jpg)

Open Input.js file

nano Input.js

Copy and paste the following


import React, { Component } from 'react';
import axios from 'axios';
 
class Input extends Component {
 
state = {
action: ""
}
 
addTodo = () => {
const task = {action: this.state.action}
 
    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }
 
}
 
handleChange = (e) => {
this.setState({
action: e.target.value
})
}
 
render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}
 
export default Input

To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.

Move your directory to client and paste the command below:


npm install axios


![images](images/39-axios.jpg)

Go to ‘components’ directory

cd src/components

After that open your ListTodo.js

vi ListTodo.js

in the ListTodo.js copy and paste the following code


import React from 'react';
 
const ListTodo = ({ todos, deleteTodo }) => {
 
return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}
 
export default ListTodo

![images](images/40.jpg)

Then in your Todo.js file, write the following code


import React, {Component} from 'react';
import axios from 'axios';
 
import Input from './Input';
import ListTodo from './ListTodo';
 
class Todo extends Component {
 
state = {
todos: []
}
 
componentDidMount(){
this.getTodos();
}
 
getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}
 
deleteTodo = (id) => {
 
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))
 
}
 
render() {
let { todos } = this.state;
 
    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )
 
}
}
 
export default Todo;


![images](images/ere.jpg)

We need to make little adjustment to our react code. Delete the logo and adjust our App.js to look like this. Move to the src folder cd .. Make sure that you are in the src folder and run

vi App.js

Copy and paste the code below into it



import React from 'react';
 
import Todo from './components/Todo';
import './App.css';
 
const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}
 
export default App;


![images](images/41-appjs.jpg)

In the src directory open the App.css

vi App.css

Then paste the following code into App.css:



.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}
 
input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}
 
input:focus {
outline: none;
}
 
button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}
 
button:focus {
outline: none;
}
 
ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}
 
li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}
 
@media only screen and (min-width: 300px) {
.App {
width: 80%;
}
 
input {
width: 100%
}
 
button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}
 
@media only screen and (min-width: 640px) {
.App {
width: 60%;
}
 
input {
width: 50%;
}
 
button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}

![images](images/42-appcss.jpg)

In the src directory open the index.css

vi index.css

Copy and paste the code below:


body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}
 
code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}


![images](images/43-indexcss.jpg)

Go to the Todo directory

cd ../..

When you are in the Todo directory run:

npm run dev

![images](images/44-npmrundev.jpg)

Assuming no errors when saving all these files, our To-Do app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your tasks.

![images](images/final.jpg)


CONCLUSION

This project demonstrates a full implementation of a MERN stack application deployed on AWS EC2.

Throughout the process, I configured a backend server using Node.js and Express, connected it to MongoDB Atlas, tested API endpoints using Postman, and built a React frontend to interact with the API.

I also gained hands-on experience with:

Linux server configuration

Environment variables and secure credential management

AWS Security Groups and port configuration

Full-stack debugging

RESTful API architecture

CHALLENGES AND SOLUTIONS

During the development and deployment of this project, I encountered a few challenges, which I resolved as follows:

1 MongoDB Authentication Error

Challenge: When connecting the Node.js backend to MongoDB Atlas, I received authentication errors. The cause was that I had registered my MongoDB account using Gmail sign-up, which did not set a password during registration.

Solution: I created a new password for my MongoDB account in Atlas and updated the connection string in the .env file. This resolved the authentication issue and allowed the backend to connect to the database successfully.

2 Port Access Issues on AWS EC2

Challenge: Initially, I was unable to access the backend on port 5000 or the frontend on port 3000 from my browser.

Solution: I updated the EC2 instance Security Groups to allow inbound TCP traffic on ports 5000 and 3000, which enabled browser access to the applications.

3 Syntax Error in POST Request

Challenge: While testing the backend API, I encountered issues sending a POST request. The problem was a syntax mistake: there was an extra full stop at the end of the port 5000 in the URL, which caused the request to fail.

Solution: I removed the extra full stop, corrected the URL, and the POST request started working as expected.

LESSONS LEARNED

Importance of AWS Security Groups

Proper environment variable management

REST API structure and CRUD implementation

Debugging using error messages

Difference between local development and cloud deployment

From the challenges I faced during this project, I learned the importance of careful debugging and thorough verification of configurations. A single overlooked error can consume a significant amount of time and effort.

For example, the MongoDB authentication error took me almost two hours to resolve. Because I had signed up for MongoDB using a Google account, no password was initially assigned. I tried multiple fixes, including creating admin accounts and testing connection strings, before realizing that the root cause was simply the lack of a password due to the sign-up method. This experience highlighted the value of checking basic configurations first before diving into complex troubleshooting.

Overall, this project reinforced the need for patience, systematic debugging, and attention to detail when developing full-stack applications.







