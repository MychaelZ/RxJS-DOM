[![Build Status](https://travis-ci.org/Reactive-Extensions/RxJS-DOM.png)](https://travis-ci.org/Reactive-Extensions/RxJS-DOM)
[![GitHub version](http://img.shields.io/github/tag/reactive-extensions/rxjs-dom.svg)](https://github.com/Reactive-Extensions/RxJS-DOM)
[![NPM version](http://img.shields.io/npm/v/rx-dom.svg)](https://npmjs.org/package/rx)
[![Downloads](http://img.shields.io/npm/dm/rx-dom.svg)](https://npmjs.org/package/rx)
[![NuGet](http://img.shields.io/nuget/v/RxJS-Bridges-HTML.svg)](http://www.nuget.org/packages/RxJS-Bridges-HTML/)
[![Built with Grunt](https://cdn.gruntjs.com/builtwith.png)](http://gruntjs.com/)

RxJS-DOM <sup>7.0</sup> - HTML DOM Bindings for the Reactive Extensions for JavaScript
==========================================================
## OVERVIEW

This project provides Reactive Extensions for JavaScript (RxJS) bindings for HTML DOM objects to abstract over the event binding, Ajax requests, Web Sockets, Web Workers, Server-Sent Events, Geolocation and more.  
## Batteries Included ##

Sure, there are a lot of libraries to get started with the RxJS bindings for the HTML DOM. Confused on where to get started?  Start out with the complete set of functionality with [`rx.dom.js`](doc/readme.md), then you can reduce it to the functionality you require such as only events, or ajax.  If you use RxJS Lite, you can start with the [`rx.lite.dom.js`](modules/lite/readme.md) file and then select the functionality you want from there.

This set of libraries include:

### Main Libraries:
- [`rx.dom.js`](doc/readme.md)
- [`rx.dom.compat.js`](doc/readme.md)
- [`rx.dom.ajax.js`](modules/main-ajax/readme.md)
- [`rx.dom.ajax.compat.js`](modules/main-ajax-compat/readme.md)
- [`rx.dom.concurrency.js`](modules/main-ajax-concurrency/readme.md)
- [`rx.dom.events.js`](modules/main-events/readme.md)
- [`rx.dom.events.compat.js`](modules/main-events-compat/readme.md)
- [`rx.dom.html.js`](modules/main-html/readme.md)

### Lite Libraries:
- [`rx.lite.dom.js`](modules/lite/readme.md)
- [`rx.lite.dom.compat.js`](modules/lite-compat/readme.md)
- [`rx.lite.dom.ajax.js`](modules/lite-ajax/readme.md)
- [`rx.lite.dom.ajax.compat.js`](modules/lite-ajax-compat/readme.md)
- [`rx.lite.dom.concurrency.js`](modules/lite-ajax-concurrency/readme.md)
- [`rx.lite.dom.events.js`](modules/lite-events/readme.md)
- [`rx.lite.dom.events.compat.js`](modules/lite-events-compat/readme.md)
- [`rx.lite.dom.html.js`](modules/lite-html/readme.md)

## GETTING STARTED

There are a number of ways to get started with the HTML DOM Bindings for RxJS.  The files are available on [cdnjs](http://cdnjs.com/) and [jsDelivr](http://www.jsdelivr.com/#!rxjs-dom).

### Download the Source

To download the source of the HTML DOM Bindings for the Reactive Extensions for JavaScript, type in the following:
```bash
git clone https://github.com/Reactive-Extensions/rxjs-dom.git
cd ./rxjs-dom
```
### Installing with [NPM](https://npmjs.org/)
```bash
npm install rx-dom
```
### Installing with [Bower](http://bower.io/)
```bash
bower install rx-dom
```
### Installing with [Jam](http://jamjs.org/)
```bash
jam install rx-dom
```
### Installing with [NuGet](http://nuget.org)
```bash
PM> Install-Package RxJS-Bridges-HTML
```
### Getting Started with the HTML DOM Bindings

Let's walk through a simple yet powerful example of the Reactive Extensions for JavaScript Bindings for HTML, autocomplete.  In this example, we will take user input from a textbox and trim and throttle the input so that we're not overloading the server with requests for suggestions.

We'll start out with a basic skeleton for our application with script references to RxJS Lite based methods, and the RxJS Bindings for HTML DOM, along with a textbox for input and a list for our results.
```html
<script type="text/javascript" src="rx.lite.js"></script>
<script type="text/javascript" src="rx.dom.js"></script>
<script type="text/javascript">

</script>
...
<input id="textInput" type="text"></input>
<ul id="results"></ul>
...
```
The goal here is to take the input from our textbox and debounce it in a way that it doesn't overload the service with requests.  To do that, we'll get the reference to the textInput using the document.getElementById method, then bind to the 'keyup' event using the `Rx.DOM.fromEvent` specialization shortcut for keyups called `Rx.DOM.keyup` which then takes the DOM element event handler and transforms it into an RxJS Observable.
```js
var textInput = document.querySelector('#textInput');
var throttledInput = Rx.DOM.keyup(textInput);
```
Since we're only interested in the text, we'll use the `map` method to take the event object and return the target's value, or we can call `pluck` to the same effect.
```js
	.pluck('target','value')
```
We're also not interested in query terms less than two letters, so we'll trim that user input by using the `where` or `filter` method returning whether the string length is appropriate.
```js
	.filter( function (text) {
		return text.length > 2;
	})
```
We also want to slow down the user input a little bit so that the external service won't be flooded with requests.  To do that, we'll use the `debounce` method with a timeout of 500 milliseconds, which will ignore your fast typing and only return a value after you have paused for that time span.  
```js
	.debounce(500)
```
Lastly, we only want distinct values in our input stream, so we can ignore requests that are not unique, for example if I copy and paste the same value twice, the request will be ignored using the `distinctUntilChanged` method.
```js
	.distinctUntilChanged();
```
Putting it all together, our throttledInput looks like the following:

```js
var textInput = document.querySelector('#textInput');
var throttledInput = Rx.DOM.keyup(textInput)
	.pluck('target','value')
	.filter( function (text) {
		return text.length > 2;
	})
	.debounce(500)
	.distinctUntilChanged();
```

Now that we have the throttled input from the textbox, we need to query our service, in this case, the Wikipedia API, for suggestions based upon our input.  To do this, we'll create a function called searchWikipedia which calls the `Rx.DOM.jsonpRequest` method which wraps making a JSONP call.

```js
function searchWikipedia(term) {
  var url = 'http://en.wikipedia.org/w/api.php?action=opensearch&format=json&search='
    + encodeURIComponent(term) + '&callback=JSONPCallback';
  return Rx.DOM.jsonpRequest(url);
}
```

Now that the Wikipedia Search has been wrapped, we can tie together throttled input and our service call.  In this case, we will call select on the throttledInput to then take the text from our textInput and then use it to query Wikipedia, filtering out empty records.  Finally, to deal with concurrency issues, we'll need to ensure we're getting only the latest value.  Issues can arise with asynchronous programming where an earlier value, if not cancelled properly, can be returned before the latest value is returned, thus causing bugs.  To ensure that this doesn't happen, we have the `flatMapLatest` method which returns only the latest value.

```js
var suggestions = throttledInput.flatMapLatest(searchWikipedia);
```

Finally, we'll subscribe to our observable by calling subscribe which will receive the results and put them into an unordered list.  We'll also handle errors, for example if the server is unavailable by passing in a second function which handles the errors.

```js
var resultList = document.getElementById('results');

function clearSelector (element) {
  while (element.firstChild) {
    element.removeChild(element.firstChild);
  }
}

function createLineItem(text) {
	var li = document.createElement('li');
	li.innerHTML = text;
	return li;
}

suggestions.subscribe(
	function (data) {
	  var results = data.response[1];

	  clearSelector(resultList);

	  for (var i = 0; i < results.length; i++) {
	    resultList.appendChild(createLineItem(results[i]));
	  }
	},
	function (e) {
		clearSelector(resultList);
  	resultList.appendChild(createLineItem('Error: ' + e));
	}
);

```

We've only scratched the surface of this library in this simple example.

## Dive In! ##

Please check out:

 - [Our Code of Conduct](https://github.com/Reactive-Extensions/RxJS/tree/master/code-of-conduct.md)
 - [The full documentation](https://github.com/Reactive-Extensions/RxJS-DOM/tree/master/doc)
 - [Our many great examples](https://github.com/Reactive-Extensions/RxJS-DOM/tree/master/examples)
 - [Our design guidelines](https://github.com/Reactive-Extensions/RxJS/tree/master/doc/designguidelines)
 - [Our contribution guidelines](https://github.com/Reactive-Extensions/RxJS/tree/master/contributing.md)
 - [Our complete Unit Tests](https://github.com/Reactive-Extensions/RxJS/tree/master/tests)

## LICENSE

Copyright Microsoft

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
