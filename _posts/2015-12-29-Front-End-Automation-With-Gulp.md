Gulp is a great tool for front end automation tasks. In this article I will show you how I use Gulp to boost my front end development efficiency. 

## Gulp Plugins
{% highlight js %}
var gulpLoadPlugins = require('gulp-load-plugins');
var plugins = gulpLoadPlugins();
{% endhighlight %}

We will use lots of Gulp plugins in our tasks. With gulp-load-plugins we don't need to import them mannually.

## Config
{% highlight js %}
var config = {
    production: !!plugins.util.env.production,
	port: 9005,
	devBaseUrl: 'http://localhost',
	paths: {
		mainJs: './src/main.js',
		appJs: './src/app/**/*.js',
		scss: './src/**/*.scss',
		html: './src/*.html',
		build: './build'
	}
}
{% endhighlight %}
Define a config variable for all our settings so we can reuse them throughout the file and it makes the code more friendly. We use gulp-util to set our environment. Append --production to ./node_modules/.bin/gulp command to tell him, "We are now on production. Don't generate source maps. Don't instantiate local server. Thanks!" 

## Sass
{% highlight js %}
gulp.task('scss', function () {
    return gulp.src(config.paths.scss)
        .pipe(plugins.if(!config.production, plugins.sourcemaps.init()))
        .pipe(plugins.sass().on('error', plugins.sass.logError))
        .pipe(plugins.concat('bundle.min.css'))
        .pipe(plugins.minifyCss())
        .pipe(plugins.rev())
        .pipe(plugins.if(!config.production, plugins.sourcemaps.write('.')))
        .pipe(gulp.dest(config.paths.build))
        .pipe(plugins.rev.manifest(config.paths.build + "/rev-manifest-css.json", {cwd: config.paths.build + "/", base: config.paths.build + "/", merge: true}))
        .pipe(gulp.dest(config.paths.build));
});
{% endhighlight %}
This task takes all scss files, parse/bundle/minify/revision, and generate source map. When we use "inspect" in browser, brower will show us the original scss file instead of minified css thanks to source map. And we use gulp-if to prevent generating source map on production.

## React
{% highlight js %}
gulp.task('js', function() {
    return browserify(config.paths.mainJs, {paths: ['./bower_components/react']})
        .transform("babelify", {presets: ["react"]})
        .bundle()
        .on('error', console.error.bind(console))
        .pipe(source('bundle.min.js'))
        .pipe(buffer())
        .pipe(plugins.if(!config.production, plugins.sourcemaps.init()))
        .pipe(plugins.uglify())
        .pipe(plugins.rev())
        .pipe(plugins.if(!config.production, plugins.sourcemaps.write('.')))
        .pipe(gulp.dest(config.paths.build))
        .pipe(plugins.rev.manifest(config.paths.build + "/rev-manifest-js.json", {cwd: config.paths.build + "/", base: config.paths.build + "/", merge: true}))
        .pipe(gulp.dest(config.paths.build));
});
{% endhighlight %}
This task uses babel and browserify to transform all JSX files, bundle/uglify/revision, and generate source map as well. We have to use vinyl-buffer to change a stream to a buffer.

## HTML
{% highlight js %}
gulp.task('html', function() {
    var handlebarOptions = {
        helpers: {
            versionPath: function(path, context) {
                return context.data.root[path];
            }
        }
    };
    var manifestCss = JSON.parse(fs.readFileSync(config.paths.build + '/rev-manifest-css.json', 'utf8'));
    var manifestJs = JSON.parse(fs.readFileSync(config.paths.build + '/rev-manifest-js.json', 'utf8'));
    return gulp.src(config.paths.html)
        .pipe(plugins.compileHandlebars(objectAssign(manifestCss, manifestJs), handlebarOptions))
        .pipe(gulp.dest(config.paths.build))
        .pipe(plugins.connect.reload());
});
{% endhighlight %}
Js and Css file paths are always changing due to revision. We update html file to use the new paths with the help of gulp-compile-handlebars.

## Watch
{% highlight js %}
gulp.task('watch', function() {
	gulp.watch(config.paths.html, ['html']);
	gulp.watch(config.paths.scss, function(event) {
            runSequence('scss', 'html');
        });
	gulp.watch([config.paths.mainJs, config.paths.appJs], function(event) {
            runSequence('js', 'html');
        });
});
{% endhighlight %}
This task watches all src file changes and automatically run related tasks. Notice that all above tasks have plugins.connect.reload() in the pipe to refresh the browser for us. We can open up browswer, code editor, terminal side by side. Whenever we make a change in the editor, terminal will show us the result of each task, browser will auto refresh and reflect the changes. This is like a magic. We use run-sequence to make sure gulp runs html task after js task is finished so that we can get the correct revision file path. 

## Connect
{% highlight js %}
gulp.task('connect', function() {
    return plugins.connect.server({
        root: ['build'],
        port: config.port,
        base: config.devBaseUrl,
        livereload: true
    });
});
{% endhighlight %}
gulp-connect will create an instance of local web server for development and support live reload.

## Open
{% highlight js %}
gulp.task('open', function() {
    gulp.src('')
        .pipe(plugins.open({
            uri: config.devBaseUrl + ':' + config.port + '/',
            app: browser
        }));
});
{% endhighlight %}
gulp-open opens up development browser for us.

## Default
{% highlight js %}
gulp.task('default', function() {
    if (config.production) {
        runSequence('clean', 'scss', 'js', 'html');
    } else {
        runSequence('clean', 'scss', 'js', 'html', 'connect', 'open', 'watch');
    }
});
{% endhighlight %}

Finally we set default tasks for Gulp. We run `./node_modules/.bin/gulp` for development and `./node_modules/.bin/gulp --production` for production.

## Github
[https://github.com/dujushi/snippets/tree/master/Gulp](https://github.com/dujushi/snippets/tree/master/Gulp){:target="_blank"}

## Reference
1. [Google Web Starter Kit](https://github.com/google/web-starter-kit/blob/master/gulpfile.babel.js){:target="_blank"}
2. [Gulp Docs](https://github.com/gulpjs/gulp/blob/master/docs/README.md){:target="_blank"}
3. [Gulp Recipes](https://github.com/gulpjs/gulp/tree/master/docs/recipes#recipes){:target="_blank"}
4. [Deploying to Azure](https://github.com/aranasoft/todo-azurewebsites/wiki/Deploying-to-Azure){:target="_blank"}
5. [Gulp! Refreshment for Your Frontend Assets](https://knpuniversity.com/screencast/gulp){:target="_blank"}
6. [coryhouse/react-flux-starter-kit](https://github.com/coryhouse/react-flux-starter-kit/blob/master/gulpfile.js){:target="_blank"}