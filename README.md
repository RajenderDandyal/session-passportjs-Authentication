11 Simple session auth with node express w/o passport
Session handling in any web application is very important and must have thing, without it we won’t be able to track user and it’s activity. In my previous article i have explained Session handling in PHP.
In this article i am going to explain how to handle Session in ExpressJS 4 and above. Express 3 deprecate many dependencies like ‘bodyParser‘ , ‘logger‘ etc. Our code is written by taking consideration of latest Express JS framework.
At the time of writing article, latest version of Express is 4.8.7.
To demonstrate Session handling in Node, i have developed small Log-in and log-out System. In this User can log-in by providing their email, and that email will be used for further Session tracking. Once User log-out, Session will be destroyed and User will be redirected to home page.
Important !
If you are familiar with Express and using its in built body-parser and session, then it’s no more and you have to now install those dependencies separately.  Following are the dependencies which we have used for this System.
•	express-session
•	body-parser
•	ejs
package.json
{
    "name": "Node-Express4-Session",
    "version": "0.0.1",
    "main": "server.js",
    "dependencies": {
        "body-parser": "^1.7.0",
        "ejs": "^1.0.0",
        "express": "^4.8.7",
        "express-session": "^1.7.6"
    }
}
Run
npm install
to install dependencies for the project.
How to use Express Session ?
Before heading to actual code, i want to put few words about express-session module. to use this module, you must have to include express in your project. Like for all packages, we have to first include it.
server.js
var express = require('express');
var session = require('express-session');
var app = express();
After this, we have to initialize the session and we can do this by using following.
app.use(session({secret: 'ssshhhhh'}));
Here ‘secret‘ is used for cookie handling etc but we have to put some secret for managing Session in Express.
Now using ‘request‘ variable you can assign session to any variable. Just like we do in PHP using $_SESSION variable. for e.g
var sess;
app.get('/',function(req,res){
    sess=req.session;
    /*
    * Here we have assign the 'session' to 'sess'.
    * Now we can create any number of session variable we want.
    * in PHP we do as $_SESSION['var name'].
    * Here we do like this.
    */
    sess.email; // equivalent to $_SESSION['email'] in PHP.
    sess.username; // equivalent to $_SESSION['username'] in PHP.
});
After creating Session variables like sess.email , we can check whether this variable is set or not in other routers and can track the Session easily.
Project Directory Structure :
 
server.js
var express = require('express');
var session = require('express-session');
var bodyParser = require('body-parser');
var app = express();

app.set('views', __dirname + '/views');
app.engine('html', require('ejs').renderFile);

app.use(session({secret: 'ssshhhhh'}));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended:true}));

var sess;

app.get('/',function(req,res){
sess = req.session;
//Session set when user Request our app via URL
if(sess.email) {
/*
* This line check Session existence.
* If it existed will do some action.
*/
    res.redirect('/admin');
}
else {
    res.render('index.html');
}
});

app.post('/login',function(req,res){
  sess = req.session;
//In this we are assigning email to sess.email variable.
//email comes from HTML page.
  sess.email=req.body.email;
  res.end('done');
});

app.get('/admin',function(req,res){
  sess = req.session;
if(sess.email) {
res.write('
<h1>Hello '+sess.email+'</h1>
');
res.end('<a href="+">Logout</a>');
} else {
    res.write('
     <h1>Please login first.</h1>
    ');
    res.end('<a href="+">Login</a>');
}
});

app.get('/logout',function(req,res){
req.session.destroy(function(err) {
  if(err) {
    console.log(err);
  } else {
    res.redirect('/');
  }
});

});
app.listen(3000,function(){
console.log("App Started on PORT 3000");
});
In code there are three routers. First which render the home page, second router is use for admin area where user can only go if he/she is log-in and third is to perform session destruction and logging out the user.
Each router checks whether the sess.email variable is set or not and that could be set only by logging in through front-end. Here is my HTML code which resides in views directory.
views/index.html
<html>
<head>
<title>Session Management in NodeJS using Express4.2</title>
<scriptsrc="//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script> 
<script>
$(document).ready(function(){
    var email,pass;
    $("#submit").click(function(){
        email=$("#email").val();
        pass=$("#password").val();
        /*
        * Perform some validation here.
        */
        $.post("http://localhost:3000/login",{email:email,pass:pass},function(data){        
            if(data==='done')           
            {
                window.location.href="/admin";
            }
        });
    });
});
</script>
</head>
<body>
<input type="text" size="40" placeholder="Type your email" id="email"><br />
<input type="password" size="40"placeholder="Type your password" id="password"><br />
<input type="button" value="Submit"id="submit">
</body>
</html>
In jQuery code, we are calling our Router ‘/login’ and redirecting it to the ‘admin‘ if log-in is successful, you can add validation to fields as per your requirement, for demo purpose i have not added any.

12 session and passportjs
Though I'd setup ExpressJS and PassportJS in 2013 for the v0 version of airpair.com, I didn't deeply understand each or their relationship with one another. Around October 2014 I noticed some 10,000,000 active session documents in MongoDB, which was obviously not right.
Luckily it never affected us in a material way, and at the time I didn't want to take my chances. So I spent some time observing how Express and PassportJS plug into each other, here's what I learned on the way to uncovering what was going wrong.
There is only ONE Session
As per the PassportJS docs[1] configuring Passport via express middleware looks something like this:
app.configure(function() {
  app.use(express.static('public'));
  app.use(express.cookieParser());
  app.use(express.bodyParser());
  app.use(express.session({ secret: 'keyboard cat' }));
  app.use(passport.initialize());
  app.use(passport.session());
  app.use(app.router);
});
The syntax is a bit misleading I think ...
The first thing to conceptually get your head around is that even though you configure express.session() andpassport.session(), there is really only one session, which is managed by Express. Passport merely piggy backs off the ExpressJS session to store data for authenticated users.
Let's take a look at how:
ExpressJS Sessions & req.session
req.session is just a json object that gets persisted by the express-sessionmiddleware, using a store of your choice e.g. Mongo or Redis. Logging to the terminal, you'll see the default session object looks something like:
req.session = { cookie: 
  { path: '/',
    _expires: Thu Nov 07 2014 09:39:45 GMT-0700 (PDT),
    originalMaxAge: 100000,
    httpOnly: true } }
If you open up your persistence store and look at the documents, you'll see the req.session object is an attribute of the the entire document that also comes with an id and expires attribute.
{
    _id : "svZGs99EIvDkwW_60PRwlafU9Y_7_m-N",
    session : { cookie:{originalMaxAge:100000,expires:"2014-11-07T02:11:16.320Z",httpOnly:true,path:"/"},
    expires : ISODate("2014-11-07T02:11:16.320Z")
}
Express stuffs the id of the session object into a cookie on the client's browser, which gets passed back to express in a header on every request. This is how Express identifies multiple requests as belonging to a single session even if the user is not logged in. With the id from the cookie header, Express reads the session store and puts all the info onto req.session, which is available to you on each request.
Harnessing req.session yourself
You can stuff anything you like onto the req.session object and Express will automatically persist it back to the session store for a given session (unique id). For example, if a user can't access a page because they are not logged in, airpair.com uses a custom made-up attribute called return_to to direct the user to the right page after login.
{ cookie: 
  { path: '/',
    _expires: Thu Oct 09 2014 09:39:45 GMT-0700 (PDT),
    originalMaxAge: 100000,
    httpOnly: true },
    passport: { user: { _id: '5175efbfa3802cc4d5a5e6ed' } },
    return_to: '/workshops' }
It's worth noting that since you can put anything you want into an anonymous session, provided you can correlate it to a user in your database on login or signup, you can start persisting info for a user even though you don't know who they are straight away. You will see some cool interactions for anonymous users on airpair.com coming in the next month or two.
Passport also harnesses req.session
As you can see on line 6 in the code snippet above, using the passport attribute allows PassportJS to use the session object to keep track of a logged in user associated with a given session.
It then uses the deserializeUser function, which receives req.session.passport.user as its first parameter, and as the default behavior suggested in the PassportJS documentation, makes an additional read to your persistence store to retrieve the user object associated with the userId.
passport.serializeUser(function(user, done) {
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  User.findById(id, function(err, user) {
    done(err, user);
});
Passport req.user
req.user is a PassportJS specific property that is the result of the deserializeUserfunction using the data from req.session.passport.user
Optimize PassportJS
I realized that in the old app, we followed the default suggestion and were hitting the database twice on every single API call to populate all the user's information in memory. But in practice, we rarely needed more than the userId in our backend code. So this time around, I've made the decision to stuff the name and email into the session object and avoid making multiple database trips on every single API call. With many pages on the site making 5-10 calls to render a single page, this seemed like a cheap way to significantly reduce database load. Here's what the new app looks like:
passport.serializeUser( (user, done) => {
  var sessionUser = { _id: user._id, name: user.name, email: user.email, roles: user.roles }
  done(null, sessionUser)
})

passport.deserializeUser( (sessionUser, done) => {
  // The sessionUser object is different from the user mongoose collection
  // it's actually req.session.passport.user and comes from the session collection
  done(null, sessionUser)
})
Optimize ExpressJS Sessions
When are sessions created?
Express will create a new session (and write it to the database) whenever it does not detect a session cookie. As it turns out, the order in which you set the session middleware and tell Express where your static directory is has some pretty dramatic nuances. Here's what the new AirPair index.js looks like:
var express = require('express')
var app = express()

//-- We don't want to serve sessions for static resources
//-- Save database write on every resources
app.use(express.static(config.appdir+'/public'))

mongo.connect()
session(app, mongo.initSessionStore)

//-- Do not move connect-livereload before session middleware
if (config.livereload) app.use(require('connect-livereload')({ port: 35729 }))

hbsEngine.init(app)
routes.init(app)    
app.listen(config.port, function() {})
Avoid Sessions for Static Resources
I learned that if you add the session middleware before your static directory, Express will generate sessions for requests on static files like stylesheets, images, and JavaScript.
If a new visitor without a session loads a page with 10 static files, the client's browser will not yet have a cookie and will send 10 cookieless requests all triggering Express to create sessions. Ouch! So that's what was happening... If you haven't done something smart to detect bots and scrapers, things can blow out pretty quickly!
Simply put your static files first, or better yet on a CDN that has nothing to do with your Node.js app and your session collection should stay much healthier.
ExpressJS 4.0 Middleware Order
The middleware setup for ExpressJS 4.0 is quite different from ExpressJS 3.0 where everything came baked in. You now need to include each piece manually with its own NPM package. In case you want to see an up-to-date version of how each piece is chained together, because there are many non-working legacy examples floating around, this is what I ended up with.
A couple of gotchas sunk half an hour or so for me, including that Cookie Parser now requires a secret and Body Parser required and Extend Url.
// Passport does not directly manage your session, it only uses the session.
// So you configure session attributes (e.g. life of your session) via express
var sessionOpts = {
  saveUninitialized: true, // saved new sessions
  resave: false, // do not automatically write to the session store
  store: sessionStore,
  secret: config.session.secret,
  cookie : { httpOnly: true, maxAge: 2419200000 } // configure when sessions expires
}

app.use(bodyParser.json())
app.use(bodyParser.urlencoded({extended: true}))
app.use(cookieParser(config.session.secret))
app.use(session(sessionOpts))

app.use(passport.initialize())
app.use(passport.session())
Strategies
Passport uses what are termed strategies to authenticate requests. Strategies range from verifying a username and password, delegated authentication using OAuth or federated authentication using OpenID.
Before asking Passport to authenticate a request, the strategy (or strategies) used by an application must be configured.
Strategies, and their configuration, are supplied via the use() function. For example, the following uses the LocalStrategy for username/password authentication.
var passport = require('passport')
  , LocalStrategy = require('passport-local').Strategy;

passport.use(new LocalStrategy(
  function(username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) { return done(err); }
      if (!user) {
        return done(null, false, { message: 'Incorrect username.' });
      }
      if (!user.validPassword(password)) {
        return done(null, false, { message: 'Incorrect password.' });
      }
      return done(null, user);
    });
  }
));
Verify Callback
This example introduces an important concept. Strategies require what is known as a verify callback. The purpose of a verify callback is to find the user that possesses a set of credentials.
When Passport authenticates a request, it parses the credentials contained in the request. It then invokes the verify callback with those credentials as arguments, in this case username and password. If the credentials are valid, the verify callback invokes done to supply Passport with the user that authenticated.
return done(null, user);
If the credentials are not valid (for example, if the password is incorrect), done should be invoked with false instead of a user to indicate an authentication failure.
return done(null, false);
An additional info message can be supplied to indicate the reason for the failure. This is useful for displaying a flash message prompting the user to try again.
return done(null, false, { message: 'Incorrect password.' });
Finally, if an exception occurred while verifying the credentials (for example, if the database is not available), done should be invoked with an error, in conventional Node style.
return done(err);
Note that it is important to distinguish the two failure cases that can occur. The latter is a server exception, in which err is set to a non-null value. Authentication failures are natural conditions, in which the server is operating normally. Ensure that err remains null, and use the final argument to pass additional details.
By delegating in this manner, the verify callback keeps Passport database agnostic. Applications are free to choose how user information is stored, without any assumptions imposed by the authentication layer.
Middleware
In a Connect or Express-based application, passport.initialize() middleware is required to initialize Passport. If your application uses persistent login sessions, passport.session()middleware must also be used.
app.configure(function() {
  app.use(express.static('public'));
  app.use(express.cookieParser());
  app.use(express.bodyParser());
  app.use(express.session({ secret: 'keyboard cat' }));
  app.use(passport.initialize());
  app.use(passport.session());
  app.use(app.router);
});
Note that enabling session support is entirely optional, though it is recommended for most applications. If enabled, be sure to use session() before passport.session() to ensure that the login session is restored in the correct order.
In Express 4.x, the Connect middleware is no longer included in the Express core, and the app.configure() method has been removed. The same middleware can be found in their npm module equivalents.
var session = require("express-session"),
    bodyParser = require("body-parser");

app.use(express.static("public"));
app.use(session({ secret: "cats" }));
app.use(bodyParser.urlencoded({ extended: false }));
app.use(passport.initialize());
app.use(passport.session());
Sessions
In a typical web application, the credentials used to authenticate a user will only be transmitted during the login request. If authentication succeeds, a session will be established and maintained via a cookie set in the user's browser.
Each subsequent request will not contain credentials, but rather the unique cookie that identifies the session. In order to support login sessions, Passport will serialize and deserialize user instances to and from the session.
passport.serializeUser(function(user, done) {
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  User.findById(id, function(err, user) {
    done(err, user);
  });
});
In this example, only the user ID is serialized to the session, keeping the amount of data stored within the session small. When subsequent requests are received, this ID is used to find the user, which will be restored to req.user.
The serialization and deserialization logic is supplied by the application, allowing the application to choose an appropriate database and/or object mapper, without imposition by the authentication layer.

