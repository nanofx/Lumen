
<div align="center">
<h2>Building a social network using React.js and Firebase</h2>
</div>

<p align="center">
 <img src="https://i.imgur.com/JISdJFy.png" alt="hero">
</p>

<p align="center">
 <img src="https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg" alt="badge">
</p>

### [Live Demo](https://csb-zxq0yon9m4-amzzgprnss.now.sh/)

### Introduction

We’re going to be creating a basic list of items that you do CRUD to - create, read (update and delete in the next chapter). The feature itself is nothing special, it’s really just a skeleton that demonstrates how to do the most common operations with React and Firebase. 

*Here are a few important concepts for this lesson.*

Firebase is a real-time **NoSQL** cloud database (but also `cloud storage` , `messaging` and `authentication service`) that helps you build apps without building the backend. It abstracts away most of your complex server-side features. You can save and retrieve **JSON** objects, build user authentication, and get data updates in real-time across connected devices.

This tutorial will take you from zero to a realtime React app with just a few lines of code. This tutorial assumes you have Node.js and NPM installed on your local system and a basic understanding of how React works and ES6 syntax. First thing we need to do is create a fresh new app, let's call it lumen. 

##### Kicking things off from the command line:


    npx create-react-app lumen
    cd lumen

Installing Firebase and Bootstrap

    yarn add firebase bootstrap reactstrap
    yarn start

Lets's create a basic form so we can handle user authentication:

    <div class="form">
        <input type="text" onChange={this.handleChange}
	         placeholder="Email" name="email" autoComplete="username" />
        <input type="password" onChange={this.handleChange} 
	        placeholder="Password" name="password" autoComplete="current-password" />
	        
        {this.state.loading ? <p>Loading ...</p> : null}
        
        <button onClick={this.login}>login</button>
        <button onClick={this.signup}>signup</button>
        
        <p class="message">
          Your password must not contain spaces, special characters, or emojis. 
          By continuing you agree to our <a href="#">terms of services</a>
        </p>
    </div>
    
 <img src="https://i.imgur.com/kv791qG.jpg" alt="login">

Before writing any more code, let's setup our Firebase project and config file: 

 1. Go to https://console.firebase.google.com/ 
 2. Create new project 
 3. Click on Auth then 'Set up sign-in method'

In this example we'll only need email/password method
So go ahead and enable that bad boy
On the top right you'll see a "Web Setup" button click on that.

Copy paste your credentials using the template below and save the file as `Fire.js` in your root folder:

    import firebase from 'firebase';
    
    const config = {
      apiKey: 'xxxxxx',
      authDomain: 'xxxxxx',
      databaseURL: 'xxxxxx',
      projectId: 'xxxxxx',
      storageBucket: 'xxxxxx',
      messagingSenderId: 'xxxxxx'
    };
    const fire = firebase.initializeApp(config);
    export default fire;

 1. Go back to Firebase website and navigate to Database
 2. Create a new one and choose "Start in test mode"

<p align="center">
 <img src="https://i.imgur.com/F5xrrJQ.jpg" alt="db">
</p>

That's it you have a fully working db!
You can later change those settings when you're ready to deploy.

Now we can add more code to our app, let's focus on the login form for a moment, you can see that we have two inputs and two buttons for signin/signup, that's the bare minimum for a function auth flow. You may add forget password if you wish, google provide everything to make that process very easy but we're not covering this today.

Let's initialize a `constructor()` with empty values:

      constructor(props) {
        super(props);
        this.state = {
          email: '',
          password: '',
          loading: false
        };
      }

We need a way to handle user input so let's add this function:

      handleChange = e => {
        this.setState({ [e.target.name]: e.target.value });
      };

>   As you can see I'm using **arrow functions** and that's because manual
> binding is confusing for beginners but you should definitely [learn more](https://reactjs.org/docs/faq-functions.html)
> about the subject so you can understand what's going on.

### LogIn And SingUp functions

Do not forget to import your `Fire.js` config file

    import fire from './Fire';
Now jump to your class and add our main auth functions:

  

    login = e => {
        this.setState({ loading: true });
        e.preventDefault();
        fire
          .auth()
          .signInWithEmailAndPassword(this.state.email, this.state.password)
          .then(u => {
            this.setState({ loading: false });
          })
          .catch(error => {
            console.log(error);
          });
      };
    
      signup = e => {
        this.setState({ loading: true });
        e.preventDefault();
        fire
          .auth()
          .createUserWithEmailAndPassword(this.state.email, this.state.password)
          .then(u => {})
          .then(u => {
            this.setState({ loading: false });
          })
          .catch(error => {
            console.log(error);
          });
      };
<p align="center">
 <img src="https://i.imgur.com/MIiERVC.jpg" alt="db">
</p>
Boom! We got our first user. Let's move to the next step.

### App Scaffold

Now that everyone can create an account and login to our app, we need a better structure so let's create a folder called `routes` where we're going to put our two mains screens

*Home.js ( social feed )
Login.js* ( auth page )

 <img src="https://i.imgur.com/zyVrXUu.jpg" alt="app">

Don't forget to fix import paths since we're moving our files inside a folder!
Let’s make a few updates to `App.js` to let React decide which screen to display based on Auth status:

    class App extends Component {
      constructor(props) {
        super(props);
        this.state = {
          error: null,
          loggedIn: null
        };
      }
      componentDidMount = () => {
        this.setState({ loading: true });
        fire.auth().onAuthStateChanged(user => {
          if (user) {
            this.setState({ loggedIn: true, loading: false });
          } else {
            this.setState({ loggedIn: false, loading: false });
          }
        });
      };
    
      render() {
        return (
          <div>
           {this.state.loggedIn ? <Home /> : <Login />}
          </div>
        );
      }
    }
    
    export default App;


You will notice that if the user is logged in, the app will show `<Home /`> otherwise the `<Login />` form will be displayed.  Adding a loading text or spinner would also be great.

> This is an easy way to handle AuthChange but it's not suitable for
> production, we're using it for demonstration purpose and learn how to
> communicate with Firebase services.

In this example we'll create basic social cards with a title, description, an image and the author name. Open `Home.js` file and let's write some code:

Starting by importing firebase config file and a bunch of components from bootstrap:

    import fire from '../Fire';
    import { Card, CardImg, CardText,
	    CardBody, CardTitle, CardSubtitle, 
	    CardFooter, CardColumns, Button,
	    Container, Row, Col } from 'reactstrap';

### Adding data to Firebase

Like our login page we need a form, some buttons and of course a function to handle user input and send data to Google.  The code below is our ui elements: 

    <input value={this.state.title} onChange={this.handleChange} name="title"
	     class="form-control mb-2" placeholder="Title" required />
	     
    <input value={this.state.picture} onChange={this.handleChange} name="picture" 
	    class="form-control mb-2" placeholder="Picture URL" required />
	    
    <textarea row="9" value={this.state.description} onChange={this.handleChange} name="description" 
	    class="form-control mb-2" placeholder="Description" required />

The `handleChange` is similar to the last one so  just copy that in your `Home.js` file and initialize values in constructor:

     constructor(props) {
        super(props);
        this.state = {
          loading: true,
          title: '',
          description: '',
          picture: '',
          list: [],
          keys: [],
          show: false
        };
      }
      
`list` is an empty array where we're going to store our social cards returned from the database, in this chapter we're not going to use `keys`, it's where the id keys are stored, we need it to identify an item so we can update it or delete it. We"ll cover that in the next chapter.

Our form is ready but the application still don't know how to send data to firebase, the function below is where all the magic happens, I decided to call it `new` but you can choose whatever you want

      new = e => {
        this.setState({ loading: true });
        e.preventDefault();
        let title = this.state.title;
        let description = this.state.description;
        let picture = this.state.picture;
        const { currentUser } = fire.auth();
        fire
          .database()
          .ref(`feed/`)
          .push({ title, description, picture, user: currentUser.email })
          .then(() => {
            this.setState({ loading: false });
          });
      };


.ref(`feed/`) is the path where we choose to put our data.

Before you can test your app add this button to your view:

    <Button color="dark" onClick={this.new} className="mb-5" block>
      SUBMIT
    </Button>

If you try testing it you'll run into some issues and the data won't show up because you have to configure the database rules, so let's open the [firebase console](https://console.firebase.google.com/) and navigate to Rules tab: 

Change the default values to `true` like so:

    {
      "rules": {
        ".read": true,
        ".write": true
      }
    }

Now if you click on create post button you should see the data submitted in your firebase console, remember that we're using **Realtime Database** and not Firestore. You can eventually CRUD values right there but it's not our goal. 

![db](https://i.imgur.com/ltJ8EKk.jpg)

### Retrieving data from Firebase

Awesome! Now that everything is working correctly let's create our social network, in order to to that we have to display data on page load and for that we use the famous`componentDidMount()`
 

    componentDidMount = () => {
        this.setState({ loading: true });
        const { currentUser } = fire.auth();
        fire
          .database()
          .ref(`/feed/`)
          .on('value', snapshot => {
            var obj = snapshot.val();
            var list = [];
            var keys = [];
            for (let a in obj) {
              list.push(obj[a]);
              keys.push(a);
            }
            this.setState({
              list: list,
              keys: keys,
              loading: false
            });
          });
      };

  Pretty easy, right?  Notice that `snapshot.val()` is our actual values. 
The logout function is very straightforward to implement, it's just one line of code:

    logout = () => { 
	    fire.auth().signOut();
    };

### User Interface
![main-app](https://i.imgur.com/VFviw5j.png)


Putting our `list` in a separate constant for best practices and good code readability, also `list` can be named whatever, I choose list for the sake of simplicity.

    const listItems = this.state.list.map((item, index) => (
      <Card style={{ borderWidth: 0, borderRadius: 8, marginBottom: 25 }}>
        <CardImg top width="100%" id="mainImage" src={item.picture} style={{ borderTopLeftRadius: 8, borderTopRightRadius: 8 }} />
        <CardBody>
          <CardTitle>{item.title}</CardTitle>
          <CardText>{item.description}</CardText>
        </CardBody>
        <CardFooter>Posted by {item.user}</CardFooter>
      </Card>
    ));

Now render that:

      {this.state.loading ? <h6 class="mb-5">Loading ...</h6> : null}
      <CardColumns className="mb-5">{listItems}</CardColumns>
      
#### Addtional Tweaks

I love to add some interactivity to my apps and that's why I hide the forms and display it when the user decide to, to make this works nothing complicated, just some changes to the state and we're good to go:

    {!this.state.show ? (    
    <Button color="dark" onClick={this.show} className="mb-5" block>
        CREATE POST
    </Button>
    ) : null}
    
    {this.state.show ? (
    <Button color="dark" onClick={this.new} className="mb-5" block>
	    SUBMIT
    </Button>    
    ) : null}
      
    {this.state.show ? (
    <SubmitForm />
    ) : null}

And then of course the `show()` function: 

    show = () => {
    this.setState({ show: !this.state.show });
    };

To give the web app a fancy UI you can copy my `app.css` file or create your own. The complete file can be found in the Github repository. 


    @import url(https://fonts.googleapis.com/css?family=Roboto:300);
    
    .form {
    	position: relative;
    	z-index: 1;
    	background: #ffffff;
    	max-width: 360px;
    	margin: 0 auto 100px;
    	padding: 45px;
    	text-align: center;
    	box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.2), 0 5px 5px 0 rgba(0, 0, 0, 0.24);
    }
    .form input {
    	font-family: 'Roboto', sans-serif;
    	outline: 0;
    	background: #f2f2f2;
    	width: 100%;
    	border: 0;
    	margin: 0 0 15px;
    	padding: 15px;
    	box-sizing: border-box;
    	font-size: 14px;
    }
    body {
    	background: -webkit-linear-gradient(right, #53d769, #fecb2e);
    }
    
    #mainImage {
    	object-fit: cover;
    	height: 200px;
    	background-color: #fff;
    }
Congrats! You now have a full-stack realtime React app that can scale to millions of users. Thanks Google! 

## Conclusion

This article should have given you a good introduction to components, state, working with form data, pulling data from Firebase services, and deploying an app. There is much more to learn and do with React, but I hope you feel confident  playing around with React and Firebase yourself now.

***If you enjoy my content, please consider supporting what I do***


<div align="center">
<h2>Chapter 2 (Update and Delete)</h2>
</div>

This is the second chapter of "Building a social network with Reajct.js and Firebase", in **[Chapter One](https://usfslk.github.io/lumen.html)** we covered the following topics:

 - Enable Firebase Auth and Database 
 - Handle forms data 
 - Login and create  an account 
 - Send and retrieve data using firebase

The app we built was a great starting point to understand how to communicate with Firebase services and use this platform as a cloud backend but most
of the big concepts of a social network wasn't there, like a public feed, editing and deleting posts, ect.. so in this article we'll focus on these features.

I pretty much ditched all the front-end code in the old repo because bootstrap wasn't really helping me, I came across an awesome library called
Semantic UI and that's what we're going to use. 

I did find the number of components greater than what's available in bootstrap, and also the customization was really fun because this ui library works great with React. The only downside is the complexity of responsive grids and changing default theme variables, not that straightforward compared to bootstrap.

> This is more an overview lesson than a step by step tutorial

PR are welcome, here's some quick and easy-to-implement features:

- Invert the feed list (most recent on top)
- Adding a username / displayName on signup
- Adding timestamp to user posts
- Create separate components files

Let's open a text editor and start writing some magic, first thing you'll notice is that we now have a public feed that anyone can view without creating an account which is pretty normal for a social network, second important thing is the ability to delete and edit personal posts. 

For the editing part, I didn't come up with an intuitive way to edit each post separately, I tried to keep it simple and not use props and dynamic navigation. Maybe in the future, I'll create a separate blog post to introduce you guys to this concept.

Anyway let's start with the header, nothing complicated there, few buttons to navigate in our app, as mentioned previously the ui components are now built with semantic-ui-react. Also you'll need react-router-dom in order to handle navigation so go ahead and install those libraries

    yarn add react-router-dom semantic-ui-react

Home

    <Button
      id="headerBTN"
      compact
      color="black"
      href="/"
    >
      Home
    </Button>


Login / Signup / Account

    <Button
      id="headerBTN"
      compact
      color="green"
      href="/account"
    >
      {this.state.loggedIn ? 'Account' : 'Log in'}
    </Button>

Next thing is to create main routes like so:

<Route exact path="/" component={Home} />
<Route path="/Account" component={Account} />

Of course you'll have to import those two pages in the top of your App.js and wrap everything inside `<Router> ... </Router>`

This app is basically a social network so when users visit the website they expect to see some kind of posts and that's exactly what we're going to build, like the function we used in the previous article, we need to retrieve data from Firebase but this time each user have a separate feed and that's a great solution because it makes our app scalable (if you wish to add 'follow' feature for example).

![enter image description here](https://i.stack.imgur.com/IOpOL.png)

The data is still coming form the same path but since each users have a separate feed we need to loop through a parent object that contains an object for each user which contains an array of posts, to render our view we have to write something similar to this:


        const listItems = this.state.list.map((item, index) =>
            Object.values(item).map(nestedItem => (
              <p>
                {nestedItem.title}
              </p>
            ))
          );

The social cards are very similar to the ones we created with bootstrap, the Semantic UI docs provide many great [examples](http://react.semantic-ui.com/views/card/) to quickstart your project, in this lesson we'll be using this:

    <Card>
      <Image
        id="mainIMG"
        src={nestedItem.picture}
      />
      <Card.Content>
        <Card.Header>
          {nestedItem.title}
        </Card.Header>
        <Card.Description>
          {nestedItem.description}
        </Card.Description>
      </Card.Content>
      <Card.Content extra>
        <a>
          <Icon name="user" />
          by {nestedItem.user}
        </a>
      </Card.Content>
    </Card>

That's it for the homepage, we'll keep it simple. Now navigate to account page and import this:

    import {
      Button,
      Form,
      Grid,
      Header,
      Image,
      Icon,
      Card,
      Loader,
      Container,
      Divider,
      Label,
      Feed,
    } from 'semantic-ui-react';

The first element on the page is the 'CREATE LUMEN' button that provide us the ability to send data to firebase, it's the same function used previously we only change the path of the database to create separate feeds, we can achieve that by requesting `loggedIn` user information:

    const { currentUser } = fire.auth(); 

and then inside the `new()` function we use:

    fire
      .database()
      .ref(`feed/${currentUser.uid}/`)
      .push({
        title,
        description,
        picture,
        user: currentUser.email,
      })

`auth()` method can also provide `displayName`, `email`, `photoURL`, `emailVerified` and `providerId`.

If you wish to learn more : https://firebase.google.com/docs/auth/web/manage-users

A header, icon and divider to create a separation between the compose component and the settings one. The main function of that blue button is to make the api call when the user decide to edit personal lumens, if that component was in a separate page we rather write the api call inside `componentDidMount`, anyhow here we are looking at the `account()` function, very similar to one we used on `Home.js`, the only difference is the database path.

To make the settings component appear onClick I used this:

    showSetting = () => {
      this.setState({
        showSetting: !this.state.showSetting,
        loading: true,
      });
      this.account();
    };

The `listItems` const is very long because I always promise to myself to keep things simple but end up with a hundred lines of code, this is a minimal version of it so you can understand what's going on:
![fun](https://i.imgur.com/QUy6npF.jpg)

    const listItems = this.state.list.map((item, index) => (
    <Card fluid>

        <h1>{item.title}</h1>
        <Image
          src={item.picture}
          size="tiny"
        />
        {item.description}

        <a onClick={() => this.delete(index)}
        class="ui white label">
          DELETE
        </a>

        {this.state.showEdit ? (
          <Form>
              <Form.Input
                onChange={this.handleChange}
                name="updateTitle"
                label="A cool title"
                placeholder={item.title}
                required
              />
              <Form.Input
                fluid
                onChange={this.handleChange}
                name="updatePicture"
                label="Picture URL"
                placeholder={item.picture}
                required
              />
            <Form.TextArea
              autoHeight
              rows={4}
              onChange={this.handleChange}
              name="updateDescription"
              label="What's on your mind?"
              placeholder={item.description}
              required
            />

            <Button
            onClick={() => this.update(index)}
            >
              UPDATE
            </Button>
          </Form>
        ) : null}

    </Card>
    ));

  

Two mains buttons there **DELETE** and **UPDATE**, first one we have to provide the *index* so we can tell firebase which post to delete and same thing for update:

    delete = index => {
      const { currentUser } = fire.auth();
      fire
        .database()
        .ref(
          `feed/${currentUser.uid}/${this.state.keys[index]}`
        )
        .remove();
    };

The technique below to edit posts is working but not suitable for large scale app, we'll try to focus on how to perform the operations and forget a little bit about user experience, so I create this function called `edit` to switch the state of **EDIT MODE**:

    edit = index => {
      this.setState({ showEdit: !this.state.showEdit });
    };

when it's enabled the form will appear below the post and then you click on the update button which execute this code:

    update = index => {
      const { currentUser } = fire.auth();
      this.setState({ loading: true });
      fire
        .database()
        .ref(
          `feed/${currentUser.uid}/${this.state.keys[index]}`
        )
        .update({
          title: this.state.updateTitle,
          description: this.state.updateDescription,
          picture: this.state.updatePicture,
        })
        .then(() => {
          this.setState({
            loading: false,
          });
        });
    };

You can see the full implementation [here](https://github.com/usfslk/Lumen)

***If you enjoy my content, please consider supporting what I do.***

<p align="center">
 <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="badge">
</p>


Thanks for reading!

