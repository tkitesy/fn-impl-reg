# fn-impl-reg
use function implement regular expression

```js
// a
const alpha = (ch) => (str, index, succ, fail) =>
  ch === str[index] ? succ(str, index + 1) : fail(str, index);
// ""
const epsilon = () => (str, index, succ, fail) => succ(str, index);

// ab
const concat = (head, tail) => (str, index, succ, fail) =>
  head(str, index, (s, i) => tail(s, i, succ, fail), fail);

// a|b
const alter = (head, tail) => (str, index, succ, fail) =>
  head(str, index, succ, (s, i) => tail(str, index, succ, fail));

// a*
const kleene = (m) => (str, index, succ, fail) =>
  m(str, index, (s, i) => kleene(m)(s, i, succ, fail), succ);

// $
const end = () => (str, index, succ, fail) =>
  str.length === index ? succ(str, index) : fail(str, index);


// for test
const log = (prefix) => (str, index) => {
  console.log(
    `${str}
${new Array(index).fill(" ").join("")}^   --${prefix}`
  );
};

const test = (name, m, ...strs) => {
  console.log(`==test: ${name} =================================`);
  strs.forEach((str) => m(str, 0, log("success"), log("fail")));
  console.log(`=================================================`);
};


const seq = (f) => (m1, ...m2) => m2.reduce((prev, curr) => f(prev, curr), m1);

const m1 = concat(alpha("a"), alpha("b"));
const m2 = seq(concat)(alpha("a"), alpha("b"), alpha("c"));
const m3 = kleene(m2);
const m4 = seq(alter)(alpha("d"), alpha("e"));
const m6 = seq(alter)(alpha("f"), epsilon());

const m = seq(concat)(m1, m3, m4, m6, end());

test(
  "/^ab(abc)*(d|e)f?$/",
  m,
  "ababcabcabc",
  "ababcabcabce",
  "ababcabcabcf",
  "ababcabcabcd",
  "ababcabcabcdf",
);

```
output 


```
==test: /^ab(abc)*(d|e)f?$/ =================================
ababcabcabc
           ^   --fail
ababcabcabce
            ^   --success
ababcabcabcf
           ^   --fail
ababcabcabcd
            ^   --success
ababcabcabcdf
             ^   --success
=================================================
```