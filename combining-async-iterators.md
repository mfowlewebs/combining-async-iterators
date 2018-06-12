# Combining Async Iterators

Hey welcome let's talk about iterating through things in javascript.

# Basics

Let's rush through some basics,

## For & for-in loops

In classic js, we had the simple for loop.

```
for (let i= 0; i< 6; ++i) {
	console.log(i); //=> 0, 1, 2, 3, 4, 5
}
```

It can also iterate through a thing with for-in:

```
let folks = ["arthur, "ford", "trillian"]
for (let i in folks) {
	console.log( folks[i] ); //=> arthur, ford, trillian
}
```

## For-of loop

Recently-ish JS got for-of loops.

```
let folks = ["arthur, "ford", "trillian"]
for (let f of folks) {
	console.log( f ) //=> arthur, ford, trillian
}
```

But uh... how do these work?

## [Symbol.iterator]

Turns out JS objects received a new feature to support this, one that arrays happen to implement. The `Symbol.iterator` symbol was defined as a common protocol for iterating through things. Objects (or arrays) that can be iterated through define this Symbol on themselves. It's value is a function that, when called, returns an iterator, which is an object with a .next().


```
let iterator = ["heart of gold", "ark fleet ship b"][Symbol.iterator]
iterator //=> [Function: values]
iterator.next() //=> { value: 'heart of gold', done: false }
iterator.next() //=> { value: 'ark fleet ship b', done: false }
iterator.next() //=> { value: undefined', done: true }
```

We saw in the previous section how this would be consumed by a for-of loop.

## Spread

In addition to being consumeable by for-of grammer, we can also consume iterables via the spread operator.

```
function print(...names){
	console.log(...names) //=> hotblack marvin zarniwoop
}
print(...["hotblack", "marvin", "zarniwoop"])
```

## DIY

We can also create our own iterables. Anything can be made iterable by adding the Symbol.iterator property.

```
const dice = {}
dice[Symbol.iterator]= function(){
	const iterator = {
		next: function(){
			var value = Math.floor(Math.random() * 6) + 1
			if (n == 1) {
				iterator.next= function(){
					return { value: undefined, done: true }
				}
			}
			return { value, done: false }
		}
	}
	return iterator
}

// for-of
for (let roll of dice){
	console.log(roll) //=> 4, 6, 6, 3, 1
}
// spread
function print(...names){
	console.log(...names) //=> 2, 2, 3, 2, 5, 1
}
print(...dice)
```

# Generators

Nice, nice. We understand iterables pretty well now. They're a protocol- an expected series of interactions that things ought to implement to do a standard behavior.

We've also seen creating our own iterable.

It was a little verbose.

ES6 also introduced **generators,** which is a grammar to make creating iterables easier.

Generators | Iterable
--- | ---
```
function * dice2(){
	let value
	while (value != 1) {
		value = Math.floor(Math.random() * 6) + 1
		yield value // <----- GENERATOR MAGIC HERE
	}
}
``` | ```
const dice = {}
dice[Symbol.iterator]= function(){
	const iterator = {
		next: function(){
			var value = Math.floor(Math.random() * 6) + 1
			if (n == 1) {
				iterator.next= function(){
					return { value: undefined, done: true }
				}
			}
			return { value, done: false }
		}
	}
	return iterator
}
```

Sooo this is generators. They implement the same protocol we explicitly implemented ourselves already, more or less:

```
// (dice2 implemented as previous)
var d = dice2()
d.next() //=> { value: 3, done: false }
d.next() //=> { value: 1, done: false }
d.next() //=> { value: undefined, done: true }
```

There's one little twist here- maybe you were wondering- where is the Symbol.iterator?

```
// (d borrowed from previous)
d[Symbol.iterator]() === d //=> true
```

So we executed `dice2` generator, and got an object which appears to be an iterator (it has `.next()`). But... that iterator is also "iterable" in that if you try to iterate it, it returns itself.

Sneaky. :)

## Why generators?

Any kind of stream of data might be an appropriate thing to model via generators. Trying to tokenize an incoming document might yield a stream of tokens. Reading framed messages out of a large buffer might yield the individual packets.

# Async Iteration

More recently, JS has gained async iteration & async generators.

Async generators have one twist to generators: the returned `value` of `{ value, done }` is a promise.

```
async function * minutes(){
	let n = 0
	let defer = Promise.defer()
	setInterval(function(){
		defer.resolve(++n)
	}, 60000}
	while(true){
		yield defer.promise
		defer = Promise.defer()
	}
}
for await( let n of minutes()){
	console.log(n) //=> 1 .... 2 ....
}
```

## Libraries

https://github.com/bhoriuchi/async-iterator-from
