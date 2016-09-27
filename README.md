# biggie
Biggie is a JavaScript application boilerplate written in ES6 based on [bigwheel](https://github.com/bigwheel-framework), a minimalist framework from [Jam3](http://www.jam3.com/).
Be sure to check out the full [documentation](https://github.com/bigwheel-framework/documentation) for bigwheel before you're getting started.

## Getting Started

```sh
git clone https://github.com/baptistebriel/biggie.git

# move into directory
cd biggie

# install dependencies
npm i

# start gulp
gulp
```

Your site will be at `http://localhost:3000` by default using [browser-sync](http://www.browsersync.io)

## Examples

- [oursroux.com](http://oursroux.com)
- [buildinamsterdam.com](http://buildinamsterdam.com)
- [flavinsky.com](http://flavinsky.com)
- [pierrelevaillant.me](http://pierrelevaillant.me)
- [alisharaf.com](http://alisharaf.com)
- [bbriel.me](http://bbriel.me)
- [bigwheel-framework/built-with-bigwheel](https://github.com/bigwheel-framework/built-with-bigwheel)
- & more to come!

## Documentation

### Loading your data

> `/assets/js/utils.js`

- `loadPage(req, view, options, done)`

This function is called on `init`, for all sections that extends default.js
- 1) retrieve the slug of the current route, from the `req` parameter 
- 2) get the HTML content using AJAX, or from biggie's cache if this route already has been resolved  
- 3) create an HTML element and append it to the view  

#### Default Setup

With the default setup, it's loading static `.html` files under the `/templates` folder as such:  
i.e. with a route as `http://localhost:3000/home`, it will load `/templates/home.html`

> `assets/js/utils.js`

```js
ajax.get(`/templates/${slug}.html`, {
  success: (object) => {
    const html = object.data
    page.innerHTML = html
    if(options.cache) cache[slug] = html
    done()
  }
})
```

#### WordPress

If you're using WordPress for example, it would look like this:

> `assets/js/utils.js`

```js
ajax.get(`/${slug}`, {
  success: (object) => {
    const html = object.data.split(/(<body>|<\/body>)/ig)[2]
    page.innerHTML = html
    done()
  }
})
```

> :exclamation: Be sure that your permalinks are set to use the custom structure: all your WordPress urls needs to look like `/home`, `/work/name` etc. It will not work using post id.

You might need to update your `.htaccess` as well:

```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]

RewriteCond %{REQUEST_FILENAME} -f [OR]
RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^ - [L]
RewriteRule . index.php [L]
</IfModule>
```

#### .JSON & Mustache.js

Another thing you could possibly do is use a single `.json` file for all your data and a template engine like [`mustache.js`](https://github.com/janl/mustache.js)

> `assets/js/sections/preloader.js`

```js
init(req, done) {
  classes.add(config.$body, 'is-loading')
  ajax.get(`/data/data.json`, {
    success: (object) => {
      window._data = object.data
      done()
    }
  })
}
```

> `assets/js/utils.js`

```js
ajax.get(`/templates/${slug}.mst`, {
  success: (object) => {
    const rendered = Mustache.render(object.data, window._data)
    page.innerHTML = rendered
    if(options.cache) cache[slug] = rendered
    done()
  }
})
```

### Animations

All sections that extends the default.js have an object called `this.ui` -- using [`query-dom-components`](https://github.com/dcamilleri/query-dom-components) -- so you can access DOM nodes easily.

Here's how most of my `animateIn` & `animateOut` functions looks like:

```js
animateIn(req, done) {
  const tl = new TimelineMax({ paused: true, onComplete: done })
  tl.to(this.page, 1, { autoAlpha: 1 })
  tl.to(this.ui.foo, 1, { y: 0 })
  tl.to(this.ui.bar, 1, { x: 0 })
  // etc...
  tl.restart()
}
```

### Utils

> CSS

- `getRect(top, right, bottom, left)`

Returns the CSS rect string with clip values

```js
const rect = utils.css.getRect(0, 300, 10, 0)
div.style.clip = rect
```

> JS

- `arrayFrom(opt)`

Returns a new Array from an argument (usually a `NodeList`)

```js
const els = document.querySelectorAll('.el')
const arr = utils.js.arrayFrom(els)

// arr.forEach(...)
```

- `clamp(min, value, max)`

Returns a clamped value between min and max values

```js
const min = 0
const max = 200
const value = e.deltaY
const clamped = utils.js.clamp(0, value, 200)
```

- `scrollTop`

Returns either `pageYOffset` or `document.documentElement||document.body.scrollTop`, based on your browser

```js
const scrollY = utils.js.scrollTop()
```

### License

MIT, see [LICENSE.md](https://github.com/baptistebriel/biggie/blob/gh-pages/LICENSE).