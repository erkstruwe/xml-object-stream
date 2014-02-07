# xml-object-stream-sax

xml-object-stream-sax is a simple wrapper on top of sax.js that extracts json objects that match a given xpath-ish expression from an xml stream and passes them to a callback or stuffs them into a promised array.

## Usage

```javascript
var xos = require('xml-object-stream-sax')(/* config object */);
var xml = '<library>' +
          '<book published="1622"><author>John</author></book>' +
          '<book author="Jane"><author country="Uganda">Jane</author></book>' +
          '</library>';
xos(xml, '//book', function(book) { console.log(book); });
```

## Configuration

The config object is passed straight through to sax.js createStream, so any additional options applicable there are also applicable here.

### strict
**default: true**

Passed through to sax.js createStream to turn on strict parsing. Non-strict parsing assumes icase is true.

### pojo
**default: false**

Instead of creating a dom-ish tree, plain object trees will be assembled with xml attributes and children set as properties. Text is available as the text property. Same-named attributes and children are turned into array properties. If there are no children or attributes, the text will be used instead of an object.

### icase
**default: true**

If set to true, the xpath-ish query will be case-insensitive. Non-strict mode enables this by default.

## Arguments

function(xml, xpath[, callback]);

### xml

This can be a string containing xml, in which case, it will be written directly to the sax stream.

It can be a string starting with 'http://', which will be opened with the node http module as a stream and piped to the sax stream.

It can be a string starting with 'file://', which will have the first 7 characters stripped off, be opened with the node fs module as a stream, and piped to the sax stream.

It can be an object with a pipe method, which will be called with the sax stream.

Anything else will cause an exception to be thrown.

### xpath

This is a very limited subset of xpath-ish syntax. /foo/bar will match bar nodes attached to a foo node, which is the root element. //bar will match any bar node in the hierarchy, including nested bar nodes. The direct descendent / and descendent // selectors may be intermingled as needed. /foo//bar//baz/bat will match bat nodes directly descended from baz nodes, which have bar ancestors, which have a foo root ancestor.

The matching is done using a non-gready regular expression made out of the xpath string. Only nodes that match or descend from a match of the xpath are kept around while the stream is being processed, so (theoretically) very large chunks of xml that have bits that are interesting deeply embedded in them should not consume too many resources.

### callback

For each matching object, the callback will be called with the matched object.

If no callback is provided a promise will be returned. If the promise is resolved, it will contain an array of all matched objects.
