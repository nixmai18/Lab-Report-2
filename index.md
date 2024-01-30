# CSE 15L Lab Report 2: Servers and SSH Keys 

## Part 1: Web Server: ChatServer

**ChatServer.java Code:**

```
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.net.URI;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

class StringHandler implements URLHandler {
  List<String> lines;
  String path;
  StringHandler(String path) throws IOException {
    this.path = path;
    this.lines = Files.readAllLines(Paths.get(path));
  }
  public String handleRequest(URI url) throws IOException {
    String query = url.getQuery();
    if(url.getPath().equals("/add-message")) {
      if(query.startsWith("s=")) {
        String[] args = query.split("&");
        String[] userArr = args[1].split("=");
        String user = userArr[1].replace("+", " ");
    
        String[] valueArr = args[0].split("=");
        String statement = valueArr[1].replace("+", " ");

        this.lines.add(user + ": " + statement);
    
        return String.join("\n", lines) + "\n";
      }
      else {
        return "requires a parameter";
      }
    }
    else {
      return String.join("\n", lines) + "\n";
    }
  }
}

class ChatServer {
  public static void main(String[] args) throws IOException {
    if(args.length == 0){
      System.out.println("Missing both port number and file path! For the first argument (port number), try any number between 1024 to 49151. For the second argument (file path), give a path to a text file.");
      return;
    }
    if(args.length == 1){
      System.out.println("Missing port number or file path! For the first argument (port number), try any number between 1024 to 49151. For the second argument (file path), give a path to a text file.");
      return;
    }

    int port = Integer.parseInt(args[0]);

    Server.start(port, new StringHandler(args[1]));
  }
}
```

**Server.java Code:**

```
// A simple web server using Java's built-in HttpServer

// Examples from https://dzone.com/articles/simple-http-server-in-java were useful references

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.URI;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

interface URLHandler {
    String handleRequest(URI url);
}

class ServerHttpHandler implements HttpHandler {
    URLHandler handler;
    ServerHttpHandler(URLHandler handler) {
      this.handler = handler;
    }
    public void handle(final HttpExchange exchange) throws IOException {
        // form return body after being handled by program
        try {
            String ret = handler.handleRequest(exchange.getRequestURI());
            // form the return string and write it on the browser
            exchange.sendResponseHeaders(200, ret.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(ret.getBytes());
            os.close();
        } catch(Exception e) {
            String response = e.toString();
            exchange.sendResponseHeaders(500, response.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(response.getBytes());
            os.close();
        }
    }
}

public class Server {
    public static void start(int port, URLHandler handler) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        //create request entrypoint
        server.createContext("/", new ServerHttpHandler(handler));

        //start the server
        server.start();
        System.out.println("Server Started!");
    }
}
```


**First add message screenshot:**

![Image](1st.png)


**Explanation:**
The method in my code that is called is the ```handle``` method in the ```ServerHttpHandler``` class which implements the ```HttpHandler``` interface. 
The relevant argument to that method is the ```HttpExchange exchange``` which represents the Http request and response. Some of the relevant fields include 
the ```URLHandler handler``` in the ```ServerHttpHandler```. This is an instance of the ```StringHandler```. Another relevant field is the ```List<String> lines``` and ```String path```
in ```StringHandler```. These represent the the lines read from a file path. If the request is valid, then the new statement or message is added to the ```lines``` list. Then this 
list is later returned. 

**Second add message screenshot:**

![Image](2nd.png)


**Explanation:**
The method in my code that is called is the ```handle``` method in the ```ServerHttpHandler``` class which implements the ```HttpHandler``` interface. 
The relevant argument to that method is the ```HttpExchange exchange``` which represents the Http request and response. Some of the relevant fields include 
the ```URLHandler handler``` in the ```ServerHttpHandler```. This is an instance of the ```StringHandler```. Another relevant field is the ```List<String> lines``` and ```String path```
in ```StringHandler```. These represent the the lines read from a file path. If the request is valid, then the new statement or message is added to the ```lines``` list. Then this 
list is later returned. 


**Private Key Absolute Path using ```ls```:**

**Public Key Absolute Path using ```ls```:**

**Login without using password:**





