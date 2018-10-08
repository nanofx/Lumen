## Lumen

We’re going to be building a basic list of items that you do CRUD to - create, read (update and delete in the next chapter). The feature itself is nothing special, it’s really just a skeleton that demonstrates how to do the most common operations with React and Firebase. 

Here are a few important concepts for this lesson.

Firebase is a real-time NoSQL cloud database (but also cloud storage and messaging and authentication service) that helps you build apps without building the backend. It abstracts away most of your complex server-side features. You can save and retrieve JSON objects, build user authentication, and get data updates in real-time across connected devices.

This tutorial will take you from zero to a realtime React app with just a few lines of code. This tutorial assumes you have Node.js and NPM installed on your local system. First thing we need to do is create a fresh new app, let's call it lumen:

    npx create-react-app lumen
    cd lumen

Installing Firebase and Bootstrap
Let’s kick things off from the command line:

    yarn add firebase bootstrap reactstrap
    yarn start

Lets's create a basic form so we can handle user authentication:

    <div class="form">
        <input type="text" onChange={this.handleChange} placeholder="Email" name="email" autoComplete="username" />
        <input type="password" onChange={this.handleChange} placeholder="Password" name="password" autoComplete="current-password" />
        {this.state.loading ? <p>Loading ...</p> : null}
        <button onClick={this.login}>login</button>
        <div class="m-3" />
        <button onClick={this.signup}>signup</button>
        <p class="message">
          Your password must not contain spaces, special characters, or emojis. By continuing you agree to our <a href="#">terms of services</a>
        </p>
    </div>
![login-form](https://i.imgur.com/uhJWgTm.jpg)

Before writing any more code, let's setup our Firebase project and config file: 

 1. Go to https://console.firebase.google.com/ 
 2. Create new project 
 3. Click on Auth then 'Set up sign-in method'

In this example we'll only need email/password
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

![firebase-db](https://i.imgur.com/F5xrrJQ.jpg)

That's it you have a fully working db!
You can later change those settings when you're ready to deploy.

Now we can more add code to our app, let's focus on the main form, you can see that we have two inputs and two buttons for signin/signup:

Let's initialize a constructor() with empty values:

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
> binding is confusing for begginers but you should defintely [learn more](https://reactjs.org/docs/faq-functions.html)
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
![firebase-auth](https://i.imgur.com/MIiERVC.jpg)

### App structure

Now that everyone can create an account and login to our app, we need a better structure so let's create a folder called `routes` where we're going to put our two screens

*Home.js
Login.js*

![architecture](https://i.imgur.com/zyVrXUu.jpg)
Dont forget to fix import paths since we're moving our files inside a folder!
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


You will notice that if the user is logged in, the app will show `<Home /`> otherwise the `<Login />` form will be displayed. This is an easy way to handle AuthChange but it's not suitable for production, we're using it for demonstration purpose and learn how to communicate with Firebase services.

In this example we'll create basic social cards with a title, description, an image and the author name. Open `Home.js` file and let's write some code:

Starting by importing firebase config file and a bunch of components from bootstrap:

    import fire from '../Fire';
    import { Card, CardImg, CardText, CardBody, CardTitle, CardSubtitle, CardFooter, CardColumns, Button, Container, Row, Col } from 'reactstrap';

### Adding data to firebase

Like our login page we need a form, some buttons and of course a function to handle user input and send data to Google.  The code below is our ui elements: 

    <input value={this.state.title} onChange={this.handleChange} name="title"
	     class="form-control mb-2" placeholder="Title" required />
    <input value={this.state.picture} onChange={this.handleChange} name="picture" 
	    class="form-control mb-2" placeholder="Picture URL" required />
    <textarea row="9" value={this.state.description} onChange={this.handleChange} name="description" 
	    class="form-control mb-2" placeholder="Description" required />
The `handleChange` is similar to the last one so copy that in your file and initialize values in constructor:

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

Now if you click on create post button you should see the data submitted in your firebase console, remember that we're using **Realtime Database** and not Firestore.

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

### Social Cards UI
![main-app](https://i.imgur.com/HdeDn8l.jpg)
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

To make a fancy UI for our web app you can copy my `app.css` file or create your own

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
    	background: #53d769; /* fallback for old browsers */
    	background: -webkit-linear-gradient(right, #53d769, #fecb2e);
    	background: -moz-linear-gradient(right, #53d769, #fecb2e);
    	background: -o-linear-gradient(right, #53d769, #fecb2e);
    	background: linear-gradient(to left, #53d769, #fecb2e);
    	font-family: 'Roboto', sans-serif;
    }
    
    #mainImage {
    	object-fit: cover;
    	height: 200px;
    	background-color: #fff;
    }
The complete file can be found in the repository. 
Congrats! You now have a full-stack realtime React app that can scale to millions of users. 





