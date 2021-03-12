[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# Database Seeding

So far, we have been able to interact with our database through the Mongo CLI and through Mongoose queries. As we move forward with our back end, we will be building APIs that feature full CRUD functionality, implemented directly from our applications.

To ensure our functionality works the way that we want it to, it's a good idea to test it on actual data. That's where database seeding comes in. **Database seeding** is a process in which an initial set of data is provided to a database during development.

## Prerequisites

- Intro to MongoDB
- MongoDB CRUD
- Mongoose

## Objectives

By the end of this, developers should be able to:

- Fetch data from an API using server-side JavaScript
- Write a data set to the filesystem using a Promise
- Define a model based on a few properties of the data set fetched from the API
- Seed data to a local MongoDB database

## [Countries API App](https://git.generalassemb.ly/dc-wdi-node-express/countries-api)

### Getting Started

Create a new Node project by creating a directory called `countries-api` and run `npm init` inside of it. It will create a `package.json` file for you.


### Install Our First Dependency: `node-fetch`
Node does not have a native `fetch` function like web browsers do, so we will install a package called `node-fetch` via `npm install node-fetch`. Check your `package.json` and `node_modules` directory to see the module is installed.


### Fetch Data from the API

For this application, we will be using the [REST Countries API](https://restcountries.eu/). Take a few moments to familiarize yourself with the API. What is the endpoint to fetch all countries from the API?

Next, we will need to import the `node-fetch` dependency using the `require()` function. Let's do this in a file called `download.js`. This is where we will  retrieve data from the API.

```js
let fetch = require('node-fetch')
```

#### Fetch Request

Use the `fetch()` method to retrieve data on **all countries** from the REST Countries API and log the data to the console. This is something you done many times in the past, so it should be familiar! We can program this functionality the same way we have done many times before.
```js
let fetch = require('node-fetch')

fetch('https://restcountries.eu/rest/v2/all')
    .then(response => response.json())
    .then(data => console.log(data))
```

Take a look at the data that is being returned in the terminal. What is the structure of the JSON. What are the properties? How would we access the values within this data set?

### Write Data to the Filesystem

Earlier, we used the `fs` module to read and write files. Let's implement this with `fetch()` and write the data to a `.json` file.

```js
let fetch = require('node-fetch')
let fs = require('fs').promises

fetch('https://restcountries.eu/rest/v2/all')
  .then(response=> response.json())
  .then(data=> fs.writeFile("./countries.json", JSON.stringify(data)))
```

Let's review what's happening here:

1. First we import our `node-fetch` module as `fetch`.
1. Next we import the `fs` module. Note that we use `require('fs').promises` instead of `require('fs')`. The former lets us use `fs` with promises rather than the (older, messier) callback pattern.
1. Next we make our fetch request, then parse the JSON in the request body.
1. Finally, we change the `data` into a string with `JSON.stringify(data)` and write it to a file called `data.json` with `fs.writeFile()`.

Let's make our code a little more robust by handing errors. We can do that by adding a `catch` to our promise chain.

```js
fetch('https://restcountries.eu/rest/v2/all')
  .then(response=> response.json())
  .then(data=> fs.writeFile("./data.json", JSON.stringify(data)))
  .catch(error=> console.log(error))
```

Now if either the fetch fails, the JSON parse fails or the write file fails we will handle the error. 

### Running the Code

In your terminal, run the script via `node download.js`.

Next, let's take a look at our `data.json` file. It has been populated with all of the data we pulled from the REST Countries API!

> Was our response successful?
> When we are writing to a file that does not yet exist, what will happen?

### Create a Country Model

Now that we have all of the data in `data.json`, let's pick out the properties we want to include in our database. Oftentimes, you will find an API or dataset that has exactly the information you want - and then some. We want to create a database of countries that only includes the country's `name`, `capital`, `region` and `population`.

In `Country.js`, build a model to include the above properties. Pay attention to the data types from the data we have in our `data.json` file. In addition to the the Schema, what else do you need in this file?

```js
let mongoose = require('mongoose')

const countrySchema = new mongoose.Schema({
    name: String,
    capital: String,
    region: String,
    population: Number
})

module.exports = mongoose.model('Country', countrySchema)
```

What's happening here? We create our `Country` Schema with the appropriate properties and data types, then we build and export our `Country` model to make it available to other files in our application.

### Create a New Data Set

We have a TON of data in our `data.json` file. We need some of it, but most of it we don't. We can see that the data set we have consists of an array of objects. This means we can use the `.map` array method to create a new array with only the properties we care about!

> Regardless of where you get your data, it is **extremely** important to examine it before using it. Every data set is structured differently, and only when you familiarize yourself its architecture can you successfully access the values you want.

In `seed.js`, add the following code:
```js
let data = require('./data.json')

let countryData = data.map(item => {
    const country = {}
    country.name = item.name
    country.capital = item.capital
    country.region = item.region
    country.population = item.population
    return country
})

console.log(countryData)
```

Run `node seed.js` in the terminal. What do you see?

### Create a Database Connection
As we've done before, we'll write a `connection.js` file to connect to the database with Mongoose.

```js
let mongoose = require('mongoose')

let mongooseConnectionConfig = { useNewUrlParser: true, useUnifiedTopology: true }
mongoose.connect("mongodb://localhost/countries", mongooseConnectionConfig)
```
In the previous code we import Mongoose and connect to our local MongoDB for a database called `countries`. We also pass in some extra configuration variables we can otherwise ignore; they're for newer features of Mongoose that we want to use that are not backwards-compatible with older versions.

Let's add some event listeners to `connection.js` that will let us know when the database is connected, disconnected, or if an error occurs.

```js
mongoose.connection.on('connected', ()=> console.log("Connected to database"))
mongoose.connection.on('disconnected', ()=> console.log("Disconnected from database"))
mongoose.connection.on('error', error=> console.error("Database error", error))
```

We can run `node connection.js` and see if our connection works. Remember to use `<ctrl>+C` to quit with an open database connection.

### Let's Seed Our Data to the Database!

Now that we have everything set up the way we want, it's time to actually seed our data - meaning we are going to write code to add our array of countries to the local database with Mongoose.

```js
require('./connection')
let mongoose = require('mongoose')
let data = require('./data.json')
let Country = require('./Country')

let countryData = data.map(item => {
    const country = {}
    country.name = item.name
    country.capital = item.capital
    country.region = item.region
    country.population = item.population
    return country
})

Country
  .deleteMany({})
  .then(()=> Country.create(countryData))
  .then(mongoose.disconnect)
  .then(()=> console.log("Done!"))
  .catch(()=> console.log("Error", error))

```

Let's break this down:

1. First is our dependencies. We import `connection.js` to run the code that connects Node to MongoDB. Note that we don't need to return any variables so we don't do an assignment. We also import Mongoose, our JSON data and our Country model.
1. Then we parse our JSON data into our smaller `countryData` array that fits with our Country model's schema.
1. Now we remove all the documents from the `countries` collection in the database with `Country.deleteMany({})`. This is a common step when seeding your database to avoid duplicated data.
1. Then, we insert our `countryData` into the database with `Country.create`. 
1. Next we are adding a Promise to add country records with the `countryData` array that we just created.
1. Then, since we are done with our database we will disconnect from it with `mongoose.disconnect`.
1. Finally, if the request is successful, we will log "Done!". We use `.catch` to catch and log an error if any of the preceeding steps fail.

In the terminal, run your MongoDB server. Then, run the command `node seed.js`. What do you see? Where is this output coming from?

### Verify in the Mongo Shell

We see the output in the terminal (which is coming from `console.log(countries)` within our Promise), but let's verify to see if we have seeded our data successfully. In the terminal open the Mongo shell using the command `mongo`.

Next, look for the database we have created:

```bash
show dbs
```

Connect to the database:

```bash
use countries
```

List your collections:

```bash
show collections
```

Last but not least, read the data you have just added!

```bash
db.countries.find().pretty()
```

What you should see is a list of records of the countries we seeded that include the properties we specified earlier in addition to `ObjectId` and `__v` properties.

```bash
{
	"_id" : ObjectId("5dc2db6224a43721f694e471"),
	"name" : "Afghanistan",
	"capital" : "Kabul",
	"region" : "Asia",
	"population" : 27657145,
	"__v" : 0
}
```

**SUCCESS!!** You just seeded a database! Now, you can use what you already learned to interact with this new data set in MongoDB.

![Success!](https://media.giphy.com/media/vViFKLAOQdDlS/giphy.gif)

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
   alternative licensing, please contact legal@ga.co.
