---
layout: post
comments: true
title: A TDD approach to develop a Pomodoro API using NodeJS part 1 Classes
---

This article is an introduction to the TDD approach. It'll explain how to implement a pomodoro class in NodeJS using TDD.

### Introduction

This week I started a new personal project. I want to develop a Pomodoro system by myself, using NodeJS, MongoDB. And I think that would be a good idea to post about things I've been learning with this project. You can find it here on my [github repository](https://github.com/gbrmachado/tomapp).

In this week I learned mainly about how to use a TDD approach to implement classes. TDD stands for Test Driven Development. It's a software development process focused on the repetition of a very short cycle:

1. Firstly, you create a test based in what you whant to implement.
2. Then you implement it, based on the test you have already done.
3. When you implementation has passed in all tests, you can improve it, by refactoring.

So, in this article, I'm gonna implement a really simple class to represent a pomodoro using a TDD approach. 
Be aware we'll gonna implement just the class and the tests on this article, we gonna implement a whole API for it in the future.

By the way, if you are not familiar with pomodoro, it's a technique which uses a timer to break down work into intervals, usually 25 minutes in length, separated by short breaks. It's really useful to prevent procrastination.

So, the idea is to implement a class to represent a **timer**.

As we are going to use a TDD approach, firstly we implement the **tests**.

For the tests we'll use the [Chai](http://chaijs.com/), which is a good framework for tests in NodeJS. Besides, we'll use [Moment](http://moment.js) to deal with time operations.

So, let's start the project

This is the directory structure(just to make it easier)

```
├── pomodoro/
│   ├── tests
│   │   ├── class_pomodoro.js
│   ├── models
│   │   ├── pomodoro.js
```

### Test Implementation

```bash
npm init
npm install --save-dev moment
npm install --save-dev mocha chai
```

Alright, let's code the tests.

```bash
mkdir test
touch test/class_pomodoro.js
```

Now, use your favorite text editor to code the following code on test/class_pomodoro

```javascript
var chai   = require('chai'),
	expect = chai.expect;

var moment   = require('moment');
var Pomodoro = require('../models/pomodoro');

describe('Pomodoro Class', function() {
    var pomodoro = new Pomodoro(['test'], 'test');  //[tags], description
    it('Test Initial Values', function() {
        expect(pomodoro.start_time).to.be.a('Date');
        expect(pomodoro.start_time.getDate()).to.be.a('number');
        expect(pomodoro.tags).to.be.instanceof(Array);
        expect(pomodoro.tags[0]).to.equal('test');
        expect(pomodoro.description).to.equal('test');
        expect(pomodoro.interruptions).to.equal(0);
    });
    it('Test increase interruptions', function() {
        pomodoro.inc_interruption();
        expect(pomodoro.interruptions).to.equal(1);
    });
    it('Test if date is within pomodoro interval', function() {
        var new_date = moment().add(10, 'minutes').toDate();
        expect(pomodoro.running(new_date)).to.equal(true);
        new_date = moment().add(20, 'minutes').toDate();
        expect(pomodoro.running(new_date)).to.equal(true);
        new_date = moment().add(30, 'minutes').toDate();
        expect(pomodoro.running(new_date)).to.not.equal(true);
        new_date = moment().subtract(2, 'minutes').toDate();
        expect(pomodoro.running(new_date)).to.not.equal(true);
        new_date = moment().subtract(1, 'minutes').toDate();
        expect(pomodoro.running(new_date)).to.not.equal(true);
    });
    it('test end_time', function() {
        pomodoro.stop();
        var new_date = moment().add(2, 'minutes').toDate();
        expect(pomodoro.running(new_date)).to.equal(false);
    });
});
```

The main advantage of this approach is that it forces you to think about the general behaviour before you implement it. It becomes way easier to think and implement it when you already know what you should code. Besides, it gives you confidence as your software becomes bigger. If a new functionality can break an old one, you can know beforehand by running the old tests.

About the code for the test, it's not really hard, isn't? It's really close to a human language. 
You imagine a class, how it will work on your code, and you code how you expect the behaviour if this class in certain situations.

Now you can run the code:

```bash
node_modules/mocha/bin/mocha test/class_pomodoro.js
```

Annnnd... BAMMM! You failed it!!

### Class Implementation

Now we have the tests, let's implement the real code. Implement it on models/pomodoro.js

```bash
mkdir models/
touch models/pomodoro.js
```

```javascript
var moment = require('moment');

module.exports = class Pomodoro {
	constructor(tags, description) {
		this._start_time    = new Date();
		this._end_time      = moment(this.start_time).add(25, 'minutes').toDate();
		this._tags          = tags;
		this._description   = description;
		this._interruptions = 0;
	}

	get end_time() {
		return this._end_time;
	}

	get start_time() {
		return this._start_time;
	}

	get tags() {
		return this._tags;
	}

	get interruptions() {
		return this._interruptions;
	}

	get description() {
		return this._description;
	}

	inc_interruption() {
		this._interruptions += 1;
	}

	set interruptions(value) {
		this._interruptions = value;
	}

	running(now) {  //If pomodoro is running at the moment
		var now_moment = moment(now);
		var start_time = moment(this.start_time);
		var end_time   = moment(this.end_time);
		return now_moment.isBetween(start_time, end_time)
	}

	stop() {
		this._end_time = new Date();
	}
}
```

Now we can run the tests again

```bash
node_modules/mocha/bin/mocha test/class_pomodoro.js
```

And... 

![Step 1](/images/test_pomodoro.png "We made it!")

That's it. We created a class to represent a pomodoro using TDD. We can refactor the code now. Change some stuffs, improve another. As long as the new code pass on the tests, we are cool(or almost :P)

That's it. It doesn't look really useful, but on next post we gonna create an API to control this class using Express.


