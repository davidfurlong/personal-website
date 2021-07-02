---
template: page
title: Javascript in Sheets fast and easy with Formulas
slug: javascript-in-google-sheets
draft: false
---
"Javascript in Sheets fast and easy with Formulas" is a google sheets add-on that provides you with helpful formulas to use javascript in your sheets.

After installing this add-on you should have access to the following formulas in google sheets.

## Formulas
### \=JS(jsArrowFunctionInQuotes: string, ...args)

Write a javascript function.

For example 

```
=JS("(a) => a.join('')", C1:C3)
=JS("(a, b) => Math.pow(a, b)", C1, C2)
=JS("(a) => { var x = 10; x = a + x; return x }", C1)
```

@param {jsArrowFunctionInQuotes} a javascript arrow function surrounded by quotation marks

@param {...additionalArguments} (optional) any additional arguments to JS will be passed to the javascript expression provided as arguments

@return the return value of the javascript code run with \`eval\` and invoked as a function with the provided arguments, essentially eval((arg1)(arg2, arg3, ...))


### \=JSE(jsExpressionInQuotes: string, ...args)

Write a javascript expression. 


For example 

```
=JSE("$1.length", C1:C3)
=JSE("$1.join('')", C1:C3)
=JSE("Math.pow($1, $2)", C1, C2)
=JSE("{ var x = 10; x = $1 + x; return x }", C1)
```

@param {jsExpressionInQuotes} a javascript expression surrounded by quotation marks, in which $1, $2, $3... will be replaced by additional arg1, arg2, arg3 respectively

@param {...additionalArguments} (optional) any additional arguments will be substituted into the javascript expression for their substitution tokens $1, ...

\* @return the return value of the javascript code run with \`eval\` after substituting in the provided arguments

### \=FETCH(url: string, params?: string)

Makes a request to fetch a URL

For example
```
=FETCH('www.google.com', '{ method: "POST", payload: { cat: true } }')
```

@param {url} url to fetch

@param {params} a JS object in quotes. For example '{ method: "POST", payload: { cat: true } }' See all options at https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app#advanced-parameters

 @return the response body of the completed request as text


### \=FETCHJSON(url: string, params?: string)

Makes a request to fetch a URL with a contentType: application/json header.

For example
```
=FETCHJSON("https://aws.random.cat/meow")
```

Is shorthand for =FETCH(<URL>, "{ contentType: application/json }")

@param {url} to fetch

@param {params}  a JS object in quotes. For example '{ method: "POST", payload: { cat: true } }' See all options at https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app#advanced-parameters

@return the response body of the completed request as text

## Additional Formulas for debugging

### \=JS_DEBUG(jsArrowFunctionInQuotes: string, ...args)

Debug a javascript function by seeing the generated JS code that will be executed. Takes the same arguments as =JS() except will return the string of the JS that will be \`eval\`ed instead of invoking \`eval\`

@param {value} any cell or value

@return a JSON.stringify of an object with the provided argument's value and typeof for debugging what values JS gets for a given cell

### \=JSE_DEBUG(jsExpressionInQuotes: string, ...args)

Debug a javascript expression by seeing the generated JS code that will be executed. Takes the same arguments as =JS() except will return the string of the JS that will be \`eval\`ed instead of invoking \`eval\`

### \=JS\_DEBUG\_TYPE(arg: any)

Debug a value and typeof when a cell or other value is passed to a javascript function

## Alternatives to this add-on

Everything this add-on does you can already do in google sheets, but it takes a bit more work. In any spreadsheet you can go to Tools > Script Editor and then write a javascript function and then use the corresponding function as a formula in your sheet.
However this approach is a) slower b) harder to transfer between sheets as you need to copy and paste not just the sheet data but also the contents of the Script Editor c) harder to share between people d) has slower feedback loops with changing a formula and seeing the updates. However the approach of using the Script Editor has the upsides of a) being able to read the syntax clearer and with syntax highlighting and IDE features b) being able to easily reuse formulas. Thus sometimes using this add-on is better, especially for lots of small quick changes and sometimes using the google sheets script editor is better.

\-----------

## Support, Bug reporting & Feedback

If something isn't working or you have some feedback about the add-on, you can send me an email at dvfurlong@gmail.com :)

## Terms

You can use the extension at your own risk, I am not liable for any damages or loss of data.

## Privacy Policy

We don't collect, store, send nor aggregate any of your information whatsoever. All your code is run by google apps script and is managed by google. "Javascript in Sheets fast and easy with Formulas" only does what your formulas instruct it to do. For example if you use the =FETCH() formula to POST data to a URL then that data is sent via a POST request there. This add-on doesn't make any requests nor send any data anywhere which you do not write yourself as a FORMULA. This add-on ONLY provides formulas and these formulas ONLY do what you instruct them to do. In order for the formulas to work the add-on uses a google apps script which runs javascript for your sheet. Your data is never sent by "Javascript in Sheets fast and easy with Formulas" to any of our servers nor do we collect any data whatsoever. This add-on does not store nor share Google user data.

_Policies effective as of 27 June 2021_
