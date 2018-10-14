# jsx-compress-loader
----
JSX optimization webpack loader

## What it does

Replaces `React.createElement` by a local variable, thus __reduce__ "number of characters" per React Element
from __17__ to __1__, as long local variable would be _uglified_.
> "a.b.createElement".length === 17, "React.default.createElement".length === 27. Usually - about 23

That is not a problem for Preact or Inferno, only to "default" React, as long only React has got "long element creation".
See [this Dan's twitt](https://twitter.com/dan_abramov/status/841266032576724992), to find more about it.

This technique also is almost NOT affecting gzipped size, only the _react_ amount of js code, browser has to parse.

### Bonus 

This also removes object property access (ie React.createElement), thus:
- speeding up `Chrome` by 5%
- speeding up `Safary 11` by 15%
- speeding up `Safary 12` by 35%
- not speeding up Mobile Safary 12(iPhone XS)
- here is [the test](https://jsperf.com/single-dot-property-access/1)

# Would it help?
Just open your bundle, and count `createElement` inside. Or open any page, and count closing tags `</`.
Next multiply by 22. Result is - amount of bytes you would remove from your bundle. For free.  

## Usage
Just add this webpack loader AFTER all other.
- after ts
- after js
- after svg -> react -> babel -> js
- and dont forget to apply it to node_modules as well.

### As separate loader
```js
rules: [
  {
    test: /\.js$/,
    use: 'jsx-compress-loader',
  },
  {
    test: /\.js$/,
    exclude: /node_modules/,
    use: 'babel-loader',    
  },
];
``` 

### As chained loader
```js
rules: [
  {
    test: /\.js$/,
    exclude: /node_modules/,
    use: [    
      'jsx-compress-loader'
      'babel-loader',
    ],
  },
];
```

## Other ways

- [babel-plugin-transform-react-inline-elements](https://babeljs.io/docs/en/next/babel-plugin-transform-react-inline-elements.html)
would inline "createElement", achieving almost the same result
- [babel-plugin-transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)
has a `pragma jsx`, letting you to change JSX compilation rules. Preact, for example, configure it to produce just `h`.
- [babel-plugin-transform-react-constant-elements](https://babeljs.io/docs/en/babel-plugin-transform-react-constant-elements)
would hoist Elements creating, thus removes _object property access_, as long it would be called
only once. Plus - would remove GC pressure due to the same "one time" element creation.

# Licence
MIT