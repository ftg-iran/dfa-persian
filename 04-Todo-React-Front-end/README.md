<div dir="rtl">

# Chapter 4: Todo React Front-end

An API exists to communicate with another program. In this chapter we will consume our Todo
API from the last chapter via a [React](https://reactjs.org/) front-end so you can see how everything actually works
together in practice.




I’ve chosen to use React as it is currently the most popular JavaScript front-end library but the
techniques described here will also work with any other popular front-end framework including
[Vue](https://vuejs.org/), [Angular](https://angular.io/), or [Ember](https://emberjs.com/). They will even work with mobile apps for iOS or Android, desktop apps, or really anything else. The process of connecting to a back-end API is remarkably similar.


If you become stuck or want to learn more about what’s really happening with React, check out
the [official tutorial](https://reactjs.org/tutorial/tutorial.html) which is quite good.

### Install Node

We’ll start by configuring a React app as our front-end. First open up a new command line console
so there are now **two consoles open**. This is important. We need our existing Todo back-end
from the last chapter to still be running on the local server. And we will use the second console
to create and then run our React front-end on a separate local port. This is how we locally mimic
what a production setting of a dedicated and deployed front-end/back-end would look like.


In the new, second command line console install [NodeJS](https://nodejs.org/en/), which is a JavaScript runtime engine.
It lets us run JavaScript outside of a web browser.


On a Mac computer we can use [Homebrew](https://brew.sh/), which should already be installed if you followed
the [Django for Beginners instructions](https://djangoforbeginners.com/initial-setup/) for configuring your local computer


<div dir="ltr">

Command Line

```shell
$ brew install node
```

</div>


On a Windows computer there are multiple approaches but a popular one is to use [nvm-windows](https://github.com/coreybutler/nvm-windows). Its repository contains detailed, up-to-date installation instructions.


If you are on Linux use [nvm](https://github.com/nvm-sh/nvm). As of this writing, the command can be done using either cURL:


<div dir="ltr">

Command Line

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/\
install.sh | bash
```

</div>


or using Wget



<div dir="ltr">

Command Line

```shell
$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/\
install.sh | bash
```

</div>


Then run:

<div dir="ltr">

Command Line

```shell
$ command -v nvm
```

</div>

Close your current command line console and open it again to complete installation.



### Install React

We will use the excellent [create-react-app](https://github.com/facebookincubator/create-react-app) package to quickly start a new React project. This
will generate our project boilerplate **and** install the required dependencies with one command!


To install React we’ll rely on [npm](https://www.npmjs.com/) which is a JavaScript package manager. Like pipenv for Python,
npm makes managing and installing multiple software packages much, much simpler. Recent
versions of npm also include [npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b), which is an improved way to install packages locally without
polluting the global namespace. It’s the recommended way to install React and what we’ll use
here!


Make sure you are in the correct directory by navigating to the Desktop (if on a Mac) and then
the todo folder.


<div dir="ltr">

Command Line

```shell
$ cd ~/Desktop
$ cd todo
```

</div>


Create a new React app called frontend.

<div dir="ltr">

Command Line

```shell
$ npx create-react-app frontend
```

</div>


Your directory structure should now look like the following:


<div dir="ltr">

Diagram

```code
todo
|   ├──frontend
|       ├──React...
|   ├──backend
|       ├──Django...
```

</div>

Change into our frontend project and run the React app with the command npm start.

<div dir="ltr">

Command Line

```shell
$ cd frontend
$ npm start
```

</div>

If you navigate to http://localhost:3000/ you will see the create-react-app default homepage.

![React welcome page](images/1.png)



### Mock data

If you go back to our API endpoint you can see the raw JSON in the browser at:

http://127.0.0.1:8000/api/?format=json


<div dir="ltr">

Code
```json
[
    {
        "id":1,
        "title":"1st todo",
        "body":"Learn Django properly."
    },
    {
        "id":2,
        "title":"Second item",
        "body":"Learn Python."
    },
    {
        "id":3,
        "title":"Learn HTTP",
        "body":"It's important."
    }
]

```

</div>


This is returned whenever a GET request is issued to the API endpoint. Eventually we will consume
the API directly but a good initial step is to mock the data first, then configure our API call.

The only file we need to update in our React app is `src/App.js`. Let’s start by mocking up the API
data in a variable named list which is actually an array with three values.


<div dir="ltr">

Code
```js
// src/App.js
import React, { Component } from 'react';

const list = [
    {
        "id":1,
        "title":"1st todo",
        "body":"Learn Django properly."
    },
    {
        "id":2,
        "title":"Second item",
        "body":"Learn Python."
    },
    {
        "id":3,
        "title":"Learn HTTP",
        "body":"It's important."
    }
]

```

</div>

Next we load the `list` into our component’s state and then use the JavaScript array method
[map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) to display all the items.


I’m deliberately moving fast here so if you’ve never used React before, just copy the code so that
you can see how it “would” work to wire up a React front-end to our Django back-end.


Here’s the complete code to include in the `src/App.js` file now.


<div dir="ltr">

Code
```js
// src/App.js
import React, { Component } from 'react';

const list = [
    {
        "id":1,
        "title":"1st todo",
        "body":"Learn Django properly."
    },
    {
        "id":2,
        "title":"Second item",
        "body":"Learn Python."
    },
    {
        "id":3,
        "title":"Learn HTTP",
        "body":"It's important."
    }
]

class App extends Component {
    constructor(props) {
        super(props);
        this.state = { list };
    }
render() {
    return (
        <div>
            {this.state.list.map(item => (
                <div key={item.id}>
                    <h1>{item.title}</h1>
                    <p>{item.body}</p>
                </div>
            ))}
        </div>
        );
    }
}

export default App;

```

</div>



We have loaded list into the state of the App component, then we’re using map to loop over each
item in the list displaying the `title` and `body` of each. We’ve also added the id as a key which is a
React-specific requirement; the id is automatically added by Django to every database field for
us.

You should now see our todos listed on the homepage at http://localhost:3000/ without
needing to refresh the page.

![Dummy data](images/2.png)

**Note**: If you spend any time working with React, it’s likely at some point you will see the error
message sh: `react-scripts: command not found while running npm start`. Don’t be alarmed.
This is a very, very common issue in JavaScript development. The fix is typically to run `npm install`
and then try `npm start` again. If that does not work, then delete your `node_modules`
folder and run `npm install`. That solves the issue 99% of the time. Welcome to modern JavaScript
development :).


### Django REST Framework + React

Now let’s hook into our Todo API for real instead of using the mock data in the list variable. In
the other command line console our Django server is running and we know the API endpoint
listing all todos is at http://127.0.0.1:8000/api/. So we need to issue a GET request to it.


There are two popular ways to make HTTP requests: with the [built-in Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) or with [axios](https://github.com/axios/axios),
which comes with several additional features. We will use axios in this example. Stop the React
app currently running on the command line with `Control+c`. Then install `axios`.



<div dir="ltr">

Command Line

```shell
$ npm install axios
```

</div>


Start up the React app again using npm start.


<div dir="ltr">

Command Line

```shell
$ npm start
```

</div>


Then in your text editor at the top of the App.js file import Axios.


<div dir="ltr">

Code
```js
// src/App.js
import React, { Component } from 'react';
import axios from 'axios'; // new
...

```

</div>

There are two remaining steps. First, we’ll use axios for our GET request. We can make a dedicated
getTodos function for this purpose.


Second, we want to make sure that this API call is issued at the correct time during the React
lifecycle. HTTP requests should be made using [componentDidMount](https://reactjs.org/docs/state-and-lifecycle.html) so we will call getTodos there.

We can also delete the mock list since it is no longer needed. Our complete `App.js` file will now
look as follows.


<div dir="ltr">

Code
```js
// src/App.js
import React, { Component } from 'react';
import axios from 'axios'; // new

class App extends Component {
    state = {
        todos: []
    };

    // new
    componentDidMount() {
    this.getTodos();
    }
    
// new
    getTodos() {
        axios
            .get('http://127.0.0.1:8000/api/')
            .then(res => {
            this.setState({ todos: res.data });
            })
            .catch(err => {
                console.log(err);
        });
    }

    render() {
        return (
            <div>
                {this.state.todos.map(item => (
                    <div key={item.id}>
                        <h1>{item.title}</h1>
                        <span>{item.body}</span>
                    </div>
                ))}
            </div>
            );
        }
    }

export default App;

```

</div>

If you look again at http://localhost:3000/ the page is the same even though we no longer
have hardcoded data. It all comes from our API endpoint and request now.

![API Data](images/3.png)

And that is how it’s done with React!

### Conclusion

We have now connected our Django back-end API to a React front-end. Even better, we have
the option to update our front-end in the future or swap it out entirely as project requirements
change.


This is why adopting an API-first approach is a great way to “future-proof” your website. It may
take a little more work upfront, but it provides much more flexibility. In later chapters we will
enhance our APIs so they support multiple HTTP verbs such as POST (adding new todos), PUT
(updating existing todos), and DELETE (removing todos).


In the next chapter we will start building out a robust Blog API that supports full CRUD (CreateRead-Update-Delete) functionality and later on add user authentication to it so users can log in,
log out, and sign up for accounts via our API.


</div>