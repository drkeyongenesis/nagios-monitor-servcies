# JavaScript DOM Manipulation Cheat Sheet

## Selecting DOM Elements
- **getElementById**: `document.getElementById('id')`
- **getElementsByClassName**: `document.getElementsByClassName('class')`
- **getElementsByTagName**: `document.getElementsByTagName('tag')`
- **querySelector**: `document.querySelector('css query')`
- **querySelectorAll**: `document.querySelectorAll('css query')`

## Modifying DOM Elements
- **textContent**: `element.textContent = 'new text'`
- **innerHTML**: `element.innerHTML = '<h1>New Heading</h1>'`
- **setAttribute**: `element.setAttribute('attribute', 'value')`
- **style**: `element.style.color = 'red'`

## Adding and Deleting Elements
- **createElement**: `let element = document.createElement('tag')`
- **appendChild**: `parent.appendChild(element)`
- **removeChild**: `parent.removeChild(element)`

## Event Handling
- **addEventListener**: `element.addEventListener('event', function)`
- **onload**: `window.onload = function() { ... }`
- **onclick**: `element.onclick = function() { ... }`

## Working with Dates
- **Creating a new date**: `let date = new Date()`
- **Getting the day**: `date.getDay()`
- **Getting the month**: `date.getMonth()`
- **Getting the year**: `date.getFullYear()`
- **Setting the date**: `date.setDate(day)`
- **Setting the month**: `date.setMonth(month)`
- **Setting the year**: `date.setFullYear(year)`

## Showing and Hiding Elements
- **Show an element**: `element.style.display = 'block'`
- **Hide an element**: `element.style.display = 'none'`

## Examples
```javascript
// Selecting an element
let heading = document.getElementById('myHeading');

// Modifying the element
heading.textContent = 'Hello, World!';
heading.style.color = 'red';

// Adding a new element
let newParagraph = document.createElement('p');
newParagraph.textContent = 'This is a new paragraph.';
document.body.appendChild(newParagraph);

// Adding an event listener
heading.addEventListener('click', function() {
    heading.style.color = heading.style.color == 'red' ? 'blue' : 'red';
});

// Adding an onload event
window.onload = function() {
    alert('Page is loaded');
}

// Adding an onclick event
let button = document.getElementById('myButton');
button.onclick = function() {
    alert('Button is clicked');
}

// Working with dates
let date = new Date();
console.log('Today is: ' + date.getDay());

// Showing and hiding elements
let div = document.getElementById('myDiv');
div.style.display = 'none'; // Hide
div.style.display = 'block'; // Show
```

- **querySelectorAll**:
```Javascript
// Assume below are found in HTML.
// <ul id="myList">
//     <li>Item 1</li>
//     <li>Item 2</li>
//     <li>Item 3</li>
// </ul>
let listItems = document.querySelectorAll('#myList li');

// Loop through the NodeList and modify elements
listItems.forEach(function(item, index) {
    item.textContent = 'Updated Item ' + (index + 1);
});
```

- **getElementsByClassName**:
```Javascript
// Selecting elements by class name
// Assume below are found in HTML.
// <ul>
//     <li class="list-item">Item 1</li>
//     <li class="list-item">Item 2</li>
//     <li class="list-item">Item 3</li>
// </ul>
let listItems = document.getElementsByClassName('list-item');

// Loop through the HTMLCollection and modify elements
for (let i = 0; i < listItems.length; i++) {
    listItems[i].textContent = 'Updated Item ' + (i + 1);
}
