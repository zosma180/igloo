<p align="center"><img src="logo.png" alt="jgloo logo" height="120"></p>

# jgloo

![npm](https://img.shields.io/npm/v/jgloo?style=flat-square)
![npm bundle size](https://img.shields.io/bundlephobia/min/jgloo?style=flat-square)
![license](https://img.shields.io/npm/l/jgloo?style=flat-square)
[![Issues](https://img.shields.io/github/issues/zosma180/jgloo.svg?style=flat-square)](https://github.com/zosma180/jgloo/issues)

---

## Description

**jgloo** is a local HTTP server useful to mock your backend API and speed up the client development.  
This project is based on the Node framework Express. The highlights are:

- Create a ReST API with two rows of code
- Create custom API easily
- Create custom middleware easily (e.g. auth check)
- Store data in accessible JSON files
- Reload the live changes of your mocks automatically (thanks to nodemon package)
- Expose a dedicated folder for the static files (images, assets etc...)
- Support to multipart/form-data requests

---

## Installation

```bash
npm i -D jgloo
```

After the installation create a folder "mock" in your project root (you can use another path and folder name if you want).  
The **only requirement** is to create a subfolder "api" in your chosen path (e.g. "mock/api").  
Now you are ready to [create your first API](#create-a-simple-api).

---

## Guide

- [Create a simple API](#create-a-simple-api)
- [Create a default ReST API](#create-a-default-rest-api)
- [Create a custom ReST API](#create-a-custom-rest-api)
- [Create a middleware](#create-a-middleware)
- [Where data are stored](#where-data-are-stored)
- [Expose the static files](#expose-the-static-files)
- [Handle requests with file uploads](#handle-requests-with-file-uploads)
- [Simulate network delay](#simulate-network-delay)
- [Manage path conflicts](#manage-path-conflicts)
- [Run the server](#run-the-server)

---

### Create a simple API

To setup your first API create a new file "hello.js" in the "api" folder. The name of the file does not matter. Then insert the following snippet:

```javascript
export default {
  path: '/hello',
  method: 'get',
  callback: (req, res) => {
    res.json({ message: 'Hello World!' });
  },
};
```

This code define the GET route http://localhost:3000/hello that returns a JSON with data "{ message: 'Hello World!' }".  
You are ready to [run the server](#run-the-server) now.

---

### Create a default ReST API

To setup a ReST API, you have to create a new file in the "api" folder with the name you prefer and the following snippet:

```javascript
export default {
  path: '/user',
  method: 'resource',
};
```

With these few rows of code will be created 6 routes:

- GET /user : return the list of users;
- GET /user/:id : return the specific user;
- POST /user : allow to store the full request body as new user and return it as response;
- PUT /user/:id : allow to replace an existent user with the full request body and return it as response;
- PATCH /user/:id : allow to merge an existent user data with the request body values and return it as response;
- DELETE /user/:id : allow to delete an existent user and return the deleted id.

If you want to skip any of the previous routes, you can add the "not" property:

```javascript
export default {
  path: '/user',
  method: 'resource',
  not: ['LIST']
};
```
The available values of the "not" property are ['LIST', 'READ', 'CREATE', 'UPDATE', 'PATCH', 'DELETE']

---

### Create a custom ReST API

If you need to control the logic of your resources, you can create a custom API that read and/or write the data in the JSON database.  
To achieve it create a new file in the "api" folder with the name you prefer and the following snippet:

```javascript
import { getResource, setResource } from 'jgloo';

export default {
  path: '/user',
  method: 'post',
  callback: (req, res) => {
    // Get the existent resource list or instantiate it
    const list = getResource('user') || [];

    // Get the data of the request and add a new field
    const user = req.body;
    user.extraField = 'value';

    // Push the new model to the list
    list.push(user);

    // Store the updated list in the "user.json" file
    setResource('user', list);

    // Return the model as JSON
    res.json(user);
  },
};
```

---

### Create a middleware

To add a middleware, you have to create a folder "middlewares" in your chosen root path (e.g. "mock/middlewares").  
Then create a new file inside with the name you prefer and the following sample snippet:

```javascript
export default function(req, res, next) {
  const isAuthorized = req.get('Authorization') === 'my-token';
  isAuthorized ? next() : res.sendStatus(401);
};
```

This sample code check for all routes if the "Authorization" header is set and it has the value "my-token".

---

### Where data are stored

The resources are stored in JSON files placed in the subfolder "db" of your chosen root path (e.g. "mock/db").  
The [default ReST API](#create-a-default-rest-api) store the JSON file with the name generated by resource path replacing the slashes with the minus sign (e.g. "/auth/user" will be stored as "auth-user.json").  
If you want to specify the file name of the resources, you can set it as the "name" property of the API:

```javascript
export default {
  path: '/my/long/path',
  method: 'resource',
  name: 'user',
};
```

With this code, the JSON file will be stored as "user.json".

---

### Expose the static files

To expose any static files you have to create the subfolder "static" in your chosen root path (e.g. "mock/static") and put all the resources inside it.  
The static content is reachable by "http://localhost:3000/static/...". That's it.

---

### Handle requests with file uploads

The multipart/form-data requests are supported by default. The `req.body` will be filled with the right data.  
The data of the uploaded files are placed in the `req.files` property and the files are saved in the `static` folder with a temporary name.  
It's recommended to add the `static` folder in the `.gitignore` file.

---

### Simulate network delay

If you want to simulate a network delay, you can add the `delay` property to your API configuration:

```javascript
export default {
  ...
  delay: 3 // Seconds
};
```

---

### Manage path conflicts

If you have a scenario where two or more paths have conflicting values, e.g.:
- /my-path/:id
- /my-path/my-sub-path

you can add the `priority` property to your API configuration:

```javascript
export default {
  ...
  priority: 2
};
```

The default value is 0. The api with the higher value will be used.

---

### Run the server

To run the server execute the following command in your project root:

```shell
npx jgloo
```

The full optional parameters are:

```shell
npx jgloo -f [FOLDER] -p [PORT] -s [STATIC_URL]
```

For example:

```shell
npx jgloo -f 'mock' -p 3000 -s 'static'
```

- "**-f**" or "**--folder**": the folder where your mocks are placed. It's optional, by default it's the folder "mock".
- "**-p**" or "**--port**": the port of running server. It's optional, by default it's "3000".
- "**-s**" or "**--static**": the url prefix of the static resources. It's optional, by default it's "static".
