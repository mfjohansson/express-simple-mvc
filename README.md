Express Simple MVC
==================

Very simple and lightweight framework to improve Express JS MVC's architecture written in CoffeeScript.
It provides a simple way to create models and controllers with an object-oriented approach (CoffeeScript).

## How to install

    npm install express-simple-mvc

## How it works

Create your express application as usual and use the "simple_mvc" function and pass the path to the controllers, models and routes directories.

```coffeescript
express      = require 'express'
{simple_mvc} = require 'express-simple-mvc'
require 'namespace'

app = express()
app.listen 3000

app.configure ->
  app.use express.methodOverride()
  app.use app.router

simple_mvc app, __dirname + '/controllers', __dirname + '/models', __dirname + '/config/routes'

```

You must configure your application before using simple_mvc.
You'll also need a namespace pattern for coffeescript, you can install it with npm install

Create a directory called "config" with a file called "routes" inside.
Define your routes (inspired by Play! Framework), for example :

    # Routes
    # This file defines all application routes (Higher priority routes first)
    # ~~~~

    GET       /                    controllers.Players.index
  
    GET       /players             controllers.Players.index
    GET       /players/new         controllers.Players.new
    GET       /player/:id          controllers.Players.show
    GET       /player/:id/edit     controllers.Players.edit
    POST      /players             controllers.Players.create
    PUT       /player/:id          controllers.Players.update
    DELETE    /player/:id          controllers.Players.delete

Create a models and a controllers directory with a Players controller and a Player model.
For example, simple CRUD (don't forget the namespace)

```coffeescript
namespace controllers:

  class Players

    # Import Player model from models namespace
    {Player} = models

    # GET /players
    @index: (req, res) ->
      Player.all (players) ->
        res.send players

    # GET /player/:id
    @show: (req, res) ->
      Player.read req.params.id, (player) ->
        res.send player

    # GET /players/new
    @new: (req, res) ->
      res.render 'newForm'

    # GET /player/:id/edit
    @edit: (req, res) ->
      Player.read req.params.id, (player) ->
        res.render 'editForm'
          player: player

    # POST /players
    @create: (req, res) ->
      Player.create req.param('player'), ->
        res.send 'New player successfully created !'

    # PUT /player/:id
    @update: (req, res) ->
      Player.update req.params.id, req.param('player'), ->
        res.send "Player #{req.param('id')} successfully updated !"

    # DELETE /player/:id
    @delete: (req, res) ->
      Player.delete req.params.id, ->
        res.send "Player #{req.param('id')} successfully deleted !"
```

And the for the model (I use mongoose here, but you can adapt it)

```coffeescript
mongoose = require 'mongoose'

namespace models:

  class Player

    @schema =
      pseudo: String
      x:
        type: Number
        default: 0
      y:
        type: Number
        default: 0

    @model = mongoose.model 'Player', new mongoose.Schema @schema

    @all: (cb) ->
      @model.find null, (err, docs) ->
        if err then throw err
        cb docs

    @create: (doc, cb) ->
      new @model(doc).save (err) ->
        if err then throw err
        do cb

    @read: (_id, cb) ->
      @model.findById _id, (err, doc) ->
        if err then throw err
        cb doc

    @update: (_id, doc, cb) ->
      @model.findByIdAndUpdate _id, doc, (err) ->
        if err then throw err
        do cb

    @delete: (_id, cb) ->
      @model.findByIdAndRemove _id, (err) ->
        if err then throw err
        do cb
```

And that's all ! Looks difficult ? Have a look at the source code in /src (only about 20 lines !) or run the example.