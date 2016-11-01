# namm

sample usage:
```
require('namm')
    .public(__dirname + '/public')
    .layout(__dirname + '/public/base.html')
    .partials(__dirname + '/public/partials')
    .favicon(__dirname + '/public/img/favicon.ico')
    .models(__dirname + "/models")
    .config(require('./config.js'))
    //.stripe(require('./namm/samples/stripeOptions'))
    //.share({plans: require('./namm/samples/stripeOptions').planData})
    .routes(__dirname + '/routes')
    .services(__dirname + '/services')
    .views(__dirname + '/views')
    .sockets()
    //.connectors('./connectors')
    .init();
```
