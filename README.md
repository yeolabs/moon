# moon v1.1.4

moon is a minimal web application framework written in Go and React ES6. It uses Node.js buildtools extensively during the development process (ie. webpack). Its purpose is to implement Single Page Applications (SPA) in a way that optimizes for rapid prototyping. It achieves this with hot module replacement, webpack, react, and opinionated routing.

Once your application is ready for production you can remove Node.js as a dependency. 

# Frontend Technologies 

The moon frontend uses [React 0.14.2](https://facebook.github.io/react/). React components are created using ES6 modules, see [Babel](https://babeljs.io/). moon has default support for [SASS](http://sass-lang.com/), but this can be configured to LESS or Stylus through webpack config.

# Hot Module Replacement (HMR)

moon implements HMR quite efficiently. It combines [webpack-hot-middleware](https://github.com/glenjamin/webpack-hot-middleware) and [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) to serve hot client bundles from an HMR express server. The HMR server runs on a separate port from your Go server and pushes updates to the client via CORS.

To update react components it utilizes a stack based on [react-transform-hmr](https://github.com/gaearon/react-transform-hmr). 
 
This stack is very lightweight, configurable, and pluggable.

_NB. Your Go server will still serve API endpoints and other static assets during hot mode._

_NB2. If you turn off `hot` mode while the servers are running, they will need to be restarted in order to serve the proper bundles.`_

# Config

moon apps are configured using _toml_.

```toml
[common]	
# development|production
env = "development"
# Bundles, images, stubs, etc. 
static = "static"
# Auto-prefixed with static 
js = "bundle.js"
# Auto-prefixed with static 
style = "bundle.css"
# prefix for api endpoints 
api = "/api/"
# when server.hot is true the bundle is served via jsonp from webpack server
hmr = "localhost:8889" 
# enables live reload. NB. env must be development and hmr must be set
hot = true 

[server]
template = "template.html.tpl"
address = "localhost:8888"
# when env is production it will prefix bundles with the serve option
# this is useful for reverse proxies
serve = "example.com"

[client]
entry = "entry.js"
```

# Routing

1. API endpoints - prefixed with the `api` config entry 
2. Static files stored in the directory set in the `static` config entry
3. Default to client routing. If the prior two cases are not satisfied, the server will send the frontend application code. The application takes the form of an html5 template with a link and script tag pointing to the css and js bundles that were output by webpack. (when in hot mode they are served from memory instead) 

On the frontend, assuming the client bundle is served, the routing is deferred to react-router. Routes can be added per component from `/client/entry.js`:

```javascript
const routes = {
	path: '/',
	component: App,
	childRoutes: [
		{ path: 'about', component: About },
		{ path: 'contact', component: Contact },
	]
}
```

# Usage

Things you will need:

1. Go
2. Node.js (4.0+ recommended)
3. npm

First thing you want to do is clone this repo. Enter the cloned directory and type:

`npm install`

Open up a separate tty and type:

`npm start`

Note that we aren't running webpack directly. This gives us a lot more power to control the environment.

Make sure you have the Go dependencies to run the server. Assuming you have set your GOPATH, just manually install the 3 dependencies:

```bash
go get github.com/Sirupsen/logrus
go get github.com/BurntSushi/toml
go get github.com/julienschmidt/httprouter
```

A simple Makefile is provided to build and run the server:

`make run-server`

Navigate to `http://localhost:8888` and verify that "Hello World!" is displayed.

Check `http://localhost:8888/api/version` to see the endpoint support.

Edit `/client/components/app.js` and see that the code is automatically updated in your browser without a refresh.

# Adding Endpoints

A simple version endpoint is provided in the `/server/api.go` file. You can manage all of your endpoints from this file. Endpoints must implement `httprouter.Handle`. In other words they need the same signature as the `VersionEndpoint` function. 

To add your own endpoint, create a new function with the same signature as `VersionEndpoint`. Then add the endpoint as shown below. 

Endpoints can have parameterized syntax provided by `httprouter`. Just remember that the endpoint will be prefixed with whatever you have set in your config. For example:

```go
s.Endpoint("/user/:id/:action", API_POST, UserEndpoint)
```

Could be reached at `http://localhost/api/user/1/edit`

Note that `API_GET`, `API_POST`, and `API_BOTH` are bit masked in order to determine allowed methods for the router.
