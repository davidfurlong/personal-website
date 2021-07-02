---
template: page
title: Javascript in Sheets fast and easy with Formulas
slug: javascript-in-google-sheets
draft: false
---
Javascript in Sheets fast and easy with Formulas is a google sheets add-on that provides you with helpful formulas to use javascript in your sheets.

After installing this add-on you should have access to the following formulas in google sheets.

## Formulas

### \=JS(jsArrowFunctionInQuotes: string, ...args)

Write a javascript function. You can also pass in cell ranges which will call your arrow function with an array of the cell values. 

@param {jsArrowFunctionInQuotes} a javascript arrow function surrounded by quotation marks

@param {...additionalArguments} (optional) any additional arguments to JS will be passed to the javascript expression provided as arguments

@return the return value of the javascript code run with \`eval\` and invoked as a function with the provided arguments, essentially eval((arg1)(arg2, arg3, ...))

For example 

```
=JS("(a) => a.join('')", C1:C3)
```

### \=JSE(jsExpressionInQuotes: string, ...args)

Write a javascript expression. You can also pass in cell ranges which will replace your subsitution tokens with an array of the cell values. 

@param {jsExpressionInQuotes} a javascript expression surrounded by quotation marks, in which $1, $2, $3... will be replaced by additional arg1, arg2, arg3 respectively

@param {...additionalArguments} (optional) any additional arguments will be substituted into the javascript expression for their substitution tokens $1, ...

\* @return the return value of the javascript code run with \`eval\` after substituting in the provided arguments

For example 

```
=JSE("$1.length", C1:C3)
```

### \=FETCH(url: string, params?: string)

Makes a request to fetch a URL

@param {url} url to fetch

@param {params} a JS object in quotes. For example '{ method: "POST", payload: { cat: true } }' See all options at https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app#advanced-parameters

 @return the response body of the completed request as text

For example

```
=FETCH('www.google.com', '{ method: "POST", payload: { cat: true } }')
```

### \=FETCHJSON(url: string, params?: string)

Makes a request to fetch a URL with a contentType: application/json header. Is shorthand for =FETCH(<URL>, "{ contentType: application/json }")

@param {url} to fetch

@param {params}  a JS object in quotes. For example '{ method: "POST", payload: { cat: true } }' See all options at https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app#advanced-parameters

@return the response body of the completed request as text

For example

```
=FETCHJSON("https://aws.random.cat/meow")
```

## Additional Formulas for debugging

### \=JS_DEBUG(jsArrowFunctionInQuotes: string, ...args)

Debug a javascript function by seeing the generated JS code that will be executed. Takes the same arguments as =JS() except will return the string of the JS that will be \`eval\`ed instead of invoking \`eval\`

@param {value} any cell or value

@return a JSON.stringify of an object with the provided argument's value and typeof for debugging what values JS gets for a given cell

### \=JSE_DEBUG(jsExpressionInQuotes: string, ...args)

Debug a javascript expression by seeing the generated JS code that will be executed. Takes the same arguments as =JS() except will return the string of the JS that will be \`eval\`ed instead of invoking \`eval\`

### \=JS_DEBUG_TYPE(arg: any)

Debug a value and typeof when a cell or other value is passed to a javascript function

\-----------

## Support, Bug reporting & Feedback

If something isn't working or you have some feedback about the add-on, you can send me an email at dvfurlong@gmail.com :)

## Terms

You can use the extension at your own risk, I am not liable for any damages or loss of data.

## Privacy Policy

We don't collect, store, send nor aggregate any of your information whatsoever. All your code is run by google apps script and is managed by google.

Policies effective as of 27 June 2021

##
