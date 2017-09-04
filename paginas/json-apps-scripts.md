# Usar JSON con Apps Scripts

Fuente original: http://googleappscripting.com/json/


## Downloading JSON from an API with UrlFetchApp

APIs are probably the most entertaining way to get JSON. For this example, we will use the Chuck Norris Quote API. Calls to the http://api.icndb.com/jokes/random API endpoint return JSON that looks like this:


{
    "type": "success",
    "value": {
        "id": 149, 
        "joke": "Chuck Norris proved that we are alone in the universe. We weren't before his first space expedition.",
        "categories": []
    }
}
{
    "type": "success",
    "value": {
        "id": 149, 
        "joke": "Chuck Norris proved that we are alone in the universe. We weren't before his first space expedition.",
        "categories": []
    }
}
(This JSON is pretty printed but the API does not actually return pretty-printed JSON)

To get request JSON from this API, all you have to do is:


// make a GET request to the API
var chuckNorrisJSON = UrlFetchApp.fetch("http://api.icndb.com/jokes/random");

// log the text that you retrieved
Logger.log(chuckNorrisJSON)
1
2
3
4
5
// make a GET request to the API
var chuckNorrisJSON = UrlFetchApp.fetch("http://api.icndb.com/jokes/random");
 
// log the text that you retrieved
Logger.log(chuckNorrisJSON)
We will see how to parse and query this later.

 

Receiving JSON with doPost
If you want to POST JSON from your computer or another cloud application, you can use the Google Spreadsheet Web App feature to receive the data. From there, it is quite easy to save that JSON text to a file in Google Drive.

The doPost() function runs anytime the web app receives a POST request. It takes one argument. That argument can be named anything you want but it is an object that stores information about the request. In this case, we access the string representation of the JSON data from the request parameter.


function doPost(request){
  
  // get the sting value of the POST data
  var postJSON = request.postData.getDataAsString();

  // save that JSON to a file
  DriveApp.createFile('post.json', postJSON);
  
}
1
2
3
4
5
6
7
8
9
function doPost(request){
  
  // get the sting value of the POST data
  var postJSON = request.postData.getDataAsString();
 
  // save that JSON to a file
  DriveApp.createFile('post.json', postJSON);
  
}
To test this little script, you can use Postman to send a POST request with raw JSON data. Or you can copy this into your (Mac) terminal.


curl -H "Content-Type: application/json" -X POST -d '{"greeting":"hello","recipient":"world"}' https://script.google.com/yourscriptURL
1
curl -H "Content-Type: application/json" -X POST -d '{"greeting":"hello","recipient":"world"}' https://script.google.com/yourscriptURL
When the above POST request is made, a file named post.json will be created in Google Drive with the contents:


{"greeting":"hello","recipient":"world"}
1
{"greeting":"hello","recipient":"world"}
 

Accessing JSON Files From Google Drive
Let’s continue from the example above. If you wanted to access the file called post.json from Google Drive,  you can then read the file in as JSON and work with its contents in your script.


  // define a file iterator
  // that will iterate through all files with the name 'post.json'
  var iter = DriveApp.getFilesByName("post.json");
  
  // iterate through all the files named 'post.json
  while (iter.hasNext()) {
    
    // define a File object variable and set the Media Tyep
    var file = iter.next();
    var jsonFile = file.getAs('application/json')
    
    // log the contents of the file
    Logger.log(jsonFile.getDataAsString());
  
  }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
  // define a file iterator
  // that will iterate through all files with the name 'post.json'
  var iter = DriveApp.getFilesByName("post.json");
  
  // iterate through all the files named 'post.json
  while (iter.hasNext()) {
    
    // define a File object variable and set the Media Tyep
    var file = iter.next();
    var jsonFile = file.getAs('application/json')
    
    // log the contents of the file
    Logger.log(jsonFile.getDataAsString());
  
  }
 

Parsing and Querying JSON with Google Apps Script
If you have any experience with Javascript, this should be a walk in the park. But in case you are new to it, let’s take a look at the JSON Google Apps Script Class.

The JSON Class offers two different methods for working with JSON. JSON.parse() reads in the JSON text into a Javascript object, and JSON.stringify() serializes (a.k.a. converts) Javascript objects into JSON text strings.

 

Parsing JSON with JSON.parse
As we saw above, when you read in or receive JSON, it will be represented as a string. But in order to use that within your script, you will need to convert it into an object. That is really easy to do. Let’s use the Chuck Norris JSON to examine this.


var chuckNorrisObject = JSON.parse(chuckNorrisJSON);
1
var chuckNorrisObject = JSON.parse(chuckNorrisJSON);
Now we have Javascript object that we can use in our script.

 

Querying JSON
Querying just means looking something within a data source. In our case, we are less querying than JSON than a Javascript object, but you get the idea. Following from the example above:


Logger.log(chuckNorrisObject.value.id)             // logs 149
Logger.log(chuckNorrisObject.value['joke'])        // logs "Chuck Norris proved...."
Logger.log(chuckNorrisObject.value.categories[0])  // logs undefined
Logger.log(chuckNorrisObject.value.shoeSize)       // also logs undefined
1
2
3
4
Logger.log(chuckNorrisObject.value.id)             // logs 149
Logger.log(chuckNorrisObject.value['joke'])        // logs "Chuck Norris proved...."
Logger.log(chuckNorrisObject.value.categories[0])  // logs undefined
Logger.log(chuckNorrisObject.value.shoeSize)       // also logs undefined
Notice, that with Javascript you can access object properties with either “dot notation” or “bracket notation.” Dot notation is convenient but bracket notation is useful when you need to access an object’s property with a variable rather than a string.

Arrays can only be accessed with bracket notation. Note that the index for the array starts with zero and if an array doesn’t have a specified index or an object doesn’t have a specified property, the result will be undefined. JSON uses null, but undefined is the Javascript equivalent.

 

Creating JSON with Google Apps Script
Now that we have seen a few ways that we can get JSON, let’s see a few ways that we can create it. When you have a Javascript object, serializing JSON is very easy:


var myDog = {"name": "Rhino", "breed": "pug", "age": 8}

var myJSON = JSON.stringify(myDog);
1
2
3
var myDog = {"name": "Rhino", "breed": "pug", "age": 8}
 
var myJSON = JSON.stringify(myDog);
To provide a user or another application we can reverse the methods that we used to get JSON above.

 

Return JSON with doGet
doGet, similar to doPost, responds to requests to your web app. To return JSON to a GET request, we simply handle the request with doGet, then return JSON text with the ContentService.


var myDog = {"name": "Rhino", "breed": "pug", "age": 8}

var myJSON = JSON.stringify(myDog);

function doGet(request){
  
  // return JSON text with the appropriate Media Type
  return ContentService.createTextOutput(myJSON).setMimeType(ContentService.MimeType.JSON);

}
1
2
3
4
5
6
7
8
9
10
var myDog = {"name": "Rhino", "breed": "pug", "age": 8}
 
var myJSON = JSON.stringify(myDog);
 
function doGet(request){
  
  // return JSON text with the appropriate Media Type
  return ContentService.createTextOutput(myJSON).setMimeType(ContentService.MimeType.JSON);
 
}
The ContentService class provides a set of utilities to create and define several types of output, including images, HTML, and of course, JSON. Whatever the doGet function returns is what is returned the client that makes the request.

 

Send JSON with UrlFetchApp
Earlier, we saw how to receive POST requests and accept the JSON payload data. Here we see how to send the POST request using the Google Apps Script UrlFetchApp function.


var options = {
  'method' : 'post',                  // specify the request type
  'contentType': 'application/json',  // specify the Media Type 
  'payload' : myJSON                  // my JSON stringified object
};

// send the request
UrlFetchApp.fetch('https://example.com/post', options);
1
2
3
4
5
6
7
8
var options = {
  'method' : 'post',                  // specify the request type
  'contentType': 'application/json',  // specify the Media Type 
  'payload' : myJSON                  // my JSON stringified object
};
 
// send the request
UrlFetchApp.fetch('https://example.com/post', options);
The second argument for UrlFetchApp.fetch is an object the specifies the details of the request. In this example, we specified a POST requests with the JSON content-type header and the JSON data from the example above.

 

Save a JSON file to Google Drive
This was show above, but since you’re here, let’s look at it again. To save a JSON file to Google Drive, serialize the Javascript object to text, and use the DriveApp class to save the file to Drive. Again, we will use the JSON from myDog object above.


  // save that JSON to a file
  DriveApp.createFile('rhino.json', myJSON);
1
2
  // save that JSON to a file
  DriveApp.createFile('rhino.json', myJSON);
Yes, it is that simple. Just search for “rhino.json” in Google Drive and you will see a file with that name. You can also dynamically name the file by passing in a variable to the first argument to DriveApp.createFile.

 

Printing a JSON data to Google Spreadsheets
One of the best things about Google Apps Script and Google Spreadsheets is that it is so easy to get data from APIs into spreadsheets. Here’s an example to create a spreadsheet header with the object keys and set the row values to the values of the object.


// define an array of all the object keys
var headerRow = Object.keys(myObject);

// define an array of all the object values
var row = headerRow.map(function(key){ return myObject[key]});

// define the contents of the range
var contents = [
 headerRow,
 row
];

// select the range and set its values
var ss = SpreadsheetApp.getActive();
var rng = ss.getActiveSheet().getRange(1, 1, contents.length, headerRow.length )
rng.setValues(contents)
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
// define an array of all the object keys
var headerRow = Object.keys(myObject);
 
// define an array of all the object values
var row = headerRow.map(function(key){ return myObject[key]});
 
// define the contents of the range
var contents = [
 headerRow,
 row
];
 
// select the range and set its values
var ss = SpreadsheetApp.getActive();
var rng = ss.getActiveSheet().getRange(1, 1, contents.length, headerRow.length )
rng.setValues(contents)
If you have an array of multiple objects, you could run them all through a loop to populate the contents array and then write the values of all the objects to the spreadsheet.
