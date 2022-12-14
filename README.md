# Setting up Sequelize v6, Express.js and Password.js 'local'
## Installation and Setting up a project

[Initialize a project:](https://docs.npmjs.com/cli/v8/commands/npm-init)
```bash
  npm init
```
Generate [Express.js](https://expressjs.com/):
```bash
  npm i express
  npx express-generator
  npm i nodemon
  npm i dotenv
```
Set `nodemon` in `package.json`.

Make `.env` file in `main` folder and set `NODE_ENV`, `PORT` and `DEBUG`.
Generate [Express.js](https://expressjs.com/)

Setup [Sequelize-CLI](https://sequelize.org/docs/v6/other-topics/migrations/):
```bash
  npm i sequelize
  npm i sqlite3
  npm i sequelize-cli
  npx sequelize-cli init
```
Check out: [The Sequelize Command Line Interface (CLI) GitHub](https://github.com/sequelize/cli)

Make changes to `config/config.json`:
```bash
{
    "development": {
        "storage": "db.sqlite3",
         "logger": true,
         "dialect": "sqlite"
    },
    "test": {
        "database": "database_test",
        "logger": true,
        "dialect": "sqlite"
    },
    "production": {
        "database": "database_production",
        "logger": true,
        "dialect": "sqlite"
    }
}
```
Generate model:
```bash
  npx sequelize-cli modle:generate --name Users -- attributes firstName:string,age:integer
  npx sequelize-cli db:migrate
  npx sequelize-cli seed:generate --name demo-user
```
Add `.map` in created seed file `xxxxxxxxxxx-demo-user.js` after object:
```javascript
await queryInterface.bulkInsert('Users', [{
    firstName: 'Tester',
    lastName: 'Testing',
    age: '30'
}].map((obj) => ({
    ...obj, createdAt: new Date(), updatedAt: new Date(),
})), {});
```

Populate database:
```bash
  npx sequelize-cli db:seed:all
```

Build REST for Users route:
```javascript
router.route('/')
    /* GET users */
    .get((req, res) => {
        db.Users.findAll().then((data) => res.json(data))
    })
    /* CREATE user */
    .post((req, res) => {
        db.Users.create(req.body)
            .then((data) => res.json(data))
            .catch((err) => {
                res.status(404).send(err)
            })
    })

router.route('/:id')
    /* GET user */
    .get((req, res) => {
        db.Users.findOne({where: {id: req.params.id}})
            .then((data) => res.json(data));
    })
    /* UPDATE user */
    .post((req, res) => {
        db.Users.update(req.body, { where: {id: req.params.id}})
            .then((data) => res.json(data))
    })
    /* DELETE user */
    .delete((req, res) => {
        db.Users.destroy({ where: {id: req.params.id} })
            .then((data) => res.json(data))
    })
```
Install [password.js](https://www.passportjs.org/) package and needed packages (npm)
```bash
  npm i passport passport-local express-session
```

Create `services` directory and create `passport.js` file in it.

Create `auth.js` file in `routes` directory and init route for use:
```bash
--> const authRouter = require('./routes/auth');

--> app.use('/auth', authRouter);
```

Import required dependencies in app.js file:
```bash
const session = require('express-session')
const passport = require('./services/passport')
```

Populate `passport.js` file with following code:
```javascript
const passport = require('passport')
const LocalStrategy = require('passport-local');
const db = require('../models')

passport.serializeUser((user, done) => {
    done(null, user);
})

passport.deserializeUser((user, done) => {
    done(null, user);
})

passport.use('local', new LocalStrategy(
    async (username, password, done) => {
        try {
            const userData = await db.Users.findOne({where : { username: username}}, {password : 1});
            const passwordData = await db.Users.findOne({where : {password:password}});
            if(!userData) {
                return done(null, false);
            }
            if(!passwordData) {
                return done(null, false);
            }
            return done(null, userData.username, passwordData.password);
        } catch (err) {
            return err;
        }
    }
));

module.exports = passport;
```

Import `passport` from `../services/passport.js` to route `auth.js` and populate it with:

```javascript
router.route('/login')
    .get((req, res) =>{
        res.render('login')
    })
    .post(passport.authenticate('local', { successRedirect: '/', failureRedirect: '/login'}))
```
Make changes in app.js:

```javascript
app.use(session({
    resave: true,
    saveUninitialized: true,
    secret: 'super secret',
    cookie: {
        httpOnly: true,
        secure: false,
        maxAge: 7 * 24 * 60 * 60 * 1000
    }
}))
app.use(passport.initialize({}))
app.use(passport.session({}))
// Route /auth to authenticate
app.use('/auth', authRouter);
app.use((req, res, next) => {
    if(req.user) {
        next()
    } else {
        next('Not authorized!')
    }
})
// Route authentication succeeded
app.use('/', indexRouter);
app.use('/users', usersRouter);
```

USE POSTMAN TO TRY - SET PARAMS WITH username and password
