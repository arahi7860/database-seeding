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

1. Fork and clone [this repository](https://git.generalassemb.ly/dc-wdi-node-express/countries-api)
1. Change into directory
1. Install all dependencies
1. Examine the codebase

Take a few minutes to examine the codebase here.

1. What dependencies are installed? What do they do and where are they required at this time?
1. What files are included? What is the purpose of each file?

### Fetch Data from the API

For this application, we will be using the [REST Countries API](https://restcountries.eu/). Take a few moments to familiarize yourself with the API. What is the endpoint to fetch all countries from the API?

<details>
    <summary><b>GET All Countries</b></summary>

```js
'https://restcountries.eu/rest/v2/all'
```
</details>


In the past, we have used the `fetch()` method in a browser environment. Now that we are using JavaScript in a server-side environment, we need to install `fetch`. First, run the command:

```bash
npm install node-fetch
```

This will install the dependency `node-fetch`. Next, we will need to import this dependency using the `require()` method. Let's do this in the `getCountries.js` file. This is where we will be retrieving data from the API.

```js
// db/getCountries.js

const fetch = require('node-fetch');
```

#### Fetch Request

Use the `fetch()` method to retrieve data on **all countries** from the REST Countries API and console log the data. This is something you done many times in the past, so it should be familiar! We can program this fucntionality the same way we have done many times before.

> Since we are in a server-side environment, where will be look for the data?

```js
// db/getCountries.js

const url = 'https://restcountries.eu/rest/v2/all'

fetch(url)
    .then(res => res.json())
    .then(res => {
        console.log(res)
    })
```

Take a look at the data that is being returned in the terminal. What is the structure of the JSON. What are the properties? How would we access the values within this data set?

### Write Data to the Filesystem

Earlier, we used the `fs` module to write and read to/from the filesystem. We were able to write simple strings to text files and turn JavaScript objects into JSON strings, writing them to `.json` files. Let's implement this functionalty into our `fetch()` and write the data we are retrieving to a `.json` file within a Promise.

Update the code in `getCountries.js`:

```js
// db/getCountries.js

const fs = require('fs')

// ...

fetch(url)
    .then(res => res.json())
    .then(res => {
        let countries = JSON.stringify(res)
        fs.writeFile('./db/data.json', countries, err => {
            if (err) {
                console.log(err)
            } else {
                console.log('success')
            }
        })
    })
```

Let's review what's happening here:

1. Within our second Promise, we are declaring the variable `countries` and saving our API response as a JSON string.
1. Next, we are writing the data from teh variable `countries` to a file named `./db/data.json`.
1. We have set up our callback function to let us know whether or not there was an error. If we successfully wrote the data to the filesystem, the string `success` will appear in the terminal.

In your terminal, run the command `node db/getCountries.js`.

Next, let's take a look at our `data.json` file. It has been populated with all of the data we pulled from the REST Countries API!

> Was our response successful?
> When we are writing to a file that does not yet exist, what will happen?

### Create a Country Model

Now that we have all of the data in `data.json`, let's pick out the properties we want to include in our database. Oftentimes, you will find an API or dataset that has exactly the information you want - and then some. We want to create a database of countries that only includes the country's `name`, `capital`, `region` and `population`.

In `models/Country.js`, build a model to include the above properties. Pay attention to the data types from the data we have in our `data.json` file. In addition to the the Schema, what else do you need in this file?

```js
// models/Country.js

const Schema = mongoose.Schema

const Country = new Schema({
    name: String,
    capital: String,
    region: String,
    population: Number
})

module.exports = mongoose.model('Country', Country)
```

What's happening here?

1. First, we a variable `Schema` and giving it a value of `mongoose.Schema`.
1. Next, we create our `Country` Schema with the appropriate properties and data types.
1. Finally, we will export our `Country` Schema to make it available to be imported - or required - within other files in our application.

### Create a New Data Set

We have a TON of data in our `data.json` file. We need some of it, but most of it we don't. We can see that the data set we have consists of an array of objects. This means we can use the `.map` array method to create a new array with only the properties we care about!

> Regardless of where you get your data, it is **extremely** important to examine it before using it. Every data set is structured differently, and only when you familiarize yourself its architecture can you successfully access the values you want.

In `seed.js`, add the following code:

```js
// db/seed.js

const Country = require('../models/Country.js')
const data = require('./data.json')

const countryData = data.map(item => {
    const country = {}
    country.name = item.name
    country.capital = item.capital
    country.region = item.region
    country.population = item.population
    return country
})
```

Next, console log your new array:

```js
console.log(countryData)
```

And run `node db/seed.js` in the terminal. What do you see?

### Let's Seed Our Data!

Now that we have everything set up the way we want, it's time to actually seed our data - meaning we are going to write some code that adds our new array of data to our local database using Mongoose queries.

Add the following code to `db/seed.js`:

```js
// db/seed.js

Country.remove({})
    .then(() => {
        Country.create(countryData)
            .then(countries => {
                console.log(countries)
            })
            .catch(err => {
                console.log(err)
            })
    })
```

Let's break this down:

1. First we are removing all records from the `countries` collection in the `countries_db` database. This is a common step when seeding your database. Clear your collection first so that you can add a fresh collection of documents.
1. Next we are adding a Promise to add country records with the `countryData` array that we just created.
1. Finally, if the request is successful, we will console log the countries. We will use `.catch` to catch an error and console log it if there is one.

In the terminal, run your MongoDB server. Then, run the command `node db/seed.js`. What do you see? Where is this output coming from?

### Verify in the Mongo Shell

We see the output in the terminal (which is coming from `console.log(countries)` within our Promise), but let's verify to see if we have seeded our data successfully. In the terminal open the Mongo shell using the command `mongo`.

Next, look for the database we have created:

```bash
show dbs
```

Connect to the database:

```bash
use countries_db
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

![Success](https://media.giphy.com/media/vViFKLAOQdDlS/giphy.gif)

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
   alternative licensing, please contact legal@ga.co.
