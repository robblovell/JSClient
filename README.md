DEPRICATED:  Use https://github.com/mojio/mojio-js 

Mojio.Client
============

We have started to develop a simple javascript client to help our website interact directly with the Mojio API.  We are releasing this client on github in the hopes of helping other developers get started with the Mojio API faster.

Installation
============

Download or checkout the mojio.client.js.  The javascript client also requires [JQuery](http://jquery.com/).


Getting Started
===============

To begin developing with our client, you will need your very own application ID and secret key.  First you will need to create an account and login to our developer center.  We recommend starting with our sandbox environment (http://sandbox.developer.moj.io/).

Once you have logged in, you can create a new Application.  From here, you will want to copy the Application ID and the Secret Key, these will be required to initialize the javascript client.


Initializing the Client
-----------------------

To get started using the client, you must first create a new instance of the Mojio.Client object.  This is where you will need to pass in the Application ID and Secret Key, as well as the developer environment you are using (Sandbox, or Live) and set other configurable settings.


```javascript
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script src="[/path/to/js]/mojio.client.js"></script>
<script>
    var appId = "{APPID}";
    var secretKey = "{SecretKey}";
	
    var settings = {
        url: Mojio.Sandbox, // or Mojio.Live
        appId: appId,
        secretKey: secretKey,
    });
	
	  // Initialize client
    var client = new Mojio.Client( settings );
	
    client.ready(function(){
        // Makes sure the client is fully loaded before attempting to make additional calls.
		
        // ...
    });
</script>
```

Authenticate a Mojio User
-------------------------

Now that you have a client that is associated with your app, you can get started making some calls.  However, many of our API calls also require an authorized user to be associated with the client sesion.  In order to authenticate a user, you must pass in the users name or email along with their password.

```javascript
// ...
// Wait till client is ready
client.ready(function(){
    // Authenticate specific user
    client.login( "demo@example.com", "mypassword");
}
	
client.onLogin(function(){
    // Do something if/when a user is successfully authenticated
    // ...
});
	
client.onLogout(function(){
    // Do something if/when a user is no longer authenticated
    // ...
});
```

Fetching Data
-------------

To retrieve a set of a particular Mojio entities, you can use the "get" method.  The returned results will depend on what user and application your client session is authorized as. Lists of data will be returned in a paginated form.  You are able to set the page size and request a particular page.  In order to keep response times fast, it is recommended to keep the page size low.

```javascript
// ...
// Fetch first page of 15 trips
client.get('trips',null,null,15).done( function( results ) {
		
    // Iterate over each trip
    for( i = 0; i &lt; results.Data.length ; i++ )
    {
        // Trip
        var trip = results.Data[i];
	
        // Do something with each trip
        // ...
    }
});
```

Fetch a specific Entity
-----------------------

By passing in the ID of an entity (often a GUID), you can fetch just that single entity from the database.

```javascript
// ...
var mojioId = "123451234512345";
	
// Fetch Mojio from database
client.get('mojios',mojioId).done( function(mojio) {
	
    // Do something with the mojio data
    // ...
});
```

Update an Entity
----------------

If you want to update and save an entity, you need to first load the entity from the API, make your changes, and then save it back.  Typically only the owner of an entity will be authorized to save changes and not all properties of an entity will be editable (for example, for an App, only the Name and Description properties can be changed).

```javascript
// ...
var appId = new Guid("0a5123a0-7e70-12d1-a5k6-28db18c10200");
	
// Fetch app from API
client.get("apps",appId).done(function(app) {
	
    // Make a change
    app.Name = "New Application Name";
		
    // Save the changes
    client.save("apps",app);
});
```

Get a list of child entities
----------------------------

If you want to fetch all the entities associated with another entity, you can call the GetBy method.  For example, if you want to fetch all the events associated with a mojio device.

```javascript
// ...
var mojioId = "123451234512345";
	
// Fetch mojio's events
client.get('mojios',mojioId,'events').done( function(events) {

    // ...
});
```

Using the Mojio Storage
-----------------------

With the Mojio API, you are able to store your own private data within our database as key value pairs.  These key value pairs will only be accessible by your application, and you can associate them with any Mojio entities (ex: Mojio Device, Application, User, Trip, Event, Invoice, Product).

```javascript
// ...
var userId = "0a5453a0-7e70-16d1-a2w6-28dl98c10200";  // Some user's ID
string key = "EyeColour";	// Key to store
string value = "Brown"; 	// Value to store

// Save user's eye colour
client.store( "users", userId, key , value );
	
// ...
// Retrieve user's eye colour
var stored = client.store( "users", userId, key );
```

Using SignalR to listen for events
----------------------------------

Instead of continuously polling the API to see if any new events have come in, our API has a signalR service you can subscribe to in order to be sent new event notifications as they happen.

```javascript
// ...
    // The Mojio ID you wish to listen to
    var mojioId = "123451234512345";
	
    // An array of event types you wish to be notified about
    var types = ["Tow","GPS"];
	
    // Setup the callback function
    var callback = function(event){
        if( event.EventType == 'GPS') {
            // A new GPS event was received!
            // ...
	    }
	    else if( event.EventType == 'Tow' )
	    {
            // Do something with the new tow alert
            // ...
        }
    }
	
    // Bind the event listener
    client.onEvent( callback );
	
    // Subscribe to a particular Mojio Device
    client.subscribe('mojios',mojioId,types);
	
    // ...
    // Unsubscribe from events.
    client.unsubscribe('mojios',mojioId,types);
```
