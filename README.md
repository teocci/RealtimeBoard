# Web Application based on WebSocket API for Java

This tutorial demonstrates how to create a simple web application that enables collaboration between client browsers that are connected to a single server. When a user draws a graphic element on the canvas in his client browser the element appears on the canvas of all connected clients. 

How does it work? 
Well, when the browser loads the web page a client-side script sends a WebSocket handshake request to the application server. The application can accept JSON and binary messages from the clients connected in the session and broadcast the messages to all the connected clients.

In this github project you can find a web application that uses the Java API for WebSocket ([JSR 356](http://www.jcp.org/en/jsr/detail?id=356)) to enable bi-directional communication between browser clients and the application server. The Java API for WebSocket provides support for creating WebSocket Java components, initiating and intercepting WebSocket events and creating and consuming WebSocket text and binary messages. This repository will also demonstrate how you can use the Java API for JSON Processing ([JSR 353](http://jcp.org/en/jsr/detail?id=353)) to produce and consume JSON. The Java API for WebSocket and the Java API for JSON Processing are part of the Java EE 7 platform ([JSR 342](http://jcp.org/en/jsr/detail?id=342)).

The application contains a WebSocket `endpoint`, a `decoder` and `encoder` interfaces, a web page and some JavaScript files that are run in the client browser when the page is loaded or when invoked from a form in the web page. You will deploy the application to GlassFish Server Open Source Edition 4, the reference implementation of Java EE 7 technology.

Note. This tutorial is based on the [Collaborative Whiteboard using WebSocket in GlassFish 4 - Text/JSON and Binary/ArrayBuffer Data Transfer (TOTD #189)](https://blogs.oracle.com/arungupta/entry/collaborative_whiteboard_using_websocket_in) blog post.


Tutorial Exercises
Content on this page applies to IntelliJ IDEA 2016-2

* [Creating the Web Application Project](#creating-the-web-application-project)
* [Creating the WebSocket Endpoint](#creating-the-websocket-endpoint)
    - [Create the Endpoint](#creating-the-endpoint)
    - [Initiate the WebSocket Session](#initiate-the-websocket-session)
    - [Test the Endpoint](#testing-the-endpoint)
* [Creating the RealtimeBoard](#creating-the-realtimeboard)
    - [Add the Canvas](#add-the-canvas-to-the-web-page)
    - [Create the POJO](#creating-the-pojo)
    - [Create a Coordinates Class](#create-a-coordinates-class)
    - [Generate the JSON String](#generate-the-json-string)
    - [Implement the Encoder and Decoder Interfaces](#implement-the-encoder-and-decoder-interfaces)
    - [Run the Application](#running-the-application)
* [Sending Binary Data to the Endpoint](#sending-binary-data-to-the-endpoint)
* [References](#references)

To follow this tutorial, you need the following software and resources.

| Software or Resource                  | Version Required  |
| ------------------------------------- | -----------------:|
| IntelliJ IDEA                         | version 2016-2    |
| Java Development Kit (JDK)            | version 7 or 8    |
| GlassFish Server Open Source Edition  | 4.1                 |

Note. GlassFish 4.1 can be [download here](https://glassfish.java.net/download.html).

## Prerequisites

This document assumes you have some basic knowledge of, or programming experience with, the following technologies:

* Java Programming
* JavaScript/HTML Programming

You can fork this repository or [download a zip archive](https://github.com/teocci/RealtimeBoard/archive/master.zip) of the finished project.

## Creating the Web Application Project
The goal of part is to create a web application project using the Project wizard in the IDE. When you create the project you will select Java EE 7 as the Java EE version and GlassFish 4.1 as the application server. GlassFish 4.1 is the reference implementation of the Java EE 7 platform. You must have an application server that supports Java EE 7 registered with the IDE to create the application in this tutorial.

1. Choose `File` > `New Project` from the main menu.
2. Select Java Enterprise. Click Next.
3. Select GlassFish Server 4.1 for the Server.
4. Set the Java EE Version to Java EE 7 Web.
5. Select `Web Application` from the `Additional Librares and Framaeworks`. Click Next.
6. Type RealtimeBoard for the the Project Name and set the Project Location. Click Finish.


<img src="http://i.imgur.com/THzfAPL.png" height="371" width="419" ><img src="http://i.imgur.com/4Yms5aP.png" height="371" width="419" >

When you click Finish, the IDE creates the project and opens the project in the Projects window.

## Creating the WebSocket Endpoint

In this section you will create a WebSocket endpoint class and a JavaScript file. The WebSocket endpoint class contains some basic methods that are run when the session is opened. You will then create a JavaScript file that will initiate the handshake with the server when the page is loaded. You will then run the application to test that the connection is successful.

For more about using WebSocket APIs and annotations, see the summary of the javax.websocket package.

## Creating the Endpoint

In this exercise you will use a wizard in the IDE to help you create the WebSocket endpoint class.

1. Right-click the Source Packages node (`src`) in the Projects window and choose New > Package.
2. Type, for example net.teocci.websocket. Click OK.
3. After the Package creation, right-click the new Package created (`net.teocci.websocket`) in the Projects window and choose New > Java Class.
4. Type BoardServer as the Class Name. Click OK.

    When you click OK the IDE generates the BoardServer class and opens the file in the source editor. In order to make this class as a Websocket Endpoint we need to add some annotations that are part of the WebSocket API. The class should be annotated with `@ServerEndpoint` to identify the class as an endpoint and the WebSocket URI (**wsURI**) must be specified as a parameter of the annotation. We also need to add a default onMessage method that is annotated with `@OnMessage`. A method annotated with `@OnMessage` is invoked each time that the client receives a WebSocket message. Now copy and past this piece of code:

    ```
    @ServerEndpoint("/actions")
    public class BoardServer {

        @OnMessage
        public String onMessage(String message) {
            return null;
        }
        
    }
    ```

5. Add the following `private static` peers field to the class.

    ```
    @ServerEndpoint("/actions")
    public class BoardServer {
        private static Set<Session> peers = Collections.synchronizedSet(new HashSet<Session>());

        @OnMessage
        public String onMessage(String message) {
            return null;
        }
    }
    ```
    
6. Add the following `onOpen` and `onClose` methods.

    ```
    @OnOpen
    public void onOpen (Session peer) {
        peers.add(peer);
    }

    @OnClose
    public void onClose (Session peer) {
        peers.remove(peer);
    }
    ```

    You can see that the `onOpen` and `onClose` methods are annotated with `@OnOpen` and `@OnClose` WebSocket API annotations. A method annotated with `@OnOpen` is called when the web socket session is opened. In this example the annotated `onOpen` method adds the browser client to the group of peers in the current session and the `onClose` method removes the browser from the group.

    Use the hints and code completion in the source editor to help you generate the methods. Click the hint glyph in the left margin next to the class declaration (or place the insert cursor in the class declaration and type Alt-Enter) and select the method in the popup menu. The code completion can help you code the method.
    screenshot of Code Hint in the Source Editor

7. Right-click in the editor and choose Fix Imports. Save your changes.

    You will see that import statements for classes in `javax.websocket` are added to the file.

The endpoint is now created. You now need to create a JavaScript file to initiate the WebSocket session.

## Initiate the WebSocket Session

We have our Endpoint ready but now we need to create a JavaScript file that will initiate a WebSocket session. This is, when the browser client joins a session via an HTTP 'handshake' with the server over TCP. In the JavaScript file you will specify the name of the **wsURI** of the endpoint and declare the WebSocket. The wsURI URI scheme is part of the WebSocket protocol and specifies the path to the endpoint for the application.

1. Right-click the web node in the Projects window and choose New > Directory.
2. Type -js- for the JavaScript Directory. Click OK.
3. Right-click the the -js- directory node in the Projects window and choose New > JavaScript File.
4. Type -websocket- for the JavaScript File Name. Click OK.
5. Add the following to the JavaScript file.

    ```
    var wsUri = "ws://" + document.location.host + document.location.pathname + "actions";
    var websocket = new WebSocket(wsUri);

    websocket.onerror = function(evt) { onError(evt) };

    function onError(evt) {
        writeToScreen('<span style="color: red;">ERROR:</span> ' + evt.data);
    }
    ```

    This script will initiate the session handshake with the server when `websocket.js` is loaded by the browser.

5. Open index.html and add the following code to the bottom of the file to load `websocket.js` when the page is finished loading.

    ```
    <body>
        <h1>Collaborative Whiteboard App</h1>
            
        <script type="text/javascript" src="js/websocket.js"></script>
    </body>
    ```

We can now test that the WebSocket endpoint is working and that the session is started and the client is added to the session.

## Testing the Endpoint
Now we will add some simple methods to the JavaScript file to print the wsURI to the browser window when it is connected to the endpoint.

1. Add the following `<div>` tag to `index.jsp` file

    ```
    <h1>Collaborative Whiteboard App</h1>
            
    <div id="output"></div>
    <script type="text/javascript" src="js/websocket.js"></script>
    ```

2. Add the following declaration and methods to `websocket.js`. Save your changes.

    ```
    // For testing purposes
    var output = document.getElementById("output");
    websocket.onopen = function(evt) { onOpen(evt) };

    function writeToScreen(message) {
        output.innerHTML += message + "<br>";
    }

    function onOpen() {
        writeToScreen("Connected to " + wsUri);
    }
    // End test functions
    ```

    When the page loads the JavaScript functions will print the message that the browser is connected to the endpoint. You can delete the functions after you confirm that the endpoint is performing correctly.

3. Now click on the green triangle from the Toolbar to run the project.

When you run the application the IDE will start the GlassFish server and build and deploy the application. The index page will open in your browser and you will see the following message in the browser window.

<img src="http://i.imgur.com/Fym2UT0.png" height="90%" width="90%" >

In the browser window you can see the following endpoint where messages are accepted: -http://localhost:8080/RealtimeBoard/actions-

## Creating the RealtimeBoard

In this section you will create the classes and JavaScript files to send and receive JSON text messages. You will also add an HTML5 Canvas element for painting and displaying some content and an HTML `<form>` with radio buttons that enable you to specify the shape and color of the paintbrush.

### Add the Canvas to the Web Page
In this exercise you add a canvas element and a form element to the default index page. The checkboxes in the form determine the properties of the paintbrush for the canvas.

1. Open index.html in the source editor.
2. Delete the `<div>` tag that you added to test the endpoint and add the following `<table>` and `<form>`elements after the opening body tag.

    ```
    <h1>Collaborative Board App</h1>
    <table>
        <tr>
            <td>
                ...
            </td>
            <td>
                <form name="inputForm">
                    ... 
                </form>
            </td>
        </tr>
    </table>
    <script type="text/javascript" src="js/websocket.js"></script>
    </body>
    ```

3. Add the following code for the canvas element.

    ```
    <table>
        <tr>
            <td>
                <canvas id="myCanvas" width="150" height="150" style="border:1px solid #000000;"></canvas>
            </td>
            ...
    ```

4. Add the following `<table>` to add radio buttons to select the color and shape. Save your changes.

    ```
    <table>
        <tr>
            <td>
                <canvas id="myCanvas" width="150" height="150" style="border:1px solid #000000;"></canvas>
            </td>
            <td>
                <form name="inputForm">
                    <table>
                        <tr>
                            <th>Color</th>
                            <td><input type="radio" name="color" value="#FF0000" checked="true">Red</td>
                            <td><input type="radio" name="color" value="#0000FF">Blue</td>
                            <td><input type="radio" name="color" value="#FF9900">Orange</td>
                            <td><input type="radio" name="color" value="#33CC33">Green</td>
                        </tr>
                        <tr>
                            <th>Shape</th>
                            <td><input type="radio" name="shape" value="square" checked="true">Square</td>
                            <td><input type="radio" name="shape" value="circle">Circle</td>
                            <td> </td>
                            <td> </td>
                        </tr>
                    </table>
                </form>
                ...
    ```

The shape, color, and coordinates of any figure drawn on the canvas will be converted to a string in a JSON structure and sent as a message to the WebSocket endpoint.

## Creating the POJO
Now let's create a simple POJO.

1. Right-click the project node and choose New > Java Class.
2. Type Figure as the Class Name and choose net.teocci.model in the Package dropdown list. Click Finish.
3. In the source editor, add the following:

    ```
    public class Figure {
        private JsonObject json;
    }
    ```
    
    When you add the code you will be prompted to add an import statement for javax.json.JsonObject. If you are not prompted, type Alt-Enter.
    
    For more about javax.json.JsonObject, see the Java API for JSON Processing (JSR 353), which is part of the Java EE 7 Specification.
    
4. Create a getter and setter for json.

    You can select getter and setter in the Insert Code popup menu to open the Generate 
    
    Getters and Setter dialog box. Alternatively, you can choose Source > Insert Code from the main menu.
    
    Generate Getter and Setter dialog box
    
5. Add a constructor for json.
    
    ```
    public Figure(JsonObject json) {
        this.json = json;
    }
    ```
    
    You can choose Constructor in the Insert Code popup menu (Ctrl-I).
    
    Generate Constructor popup menu
    
6. Add the following toString method:

    ```
    @Override
    public String toString() {
        StringWriter writer = new StringWriter();
        Json.createWriter(writer).write(json);
        return writer.toString();
    }
    ```
    
7. Right-click in the editor and choose Fix Imports. Save your changes.

### Create a Coordinates Class

You now create a class for the coordinates of the figures that are painted on the canvas.

1. Right-click the project node and choose New > Java Class.
2. In the New Java Class wizard, type Coordinates as the Class Name and select org.sample.whiteboardapp in the Package dropdown list. Click Finish.
3. In the source editor, add the following code. Save your changes.

    ```
    private float x;
    private float y;

    public Coordinates() {
    }

    public Coordinates(float x, float y) {
        this.x = x;
        this.y = y;
    }

    public float getX() {
        return x;
    }

    public void setX(float x) {
        this.x = x;
    }

    public float getY() {
        return y;
    }

    public void setY(float y) {
        this.y = y;
    }
    ```               

The class only contains a fields for the x and y coordinates and some getters and setters.

### Generate the JSON String
In this exercise you will create a JavaScript file that puts the details of the figure that is drawn on the canvas element into a JSON structure that is sent to the websocket endpoint.

1. Right-click the project node and choose New > JavaScript File to open the New JavaScript File wizard.
2. Type whiteboard for the File Name. Click Finish.
    
    When you click Finish the IDE creates the empty JavaScript file and opens the file in the editor. You can see the new file under the Web Pages node in the Projects window.

3. Add the following code to initialize the canvas and to add an event listener.

    ```
    var canvas = document.getElementById("myCanvas");
    var context = canvas.getContext("2d");
    canvas.addEventListener("click", defineImage, false);
    ```
    
    You can see that the defineImage method is invoked when the user clicks in the canvas element.
    
4. Add the following getCurrentPos, defineImage and drawImageText methods to construct the JSON structure and send it to the endpoint (sendText(json)).

    ```
    function getCurrentPos(evt) {
        var rect = canvas.getBoundingClientRect();
        return {
            x: evt.clientX - rect.left,
            y: evt.clientY - rect.top
        };
    }
                
    function defineImage(evt) {
        var currentPos = getCurrentPos(evt);
        
        for (i = 0; i < document.inputForm.color.length; i++) {
            if (document.inputForm.color[i].checked) {
                var color = document.inputForm.color[i];
                break;
            }
        }
                
        for (i = 0; i < document.inputForm.shape.length; i++) {
            if (document.inputForm.shape[i].checked) {
                var shape = document.inputForm.shape[i];
                break;
            }
        }
        
        var json = JSON.stringify({
            "shape": shape.value,
            "color": color.value,
            "coords": {
                "x": currentPos.x,
                "y": currentPos.y
            }
        });
        drawImageText(json);
            sendText(json);
    }

    function drawImageText(image) {
        console.log("drawImageText");
        var json = JSON.parse(image);
        context.fillStyle = json.color;
        switch (json.shape) {
        case "circle":
            context.beginPath();
            context.arc(json.coords.x, json.coords.y, 5, 0, 2 * Math.PI, false);
            context.fill();
            break;
        case "square":
        default:
            context.fillRect(json.coords.x, json.coords.y, 10, 10);
            break;
        }
    }
    ```
    
    The JSON structure that is sent will be similar to the following:

    ```
    {
     "shape": "square",
     "color": "#FF0000",
     "coords": {
     "x": 31.59999942779541,
     "y": 49.91999053955078
     }
    }
    ```
    
    You now need to add a sendText(json) method to send the JSON string using websocket.send().
    
5. Open websocket.js in the editor and add the following methods for sending JSON to the endpoint and for drawing the image when a message is received from the endpoint.

    ```
    websocket.onmessage = function(evt) { onMessage(evt) };

    function sendText(json) {
        console.log("sending text: " + json);
        websocket.send(json);
    }
                    
    function onMessage(evt) {
        console.log("received: " + evt.data);
        drawImageText(evt.data);
    }
    ```
    
    Note. You can delete the code that you added to websocket.js for testing the endpoint.
    
6. Add the following line to the bottom of index.html to load js/board.js.

    ```
            </table>
        <script type="text/javascript" src="js/websocket.js"></script>
        <script type="text/javascript" src="js/board.js"></script>
    <body>
    ```    

### Implement the Encoder and Decoder Interfaces
In this exercise you create classes to implement decoder and encoder interfaces to decode web socket messages (JSON) to the POJO class Figure and to encode Figure as a JSON string for sending to the endpoint.

For more details, see the section about message types and encoders and decoders in the technical article JSR 356, Java API for WebSocket.

1. Right-click the project node and choose New > Java Class.
2. Type FigureEncoder as the Class Name and choose org.sample.whiteboardapp in the Package dropdown list. Click Finish.
3. In the source editor, implement the WebSocket Encoder interface by adding the following code:

    ```
    public class FigureEncoder implements Encoder.Text<Figure> {
        
    }
    ```

4. Add an import statement for javax.websocket.Encoder and implement the abstract methods.

    Place your cursor in the class declaration and type Alt-Enter and choose Implement all abstract methods from the popup menu.

5. Modify the generated abstract methods by making the following changes. Save your changes.

    ```
    @Override
    public String encode(Figure figure) throws EncodeException {
        return figure.getJson().toString();
    }

    @Override
    public void init(EndpointConfig ec) {
        System.out.println("init");
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    }
    ```

6. Right-click the project node and choose New > Java Class.
7. Type FigureDecoder as the Class Name and choose org.sample.whiteboardapp in the Package dropdown list. Click Finish.
8. In the source editor, implement the WebSocket Decoder interface by adding the following code:

    ```
    public class FigureDecoder implements Decoder.Text<Figure> {
        
    }
    ```
9. Add an import statement for javax.websocket.Decoder and implement abstract methods.
10. Make the following changes to the generated abstract methods.

    ```
    @Override
    public Figure decode(String string) throws DecodeException {
        JsonObject jsonObject = Json.createReader(new StringReader(string)).readObject();
        return  new Figure(jsonObject);
    }

    @Override
    public boolean willDecode(String string) {
        try {
            Json.createReader(new StringReader(string)).readObject();
            return true;
        } catch (JsonException ex) {
            ex.printStackTrace();
            return false;
        }
    
    }

    @Override
    public void init(EndpointConfig ec) {
        System.out.println("init");
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    }
    ```
    
11. Fix the imports and save your changes.

You now need to modify BoardServer.java to specify the encoder and decoder.

### Running the Application

You are now almost ready to run the application. In this exercise you modify the WebSocket endpoint class to specify the encoder and decoder for the JSON string and to add a method to send the JSON string to connected clients when a message is received.

1. Open BoardServer.java in the editor.
2. Modify the `@ServerEndpoint` annotation to specify the encoder and decoder for the endopoint. Note that you need to explicitly specify the value parameter for the name of the endpoint.

    ```
    @ServerEndpoint(value="/actions", encoders = {FigureEncoder.class}, decoders = {FigureDecoder.class})
    ```    

3. Delete the onMessage method that was generated by default.
4. Add the following broadcastFigure method and annotate the method with @OnMessage.
    
    ```
    @OnMessage
    public void broadcastFigure(Figure figure, Session session) throws IOException, EncodeException {
        System.out.println("broadcastFigure: " + figure);
        for (Session peer : peers) {
            if (!peer.equals(session)) {
                peer.getBasicRemote().sendObject(figure);
            }
        }
    }
    ```

5. Right-click in the editor and choose Fix Imports. Save your changes.
6. Right-click the project in the Projects window and choose Run.

When you click Run the IDE opens a browser window to `http://localhost:8080/RealtimeBoard/`.

Note. You might need to undeploy the previous application from the application server or force reload the page in the browser.

If you view the browser messages you can see that a string is sent via JSON to the endpoint each time you click in the canvas.
screenshot of application in browser

If you open another browser to `http://localhost:8080/RealtimeBoard/` you can see that each time you click in the canvas in one browser the new circle or square is reproduced in the canvas of the other browser.
screenshot of application in two browsers

## Sending Binary Data to the Endpoint

The application can now process and send a string via JSON to the endpoint and the string is then sent to the connected clients. In this section you will modify the JavaScript files to send and receive binary data.

To send binary data to the endpoint you need to set the binaryType property of WebSocket to `arraybuffer`. This ensures that any binary transfers using WebSocket are done using `ArrayBuffer`. The binary data conversion is performed by the `defineImageBinary` method in `js/board.js`.

1. Open websocket.js and add the following code to set the binaryType property of WebSocket to `arraybuffer`.


    ```
    websocket.binaryType = "arraybuffer";
    ```

2. Add the following method to send binary data to the endpoint.

    
    ```
    function sendBinary(bytes) {
        console.log("sending binary: " + Object.prototype.toString.call(bytes));
        websocket.send(bytes);
    }
    ```

3. Modify the `onMessage` method to add the following code to select the method for updating the `canvas` according to the type of data in the incoming message.

    ```
    function onMessage(evt) {
        console.log("received: " + evt.data);
        if (typeof evt.data == "string") {
            drawImageText(evt.data);
        } else {
            drawImageBinary(evt.data);
        }
    }
    ```

    The `drawImageBinary` method is invoked if a message with binary data is received.
    
4. Open `js/board.js` and add the following methods. The drawImageBinary method is invoked to update the canvas after parsing the incoming binary data. The defineImageBinary method is used to prepare a snapshot of the canvas as binary data.

    ```
    function drawImageBinary(blob) {
        var bytes = new Uint8Array(blob);
    //    console.log('drawImageBinary (bytes.length): ' + bytes.length);
        
        var imageData = context.createImageData(canvas.width, canvas.height);
        
        for (var i=8; i<imageData.data.length; i++) {
            imageData.data[i] = bytes[i];
        }
        context.putImageData(imageData, 0, 0);
        
        var img = document.createElement('img');
        img.height = canvas.height;
        img.width = canvas.width;
        img.src = canvas.toDataURL();
    }
                        
    function defineImageBinary() {
        var image = context.getImageData(0, 0, canvas.width, canvas.height);
        var buffer = new ArrayBuffer(image.data.length);
        var bytes = new Uint8Array(buffer);
        for (var i=0; i<bytes.length; i++) {
            bytes[i] = image.data[i];
        }
        sendBinary(buffer);
    }
    ```

    You now need to add a way to invoke `defineImageBinary` when you want to generate the binary data as the type ArrayBuffer and send it to the endpoint.
    
5. Open index.html and modify the `<table>` element to add the following row to the table in the form.

    ```
    <tr>
        <th> </th>
        <td><input type="submit" value="Send Snapshot" onclick="defineImageBinary(); return false;"></td>
        <td> </td>
        <td> </td>
        <td> </td>
    </tr>
    ```             

    The new row contains a Send Snapshot button to send a binary snapshot of the canvas to the connected peers. The `defineImageBinary` method in `board.js` is invoked when the button is clicked.
    
6. Open `BoardServer.java` and add the following method that will send the binary data to peers when the endpoint receives a message with binary data.

    ```
    @OnMessage
    public void broadcastSnapshot(ByteBuffer data, Session session) throws IOException {
        System.out.println("broadcastBinary: " + data);
        for (Session peer : peers) {
            if (!peer.equals(session)) {
                peer.getBasicRemote().sendBinary(data);
            }
        }
    }
    ```

    Note. You will need to add an import statement for `java.nio.ByteBuffer`.

You can modify the application to enable the user to stop sending data to the endpoint. By default all peers are connected as soon as they open the page and data is sent from the browser to all connected peers. You can add a simple conditional so that data is not sent to the endpoint unless the option is selected. This does not affect receiving data. Data is still received from the endpoint.

1. Modify the `defineImage` method in `board.js` to add the following code.

    ```
            drawImageText(json);
        if (document.getElementById("instant").checked) {
            sendText(json);
        }
    }
    ```

    The conditional code that you checks that if the element with the id checked

2. Open index.html and modify the <table> element to add a checkbox to the form.

    ```
    <tr>
        <th> </th>
        <td><input type="submit" value="Send Snapshot" onclick="defineImageBinary(); return false;"></td>
        <td><input type="checkbox" id="instant" value="Online" checked="true">Online</td>
        <td> </td>
        <td> </td>
    </tr>
    ```

    No data is sent when the `Online checkbox` is deselected, but the client will still receive data from the endpoint.

If you add the Send Snapshot button and the `Online checkbox` and run the application again you will see the new elements in the index page. If you open another browser and deselect the `Online` button you can see that the JSON message is not sent to the endpoint when you click in the canvas.
screenshot of application in browser

If you click Send Snapshot the binary data is sent to the endpoint and broadcast to the connected clients.

## References
* [Collaborative Whiteboard using WebSocket in GlassFish 4 - Text/JSON and Binary/ArrayBuffer Data Transfer (TOTD #189) ](https://blogs.oracle.com/arungupta/entry/collaborative_whiteboard_using_websocket_in)
* [Video of Using the WebSocket API in a Web Application](https://netbeans.org/kb/docs/javaee/maven-websocketapi-screencast.html)
* [Arun Gupta's Blog](http://blog.arungupta.me/)
