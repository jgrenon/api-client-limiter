# API Client Limiter

Some web API limits the number of calls you can make in a certain period of time. After looking for a node module that just do that, I
was frustrated and created mine. All modules I saw were either very complex to use or were actually "debouncing" or throttling calls, which
means that all calls within a certain time frame get the same results instead of doing the full round trip to the API.

This simple module does only one thing: it wraps a function following the standard nodejs callback pattern, queue calls and execute the real method
every X seconds.

## Installing

Just install the module in your node project using the following call :

    npm install api-client-limiter

## Limiting your API calls

Here's a quick sample on how to transform a tight loop performing API calls into a limited one. This sample use async to control the loop, because of the provided
callback parameter. This is the ideal way of using this module.

    var Limiter = require('api-client-limiter');

    // Create a limiter for 1 call every 2 seconds
    var limiter = new Limiter(1, 2000);

    // Wrap a callback that will be called for a team collection
    var callback = limiter(function(team, done) {

        console.log("This will be queued and called every 2 seconds");

        // An API call is performed by this callback
        request.get({
            url: 'http://api.sample.com',
            json:true,
            agent:false
        }, function(err, resp, body) {
            // Let's do anything with the received data...
            db.save(body);
            done();
        });

    });

    async.forEach(teams, callback, function(err) {
        console.log("Execution is now complete. All teams have been processed");
    });

Like your can see in this sample, calling limiter with the function your want to limit is very easy. Internally, we create a queue allowing a certain number of parallel calls
every x milliseconds. Each call to limiter will enqueue a call to your original callback, wrapped in a timer of the given duration.

## Limitations

For now, only functions with a callback as the last parameter are supported. I may add support for promises in the future, but for now, that was what I needed for my use case.

