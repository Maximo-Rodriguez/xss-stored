# Stored XSS Lab

## Description

This lab demonstrates a basic **Stored Cross-Site Scripting (Stored XSS)** vulnerability.

Stored XSS happens when user-controlled input is stored and later rendered in the application without proper sanitization or escaping.

In this lab, the storage is simulated using a JavaScript array:

```js
let comments = [];
```

In a real application, this stored data could come from:

- a database
- a file
- a backend service
- a chat system
- a comment section
- a user profile field

---

## Files

- `vulnerable.html` — vulnerable version using `innerHTML`
- `secure.html` — secure version using `textContent`

---

## Vulnerable Flow

1. The user writes a comment.
2. The app stores the comment in an array.
3. The app renders all stored comments.
4. The vulnerable version uses `innerHTML`.
5. The browser interprets stored user input as HTML.
6. If the stored input contains malicious code, it can execute.

---

## Vulnerable Code

```js
let comments = [];

function addComment() {
    let value = document.getElementById("input").value;

    comments.push(value);

    render();

    document.getElementById("input").value = "";
}

function render() {
    let container = document.getElementById("comments");

    container.innerHTML = "";

    for (let c of comments) {
        container.innerHTML += "<p>" + c + "</p>";
    }
}
```

---

## Why It Is Vulnerable

The dangerous line is:

```js
container.innerHTML += "<p>" + c + "</p>";
```

The variable `c` represents each stored comment from the `comments` array.

If a user stores normal text, the application works normally:

```txt
Hello world
```

But if a user stores HTML or JavaScript-related code, `innerHTML` makes the browser interpret it as real HTML.

Example payload:

```html
<img src=x onerror="alert('Stored XSS')">
```

The browser tries to load an image from `src=x`.

Because the image does not exist, the `onerror` event runs and executes JavaScript.

That means stored user input is being executed by the browser.

---

## Example Payload

```html
<img src=x onerror="alert('Stored XSS')">
```

> Note: GitHub sanitizes HTML inside README files, so this payload should be shown as code here. It is not expected to execute on GitHub.

---

## Secure Version

The secure version avoids rendering user input as HTML.

Instead of using `innerHTML`, it creates a paragraph element and inserts the comment using `textContent`.

```js
function render() {
    let container = document.getElementById("comments");

    container.innerHTML = "";

    for (let c of comments) {
        let p = document.createElement("p");
        p.textContent = c;
        container.appendChild(p);
    }
}
```

---

## Why This Is Secure

This line is the key:

```js
p.textContent = c;
```

`textContent` treats the stored comment as plain text.

So this payload:

```html
<img src=x onerror="alert('Stored XSS')">
```

will be displayed as text instead of being executed as code.

---

## Technical Explanation

### `comments.push(value)`

```js
comments.push(value);
```

This stores the user input inside the `comments` array.

The array simulates a database.

This line is not the main vulnerability by itself.  
The problem appears later when the stored data is rendered unsafely.

---

### `render()`

```js
render();
```

This function updates what the user sees on the page.

It takes all stored comments from the `comments` array and displays them inside the `comments` div.

---

### `for (let c of comments)`

```js
for (let c of comments)
```

This loop goes through every stored comment in the array.

Each comment is temporarily stored in the variable `c`.

Example:

```js
comments = ["hello", "second comment"];
```

The loop runs like this:

```txt
First loop:  c = "hello"
Second loop: c = "second comment"
```

---

### Vulnerable Rendering

```js
container.innerHTML += "<p>" + c + "</p>";
```

This builds HTML using stored user input.

That is dangerous because `c` may contain HTML or JavaScript-related content.

---

### Secure Rendering

```js
let p = document.createElement("p");
p.textContent = c;
container.appendChild(p);
```

This safely creates a paragraph, inserts the comment as plain text, and then adds it to the page.

---

## Impact

Stored XSS can be more dangerous than reflected XSS because the payload remains stored and can affect multiple users.

A successful Stored XSS may allow an attacker to:

- execute JavaScript in other users' browsers
- modify the page interface
- inject phishing content
- perform actions in the context of a logged-in user
- access data that is available to JavaScript on the page

---

## Fix Summary

Avoid this with user-controlled data:

```js
innerHTML
```

Prefer this for plain text:

```js
textContent
```

Or use proper sanitization if rendering HTML is truly necessary.

---

## Conclusion

The issue is not only storing user input.

The real issue is storing user input and later rendering it as HTML without sanitization.

Stored data must always be treated as untrusted.

---

## Author

Maximo Rodriguez  
Junior Cybersecurity Learner