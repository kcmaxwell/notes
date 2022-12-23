# Fundamentals of Web apps

## Traditional web applications
In a traditional web application, when entering the page, the browser fetches the HTML document detailing the structure and textual content of the page from the server.
The document can be a *static* file, saved into the server's directory.
The server can also form the HTML documents *dynamically* according to the application, for example, using data from a database.

In traditional web applications, the browser is "dumb", and only fetches HTML data from the server, with all application logic on the server.

## Document Object Model
The Document Object Model, or DOM, is an Application Programming Interface (API) which enables programmatic modification of the *element trees* corresponding to web-pages.

## Manipulating the document-object from console
The topmost node of the DOM tree of an HTML document is called the `document` object.
You can use JavaScript in the console of the developer tools to manipulate the document object and get and modify its children.

## CSS
Cascading Style Sheets, or CSS, is a style sheet language used to determine the appearance of web pages.

A class selector can select certain parts of the page and define styling rules to style them.
A class selector definition always starts with a period, and contains the name of the class.
```
.container {
  padding: 10px;
  border: 1px solid;
}
```

Classes are attributes, which can be added to HTML elements.
You can examine CSS attributes on the elements tab of the console.

HTML elements can also have other attributes apart from classes, for example, id attributes, which JavaScript code can use to find an element.
