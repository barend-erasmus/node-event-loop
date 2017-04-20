# Node: Event Loop

In this document we'll cover:

1. What is the Event Loop?
2. What is single threaded?
3. Thread Pool

## What is the Event Loop?

| Event Queue                            | Order      
| ---------------------------------------|------------- |
| Write to console 'Reading file...'     | 1            |
| Queue file reading task                | 2            |
| Write to console 'Done.'               | 3            |
| .....                                  | 4            |
| .....                                  | 5            |
| Callback with file data                | 6            |
| Write to console                       | 7            |

### What is single threaded?

A single threaded application can be thought of as a long line at a cash register. Everything executes sequentially in order. The important take away is **YOUR** application does one thing at a time.

```javascript
    console.log('Reading file...');

    var fileBytes = fs.readFileSync(filename, "utf8");

    console.log('The file contains ' + fileBytes.length + ' bytes.');

    console.log('Done.');
```

```
    OUTPUT:

    Reading file...
    The file contains 142 bytes.
    Done.
```

Handling a web request

```javascript
    function handleRequest() {
        // Retrieve data from database

        // Calculate averages

        // Write data to database

        // Send JSON object to client

    }
```

In the example above, no requests will be processed while data is being retrieve or written to the database. Node is single threaded and can only perform **ONE** action at a time. 

This will reduce the number of request that your application can handle per second.

### Thread Pool

Apart from your application, Node runs an IO and Network thread pool which consists of 4 (by default) threads.

These threads can be used to offload IO and Network tasks.

Example of a non-blocking task.

```javascript
    console.log('Reading file...');

    fs.readFile('/etc/hosts', 'utf8', function (err,fileBytes) {
        console.log('The file contains ' + fileBytes.length + ' bytes');
    });

    console.log('Done.');
```

```
    OUTPUT:
    
    Reading file...
    Done.
    The file contains 142 bytes.
```

Example of a blocking task.

```javascript
    console.log('Reading file...');

    fs.readFile('/etc/hosts', 'utf8', function (err,fileBytes) {
        console.log('The file contains ' + fileBytes.length + ' bytes');
    });

    doSomeStuffThatTakesVeryLong();

    console.log('Done.');
```

What do you think the output will be?


```
    OUTPUT:
    
    Reading file...
    Done.
    The file contains 142 bytes.
```

Lets take another look at our snippet for handling a request.

```javascript

    function handleRequest() {
        // Retrieve data from database
        sql.query('SELECT * FROM ....', function (err1, result1) {

            // Calculate averages
            calculateAverages();

            // Write data to database
            sql.query('INSERT INTO ....', function (err2, result2) {
                // Send JSON object to client
                sendJSONToClient();
            }
        }
    }
```