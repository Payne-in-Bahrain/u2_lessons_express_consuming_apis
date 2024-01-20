# Consuming APIs with Express 
<img src="https://i.imgur.com/J6GgzUD.jpg" width="60%">

## Objective 

The goal of this lesson is to learn how to consume third-party APIs using an Express application.  

The backend that we've been building with our express apps is actually what is known as an application programming interface.  Application programming interfaces, or APIs as they're more commonly known, allow different software systems to communicate with each other.  For example, in our next unit we will build a React frontend that communicates with our Express backend.  Our Express backend will be an API with different restful endpoints that send information in a format called JSON to the react frontend.  Additionally, we can use APIs that others have built to get information and bring additional functionality into our applications.  

In this lesson, we will gain an understanding of API consumption using a library called Axios, as we build an app that fetches cat images.  We will use ejs templates, as we have been doing, to display these images.  We will also touch on testing APIs using an awesome tool called Postman.  

Let's dive in! 


## What Kind of API are we Talking About?

The term _API_ is quite vague and used within several contexts.

First, it's an acronym that stands for **Application Programming Interface**.

_Application Programming Interfaces_ define the set of methods and properties made available by a library, a framework, an operating system, or any piece of software that programmers can use to access the software's functionality. 

However, in today's lesson we're interested in external (third-party) APIs that respond with data when we send them requests.  This is an example of what is specifically known as RESTFUL API. 

REST, or Representational State Transfer, is an architectural style for designing networked applications—basically, applications that talk to each other over the internet. RESTful APIs, which follow the principles of REST, use standard HTTP methods (like GET, POST, PUT, DELETE) to do things with resources (which can be data, services, or anything else). The APIs usually respond with data in a structured format, commonly JSON, showing the current state of a resource (for example, fetching and displaying a specific Facebook post).

When you make a request to a RESTful API, you are usually interacting with a remote server that provides a set of endpoints representing different resources (e.g., /users, /products). Each endpoint corresponds to a specific operation or resource, and the server responds with the requested data or performs the requested action.


## Why Consume Third-Party APIs?

Lots of useful data is available via APIs across the Internet - often free of charge!

Apps we create can consume this data in interesting ways!


## 👉 You Do - Research Available APIs (2 minutes)

[This GitHub repo](https://github.com/public-apis/public-apis) maintains a list of well organized Public APIs.

Pick one that's interesting to you and identify the following:

- The name of the API
- What kind of data it makes available
- It's access requirements and limitations:
  - Is it completely free, free up to a certain usage, or paid from the start?
  - Does it require authentication (usually via a token)?


## I Have a Simple Request

It only takes a simple `GET` request to one of the API's available endpoints to retrieve data.

Since we can issue `GET` requests straight from the browser's address bar, let's retrieve all the data you'd ever want to know about Bulbasaur:

```
https://pokeapi.co/api/v2/pokemon/1
```

<details>
<summary>
❓ What data format was returned?
</summary>
<hr>

**JSON (JavaScript Object Notation)**, the same format as, for example, a `package.json` file.

<hr>
</details>


## Different Architectural Approaches

When accessing APIs, there are a few different architectural approaches we can take:

<img src="https://i.imgur.com/Hflu0K0.png">

- The top-approach is the recommended approach with traditional web apps that respond to client requests with a new HTML page.  Access tokens remain secure on the server.

- The middle-approach is recommended for single-page apps (SPAs) like the MERN-stack apps we'll develop in unit 4.

- The bottom-approach is not recommended because access tokens would have to be sent to the browser.  Because of this, many APIs will disallow this architecture by not implementing [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (discussed in a later lesson) and the request will fail.

Since we will be developing traditional web apps in this and next unit, we'll use the **top-approach** in this lesson.


## Getting Started

1. **Navigate to your code folder:**
   
Open your terminal and navigate to the folder where you want to create your project.  

2. **Create Project Directory:**

```
mkdir express-cat-api
cd express-cat-api 
```

3. **Create an App.js file:**

```
touch app.js
code . 
```


## What is an API?

An API(Application Programming Interface) allows different software systems to communicate with each other.  In this lesson, we will focus on consuming a public API to retrieve and display cat images.  


## Express App Setup 

1. **Install Express and EJS:**

```
npm i express ejs 
```

2. **Configure Express App:**

Within `app.js` add the following code to set up a basic Express application:

```
// app.js
const express = require('express');
const app = express();
const port = 3000;

app.set('view engine', 'ejs');

// Define routes and middleware here

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

3. **Install Axios:**

Axios is a promise-based HTTP client for the browers and Node.js.
We can install it using:

```
npm i axios
```


## Defining our first endpoint and integrating the Dog API 

Let's create our first endpoint.  We'll make a `get` request to the `/'.  Don't forget to import axios above our routes.  

```
// Inside app.js
const axios = require('axios');

app.get('/', async (req, res) => {
  const catFactsUrl = 'https://catfact.ninja/facts';

  try {
    const response = await axios.get(catFactsUrl);
    
    if (response.status !== 200) {
      throw new Error(`Failed to fetch cat facts. Status: ${response.status}`);
    }

    const catFacts = response.data.data;

    res.json(catFacts);
  } catch (error) {
    console.error('Error fetching cat facts:', error);
    res.status(500).send('Error fetching cat facts');
  }
});
```

Notice above how we call the get method from axios on the apiUrl.  We await the response using the `await` keyword, just as when calling asynchronous mongoose methods.  Our call to the apiUrl results in a promise that is either rseovled, or rejected.  In the case it is successful, we access the data property on the data attribute of the response object and assign it to a variable called catFacts.  For now we are simply sending a json response, which we can check in our browser, or better yet, Postman!  

_a little more on promises:_ 

Asynchronous methods in JavaScript often return promises. These promises represent the eventual completion or failure of an asynchronous operation, such as retrieving data from a database. When the asynchronous operation is successful, the promise is resolved with the result data. On the other hand, if there is an error during the operation, the promise is rejected with an error object. Developers can then use .then() to handle successful outcomes and .catch() to handle errors when working with promises.  For example, we could rewrite the `async/await` above using the following syntax that makes use of promise chaining:

```
axios.get(catFactsUrl)
  .then(response => {
    // Handle successful response here
    console.log(response.data);
  })
  .catch(error => {
    // Handle error here
    console.error(error);
  });
```

Using `async/await`, however, is a more modern, clearner syntax and the one we will give preference to in this cohort.  

Next, let's create our `index` EJS template, so that we can display our catfacts.

1. **Create a views folder:**

create a folder named views in the root of your project directory. 

```
mkdir views
```

2. **Create a template named index.ejs to display the dog image:**

```
touch views/index.ejs
```

and then past the following into your index.ejs:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kitty Facts</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      text-align: center;
    }

    .fact-container {
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <h1>≽^•⩊•^≼ Kitty Facts ≽^•⩊•^≼</h1>
  
  <!-- Display cat facts -->
  <div class="fact-container">
    <h2>All Kitty Facts:</h2>
    <% catFacts.forEach(fact => { %>
      <p><%= fact %></p>
    <% }); %>
  </div>
</body>
</html>
```

Now it's your turn.  Let's update our route to render the index.ejs, instead of responding with json.  Make sure to pass the catFacts variable to the template using the context object as the second argument of the render function. 

Awesome!  If you navigate back to `localhost:3000` you should see a list of... `[object Object]`.  

That's actually to be expected.  If you recall the structure of our json, it looked something like this:

```
"data": [
        {
            "fact": "Unlike dogs, cats do not have a sweet tooth. Scientists believe this is due to a mutation in a key taste receptor.",
            "length": 114
        },
        {
            "fact": "When a cat chases its prey, it keeps its head level. Dogs and humans bob their heads up and down.",
            "length": 97
        }, etc... 
]
```

If you don't recall that, we can always test the cat facts endpoint with Postman.

In any case, it looks like we need to access the fact property of each object.  We could do that in our ejs template, but it's usually better to do that kind of data processing in our controller. 

Let's go back and refactor our route to look like the following:

```
const catFacts = response.data.data;
formattedCatFacts = catFacts.map(fact => fact.fact);
  
res.render('index', { catFacts: formattedCatFacts })
```

Notice how we made use of the awesome map method to create a new array where every item is just the fact.  Now, since we're passing this newly formatted array to the template as catFacts, when we refresh our browser, we should see the facts being displayed!


## Testing with Postman

It's all fine and well to test our endpoints in the browser, but there's a really cool tool called Postman that helps us to not only test API endpoints, but also provides us with the code to make API requests in different programming languages and environments.  It is especially helpful when making calls to third party APIs.  

1. **Install Postman:**

Download and install Postman from [https://www.postman.com/](https://www.postman.com/).

2. **Test the following endpoint:**

Let's explore Postman to test API endpoints:

1. Open Postman and create a new request.
2. Use the request URL [https://catfact.ninja/facts](https://catfact.ninja/facts).
3. Send the request.
4. Observe the response and explore different features in Postman.


## In Conclusion 

Congratulations! In this lesson, you've learned the fundamentals of consuming APIs using an Express application. You built a simple app that fetches dog images from the Dog API and explored API testing using Postman. Now you have the ability to test a backend without a browser, or any other kind of frontend client.  Now that you know the basics, consider integrating an API in your unit 2 projects.  Check out this list of free APIs to get some ideas. 

### Free APIs for Your Projects: 

1. **REST Countries:**
   - **API Documentation:** [REST Countries](https://restcountries.com/)
   - **Description:** Get information about countries, including details such as population, area, languages, and more.

2. **OpenWeatherMap:**
   - **API Documentation:** [OpenWeatherMap API](https://openweathermap.org/api)
   - **Description:** Access weather data for any location, including current conditions, forecasts, and historical data.

3. **JokeAPI:**
   - **API Documentation:** [JokeAPI](https://jokeapi.dev/)
   - **Description:** Retrieve programming-related jokes, general jokes, and more.

4. **The Cat API:**
   - **API Documentation:** [The Cat API](https://thecatapi.com/)
   - **Description:** Get random cat images, facts, and more.

5. **PokéAPI:**
   - **API Documentation:** [PokéAPI](https://pokeapi.co/)
   - **Description:** Access detailed Pokémon data, including information about different species, abilities, and more.

6. **NewsAPI:**
   - **API Documentation:** [NewsAPI](https://newsapi.org/)
   - **Description:** Access news articles from various sources around the world.

7. **Random User Generator:**
   - **API Documentation:** [Random User Generator](https://randomuser.me/)
   - **Description:** Generate random user data, including names, addresses, and profile pictures.

8. **Unsplash API:**
   - **API Documentation:** [Unsplash API](https://unsplash.com/developers)
   - **Description:** Access a vast collection of high-quality, royalty-free images.

9. **CoinGecko API:**
    - **API Documentation:** [CoinGecko API](https://www.coingecko.com/en/api)
    - **Description:** Retrieve cryptocurrency data, including prices, market data, and more.

10. **Jikan API:**
    - **API Documentation:** [Jikan API](https://jikan.moe/)
    - **Description:** Access anime and manga-related data, including information about shows, characters, and more.

11. **OpenAI API:**
    - **API Documentation:** [OpenAI API](https://beta.openai.com/docs/)
    - **Description:** Utilize OpenAI's language models and AI capabilities for various natural language processing tasks.


Remember to review each API's documentation for information on how to make requests and any authentication requirements. These APIs cover a range of topics, so you can choose one that aligns with your interests or project goals. Happy coding!







