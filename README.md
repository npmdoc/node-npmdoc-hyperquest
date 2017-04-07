# api documentation for  [hyperquest (v2.1.2)](https://github.com/substack/hyperquest)  [![npm package](https://img.shields.io/npm/v/npmdoc-hyperquest.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-hyperquest) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-hyperquest.svg)](https://travis-ci.org/npmdoc/node-npmdoc-hyperquest)
#### make streaming http requests

[![NPM](https://nodei.co/npm/hyperquest.png?downloads=true)](https://www.npmjs.com/package/hyperquest)

[![apidoc](https://npmdoc.github.io/node-npmdoc-hyperquest/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-hyperquest_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-hyperquest/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-hyperquest/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-hyperquest/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "James Halliday",
        "email": "mail@substack.net",
        "url": "http://substack.net"
    },
    "bugs": {
        "url": "https://github.com/substack/hyperquest/issues"
    },
    "dependencies": {
        "buffer-from": "^0.1.1",
        "duplexer2": "~0.0.2",
        "through2": "~0.6.3"
    },
    "description": "make streaming http requests",
    "devDependencies": {
        "tape": "^3.0.1"
    },
    "directories": {},
    "dist": {
        "shasum": "95dbf6fcc74c749bae55cc39321335ddd6adc158",
        "tarball": "https://registry.npmjs.org/hyperquest/-/hyperquest-2.1.2.tgz"
    },
    "gitHead": "bbf1bf549e093ce2d42632c1e30e55c43a7d0e3b",
    "homepage": "https://github.com/substack/hyperquest",
    "keywords": [
        "stream",
        "http",
        "transport",
        "request",
        "get",
        "post",
        "put",
        "delete",
        "duplex",
        "pooling"
    ],
    "license": "MIT",
    "main": "index.js",
    "maintainers": [
        {
            "name": "substack",
            "email": "mail@substack.net"
        },
        {
            "name": "juliangruber",
            "email": "julian@juliangruber.com"
        },
        {
            "name": "rvagg",
            "email": "rod@vagg.org"
        }
    ],
    "name": "hyperquest",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/substack/hyperquest.git"
    },
    "scripts": {
        "test": "tape test/*.js"
    },
    "version": "2.1.2"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module hyperquest](#apidoc.module.hyperquest)
1.  [function <span class="apidocSignatureSpan">hyperquest.</span>delete (uri, opts, cb)](#apidoc.element.hyperquest.delete)
1.  [function <span class="apidocSignatureSpan">hyperquest.</span>get (uri, opts, cb, extra)](#apidoc.element.hyperquest.get)
1.  [function <span class="apidocSignatureSpan">hyperquest.</span>post (uri, opts, cb)](#apidoc.element.hyperquest.post)
1.  [function <span class="apidocSignatureSpan">hyperquest.</span>put (uri, opts, cb)](#apidoc.element.hyperquest.put)



# <a name="apidoc.module.hyperquest"></a>[module hyperquest](#apidoc.module.hyperquest)

#### <a name="apidoc.element.hyperquest.delete"></a>[function <span class="apidocSignatureSpan">hyperquest.</span>delete (uri, opts, cb)](#apidoc.element.hyperquest.delete)
- description and source-code
```javascript
delete = function (uri, opts, cb) {
    return hyperquest(uri, opts, cb, { method: 'DELETE' });
}
```
- example usage
```shell
...

Return a duplex stream from 'hyperquest(..., { method: 'PUT' })'.

## var req = hyperquest.post(uri, opts, cb)

Return a duplex stream from 'hyperquest(..., { method: 'POST' })'.

## var req = hyperquest.delete(uri, opts, cb)

Return a readable stream from 'hyperquest(..., { method: 'DELETE' })'.

# events

## req.on('request', function (req) {})
...
```

#### <a name="apidoc.element.hyperquest.get"></a>[function <span class="apidocSignatureSpan">hyperquest.</span>get (uri, opts, cb, extra)](#apidoc.element.hyperquest.get)
- description and source-code
```javascript
function hyperquest(uri, opts, cb, extra) {
    if (typeof uri === 'object') {
        cb = opts;
        opts = uri;
        uri = undefined;
    }
    if (typeof opts === 'function') {
      cb = opts;
      opts = undefined;
    }
    if (!opts) opts = {};
    if (uri !== undefined) opts.uri = uri;
    if (extra) opts.method = extra.method;

    var req = new Req(opts);
    var ws = req.duplex && through();
    var rs = through();

    var dup = req.duplex ? duplexer(ws, rs) : rs;
    if (!req.duplex) {
        rs.writable = false;
    }
    dup.request = req;
    dup.setHeader = bind(req, req.setHeader);
    dup.setLocation = bind(req, req.setLocation);

    var closed = false;
    dup.on('close', function () { closed = true });

    process.nextTick(function () {
        if (closed) return;
        dup.on('close', function () { r.destroy() });

        var r = req._send();
        r.on('error', bind(dup, dup.emit, 'error'));
        dup.emit('request', r);

        r.on('response', function (res) {
            dup.response = res;
            dup.emit('response', res);
            if (req.duplex) res.pipe(rs)
            else {
                res.on('data', function (buf) { rs.push(buf) });
                res.on('end', function () { rs.push(null) });
            }
        });

        if (req.duplex) {
            ws.pipe(r);
        }
        else r.end();
    });

    if (cb) {
        dup.on('error', cb);
        dup.on('response', bind(dup, cb, null));
    }
    return dup;
}
```
- example usage
```shell
...

Set an outgoing header 'key' to 'value'.

## req.setLocation(uri);

Set the location if you didn't specify it in the 'hyperquest()' call.

## var req = hyperquest.get(uri, opts, cb)

Return a readable stream from 'hyperquest(..., { method: 'GET' })'.

## var req = hyperquest.put(uri, opts, cb)

Return a duplex stream from 'hyperquest(..., { method: 'PUT' })'.
...
```

#### <a name="apidoc.element.hyperquest.post"></a>[function <span class="apidocSignatureSpan">hyperquest.</span>post (uri, opts, cb)](#apidoc.element.hyperquest.post)
- description and source-code
```javascript
post = function (uri, opts, cb) {
    return hyperquest(uri, opts, cb, { method: 'POST' });
}
```
- example usage
```shell
...

Return a readable stream from 'hyperquest(..., { method: 'GET' })'.

## var req = hyperquest.put(uri, opts, cb)

Return a duplex stream from 'hyperquest(..., { method: 'PUT' })'.

## var req = hyperquest.post(uri, opts, cb)

Return a duplex stream from 'hyperquest(..., { method: 'POST' })'.

## var req = hyperquest.delete(uri, opts, cb)

Return a readable stream from 'hyperquest(..., { method: 'DELETE' })'.
...
```

#### <a name="apidoc.element.hyperquest.put"></a>[function <span class="apidocSignatureSpan">hyperquest.</span>put (uri, opts, cb)](#apidoc.element.hyperquest.put)
- description and source-code
```javascript
put = function (uri, opts, cb) {
    return hyperquest(uri, opts, cb, { method: 'PUT' });
}
```
- example usage
```shell
...

Set the location if you didn't specify it in the 'hyperquest()' call.

## var req = hyperquest.get(uri, opts, cb)

Return a readable stream from 'hyperquest(..., { method: 'GET' })'.

## var req = hyperquest.put(uri, opts, cb)

Return a duplex stream from 'hyperquest(..., { method: 'PUT' })'.

## var req = hyperquest.post(uri, opts, cb)

Return a duplex stream from 'hyperquest(..., { method: 'POST' })'.
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
