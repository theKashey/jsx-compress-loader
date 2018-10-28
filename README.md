# jsx-compress-loader
----
JSX optimization webpack loader

## What it does

Replaces `React.createElement` by a local variable, thus __reduce__ "number of characters" per React Element
from __17__ to __1__, as long local variable would be _uglified_.
> "a.b.createElement".length === 17, "React.default.createElement".length === 27. Usually - about 23

That is not a problem for Preact or Inferno, only to "default" React, as long only React has got "long element creation".
See [this tweet from Dan Abramov](https://twitter.com/dan_abramov/status/841266032576724992), to find more about it.

This technique also is almost NOT affecting gzipped size, only the _real_ amount of js code, browser has to parse.

### Bonus 

This also removes object property access (ie React.createElement), thus:
- speeding up `Chrome` by 5%
- speeding up `Safari 11` by 15%
- speeding up `Safari 12` by 35%
- not speeding up Mobile Safari 12(iPhone XS)
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
> in terms of webpack configuration - "after" goes "before", ie top-most loader is the "last" one.

### Only for ESM modules!
babel "modules" should be "false" - you already should have it, for proper tree-shaking, and 
this is what this library is counting on. 

### As separate loader
```js
rules: [
  {
    test: /\.js$/, // for any js file in your project
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
    test: /\.js$/, // paired with babel loader
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
- [pragma jsx + webpack conf](https://medium.com/@jilizart/reduce-the-size-of-final-jsx-code-c39effca906f) - the same, but webpack plugin will inject right imports.
- [babel-plugin-transform-react-constant-elements](https://babeljs.io/docs/en/babel-plugin-transform-react-constant-elements)
would hoist Elements creating, thus removes _object property access_, as long it would be called
only once. Plus - would remove GC pressure due to the same "one time" element creation.
- [react-local](https://github.com/philosaf/react-local) - is doing absolutely the same, but as a babel plugin. Requires more work for proper instalation.
- [runtime-compress-loader](https://github.com/theKashey/runtime-compress-loader) - compress all inlined babel helpers. The same action, but for js sugar.

# Licence
MIT
