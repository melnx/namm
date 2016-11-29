# namm

### example reddit clone application: https://github.com/melnx/namm_example

This framework lets you build an entire single page application driven by an advanced REST api in minutes with minimal coding.

sample usage:
```javascript
require('namm')
    .public(__dirname + '/public')                // static folder completely accessible to the client
    //.layout(__dirname + '/public/base.html')    // [optional] override the layout if you want
    .partials(__dirname + '/public/partials')     // partial views to load for different model actions (e.g.: ./public/partials/posts/index.html)
    .favicon(__dirname + '/public/favicon.ico')   // [optional] favicon path
    .models(__dirname + "/models")                // models folder (e.g.: ./models/Post.js)
    .config(require('./config.js'))               // basic configuration file for mongo
    .stripe(require('./stripeOptions'))           // [optional] stripe options for recurring payment subscriptions
    .share({                                      // [optional] share data from server to client, configure menu/title
        title: "WebFone",
        menu:  {
            "Token" : "/Token",
            "Analytics" : "/Calls/pie",
            "History" : "/Calls"
        }
    })
    .routes(__dirname + '/routes')                // [optional] custom server-side endpoints
    .registrationCode('1SecretCode!')             // [optional] secret code to gate sign-ups
    //.services(__dirname + '/services')          // [optional] services shared with the angular app
    //.views(__dirname + '/views')                // [optional] login view override
    //.sockets()                                  // [optional] sockets for online status tracking/chat
    //.connectors('./connectors')
    .init();                                      // actually start the application
```

## Getting Started

Just think of what models (resources) your application uses and how they are associated,
put your schemas in the models folder

Next think of views you want for various actions of this model and put them in /public/partials/<modelname>s/<action>.html
these views can then be accessed in your browser via /<modelname>s/<action>  (index is the default action if the action is left empty)

Access the advanced REST api via built in angular helper functions create() get() list() update() delete()
or access it via the flexible action() helper function.  You can access the API manually by going to /api/v1/<model>/[[id]/action]

If the automatic REST api is not enough, you can create custom routes using the /routes folder

There's a default layout file but you can specify you own via layout(<path>) call

## Sample Code

check the `samples` folder for the bower files (bower.json + .bowerrc) server.js and stripeOptions.js,
https://github.com/melnx/namm/tree/master/samples
copy-paste them to your project's root directory for a quick start.

if you don't have bower, install it, it helps you install all the client-side dependencies for this framework:
```
npm install -g bower
```

then, to install all the components run
```
bower install
```

## Configuration File:

```javascript
module.exports = {
    port: 5555,                                   // [optional] a port to test your application locally

    mongooseTestConn: 'mongodb://MONGO_URL',      // mongo connections url

    mg_api_key: 'key-MAILGUN_KEY',                // mailgun email api credentials
    mg_domain: 'MAILGUN_DOMAIN',                  // for lost password stuff
}
```

## Model Files

model files are arranged in a flat structure in the folder you specify with your models("<path>") call
by default this folder is "/models"

```javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

module.exports = {
    Post: { //the model name (multiple models can be specified per file)

        //basic model descriptor exactly the same as mongo schemas
        name: String,
        content: String,
        url: String,
        image: String,
        points: {type:Number, default:0},

        created: Date,
        updated: Date,

        _sub: {
            type: Schema.Types.ObjectId,
            ref: 'Sub'
        },

        //access descriptor to determine which User.role can access what for this model
        $access: {
            user: {
                list: 'all',
                create: true,
                get: 'all',
                update: 'own',
                delete: 'own'
            },
            admin: {
                list: 'all',
                create: true,
                get: 'all',
                update: 'all',
                delete: 'all'
            },
        },

        //this code gets bubbled up to the angular controller for this model (each model gets a controller)
        $init: function($scope, $http, $sce) {
            $scope.$on('$routeChangeSuccess', function() {
                $scope.$parent.controllericon = "fa-file-text-o";
                $scope.$parent.controllerheader = "Posts";
                $scope.$parent.controllerdescription = "posts";
            });

            $scope.doStuff = function(param){
                console.log("look! it's the parameter:", param);
            }
        }
    }
}

```

## Partial View Files

View files are simply angular template files. They live in the folder specified by your partials(<path>) call
This example creates a new post and saves it to the database via REST api

```html
<div ng-init="new()">
    Title<br />
    <input style="width:500px;" type="text" ng-model="item.name" /><br />
    Url<br />
    <input style="width:500px;" type="text" ng-model="item.url" /><br />
    Content<br />
    <textarea style="height:120px; width:500px;" type="text" ng-model="item.content"></textarea><br />
    Image (Override)<br />
    <input style="width:500px;" type="text" ng-model="item.image" /><br />
    <br />
    <div ng-if="_parameters._sub" style="margin-bottom: 10px;" ng-init="action('sub', 'Sub', 'get', _parameters._sub)">Posting to sub: {{sub.name}}</div>
    <button ng-click="create()" type="submit" class="btn btn-primary">Post</button>
</div>
```
note the usage of item, new(), create() and action()

## Route Files (optional)

route files are not necessary but can help you add custom server-side logic if the automatic API is not enough

```javascript
function downvotePost(req,res){
    var Post = mongoose.model('Post');
    var id = req.params.id;

    vote( id, req.user._id, -1, Post, {_post:id}, function(result){
        res.send(result);
    });
}

module.exports = {
    "/util/upvoteComment/:id": upvoteComment,
    "/util/downvoteComment/:id": downvoteComment,
    "/util/upvotePost/:id": upvotePost,
    "/util/downvotePost/:id": downvotePost,
}
```

Prefix the routes with PUBLIC and/or POST/GET.
PUBLIC makes it accessible without authentication
POST/GET explicitly specify the method, by default they are private and GET.
for example "PUBLIC|POST /util/downvotePost/:id" would make it PUBLIC and POST

## Layout File (optional)

There's a default layout file but you can override it by calling layout(<path>)
The layout file is a simple angular file that uses NammApp as ng-app, MainController as ng-controller, and has an ng-view div to load the partials

```html
<html ng-app="NammApp">
    <head>
        <script src="/_models.js"></script>
        <script src="/_services.js"></script>
    </head>
    <body ng-controller="MainController">
        <div ng-view></div>

        <script src="/components/angular/angular.min.js"></script>
        <script src="/components/angular-route/angular-route.min.js"></script>
        <script src="/client.js"></script>
    </body>
</html>
```

This is the simplest layout possible, it simply loads the partials to take up the whole page
Currently these scripts are required to make the application run properly (will load them automatically in the future)




