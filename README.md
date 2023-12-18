# Consuming APIs with Express 
<img src="https://i.imgur.com/pOhOMgk.jpg" width="40%">

## Objective 

The goal of this lesson is to learn how to consume third-party APIs using an Express application.  

The backend that we've been building with our express apps is actually what is known as an application programming interface.  Application programming interfaces, or APIs as they're more commonly known, allow different software systems to communicate with each other.  For example, in our next unit we will build a React frontend that communicates with our Express backend.  Our Express backend will be an API with different restful endpoints that send information in a format called json to the react frontend.  Additionally, we can use APIs that others have built to get information and bring additional functionality into our applications.  

In this lesson, we will gain an understanding of API consumption using a library called Axios, as we build an API that fetches dog images.  We will use ejs templates, as we have been doing, to display these images.  We will also touch on testing APIs using an awesome tool called Postman.  

Let's dive in! 


## What Kind of API are we Talking About?

The term _API_ is quite vague and used within several contexts.

First, it's an acronym that stands for **Application Programming Interface**.

_Application Programming Interfaces_ define the set of methods and properties made available by a library, a framework, an operating system, or any piece of software that programmers can use to access the software's functionality. 

However, in today's lesson we're interested in external (third-party) APIs that respond with data when we send them requests.  


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
mkdir express-dog-api
cd express-dog-api 
```

3. **Create an App.js file:**

```
touch App.js
code . 
```


## What is an API?

An API(Application Programming Interface) allows different software systems to communicate with each other.  In this lesson, we will focus on consuming a public API to retrieve and display dog images.  


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
  const apiUrl = 'https://dog.ceo/api/breeds/image/random'
  try {
    const response = await axios.get(apiUrl);
    const imageUrl = response.data.message;
    res.render('index', { imageUrl });
  } catch (error) {
    console.error('Error fetching image:', error);
    res.status(500).send('Error fetching image');
  }
});
```

Notice above how we call the get method from axios on the apiUrl.  We await the response using the `await` keyword, just as when calling asynchronous mongoose methods.  Our call to the apiUrl results in a promise that is either rseovled, or rejected.  In the case it is successful, we access the message property on the data attribute of the response object and assign it to the imageUrl.  We then render an `index` template and pass the imageUrl as part of the locals object. 

Next, let's create our `index` EJS template.

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
<html>
  <head>
    <title>Dog Image Viewer</title>
  </head>
  <body>
    <img src="<%= imageUrl %>" alt="Dog" style="max-width: 100%;" />
  </body>
</html>
```

Now that we have created our template and are displaying the dog image, let's allow for some user interaction by implenting the following user story:

`As a user, I want to be able to enter a dog breed and return an image related to that breed.`

First, let's add a form in our views right below the `img` element:

```
<body>
    <img src="<%= imageUrl %>" alt="Dog" style="max-width: 100%;" />
    <form action="/" method="get">
      <input type="text" name="breed" id="dogInput" placeholder="Enter a dog breed" />
      <button type="submit">Fetch!</button>
    </form>
</body>
```

For the form action, we can actually use the same `/` endpath for intially calling our API and dispalying our index page.  However, we must do some refactoring to our route.  Adding a conditional to check for a req.body will handle the case for a user entering a dog.  

Modify the `get /` endpath to be the following. 

```
app.get('/', async (req, res) => {
  try {
    let imageUrl;

    // Check if a specific breed is requested
    if (req.query.breed) {
      const response = await axios.get(`https://dog.ceo/api/breed/${req.query.breed}/images/random`);
      imageUrl = response.data.message;
    } else {
      // If no specific breed, fetch a random dog image
      const response = await axios.get('https://dog.ceo/api/breeds/image/random');
      imageUrl = response.data.message;
    }

    res.render('index', { imageUrl });
  } catch (error) {
    console.error('Error fetching image:', error);
    res.status(500).send('Error fetching image');
  }
});
```


## Testing with Postman

It's all fine and well to test our endpoints in the browser, but there's a really cool tool called Postman that helps us to not only test API endpoints, but also provides us with the code to make those requests.  It especialy can be helpful when making calls to third party APIs.  

1. **Install Postman:**

Download and install Postman from [https://www.postman.com/](https://www.postman.com/).

2. **Test the following endpoint:**

Let's explore Postman to test API endpoints:

1. Open Postman and create a new request.
2. Use the request URL [https://dog.ceo/api/breeds/image/random](https://dog.ceo/api/breeds/image/random).
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

Remember to review each API's documentation for information on how to make requests and any authentication requirements. These APIs cover a range of topics, so you can choose one that aligns with your interests or project goals. Happy coding!







