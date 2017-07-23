---
layout: post
title:  "*this* and JavaScript"
date:   2017-07-23 12:48:12 +0000
---


If you are new to JavaScript then one of the most important concepts that you will need to learn and understand is *this*.

Figuring out what *this* is doesnt have to be hard and I'll show you why with 3 tips. You can follow along in the browser by opening the console.

Tip 1: If you use bind, call, or apply on a function *this* is the object that is passed in as an argument. 

```
function pastaParties(ingredient, dressing) {
	this.leaf = ingredient;
	this.dressing = dressing;
}

function myDressing() {
	console.log(this.dressing)
}

var pasta = new pastaParties('lettuce', 'ranch')

myDressing.call(pasta)

// ->  ranch
```

As you can see *this* is the pasta variable that is passed in as an argument to the call function. 

Tip 2: Whenever a function is invoked *this* is the object that the function belongs to.

```
function pastaParties(ingredient, dressing) {
	this.leaf = ingredient;
	this.dressing = dressing;
	this.whatAmI = function() {
		console.log(this)
	}
}

function myDressing() {
	console.log(this.dressing)
}

var pasta = new pastaParties('lettuce', 'ranch')

function func(argument) {
	console.log(this)
}

pasta.whatAmI()
// -> pastaParties {leaf: "lettuce", dressing: "ranch", whatAmI: function}

func()

// -> Window {stop: function, open: function, alert: function, confirm: function, prompt: function…}

```

Back to the previous example, I added a function 'whatAmI' to saladParties. Now when pasta.whatAmI is called the object pasta is returned since the function belongs to whatAmI. 

However, when the function 'func' is invoked it is not apart of an object so it belongs to the global object in this case, the window.

Tip 3: When using the ES6 arrow syntax the value of *this* is thats of the surrounding scope.

```
const arrowSalad = {
	ingredient: 'cheese',
	dressing: 'thousand island',
	whatAmI: function() {
		this.dressing = 'I dont like dressing';
		return ()=>{
		  console.log(this)
	  }
	}
}
console.log(arrowSalad.whatAmI()())

// -> Object {
  "dressing": "I dont like dressing",
  "ingredient": "cheese",
  "whatAmI": [Function whatAmI]
}
```

Your browser console may not support es6 syntax so try this instead. 

```
var arrowSalad = {
	ingredient: 'cheese',
	dressing: 'thousand island',
	whatAmI: function whatAmI() {
		var _this = this;

		this.dressing = 'I dont like dressing';
		return function () {
			console.log(_this);
		};
	}
};
console.log(arrowSalad.whatAmI()());
```

As you can see, the anonymous arrow function is two levels deep however it still refers to the arrowSalad object. Another interesting thing about arrow functions is that they ignore bind, apply, call. Forexample 

```
const arrowSalad = {
	ingredient: 'cheese',
	dressing: 'thousand island',
	whatAmI: function() {
		this.dressing = 'I dont like dressing';
		return ()=>{
		  console.log(this)
	  }
	}
}

function pastaParties(ingredient, dressing) {
	this.leaf = ingredient;
	this.dressing = dressing;
}

var pasta = new pastaParties('lettuce', 'ranch')

arrowSalad.whatAmI().call(pasta)

// -> Object {
  "dressing": "I dont like dressing",
  "ingredient": "cheese",
  "whatAmI": [Function whatAmI]
}
```

Notice that the arrowSalad object is returned instead of the pasta object which is a new instance of pasataParties. 

Thats all for now, hopefully these three tips will help to make *this* a lot less scary.
