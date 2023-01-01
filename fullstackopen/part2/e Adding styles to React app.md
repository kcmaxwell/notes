# Adding styles to React app

There are several ways we can add CSS styles to a React app. One way is using a single CSS file without a CSS preprocessor. We just need to import it, like so:
```
import './index.css'
```

We can add a CSS rule like this:
```
h1 {
  color: green;
}
```

CSS rules comprise of *selectors* and *declarations*. The selector defines which elements the rule should be applied to. The selector above is *h1*, which will match all of the *h1* header tags in our application. The declaration sets the `color` property to the value *green*.

One CSS rule can contain an arbitrary number of properties.

Using element types for defining CSS rules is slightly problematic, so we can also use class selectors. In React, we have to use the className attribute instead of the class attribute.

## Inline styles

React also makes it possible to write styles directly in the code as so-called inline styles. Any React component or element can be provided with a set of CSS properties as a JavaScript object through the style attribute.

CSS rules are defined slightly differently in JavaScript than in normal CSS files. For example, in a CSS file, it would look like this:
```
{
  color: green;
  font-style: italic;
  font-size: 16px;
}
```

But as a React inline-style object, it would look like this:
```
{
  color: 'green',
  fontStyle: 'italic',
  fontSize: 16
}
```

Every CSS property is defined as a separate property of the JavaScript object. Numeric values for pixels can be defined as integers. One major difference is that hyphenated/kebab case CSS properties are written in camelCase for JavaScript.

There are certain limitations to inline styles. For example, pseudo-classes (for example, `:hover`) can't be used straightforwardly.
