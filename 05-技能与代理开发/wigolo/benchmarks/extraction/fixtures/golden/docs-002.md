# Array.prototype.reduce() - MDN Web Docs

The `reduce()` method of `Array` instances executes a user-supplied "reducer" callback function on each element of the array, in order, passing in the return value from the calculation on the preceding element. The final result of running the reducer across all elements of the array is a single value.

## Syntax

```javascript
reduce(callbackFn)
reduce(callbackFn, initialValue)
```

### Parameters

**callbackFn**

A function to execute for each element in the array. Its return value becomes the value of the `accumulator` parameter on the next invocation of `callbackFn`. For the last invocation, the return value becomes the return value of `reduce()`.

The function is called with the following arguments:

- **accumulator** - The value resulting from the previous call to `callbackFn`. On the first call, its value is `initialValue` if specified; otherwise its value is `array[0]`.
- **currentValue** - The value of the current element. On the first call, its value is `array[0]` if `initialValue` is specified; otherwise its value is `array[1]`.
- **currentIndex** - The index position of `currentValue` in the array. On the first call, its value is `0` if `initialValue` is specified; otherwise `1`.
- **array** - The array `reduce()` was called upon.

**initialValue** (optional)

A value to which `accumulator` is initialized the first time the callback is called. If `initialValue` is specified, `callbackFn` starts executing with the first value in the array as `currentValue`. If `initialValue` is not specified, `accumulator` is initialized to the first value in the array, and `callbackFn` starts executing with the second value in the array as `currentValue`. In this case, if the array is empty (so there is no first value to return as `accumulator`), an error is thrown.

### Return Value

The value that results from running the "reducer" callback function to completion over the entire array.

### Exceptions

- **TypeError** - The array contains no elements and `initialValue` is not provided.

## Description

The `reduce()` method is an iterative method. It runs a "reducer" callback function over all elements in the array, in ascending-index order, and accumulates them into a single value. Every time, the return value of `callbackFn` is passed into `callbackFn` again on the next invocation as `accumulator`. The final value of `accumulator` (which is the value returned from `callbackFn` on the final iteration of the array) becomes the return value of `reduce()`.

`reduce()` does not mutate the array on which it is called, but the function provided as `callbackFn` can. Note, however, that the length of the array is saved before the first invocation of `callbackFn`.

## Examples

### Sum of Values

```javascript
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 15
```

### Flatten an Array of Arrays

```javascript
const nested = [[1, 2], [3, 4], [5, 6]];
const flat = nested.reduce((acc, curr) => acc.concat(curr), []);
console.log(flat); // [1, 2, 3, 4, 5, 6]
```

### Counting Instances of Values

```javascript
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
console.log(count); // { apple: 3, banana: 2, orange: 1 }
```

### Grouping Objects by Property

```javascript
const people = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 25 },
  { name: 'Diana', age: 30 },
];

const grouped = people.reduce((acc, person) => {
  const key = person.age;
  if (!acc[key]) acc[key] = [];
  acc[key].push(person);
  return acc;
}, {});
// { 25: [{ name: 'Alice', ... }, { name: 'Charlie', ... }], 30: [...] }
```

### Building a Pipeline

```javascript
const pipeline = [
  (x) => x + 1,
  (x) => x * 2,
  (x) => x - 3,
];

const result = pipeline.reduce((acc, fn) => fn(acc), 5);
console.log(result); // ((5 + 1) * 2) - 3 = 9
```

### Remove Duplicates

```javascript
const values = [1, 2, 3, 2, 1, 4, 3, 5];
const unique = values.reduce((acc, val) => {
  if (!acc.includes(val)) acc.push(val);
  return acc;
}, []);
console.log(unique); // [1, 2, 3, 4, 5]
```

### Running Promises in Sequence

```javascript
const urls = ['/api/first', '/api/second', '/api/third'];

const results = await urls.reduce(async (accPromise, url) => {
  const acc = await accPromise;
  const response = await fetch(url);
  const data = await response.json();
  return [...acc, data];
}, Promise.resolve([]));
```

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Empty array, no `initialValue` | Throws `TypeError` |
| Empty array, with `initialValue` | Returns `initialValue` without calling `callbackFn` |
| Single element, no `initialValue` | Returns the element without calling `callbackFn` |
| Single element, with `initialValue` | Calls `callbackFn` once |

## When Not to Use reduce()

While `reduce()` is powerful, simpler alternatives exist for common patterns:

```javascript
// Instead of reduce for sum:
const sum = numbers.reduce((a, b) => a + b, 0);
// Consider:
let sum2 = 0;
for (const n of numbers) sum2 += n;

// Instead of reduce for filtering + mapping:
const result = arr.reduce((acc, x) => {
  if (x > 5) acc.push(x * 2);
  return acc;
}, []);
// Consider:
const result2 = arr.filter(x => x > 5).map(x => x * 2);
```

## Browser Compatibility

| Browser | Version |
|---------|---------|
| Chrome | 3+ |
| Firefox | 3+ |
| Safari | 4+ |
| Edge | 12+ |
| Opera | 10.5+ |
| Node.js | All versions |

## See Also

- [Array.prototype.reduceRight()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight)
- [Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- [Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
- [Array.prototype.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
