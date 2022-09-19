# Using Jest to perform unit testing with mocks in JavaScript applications: https://zetcode.com/javascript/jest/

A unit: the smallest testable part of any software. An example is a small function used by other functions.

Mocking: a technique where code with unpredictable output/behaviour is replaced with emulated code that produces predictable output/behaviour to achieve isolation required for unit testing.

expect() function: In Jest, this object provides a number of matcher methods like toBe(), toBeFalsy(), or toEqual() to validate a variety of things.

## Setting up Jest:

Initiate a NodeJS application with a node_modules folder, if not already done:

`$ npm init -y`

Install the jest package:

`$ npm i --dev jest`

Next, we will install the jsonserver and axios packages by running the following commands. The jsonserver package allows us to mock/simulate a REST API that will return a json object on demand (https://www.npmjs.com/package/jsonserver). The axios package allows us to make http requests from node.js and XMLHttpRequests from the browser (https://www.npmjs.com/package/axios).

`$ npm i jsonserver`

`$ npm i axios`

## package.json's "scripts" object:

Nest, in package.json, you will find the scripts object. You can give the "test" key the value of "jest" like shown below, then running `$ npm test` will run all test files with ".test" in its name using jest.

```
"scripts": {
  "test": "jest"
},
```

You can also run `npm test <test file name>` to run test files individually.

## Basic testing with Jest

The following is a basic example of unit testing with Jest.

arith.js:

```
const add = (a, b) => a + b;
const mul = (a, b) => a * b;
const sub = (a, b) => a - b;
const div = (a, b) => a / b;

module.exports = { add, mul, sub, div };
```

arith.test.js:

```
const { add, mul, sub, div } = require('./arith');

test('2 + 3 = 5', () => {
  expect(add(2, 3)).toBe(5);
});

test('3 * 4 = 12', () => {
  expect(mul(3, 4)).toBe(12);
});

test('5 - 6 = -1', () => {
  expect(sub(5, 6)).toBe(-1);
});

test('8 / 4 = 2', () => {
  expect(div(8, 4)).toBe(2);
});
```

`$ npm test arith.test.js`

This will run the test and output the following to the console:
``` 
PASS  tests/arith.test.js
  ✓ 2 + 3 = 5 (2 ms)
  ✓ 3 * 4 = 12 (1 ms)
  ✓ 5 - 6 = -1 (1 ms)
  ✓ 8 / 4 = 2 (1 ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        0.482 s, estimated 1 s
Ran all test suites matching /arith.test.js/i.
Waiting for the debugger to disconnect...
```

## Skipping tests with "x" or .skip()

Some tests can take long to run, so you may find it useful to skip them. To skip a test, put the letter 'x' in front of the test or use .skip():

```
xtest('2 + 3 = 5', () => {
  expect(add(2, 3)).toBe(5);
});

test.skip('3 * 4 = 12', () => {
  expect(mul(3, 4)).toBe(12);
});
```
`$ npm test arith.test.js`

This will run the test and output the following to the console:
``` 
 PASS  tests/arith.test.js
  ✓ 5 - 6 = -1 (2 ms)
  ✓ 8 / 4 = 2 (1 ms)
  ○ skipped 2 + 3 = 5
  ○ skipped 3 * 4 = 12

Test Suites: 1 passed, 1 total
Tests:       2 skipped, 2 passed, 4 total
Snapshots:   0 total
Time:        0.409 s, estimated 1 s
Ran all test suites matching /arith.test.js/i.
Waiting for the debugger to disconnect...
```

## Parameterized tests using .each()

Parameterized tests runs the same test multiple times with different values using the `each()` global function. The `percent sign` symbols are format specifiers that expect integers to create custom output that is shown on the console.

arith-param.test.js:

```
const { add, mul, sub, div } = require('./arith')

test.each([[1, 1, 2], [-1, 2, 1], [2, 1, 3]])(
  '%i + %i equals %i', (a, b, expected) => {
    expect(add(a, b)).toBe(expected);
  },
);

test.each([[1, 1, 0], [-1, 2, -3], [2, 2, 0]])(
  '%i - %i equals %i', (a, b, expected) => {
    expect(sub(a, b)).toBe(expected);
  },
);

test.each([[1, 1, 1], [-1, 2, -2], [2, 2, 4]])(
  '%i * %i equals %i', (a, b, expected) => {
    expect(mul(a, b)).toBe(expected);
  },
);

test.each([[1, 1, 1], [-1, 2, -0.5], [2, 2, 1]])(
  '%i / %i equals %i', (a, b, expected) => {
    expect(div(a, b)).toBe(expected);
  },
);
```

The each method takes an array of arrays with the arguments that are passed into the test function for each row. Run the following:

`$ npx jest arith-param.test.js`

The result:

```
 PASS  tests/arith-param.test.js
  ✓ 1 + 1 equals 2 (6 ms)
  ✓ -1 + 2 equals 1 (1 ms)
  ✓ 2 + 1 equals 3 (1 ms)
  ✓ 1 - 1 equals 0 (1 ms)
  ✓ -1 - 2 equals -3 (1 ms)
  ✓ 2 - 2 equals 0 (1 ms)
  ✓ 1 * 1 equals 1
  ✓ -1 * 2 equals -2
  ✓ 2 * 2 equals 4
  ✓ 1 / 1 equals 1 (1 ms)
  ✓ -1 / 2 equals -0
  ✓ 2 / 2 equals 1 (1 ms)
```

## Setting up with .beforeAll()

The beforeAll function runs before any of the tests in a file to set things up like initializing test variables or making ajax requests. If the function returns a promise or is a generator, Jest waits for that it to resolve before the tests start.

As an example, we have the following module in our directory:

math-utils.js:

```
const sum = (vals) => {
    let sum = 0;
    vals.forEach((val) => {
        sum += val;
    });
    return sum;
}

const positive = (vals) => {
    return vals.filter((x) => { return x > 0; });
}

const negative = (vals) => {
    return vals.filter((x) => { return x < 0; });
}

module.exports = { sum, positive, negative };
```

This test will set up variables before each test using beforeAll():

math-utils.test.js:

```
const { sum, positive, negative } = require('./math-utils');

let vals;
let sum_of_vals;
let pos_vals;
let neg_vals;

beforeAll(() => {
    pos_vals = [2, 1, 3];
    neg_vals = [-2, -1, -1];
    vals = pos_vals.concat(neg_vals);
    sum_of_vals = vals.reduce((x, y) => x + y, 0);
})

test('the sum of vals should be 2', () => {
    expect(sum(vals)).toBe(sum_of_vals);
});

test('should get positive values', () => {
    expect(positive(vals)).toEqual(pos_vals);
});

test('should get negative values', () => {
    expect(negative(vals)).toEqual(neg_vals);
});
```

`$ npx jest math-utils.test.js`

## Grouping tests with describe()

Related tests can be grouped into a block using describe() like below.

groups.test.js:

```
const { sum, positive, negative } = require('../modules/math-utils');
const { isPalindrome, isAnagram } = require('../modules/string-utils');

describe('group testing only math utilities', () => {
    let vals;
    let sum_of_vals;
    let pos_vals;
    let neg_vals;

    beforeAll(() => {
        pos_vals = [2, 1, 3];
        neg_vals = [-2, -1, -1];
        vals = pos_vals.concat(neg_vals);
        sum_of_vals = vals.reduce((x, y) => x + y, 0);
    })

    test('the sum of vals should be 2', () => {
        expect(sum(vals)).toBe(sum_of_vals);
    });

    test('should get positive values', () => {
        expect(positive(vals)).toEqual(pos_vals);
    });

    test('should get negative values', () => {
        expect(negative(vals)).toEqual(neg_vals);
    });
});


describe('group testing only string utilities', () => {

    test.each(["racecar", "radar", "level", "refer", "deified", "civic"])(
        'testing %s for palindrome', (word) => {
            expect(isPalindrome(word)).toBeTruthy();
        },
    );

    test.each([["arc", "car"], ["cat", "act"], ["cider", "cried"]])(
        'testing if %s and %s are anagrams ', (word1, word2) => {
            expect(isAnagram(word1, word2)).toBeTruthy();
        },
    );
});
```

`$ npm test groups.test.js`

## Lastly, testing with axios and jsonserver 

Open users.json and observe the mock json data inside. We can feed this through a server enabled by the jsonserver package to simulate a REST API. Run `$ npx json-server --watch users.json` then log onto `http://localhost:3000/users` to see it in action. The browser should display the json data onto the screen. Now, with this server running on watch mode, open a new terminal instance and do the following.

The `modules/users.js`  defines a class called `Users` that has a method called `all()`, which will use axios to return any data fetched from the url: `http://localhost:3000/users`. It is a very simple piece of code. We will test this, but we have to call the class through another module: `modules/users-app.js`. This will call `Users.all()` to log the data onto the console. Run it with the following command on the new terminal:

`$ node modules/users-app.js`

This will output the below console printout. Notice how the "finished" string was logged first because of the asynchronous showData() operation.

```
finished
[
  {
    id: 1,
    first_name: 'Robert',
    last_name: 'Schwartz',
    email: 'rob23@gmail.com'
  },
  {
    id: 2,
    first_name: 'Lucy',
    last_name: 'Ballmer',
    email: 'lucyb56@gmail.com'
  },
  {
    id: 3,
    first_name: 'Anna',
    last_name: 'Smith',
    email: 'annasmith23@gmail.com'
  },
  {
    id: 4,
    first_name: 'Robert',
    last_name: 'Brown',
    email: 'bobbrown432@yahoo.com'
  },
  {
    id: 5,
    first_name: 'Roger',
    last_name: 'Bacon',
    email: 'rogerbacon12@yahoo.com'
  }
]
Waiting for the debugger to disconnect...
```
### The Mock:

Now that we see how jsonserver and axios works, we'll mock it. Open `users.test.js`. This  mock-tests the `users.js` module. The line **jest.mock("axios");** mocks the axios. The following lines specifies the response that the mocked module should return:

```
const users = [{
    "id": 1,
    "first_name": "Robert",
    "last_name": "Schwartz",
    "email": "rob23@gmail.com"
}, {
    "id": 2,
    "first_name": "Lucy",
    "last_name": "Ballmer",
    "email": "lucyb56@gmail.com"
}];

const resp = { data : users };
```

The mocked implementation returns a promise with the response:

`axios.get.mockImplementation(() => Promise.resolve(resp));`

We test the mocked Users.all function using expect():

`Users.all().then(res => expect(res.data).toEqual(users));`

Run the test. It should pass:

`$ npm test users.test.js`
