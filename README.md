# get-form-data

Gets form data - or data for a named form element - via `form.elements`.

Data is retrieved in a format similar to request parameters which would be sent
if the form was submitted, so this module is suitable for extracting form data
in the client side of projects which implement ismorphic handling of form
submissions.

## Install

### Node.js

get-form-data can be bundled for the client using an npm-compatible packaging
system such as [Browserify](http://browserify.org/) or
[webpack](http://webpack.github.io/).

```
npm install get-form-data
```

### Browser bundle

The browser bundle exposes a global `getFormData` variable.

You can find it in the [/dist directory](https://github.com/insin/get-form-data/tree/master/dist).

## Usage

### Getting form data

To get data for an entire form, use the `getFormData(form)` function:

```html
<form id="productForm">
  ...
  <label>Product:</label>
  <select name="product">
    <option value="1" selected>T-shirt</option>
    <option value="2">Hat</option>
    <option value="3">Shoes</option>
  </select>

  <label>Quantity:</label>
  <input type="number" name="quantity" min="0" step="1" value="9">

  <label>Express shipping</label>
  <p>Do you want to use <a href="/shipping#express">Express Shipping</a>?</p>
  <div class="radios">
    <label><input type="radio" name="shipping" value="express" checked> Yes</label>
    <label><input type="radio" name="shipping" value="regular"> No</label>
  </div>

  <label>Terms of Service:</label>
  <p>I have read and agree to the <a href="/">Terms of Service</a>.</p>
  <label class="checkbox"><input type="checkbox" name="tos" value="Y" checked> Yes</label>
  ...
</form>
```
```javascript
var form = document.querySelector('#productForm')

var data = getFormData(form)

console.log(JSON.stringify(data))
```
```
{"product": "1", "quantity": "9", "shipping": "express", "tos": "Y"}
```

### Getting field data

To get data for individual form elements (which may contain multiple form inputs
with the same name), use the `getNamedFormElementData(form, elementName)`
function, which is exposed as a proprety of `getFormData`:

```html
<form id="tshirtForm">
  ...
  <label>Sizes:</label>
  <div class="checkboxes">
    <label><input type="checkbox" name="sizes" value="S"> S</label>
    <label><input type="checkbox" name="sizes" value="M" checked> M</label>
    <label><input type="checkbox" name="sizes" value="L" checked> L</label>
  </div>
  ...
</form>
```
```javascript
// You probably want to alias the function :)
var getFieldData = getFormData.getNamedFormElementData

var form = document.querySelector('#tshirtForm')

var sizes = getFieldData(form, 'sizes')

console.log(JSON.stringify(sizes))
```
```
["M", "L"]
```

## API

### `getFormData(form: HTMLFormElement)`

Extracts data from a `<form>`s `.elements` collection - in order to use
`.elements`, form inputs must have `name` or `id` attributes. Since multiple
inputs can't have the same `id` and a `name` allows an input to qualify as a
successful control for form submission, `name` attributes are preferred and will
be given priority if both are present.

#### Return type: `Object<String, String|Array.<String>>`

Properties in the returned data object are mostly consistent with what would
have been sent as request parameters if the form had been submitted:

* All disabled inputs are ignored
* Text inputs will always conribute a value, which will be `''` if they are
  empty.
* Checkbox inputs will only contribute a value if they are checked, in which
  case their `value` attribute will be used.
* Form elements which represent multiple values (select-multiple, or multiple
  inputs with the same name) will only contribute a value if they have at least
  one value to submit. Their values will always be held in an `Array`, even if
  there is only one.

An exception to this is that buttons are completely ignored, as it's only
possible to determine which button counts as successful after it's been used to
submit the form.

### `getNamedFormElementData(form: HTMLFormElement, elementName: String): `

Extracts data for a named element from a  `<form>`s `.elements` collection.

#### Return type: `null|String|Array.<String>`

This function is used by `getFormData()`, so the documentation for individual
return values above also applies.

`null` will be returned if the named element is non-existent, disabled, or
shouldn't contribute a value (unchecked checkboxes, multiple selects with no
selections).

## MIT Licensed