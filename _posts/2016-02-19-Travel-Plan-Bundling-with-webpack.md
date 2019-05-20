---
title: 'Travel Plan: Bundling with webpack'
---
In [last article](/2016/02/14/Travel-Plan-React-Basic/){:target="_blank"} I built a simple Travel Plan web app with React. All Javascript code were wrote directly inside script tag. In this article  I will use webpack to bundle my front end resources.

## package.json
{% highlight js %}
{
  "name": "react-travel-plan",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "webpack --progress --colors --watch",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^0.14.7",
    "react-dom": "^0.14.7"
  },
  "devDependencies": {
    "babel-core": "^6.5.2",
    "babel-loader": "^6.2.3",
    "babel-preset-react": "^6.5.0",
    "css-loader": "^0.23.1",
    "style-loader": "^0.13.0",
    "webpack": "^1.12.13"
  }
}
{% endhighlight %}
First we need to install Node.js and run `npm init` to generate package.json file. (Refer to [npm Docs](https://docs.npmjs.com) for details.){:target="_blank"} This file will keep our dependencies. We won't add node_modules folder into repository. Whenever we start a new project, run `npm install` to generate our dependencies. We then use `npm install --save react react-dom` to install all our project dependencies and `npm install --save-dev webpack babel-core babel-loader babel-preset-react style-loader css-loader` to install dev dependencies. We add `"start": "webpack --progress --colors --watch"` in scripts. So we can run `npm start` to start webpack. 

## webpack.config.js
{% highlight js %}
module.exports = {
  entry: './index',
  output: {
    filename: 'bundle.js'
  },
  devtool: 'source-map',
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        query: {
          presets: ['react']
        }
      },
      { test: /\.css$/, loader: 'style-loader!css-loader' },
    ]
  }
};
{% endhighlight %}
webpack will use this config file by default. We add two loaders: one for JSX translation, the other to require css within js.

## Components
{% highlight js %}
var React = require('react');
var Place = require('./Place');

var PlaceList = React.createClass({
  render: function() {
    var placeNodes = this.props.places.map(function(place) {
      return (
        <Place key={place.place_id} {...place} />
      );
    });
    return (
      <div className='PlaceList'>
        {placeNodes}
      </div>
    );
  }
});

module.exports = PlaceList;
{% endhighlight %}
We split all components(Place, PlaceList, PlaceForm, PlaceBox) into 4 separate files and use `modules.exports` to export our components and `require` to include required libraries which uses [CommonJS](http://webpack.github.io/docs/commonjs.html){:target="_blank"} module system.

## entry: index.js
{% highlight js %}
require('./style.css');
var React = require('react');
var ReactDOM = require('react-dom');
var PlaceBox = require('./components/PlaceBox');

ReactDOM.render(
  <PlaceBox />,
  document.getElementById('content')
);
{% endhighlight %}
With `require('./style.css');` Webpack will include the css file into js file which will in turn generate a style tag inside html head and put all the css there.

## index.html
{% highlight js %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Travel Plan</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.5.0/css/font-awesome.min.css" />
  </head>
  <body>
    <div id='content'></div>
    <div class='PoweredBy'><i class='fa fa-google'></i> Powered by Google</div>
    <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyCO9pWIxbEw1Q9M4hBtBIjHXJDqGrr0cBc&libraries=places"></script>
    <script src="./bundle.js"></script>
  </body>
</html>
{% endhighlight %}

webpack will bundle all our js files into bundle.js. So we only need to include one js file here.

## npm start
Run `npm start` to bundle our project. Then you can open index.html file in your browser to check the result.

## Github
[https://github.com/dujushi/snippets/tree/master/ReactBundlingWithWebpack](https://github.com/dujushi/snippets/tree/master/ReactBundlingWithWebpack){:target="_blank"}

## References
1. [npm docs](https://docs.npmjs.com){:target="_blank"}
2. [CommonJS](http://webpack.github.io/docs/commonjs.html){:target="_blank"}
3. [webpack-howto](https://github.com/petehunt/webpack-howto){:target="_blank"}
4. [react-webpack-template](https://github.com/petehunt/react-webpack-template){:target="_blank"}
5. [react-howto](https://github.com/petehunt/react-howto){:target="_blank"}
