# serve-lindex

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]

  Serves pages that contain directory listings for a given path.

## Install

This is a [Node.js](https://nodejs.org/en/) module available through the
[npm registry](https://www.npmjs.com/). Installation is done using the
[`npm install` command](https://docs.npmjs.com/getting-started/installing-npm-packages-locally):

```sh
$ npm install serve-lindex
```

## API

```js
var serveIndex = require('serve-lindex')
```

### serveLindex(path, options)

Returns middlware that serves an index of the directory in the given `path`.

The `path` is based off the `req.url` value, so a `req.url` of `'/some/dir`
with a `path` of `'public'` will look at `'public/some/dir'`. If you are using
something like `express`, you can change the URL "base" with `app.use` (see
the express example).

#### Options

Serve index accepts these properties in the options object.

##### filter

Apply this filter function to files. Defaults to `false`. The `filter` function
is called for each file, with the signature `filter(filename, index, files, dir)`
where `filename` is the name of the file, `index` is the array index, `files` is
the array of files and `dir` is the absolute path the file is located (and thus,
the directory the listing is for).

##### hidden

Display hidden (dot) files. Defaults to `false`.

##### icons

Display icons. Defaults to `false`.

##### stylesheet

Optional path to a CSS stylesheet. Defaults to a built-in stylesheet.

##### template

Optional path to an HTML template or a function that will render a HTML
string. Defaults to a built-in template.

When given a string, the string is used as a file path to load and then the
following tokens are replaced in templates:

  * `{directory}` with the name of the directory.
  * `{files}` with the HTML of an unordered list of file links.
  * `{linked-path}` with the HTML of a link to the directory.
  * `{style}` with the specified stylesheet and embedded images.

When given as a function, the function is called as `template(locals, callback)`
and it needs to invoke `callback(error, htmlString)`. The following are the
provided locals:

  * `directory` is the directory being displayed (where `/` is the root).
  * `displayIcons` is a Boolean for if icons should be rendered or not.
  * `fileList` is a sorted array of files in the directory. The array contains
    objects with the following properties:
    - `name` is the relative name for the file.
    - `stat` is a `fs.Stats` object for the file.
  * `path` is the full filesystem path to `directory`.
  * `style` is the default stylesheet or the contents of the `stylesheet` option.
  * `viewName` is the view name provided by the `view` option.
  
##### long

This is a Boolean to add richer information in plain text and JSON responses.  Defualts to `false`.

##### view

Display mode. `tiles` and `details` are available. Defaults to `tiles`.

## Examples

### Serve directory indexes with vanilla node.js http server

```js
var finalhandler = require('finalhandler')
var http = require('http')
var serveLindex = require('serve-lindex')
var serveStatic = require('serve-static')

// Serve directory indexes for public/ftp folder (with icons)
var index = serveLindex('public/ftp', {'dotfiles': 'ignore', 'long': true})

// Serve up public/ftp folder files
var serve = serveStatic('public/ftp')

// Create server
var server = http.createServer(function onRequest(req, res){
  var done = finalhandler(req, res)
  serve(req, res, function onNext(err) {
    if (err) return done(err)
    index(req, res, done)
  })
})

// Listen
server.listen(3000)
```

### Serve directory indexes with express

```js
var express    = require('express')
var serveLindex = require('serve-lindex')

var app = express()

// Serve URLs like /ftp/thing as public/ftp/thing
app.use('/ftp', serveLindex('public/ftp', {'dotfiles': 'ignore', 'long': true}))
app.listen()
```

### JSON response with long set

When a request is made with the header set to accept JSON, for example:

Request Header, `Accept: application/json`

The middleware responds with a JSON format response, if the `long` flag is set to `true` then you will get a response similar to the example below:

```json
[{"dev":50,"mode":16877,"nlink":2,"uid":0,"gid":0,"rdev":0,"blksize":4096,"ino":127625,"size":4096,"blocks":8,"atime":"2017-04-11T01:38:06.168Z","mtime":"2017-04-03T18:20:14.000Z","ctime":"2017-04-11T01:38:06.118Z","birthtime":"2017-04-11T01:38:06.118Z","name":"99 Icons","ext":"","path":"/","type":"dir"},
{"dev":2049,"mode":33261,"nlink":1,"uid":0,"gid":0,"rdev":0,"blksize":4096,"ino":1184500,"size":15701,"blocks":32,"atime":"2013-03-28T01:55:32.000Z","mtime":"2013-03-28T01:55:32.000Z","ctime":"2017-04-11T01:38:05.829Z","birthtime":"2017-04-11T01:38:05.829Z","name":"CHANGELOG.txt","ext":".txt","path":"/","type":"file"}]
```

## License

[MIT](LICENSE). The [Silk](http://www.famfamfam.com/lab/icons/silk/) icons
are created by/copyright of [FAMFAMFAM](http://www.famfamfam.com/).

[npm-image]: https://img.shields.io/npm/v/serve-lindex.svg
[npm-url]: https://npmjs.org/package/serve-lindex

[downloads-image]: https://img.shields.io/npm/dm/serve-lindex.svg
[downloads-url]: https://npmjs.org/package/serve-lindex
