# Lockit

[^1] [^2]

Lockit is an authentication solution for [Express][].

It consists of multiple single purpose modules:

-   [lockit-login][]

-   [lockit-signup][]

-   [lockit-delete-account][]

-   [lockit-forgot-password][]

-   [lockit-sendmail][]

-   [lockit-couchdb-adapter][]

-   [lockit-mongodb-adapter][]

-   [lockit-sql-adapter][]

-   [lockit-utilities][]

-   [lockit-template-blank][]

## Table of contents

-   [Quickstart][]

-   [Full Installation][]

-   [Configuration][]

-   [Database connection][]

-   [Sending emails][]

-   [Custom views][]

-   [Events][]

-   [signup][]

-   [login][]

-   [logout][]

-   [delete][]

-   [REST API][]

-   [Sample config][]

-   [Features][]

-   [Routes included][]

## Quickstart

1.  Create new Express app (sessions are required).

`express -s`

1.  Install Lockit via npm.

`npm install lockit --save`

1.  Use `lockit` in your Express `app.js`.

\`\`\`js var lockit = require('lockit');

... lockit(app); app.use(app.router); // default express.js router
\`\`\`

1.  Go to [localhost:3000/signup][]

By default Lockit uses an in-memory SQLite database. So you don't have
to set up any db. Lockit will just work. Check out the [default
example][].

For production use a persistent data store!

## Full installation

1.  Install and require

`npm install lockit --save`

\`\`\`js var config = require('./config.js'); var Lockit =
require('lockit');

var app = express();

// express middleware // ... // sessions are required
app.use(express.cookieParser('your secret here'));
app.use(express.cookieSession());

// use middleware before router so your own routes have access to //
req.session.email and req.session.name var lockit = new Lockit(app,
config);

// you now have all the routes like /login, /signup, etc. // and you can
listen on events. For example 'signup' lockit.on('signup',
function(user, res) { console.log('a new user signed up');
res.send('Welcome!'); // set signup.handleResponse to 'false' for this
to work });

app.use(app.router); // continue with express middleware // ... \`\`\`

1.  Add styles

Views are built with [bootstrap][]. You can use [your own ones][Custom
views] though! Use Bootstrap CDN and add the following line to your
`layout.jade`

`jade   link(rel='stylesheet', href='//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css')`

1.  Install database adapter

`npm install lockit-[DB]-adapter` where `[DB]` can be

+-----------------------+-----------------------+
| Database              | Command               |
+-----------------------+-----------------------+
| [CouchDB][lockit-couc | `npm install lockit-c |
| hdb-adapter]          | ouchdb-adapter`       |
+-----------------------+-----------------------+
| [MongoDB][lockit-mong | `npm install lockit-m |
| odb-adapter]          | ongodb-adapter`       |
+-----------------------+-----------------------+
| [SQL (PostgreSQL,     | `npm install lockit-s |
| MySQL, MariaDB or     | ql-adapter`           |
| SQLite)][lockit-sql-a |                       |
| dapter]               |                       |
+-----------------------+-----------------------+

If you use a SQL database you also have to install the connector.

`npm install pg       # for postgres   npm install mysql    # for mysql   npm install sqlite3  # for sqlite   npm install mariasql # for mariasql`

## Configuration

You need a `config.js` somewhere in your app.

### Database connection

Add the database connection string to your `config.js`.

    // database settings for CouchDB
    exports.db = 'http://127.0.0.1:5984/';        // connection string for database

    // or if you want to use MongoDB
    // exports.db = 'mongodb://127.0.0.1/test';
    // exports.dbCollection = 'users';                // collection name for MongoDB

    // PostgreSQL
    // exports.db = 'postgres://127.0.0.1:5432/users';
    // exports.dbCollection = 'users';                // table name for SQL databases

    // MySQL
    // exports.db = 'mysql://127.0.0.1:9821/users';
    // exports.dbCollection = 'users';

    // SQLite
    // exports.db = 'sqlite://:memory:';
    // exports.dbCollection = 'users';

### Sending emails

By default the email service is stubbed and no emails are sent. That
means that you won't receive any signup and password reset tokens. You
have to look them up in your database and call the routes manually (e.g.
`/signup/:token`). To send emails you need an email server and you have
to change the settings in your `config.js`:

-   `emailType` - usually `SMTP`

-   `emailSettings` - see [nodemailer][] for more information

With [mailgun][] you can send up to 10,000 emails per month for free.

    exports.emailType = 'SMTP';
    exports.emailSettings = {
      service: 'Mailgun',
      auth: {
        user: 'postmaster@username.mailgun.org',
        pass: 'secret-password'
      }
    };

### Custom views

Lockit comes with built-in views which are based on Bootstrap. If you
want to use your own custom views you can. It is dead simple.

Put them into your `views` folder, for example
`views/lockit/myLogin.jade`. Then edit your `config.js` and set the path
to your custom view.

    exports.login = {
      route: '/login',
      logoutRoute: '/logout',
      views: {
        login: 'lockit/myLogin.jade',
        loggedOut: 'lockit/myLogoutSuccess.jade'
      }
    };

The only thing you have to keep in mind is the structure. The
`login.views.login` view, for example, needs a form element with two
input fields. The `method` has to be `POST` and `action` should point to
your `login.route`. The input fields have to have the names `login` and
`password`. If something went wrong during the login process you'll get
an `error` variable that you can use in your template.

Here is a minimalistic example for an alternative `myLogin.jade`.

    extend /layout

    block content
      h1 Login
      form(action="/login", method="POST")
        div
          label(for="login") Email or Username
          input(type="text", id="login", name="login", placeholder="Your email or username")
        div
          label(for="password") Password
          input(type="password", id="password", name="password", placeholder="Your password")
        if error
          p #{error}
        input(type="submit", value="Login")

For more information about each view see the `views` folder inside the
different repositories. Make sure your view extends `/layout` which is
different to your normal views. They extend `layout` without the slash.
This is required to find the view.

## Events

Lockit emits the most important events for user authentication. Those
are

-   `signup`

-   `login`

-   `logout`

-   `delete`

You can use these events to intercept requests and implement some custom
logic, like getting the gravatar before sending a response to the
client.

##### `signup`

A new user signed up. The callback function has two arguments.

-   `user` is an object and contains information about the new user,
    like `user.name` or `user.email`.

-   `res` is the standard Express.js `res` object with methods like
    `res.render` and `res.send`. If you've set `signup.handleResponse`
    to `false` Lockit will not handle the response for you. You
    therefore have to send the response back to the client manually or
    otherwise it will wait forever.

<!-- -->

    lockit.on('signup', function(user, res) {
      // ...
    });

##### `login`

A user logged in. Callback function this time has three arguments.

-   `user` is again the JSON object containing info about that
    particular user.

-   `res` is the normal Express.js response object with all properties
    and methods.

-   `target` is the redirect target route after a successful login, i.e.
    `/settings`

<!-- -->

    lockit.on('login', function(user, res, target) {
      // ...
    });

##### `logout`

A user logged out. Same as above without the `target` string.

    lockit.on('logout', function(user, res) {
      // ...
    });

##### `delete`

A user deleted an account. Same callback as above.

    lockit.on('delete', function(user, res) {
      // ...
    });

## REST API

In a single page application (SPA) all the routing and template
rendering is done on the client. Before version 0.5.0 Lockit caught
relevant routes, like `/login` or `/signup`, and did the entire
rendering on the server.

Starting with version 0.5.0 you're able to use Lockit as a REST API and
communicate via JSON. All you have to do is set `exports.rest = true` in
your `config.js`.

With REST enabled all default routes get a `/rest` prefix so you can
catch `/login` on the client. To allow for true page refreshes (i.e.
user is at `/login` and refreshes the page) all routes on the server,
like `/login` and `/signup`, send the `index.html` from the `public/`
folder. From there your SPA has to take over.

Here is a short example how the process works.

1.  User sends GET request for `/login`

2.  Server has a route handler for this request and sends `index.html`
    back

3.  Client router takes over and renders `/login` page

4.  User enters credentials and submits the form

5.  Client controller catches submit and sends POST via AJAX request to
    `/rest/login`

6.  Server handles POST request and validates user credentials

7.  Server sends status code `200` or some JSON with error message

8.  Client reacts to JSON from server and redirects on success or shows
    error

I've built a [simple example][] using AngularJS on the client side.

## Sample config

If you want to go crazy and customize all the things you can:

    // name for subject and email content
    exports.appname = 'lockit - Test App';

    // url for proper link generation
    exports.url = 'http://localhost:3000';

    // email settings (same as nodemailer)
    exports.emailType = 'Stub';
    exports.emailSettings = {
      service: 'none',
      auth: {
        user: 'none',
        pass: 'none'
      }
    };

    // whenever a library uses request under the hood (like nano in lockit-couchdb-adapter)
    // the following values will be used
    exports.request_defaults = {
      // proxy: 'http://someproxy'
    };

    // email template from npm
    exports.emailTemplate = 'lockit-template-blank';

    // render views or json for single page apps
    exports.rest = false;

    // or if you want to use rest
    // exports.rest = {
    //
    //   // set starting page for single page app
    //   index: 'public/index.html',
    //
    //   // use view engine (render()) or send static file (sendfile())
    //   useViewEngine: false
    //
    // }

    // signup settings
    exports.signup = {
      route: '/signup',
      tokenExpiration: '1 day',
      views: {
        signup: '',         // input fields 'name', 'email' and 'password' | local variable 'error' | POST /'signup.route'
        linkExpired: '',    // message link has expired | input field 'email' | POST /'signup.route'/resend-verification
        verified: '',       // message email is now verified and maybe link to /'login.route'
        signedUp: '',       // message email has been sent => check your inbox
        resend: ''          // input field 'email' | local variable 'error' | POST /'signup.route'/resend-verification
      },
      handleResponse: true  // let lockit handle the response after signup success
    };

    // login settings
    exports.login = {
      route: '/login',
      logoutRoute: '/logout',
      views: {
        login: '',          // input fields 'login' and 'password' | POST /'login.route' | local variable 'error'
        loggedOut: ''       // message that user logged out
      },
      handleResponse: true  // let lockit handle the response after login/logout success
    };

    // forgot password settings
    exports.forgotPassword = {
      route: '/forgot-password',
      tokenExpiration: '1 day',
      views: {
        forgotPassword: '', // input field 'email' | POST /'forgotPassword.route' | local variable 'error'
        newPassword: '',    // input field 'password' | POST /'forgotPassword.route'/#{token} | local variable 'error'
        changedPassword: '',// message that password has been changed successfully
        linkExpired: '',    // message that link has expired and maybe link to /'forgotPassword.route'
        sentEmail: ''       // message that email with token has been sent
      }
    };

    // delete account settings
    exports.deleteAccount = {
      route: '/delete-account',
      views: {
        remove: '',         // input fields 'name', 'phrase', 'password' | POST /'deleteAccount.route' | local variable 'error'
        removed: ''         // message that account has been deleted
      },
      handleResponse: true  // let lockit handle the response after delete account success
    };

    // lock account
    // show warning after three failed login attempts
    exports.failedLoginsWarning = 3;
    // lock account after five failed login attempts
    exports.failedLoginAttempts = 5;
    // lock account for 20 minutes
    exports.accountLockedTime = '20 minutes';

    // public email address of your app
    exports.emailFrom = 'welcome@lock.it';

    // email signup template
    exports.emailSignup = {
      subject: 'Welcome to <%- appname %>',
      text: [
        '<h2>Hello <%- username %></h2>',
        'Welcome to <%- appname %>.',
        '<p><%- link %> to complete your registration.</p>'
      ].join(''),
      linkText: 'Click here'
    };

    // email already taken template
    exports.emailSignupTaken = {
      subject: 'Email already registered',
      text: [
        '<h2>Hello <%- username %></h2>',
        'you or someone else tried to sign up for <%- appname %>.',
        '<p>Your email is already registered and you cannot sign up twice.',
        ' If you haven\'t tried to sign up, you can safely ignore this email. Everything is fine!</p>',
        '<p>The <%- appname %> Team</p>'
      ].join('')
    };

    // resend signup template
    exports.emailResendVerification = {
      subject: 'Complete your registration',
      text: [
        '<h2>Hello <%- username %></h2>',
        'here is the link again. <%- link %> to complete your registration.',
        '<p>The <%- appname %> Team</p>'
      ].join(''),
      linkText: 'Click here'
    };

    // forgot password template
    exports.emailForgotPassword = {
      subject: 'Reset your password',
      text: [
        '<h2>Hey <%- username %></h2>',
        '<%- link %> to reset your password.',
        '<p>The <%- appname %> Team</p>'
      ].join(''),
      linkText: 'Click here'
    };

## Features

-   responsive html email template: [lockit-template-blank][]

-   support for wide range of databases out of the box

-   email address verification

-   account locking after too many failed login attempts

-   verification link expiration

-   failed login tracking

-   /login redirection when user is unauthorized

-   password hash generation with bcrypt

-   unit tests for all modules

-   serves proper HTML views or only JSON

-   events for most important happenings `login`, `logout`, `signup` and
    `delete`

-   implementation of [lots of][] [best][] [pratices][]

## Routes included

From [lockit-signup][]

-   GET /signup

-   POST /signup

-   GET /signup/:token

-   GET /signup/resend-verification

-   POST /signup/resend-verification

From [lockit-login][]

-   GET /login

-   POST /login

-   GET /logout

From [lockit-forgot-password][]

-   GET /forgot-password

-   POST /forgot-password

-   GET /forgot-password/:token

-   POST /forgot-password/:token

From [lockit-delete-account][]

-   GET /delete-account

-   POST /delete-account

## Test

`grunt`

## License

MIT

[^1]: <https://travis-ci.org/zeMirco/lockit>

[^2]: <http://badge.fury.io/js/lockit>

  [Express]: http://expressjs.com/
  [lockit-login]: https://github.com/hdeshmukh/lockit-login
  [lockit-signup]: https://github.com/hdeshmukh/lockit-signup
  [lockit-delete-account]: https://github.com/hdeshmukh/lockit-delete-account
  [lockit-forgot-password]: https://github.com/hdeshmukh/lockit-forgot-password
  [lockit-sendmail]: https://github.com/hdeshmukh/lockit-sendmail
  [lockit-couchdb-adapter]: https://github.com/hdeshmukh/lockit-couchdb-adapter
  [lockit-mongodb-adapter]: https://github.com/hdeshmukh/lockit-mongodb-adapter
  [lockit-sql-adapter]: https://github.com/hdeshmukh/lockit-sql-adapter
  [lockit-utilities]: https://github.com/hdeshmukh/lockit-utilities
  [lockit-template-blank]: https://github.com/hdeshmukh/lockit-template-blank
  [Quickstart]: #quickstart
  [Full Installation]: #full-installation
  [Configuration]: #configuration
  [Database connection]: #database-connection
  [Sending emails]: #sending-emails
  [Custom views]: #custom-views
  [Events]: #events
  [signup]: #signup
  [login]: #login
  [logout]: #logout
  [delete]: #delete
  [REST API]: #rest-api
  [Sample config]: #sample-config
  [Features]: #features
  [Routes included]: #routes-included
  [localhost:3000/signup]: http://localhost:3000/signup
  [default example]: https://github.com/hdeshmukh/lockit/tree/master/examples/default
  [bootstrap]: http://getbootstrap.com/
  [nodemailer]: https://github.com/andris9/Nodemailer
  [mailgun]: http://www.mailgun.com/pricing
  [simple example]: https://github.com/hdeshmukh/lockit/tree/master/examples/angular
  [lots of]: https://www.owasp.org/index.php/Guide_to_Authentication
  [best]: http://stackoverflow.com/questions/549/the-definitive-guide-to-form-based-website-authentication
  [pratices]: https://www.owasp.org/index.php/Authentication_Cheat_Sheet
