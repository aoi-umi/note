
```nginx
upstream spider_server {
  server localhost:8888;
}

server {
    listen       80;
    server_name  example.com;

    location / {
      proxy_set_header  Host            $host:$proxy_port;
      proxy_set_header  X-Real-IP       $remote_addr;
      proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

      if ($http_user_agent ~* "Baiduspider|twitterbot|facebookexternalhit|rogerbot|linkedinbot|embedly|quora link preview|showyoubot|outbrain|pinterest|slackbot|vkShare|W3C_Validator|bingbot|Sosospider|Sogou Pic Spider|Googlebot|360Spider") {
        proxy_pass  http://spider_server;
      }
    }
}
```

```py
from http.server import HTTPServer, BaseHTTPRequestHandler
from selenium import webdriver
from time import sleep
from selenium.webdriver.firefox.options import Options
# import io
# import sys

# sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
options = Options()
options.add_argument('-headless')
options.add_argument('--disable-gpu')
driver = webdriver.Firefox(options=options)
def get(req):
    url = "http://localhost" + req.path
    driver.get(url)
    driver.implicitly_wait(30)
    sleep(5)
    data = driver.page_source
    # driver.close()
    return data


host = ('localhost', 8888)

class Resquest(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        # self.send_header('Content-type', 'application/json')
        self.end_headers()
        data = get(self)
        self.wfile.write(data.encode())


if __name__ == '__main__':
    server = HTTPServer(host, Resquest)
    print("Starting http server, listen at: %s:%s" % host)
    server.serve_forever()

```
