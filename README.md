# Python-HTTP-Server
This will include a basic web server in python hosted by a Linux VM, and will include steps to capture the traffic with Wireshark
      
## Create Login Page and Python Web Server _(in Linux)_
* mkdir ~/webserver
* cd ~/webserver
* nano index.html
```
<!DOCTYPE html>
<html>
<body>
  <h2>Login Page</h2>
  <form action="/login" method="POST">
    Username: <input type="text" name="username"><br>
    Password: <input type="password" name="password"><br>
    <input type="submit" value="Login">
  </form>
</body>
</html>
```
* nano server.py

```
from http.server import BaseHTTPRequestHandler, HTTPServer
import urllib.parse
 
class SimpleHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            with open("index.html", "rb") as f:
                self.send_response(200)
                self.send_header("Content-type", "text/html")
                self.end_headers()
                self.wfile.write(f.read())
 
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(length).decode('utf-8')
        data = urllib.parse.parse_qs(post_data)
        print("=== LOGIN ATTEMPT ===")
        print(f"Username: {data.get('username', [''])[0]}")
        print(f"Password: {data.get('password', [''])[0]}")
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Login submitted. Check server console.")
 
server = HTTPServer(('0.0.0.0', 8080), SimpleHandler)
print("Server started on http://localhost:8080")
server.serve_forever()
```
* python3 server.py

## Launch Wireshark
* Select interface: lo (loopback interface)
* Start Capture

## Visit Login Page
Go to: http://localhost:8080
* Enter sample credentials
* Login

## Stop Capture and Review
* Right-click a POST packet > Follow > HTTP Stream
* Voila, credentials should be visible. Easy when capturing packets in a Linux environment
* If too many packets: apply a filter such as: http, or tcp.port == 8080, or http.request.method == "POST", or frame contains "username"
* If still no POST, try incognito window and refresh the form to try again
