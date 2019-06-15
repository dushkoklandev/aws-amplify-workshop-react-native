# Building Mobile Applications with React Native & AWS Amplify

In this workshop we'll learn how to build cloud-enabled mobile applications with React Native & [AWS Amplify](https://aws-amplify.github.io/).

![](https://imgur.com/IPnnJyf.jpg)

### Topics we'll be covering:

- [Authentication](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-authentication)
- [GraphQL API with AWS AppSync](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-a-graphql-api-with-aws-appsync)
- [REST API with a Lambda Function](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-a-graphql-api)
- [Analytics](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-analytics)
- [Adding Storage with Amazon S3](https://github.com/dabit3/aws-amplify-workshop-react-native#working-with-storage)
- [Removing / Deleting Services](https://github.com/dabit3/aws-amplify-workshop-react-native#removing-services)

## Redeeming the AWS Credit   

1. Visit the [AWS Console](https://console.aws.amazon.com/console).
2. In the top right corner, click on __My Account__.
![](dashboard1.jpg)
3. In the left menu, click __Credits__.
![](dashboard2.jpg)

## Getting Started - Creating the React Native Application

To get started, we first need to create a new React Native project & change into the new directory using the [React Native CLI](https://facebook.github.io/react-native/docs/getting-started.html) (See __Building Projects With Native Code__ in the documentation) or [Expo CLI](https://facebook.github.io/react-native/docs/getting-started).

If you already have the CLI installed, go ahead and create a new React Native app. If not, install the CLI & create a new app:

```bash
npm install -g react-native-cli

react-native init RNAmplify

# or

npm install -g expo-cli

expo init RNAmplify

> Choose a template: blank
```

Now change into the new app directory & install the AWS Amplify, AWS Amplify React Native, & React Native Vector Icon libraries:

```bash
cd RNAmplify

npm install --save aws-amplify aws-amplify-react-native uuid

# or

yarn add aws-amplify aws-amplify-react-native uuid
```

Finally, if you're __not using Expo CLI__ you need to link two native libraries:

```sh
react-native link react-native-vector-icons
react-native link amazon-cognito-identity-js
```
Next, run the app:

```sh
react-native run-ios

# or if running android

react-native run-android

# or, if using expo

expo start
```

## Installing the CLI & Initializing a new AWS Amplify Project

### Installing the CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __preferred region__
- Specify the username of the new IAM user: __amplify-workshop-user__
> In the AWS Console, click __Next: Permissions__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __(default)__

### Initializing A New AWS Amplify Project

> Make sure to initialize this Amplify project in the root of your new React Native application

```bash
amplify init
```

- Enter a name for the project: __RNAmplify__
- Enter a name for the environment: __dev__
- Choose your default editor: __Visual Studio Code (or your favorite editor)__   
- Please choose the type of app that you're building __javascript__   
- What javascript framework are you using __react-native__   
- Source Directory Path: __/__   
- Distribution Directory Path: __/__
- Build Command: __npm run-script build__   
- Start Command: __npm run-script start__   
- Do you want to use an AWS profile? __Y__
- Please choose the profile you want to use: __amplify-workshop-user__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a couple of new files & folders: __amplify__ & __aws-exports.js__. These files hold your project configuration.

### Configuring the React Native application

The next thing we need to do is to configure our React application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `aws-exports.js` file that is now in our root folder.

To configure the app, open __index.js__ and add the following code below the last import:

```js
import Amplify from 'aws-amplify'
import config from './aws-exports'
Amplify.configure(config)
```

Now, our app is ready to start using our AWS services.

## Adding Authentication

To add authentication, we can use the following command:

```sh
amplify add auth
```

- Do you want to use default authentication and security configuration? __Default configuration__   
- How do you want users to be able to sign in when using your Cognito User Pool? __Username__ (keep default) 
- What attributes are required for signing up? __Email__ (keep default)

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push
```

To view the new Cognito authentication service at any time after its creation, run the following command:

```sh
amplify console auth
```

### Using the withAuthenticator component

To add authentication, we'll go into __App.js__ and first import the `withAuthenticator` HOC (Higher Order Component) from `aws-amplify-react`:

```js
import { withAuthenticator } from 'aws-amplify-react-native'
```

Next, we'll wrap our default export (the App component) with the `withAuthenticator` HOC:

```js
export default withAuthenticator(App, {
  includeGreetings: true
})
```

Now, we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up & sign in.

```sh
react-native run-ios

# or if running android

react-native run-android

# or, if using expo

expo start
```

### Accessing User Data

We can access the user's info now that they are signed in by calling `Auth.currentAuthenticatedUser()`.

```js
import { Auth } from 'aws-amplify'

async componentDidMount() {
  const user = await Auth.currentAuthenticatedUser()
  console.log('user:', user)
}
```

### Using the Auth class to sign out

We can also sign the user out using the `Auth` class & calling `Auth.signOut()`. This function returns a promise that is fulfilled after the user session has been ended & AsyncStorage is updated.

Because `withAuthenticator` holds all of the state within the actual component, we must have a way to rerender the actual `withAuthenticator` component by forcing React to rerender the parent component.

To do so, let's make a few updates:

```js
import React from 'react'
import { View, Text } from 'react-native'
import { withAuthenticator } from 'aws-amplify-react-native'

function App(props) {
  function signOut() {
    Auth.signOut()
      .then(() => props.onStateChange('signedOut', null))
      .catch(err => console.log('err: ', err))
  }
  
  return (
    <View>
      <Text>Hello World</Text>
      <Text onPress={signOut}>Sign Out</Text>
    </View>
  )
}

export default withAuthenticator(App)
```

### Custom authentication strategies

The `withAuthenticator` component is a really easy way to get up and running with authentication, but in a real-world application we probably want more control over how our form looks & functions.

Let's look at how we might create our own authentication flow.

To get started, we would probably want to create input fields that would hold user input data in the state. For instance when signing up a new user, we would probably need 4 user inputs to capture the user's username, email, password, & phone number.

To do this, we could create some initial state for these values & create an event handler that we could attach to the form inputs:

```js
// initial state
state = {
  username: '', password: '', email: '', phone_number: ''
}

// event handler
onChangeText = (key, value) => {
  this.setState({ [key]: value })
}

// example of usage with TextInput
<TextInput
  placeholder='username'
  value={this.state.username}
  style={{ width: 300, height: 50, margin: 5, backgroundColor: "#ddd" }}
  onChangeText={v => this.onChange('username', v)}
/>
```

We'd also need to have a method that signed up & signed in users. We can us the Auth class to do thi. The Auth class has over 30 methods including things like `signUp`, `signIn`, `confirmSignUp`, `confirmSignIn`, & `forgotPassword`. Thes functions return a promise so they need to be handled asynchronously.

```js
// import the Auth component
import { Auth } from 'aws-amplify'

// Class method to sign up a user
signUp = async() => {
  const { username, password, email, phone_number } = this.state
  try {
    await Auth.signUp({ username, password, attributes: { email, phone_number }})
  } catch (err) {
    console.log('error signing up user...', err)
  }
}
```

## Adding a GraphQL API with AWS AppSync

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the above mentioned services __GraphQL__   
- Provide API name: __RestaurantAPI__   
- Choose an authorization type for the API __API key__   
- Do you have an annotated GraphQL schema? __N__   
- Do you want a guided schema creation? __Y__   
- What best describes your project: __Single object with fields (e.g. “Todo” with ID, name, description)__   
- Do you want to edit the schema now? (Y/n) __Y__   

> When prompted, update the schema to the following:   

```graphql
type Restaurant @model {
  id: ID!
  clientId: String
  name: String!
  description: String!
  city: String!
}
```

Next, let's push the configuration to our account:

```bash
amplify push
```

- Do you want to generate code for your newly created GraphQL API: __Y__
- Choose the code generation language target:  <Your target>
- Enter the file name pattern of graphql queries, mutations and subscriptions: __(src/graphql/**/*.js)__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions: __Y__
- Enter maximum statement depth [increase from default if your schema is deeply nested]: __(2)__


To view the new AWS AppSync API at any time after its creation, run the following command:

```sh
amplify console api
```

### Adding mutations from within the AWS AppSync Console

In the AWS AppSync console, open your API & then click on Queries.

Execute the following mutation to create a new restaurant in the API:

```graphql
mutation createRestaurant {
  createRestaurant(input: {
    name: "Nobu"
    description: "Great Sushi"
    city: "New York"
  }) {
    id name description city
  }
}
```

Now, let's query for the restaurant:

```graphql
query listRestaurants {
  listRestaurants {
    items {
      id
      name
      description
      city
    }
  }
}
```

We can even add search / filter capabilities when querying:

```graphql
query searchRestaurants {
  listRestaurants(filter: {
    city: {
      contains: "New York"
    }
  }) {
    items {
      id
      name
      description
      city
    }
  }
}
```

### Interacting with the GraphQL API from our client application - Querying for data

Now that the GraphQL API is created we can begin interacting with it!

The first thing we'll do is perform a query to fetch data from our API.

To do so, we need to define the query, execute the query, store the data in our state, then list the items in our UI.


```js
// App.js

// imports from Amplify library
import { API, graphqlOperation } from 'aws-amplify'

// import the query
import { listRestaurants } from './src/graphql/queries'

// define some state to hold the data returned from the API
state = {
  restaurants: []
}

// execute the query in componentDidMount
async componentDidMount() {
  try {
    const restaurantData = await API.graphql(graphqlOperation(listRestaurants))
    console.log('restaurantData:', restaurantData)
    this.setState({
      restaurants: restaurantData.data.listRestaurants.items
    })
  } catch (err) {
    console.log('error fetching restaurants...', err)
  }
}

// add UI in render method to show data
  {
    this.state.restaurants.map((restaurant, index) => (
      <View key={index}>
        <Text>{restaurant.name}</Text>
        <Text>{restaurant.description}</Text>
        <Text>{restaurant.city}</Text>
      </View>
    ))
  }
```

## Performing mutations

 Now, let's look at how we can create mutations.

```js
// additional imports
import {
  // ...existing imports
  TextInput, Button
} from 'react-native'
import { graphqlOperation, API } from 'aws-amplify'
import uuid from 'uuid/v4'
const CLIENTID = uuid()

// import the mutation
import { createRestaurant } from './src/graphql/mutations'

// add name & description fields to initial state
state = {
  name: '', description: '', city: '', restaurants: []
}

createRestaurant = async() => {
  const { name, description, city  } = this.state
  const restaurant = {
    name, description, city, clientId: CLIENTID
  }
  
  const updatedRestaurantArray = [...this.state.restaurants, restaurant]
  this.setState({
    restaurants: updatedRestaurantArray,
    name: '', description: '', city: ''
    })
  try {
    await API.graphql(graphqlOperation(createRestaurant, {
      input: restaurant
    }))
    console.log('item created!')
  } catch (err) {
    console.log('error creating restaurant...', err)
  }
}

// change state then user types into input
onChange = (key, value) => {
  this.setState({ [key]: value })
}

// add UI with event handlers to manage user input
<TextInput
  onChangeText={v => this.onChange('name', v)}
  value={this.state.name}
  style={{ height: 50, margin: 5, backgroundColor: "#ddd" }}
/>
<TextInput
  style={{ height: 50, margin: 5, backgroundColor: "#ddd" }}
  onChangeText={v => this.onChange('description', v)}
  value={this.state.description}
/>
<TextInput
  style={{ height: 50, margin: 5, backgroundColor: "#ddd" }}
  onChangeText={v => this.onChange('city', v)}
  value={this.state.city}
/>
<Button onPress={this.createRestaurant} title='Create Restaurant' />

// update styles to remove alignItems property
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    justifyContent: 'center',
  },
});
```

### GraphQL Subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to define the subscription, listen for the subscription, & update the state whenever a new piece of data comes in through the subscription.

```js
// import the subscription
import { onCreateRestaurant } from './src/graphql/subscriptions'

// define the subscription in the class
subscription = {}

// subscribe in componentDidMount
componentDidMount() {
  this.subscription = API.graphql(
    graphqlOperation(onCreateRestaurant)
  ).subscribe({
      next: eventData => {
        console.log('eventData', eventData)
        const restaurant = eventData.value.data.onCreateRestaurant
        if(CLIENTID === restaurant.clientId) return
        const restaurants = [...this.state.restaurants, restaurant]
        this.setState({ restaurants })
      }
  });
}

// remove the subscription in componentWillUnmount
componentWillUnmount() {
  this.subscription.unsubscribe()
}
```

## Challenge

Recreate this functionality in Hooks

> For direction, check out the tutorial [here](https://medium.com/open-graphql/react-hooks-for-graphql-3fa8ebdd6c62)

> For the solution to this challenge, view the [hooks](hooks.js) file. __Note__ that Expo does not yet support hooks.

## Adding a Serverless Function

### Adding a basic Lambda Function

To add a serverless function, we can run the following command:

```sh
amplify add function
```

> Answer the following questions

- Provide a friendly name for your resource to be used as a label for this category in the project: __basiclambda__
- Provide the AWS Lambda function name: __basiclambda__
- Choose the function template that you want to use: __Hello world function__
- Do you want to edit the local lambda function now? __Y__

> This should open the function package located at __amplify/backend/function/basiclambda/src/index.js__.

Edit the function to look like this, & then save the file.

```js
exports.handler = function (event, context) {
  console.log('event: ', event)
  const body = {
    message: "Hello world!"
  }
  const response = {
    statusCode: 200,
    body
  }
  context.done(null, response);
}
```

Next, we can test this out by running:

```sh
amplify function invoke basiclambda
```

- Provide the name of the script file that contains your handler function: __index.js__
-  Provide the name of the handler function to invoke: __handler__

You'll notice the following output from your terminal:

```sh
Running "lambda_invoke:default" (lambda_invoke) task

event:  { key1: 'value1', key2: 'value2', key3: 'value3' }

Success!  Message:
------------------
{"statusCode":200,"body":{"message":"Hello world!"}}

Done.
Done running invoke function.
```

_Where is the event data coming from? It is coming from the values located in event.json in the function folder (__amplify/backend/function/basiclambda/src/event.json__). If you update the values here, you can simulate data coming arguments the event._

Feel free to test out the function by updating `event.json` with data of your own.

### Adding a function running an express server

Next, we'll build a function that will be running an [Express](https://expressjs.com/) server inside of it.

This new function will fetch data from a cryptocurrency API & return the values in the response.

To get started, we'll create a new function:

```sh
amplify add function
```

> Answer the following questions

- Provide a friendly name for your resource to be used as a label for this category in the project: __cryptofunction__
- Provide the AWS Lambda function name: __cryptofunction__
- Choose the function template that you want to use: __Serverless express function (Integration with Amazon API Gateway)__
- Do you want to edit the local lambda function now? __Y__

> This should open the function package located at __amplify/backend/function/cryptofunction/src/index.js__.

Here, we'll add the following code & save the file:

```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*")
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept")
  next()
});
// below the last app.use() method, add the following code 👇
const axios = require('axios')

app.get('/coins', function(req, res) {
  let apiUrl = `https://api.coinlore.com/api/tickers?start=0&limit=10`
  
  if (req.apiGateway && req.apiGateway.event.queryStringParameters) {
    const { start = 0, limit = 10 } = req.apiGateway.event.queryStringParameters
    apiUrl = `https://api.coinlore.com/api/tickers/?start=${start}&limit=${limit}`
  }
  axios.get(apiUrl)
    .then(response => {
      res.json({
        coins: response.data.data
      })
    })
    .catch(err => res.json({ error: err }))
})
```

Next, we'll install axios in the function package:

```sh
cd amplify/backend/function/cryptofunction/src

npm install axios

cd ../../../../../
```

Next, change back into the root directory.

Now we can test this function out:

```sh
amplify function invoke cryptofunction
```

This will start up the node server. We can then make `curl` requests agains the endpoint:

```sh
curl 'localhost:3000/coins'
```

If we'd like to test out the query parameters, we can update the __event.json__ to add the following:

```json
{
    "httpMethod": "GET",
    "path": "/coins",
    "queryStringParameters": {
        "start": "0",
        "limit": "1"
    }
}
```

When we invoke the function these query parameters will be passed in & the http request will be made immediately.

## Adding a REST API

Now that we've created the cryptocurrency Lambda function let's add an API endpoint so we can invoke it via http.

To add the REST API, we can use the following command:

```sh
amplify add api
```

> Answer the following questions

- Please select from one of the above mentioned services __REST__   
- Provide a friendly name for your resource that will be used to label this category in the project: __cryptoapi__   
- Provide a path, e.g. /items __/coins__   
- Choose lambda source __Use a Lambda function already added in the current Amplify project__   
- Choose the Lambda function to invoke by this path: __cryptofunction__   
- Restrict API access __Y__
- Who should have access? __Authenticated users only__
- What kind of access do you want for Authenticated users __read/create/update/delete__
- Do you want to add another path? (y/N) __N__     

Now the resources have been created & configured & we can push them to our account: 

```bash
amplify push
```

### Interacting with the new API

Now that the API is created we can start sending requests to it & interacting with it.

Let's request some data from the API:

```js
// src/App.js
import React from 'react'
import { API } from 'aws-amplify'
import { withAuthenticator } from 'aws-amplify-react'

class App extends React.Component {
  state = {
    coins: []
  }
  async componentDidMount() {
    try {
      // const data = await API.get('cryptoapi', '/coins')
      const data = await API.get('cryptoapi', '/coins?limit=5&start=100')
      console.log('data from Lambda REST API: ', data)
      this.setState({ coins: data.coins })
    } catch (err) {
      console.log('error fetching data..', err)
    }
  }
  render() {
    return (
      <div>
        {
          this.state.coins.map((c, i) => (
            <div key={i}>
              <h2>{c.name}</h2>
              <p>{c.price_usd}</p>
            </div>
          ))
        }
      </div>
    )
  }
}

export default withAuthenticator(App, { includeGreetings: true })
```

## Adding Analytics

To add analytics, we can use the following command:

```sh
amplify add analytics
```

> Next, we'll be prompted for the following:

- Provide your pinpoint resource name: __amplifyanalytics__   
- Apps need authorization to send analytics events. Do you want to allow guest/unauthenticated users to send analytics events (recommended when getting started)? __Y__   
- overwrite YOURFILEPATH-cloudformation-template.yml __Y__

### Recording events

Now that the service has been created we can now begin recording events.

To record analytics events, we need to import the `Analytics` class from Amplify & then call `Analytics.record`:

```js
import { Analytics } from 'aws-amplify'

state = {username: ''}

async componentDidMount() {
  try {
    const user = await Auth.currentAuthenticatedUser()
    this.setState({ username: user.username })
  } catch (err) {
    console.log('error getting user: ', err)
  }
}

recordEvent = () => {
  Analytics.record({
    name: 'My test event',
    attributes: {
      username: this.state.username
    }
  })
}

<Button onPress={this.recordEvent} title='Record Event' />
```

## Working with Storage

To add storage, we can use the following command:

```sh
amplify add storage
```

> Answer the following questions   

- Please select from one of the below mentioned services __Content (Images, audio, video, etc.)__
- Please provide a friendly name for your resource that will be used to label this category in the
 project: __YOURAPINAME__
- Please provide bucket name: __YOURBUCKETNAME__
- Who should have access: __Auth users only__
- What kind of access do you want for Authenticated users __read/write__   


```sh
amplify push
```

Now, storage is configured & ready to use.

What we've done above is created configured an Amazon S3 bucket that we can now start using for storing items.

For example, if we wanted to test it out we could store some text in a file like this:

```js
import { Storage } from 'aws-amplify'

// create function to work with Storage
addToStorage = () => {
  Storage.put('javascript/MyReactComponent.js', `
    import React from 'react'
    const App = () => (
      <p>Hello World</p>
    )
    export default App
  `)
    .then (result => {
      console.log('result: ', result)
    })
    .catch(err => console.log('error: ', err));
}

// add click handler
<Button onPress={this.addToStorage} title='Add to Storage' />
```

This would create a folder called `javascript` in our S3 bucket & store a file called __MyReactComponent.js__ there with the code we specified in the second argument of `Storage.put`.

If we want to read everything from this folder, we can use `Storage.list`:

```js
readFromStorage = () => {
  Storage.list('javascript/')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error fetching from S3', err))
}
```

If we only want to read the single file, we can use `Storage.get`:

```js
readFromStorage = () => {
  Storage.get('javascript/MyReactComponent.js')
    .then(data => {
      console.log('data from S3: ', data)
      fetch(data)
        .then(r => r.text())
        .then(text => {
          console.log('text: ', text)
        })
        .catch(e => console.log('error fetching text: ', e))
    })
    .catch(err => console.log('error fetching from S3', err))
}
```

If we wanted to pull down everything, we can use `Storage.list`:

```js
readFromStorage = () => {
  Storage.list('')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error fetching from S3', err))
}
```

## Removing Services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth

amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.