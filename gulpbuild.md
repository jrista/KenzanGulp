# Gulp: The JavaScript Build System

What is Gulp?
Installing Gulp
Plugins
Tasks
Pipelines
The Gulpfile
gulp-david?
More Complex Things (gulp-newer gulp-if)
- Watch
- WebServer
- Plumbing
Questions

## What is Gulp?

Gulp is a build system for JavaScript apps. More specifically, it is an orchestrated task-oriented, streaming pipeline for efficiently performing build tasks on ‘Vinyl'. Gulp is similar to Grunt, an older build system for JavaScript apps, and is more well known. Grunt is similar, in that it is also task-oriented, and it is a capable tool, however it lacks some of the amazing things that Gulp can do. Grunt is also configuration-based, and all builds are set up via JSON configuration objects. This can lead to certain complexities that simply do not exist with Gulp. Gulp, unlike Grunt, is just pure javascript code. It’s totally PC (pure code)!

Gulp is an orchestration. Gulp is task oriented. It is also streaming, and pipelined. Gulp makes use of Vinyl. Orchestration? Streaming? Pipelined? Vinyl? Let’s start with the latter. Vinyl is, as described by its creator, a “virtual file format”…but it’s really just a metadata object that describes files. The cool thing about Vinyl is it can describe any file…anywhere. Files don’t have to live on disk…they can live on a server…or on the web. On potentially anything. **Vinyl describes files.** Gulp uses streams of Vinyl to move information throughout it’s processing pipeline. 

You can think of Gulp like a *nix command line of build tools. Just like the command line, the idea is to chain together simple task-oriented commands that do one thing, and do it really, really well…and “pipe” the output of each command to a subsequent command, one after the other. With the command line, it is usually lines of text that we are piping along. Pipes can usually be 'buffered', sometimes they can be 'streamed'. You might do something like the following on a command line:

`cat *.js | ack ’task’ | less`

This simple command line will concatenate all javascript files in the current directory, pipe them through the *ack* search tool which will look for the term ‘task’, and the output of that will be piped through less, which will allow us to scroll through the results if they consume more lines than are available within the terminal. Gulp is very similar, only instead of chaining commands with the | symbol, we chain function calls via .pipe:

```javascript
gulp
  .src(‘*.js’)
  .pipe(jshint())
  .pipe(concat())
  .pipe(minify())
  .pipe(gulp.dest(‘../dest/all.min.js’));
```

This is a simple gulp pipeline. Usually wrapped in a gulp task, such as ‘build:js’, this pipeline sources all .js files in the current directory, concatenates all the .js files into a single script, minifies the .js represented by that concatenation, and finally writes the minified output to a particular destination location. Task oriented. Streaming. Pipelined. Executed within an orchestration.

## Installing Gulp

Gulp, like most build systems for JavaScript, runs with Node.js. As such, we must have Node installed. Gulp is packaged and available via NPM, or the Node Package Manager, so we need to have npm installed as well. We can easily check if these are installed with the following commands:

`node -v`

This will, if node is installed, print the installed version. We can do the same with npm:

`npm -v`

If node or npm are not installed, we can install them rather simply. With a Mac, simply use brew:

`brew install node`

This should also install npm, however if it does not:

`brew install npm`

Once node and npm are installed, we need to initialize our node project. This can be done using npm's init command, which will interactively walk you through the creation process:

`npm init`

You will be asked to supply a project name, version, and a variety of other details. Provide those now. For this project, I have used name: GulpBuild, version: 1.0.0, author: Kenzan. For the record, it is not actually necessary to go through the full initialization process if you want to get started quickly. The following will also suffice (and is much quicker):

`echo '{}' > package.json`

Now that we have initialized our node project, we can then install gulp. Gulp, being a build tool, actually has “global” components as well as local components. The global part of gulp supplies the necessary command line interface (CLI). We install this via the following command:

`sudo npm install -g gulp`

You can check that gulp installed properly by checking it’s version:

`gulp -v`

When you create a gulp build for your project, you will also need to install it locally:

`npm install gulp —save-dev`

Once both are installed, you are ready to start building your project with gulp.

## Plugins 

The Gulp package itself doesn’t do much. It provides the base system, the streaming pipelined task-oriented system. It also provides some basic file watching capabilities, which can be used to work with a few of the cooler features of JavaScript development like live reloading. Beyond that, most of the functionality is provided by plugins. Hundreds, if not thousands, of plugins exist for Gulp, and each one solves some particular problem. Such as linting (valiating) code, compiling Jade templates into HTML, compiling Less into CSS, minifying script and content, concatenation, even starting a fully featured web server to host your project during development if you so need.

Plugins for gulp are usually installed locally. They are development dependencies in most cases, as they are not required for the regular operation of the project. We can install a gulp plugin using npm again. Let’s start with a simple plugin, like validating code style with jshint:

`npm install gulp-jshint —-save-dev`

This downloads the `gulp-jshint` module from an npm repository, and installs it under the node_modules folder. The --save-dev option also adds a development dependency reference to the module in the package.json file. At this point, our package.json should look something like this:

```json
{
  "name": "GulpBuild",
  "version": "1.0.0",
  "description": "The Gulp Build System - Lunch and Learn",
  "main": "index.js",
  "author": "Kenzan",
  "devDependencies": {
    "gulp": "^3.9.0",
    "gulp-jshint": "^1.11.0"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

We can install additional plugins to perform other common tasks, such as file concatenation, obfuscation/minification, Less to CSS and Jade to HTML compilation, etc. We can actually install all of these with a single npm install command:

`npm install gulp-concat gulp-uglify gulp-less gulp-jade --save-dev`

This should update our package.json like so:

```json
{
  "name": "GulpBuild",
  "version": "1.0.0",
  "description": "The Gulp Build System - Lunch and Learn",
  "main": "index.js",
  "author": "Kenzan",
  "devDependencies": {
    "gulp": "^3.9.0",
    "gulp-concat": "^2.6.0",
    "gulp-uglify": "^1.2.0",
    "gulp-jade": "^1.0.1",
    "gulp-jshint": "^1.11.0",
    "gulp-less": "^3.0.3"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

## Tasks

When creating build projects with Gulp, we build tasks. Tasks are simple jobs that make use of one or more plugins to accomplish some unit of work that accomplishes a build task. Tasks can depend on other tasks. Tasks can and will run concurrently whenever possible. Tasks can also be set up to watch for changes in code and invoke live updates of the compiled build output. This latter one is a very important concept that we will get into later. 

Creating a task is very simple. You've actually already seen what the meat of a task looks like, in the first example above. Let's start with the most common task you'll encounter when developing javascript applications: to validate and concatenate all the application javascript together into a single .js file, then obfuscate and minify that .js file.

The heart of building gulp tasks is the `gulp.task()` function. This binds a string name for the task to a function that actually holds the task implementation. Since Gulp is just pure javascript code, you have (unlike Grunt and other config-based build systems) the full power of JavaScript at your disposal when writing tasks. There are few limitations, outside of having to use the gulp framework to actually work with gulp as a build system:

```javascript
gulp.task('build:js', function(){
	return gulp
		.src(['src/*.js', 'src/**/*.js'])
		.pipe(jshint())
		.pipe(concat())
		.pipe(uglify())
		.pipe(gulp.dest('/deploy/app.min.js'));
});
```

That's all there really is to creating a gulp task. Source your files, and pipe that stream of files (which is actually a stream of Vynil file descriptors) through various plugins, and finally write that stream to some destination location. There is one additional aspect to building tasks. You may be wondering where the jshint(), concat() and uglify() functions come from. For that matter,  you may be wondering where the gulp object comes from. This is where Node.js comes into play. Gulp runs within node, and we use node to require modules, similar to how Java requires packages or .NET requires assemblies. Our full script may look like so:

```javascript
var gulp = require('gulp'),
	jshint = require('gulp-jshint'),
	concat = require('gulp-concat'),
	uglify = require('gulp-uglify');

gulp.task('build:js', function(){
	return gulp
		.src(['src/*.js', 'src/**/*.js']) // Source all javascript in ./src directory
		.pipe(jshint()) // Validate
		.pipe(concat()) // Join all js files into one (output single Vynil descriptor)
		.pipe(uglify()) // Obfuscate and minify .js file
		.pipe(gulp.dest('./deploy/app.min.js')); // Copy final .js file to deployment directory
});
```

The require() function is a part of node, and is the means by which we "bring in" the javascript code provided by third-party modules that we install with npm. The name of the module will usually match the name of the module as referenced by dependencies or devDependencies in the package.json file. You can assign the module loaded by require to a variable...of any name, however convention is to use the name that matches the module (minus any gulp- prefix if one exists.)

### Task Dependencies

Tasks can depend on other tasks. This allows us to create root tasks, such as 'build' or 'build:all', that run other tasks. Lets say we have several other tasks, one that builds an angularjs template script for all the Jade templates of an AngularJS project, and another that compiles .less styles into a single .css file. These tasks might look like so:

```javascript
var gulp = require('gulp'),
	jshint = require('gulp-jshint'),
	concat = require('gulp-concat'),
	uglify = require('gulp-uglify'),
	jade = require('gulp-jade'),
	less = require('gulp-less'),
	minhtml = require('gulp-minify-html')
	mincss = require('gulp-minify-css'),
	templatecache = require('gulp-angular-templatecache');

gulp.task('build:js', function(){
	return gulp
		.src(['src/*.js', 'src/**/*.js'])			// Source all js in ./src directory
		.pipe(jshint()) 							// Validate
		.pipe(concat()) 							// Join all js into one (output single Vynil)
		.pipe(uglify()) 							// Obfuscate and minify .js file
		.pipe(gulp.dest('./deploy/app.min.js')); 	// Copy final .js to deployment directory
});

gulp.task('build:html', function() {
	return gulp
		.src(['src/*.jade', 'src/**/*.jade'])		// Source all .jade templates
		.pipe(jade())								// Compile .jade into .html
		.pipe(minhtml())							// Minify each .html
		.pipe(templatecache('app.tmpls.js', {		// Generate an AngularJS template cache js
			module: 'app.tmpls',
			root: '/',
			standalone: 'true'
		}))
		.pipe(gulp.dest('./deploy/'));				// Copy app.tmpls.js to deployment directory
});

gulp.task('build:css', function() {
	return gulp
		.src('src/**/*.less')						// Source all .less templates
		.pipe(concat())								// Concatenate to single .less
		.pipe(less())								// Compile .less into .css
		.pipe(mincss())								// Minify .css file
		.pipe(gulp.dest('./deploy/content/css'));	// Copy .css to deployment directory
});
```

We need some more gulp plugins, so we can install them the same as any other:

`npm install gulp-minify-html gulp-minify-css gulp-angular-templatecache --save-dev`

It should be noted that each of the node packages we are installing have their own dependencies on other packages. For example, `gulp-minify-html` depends on the `minimize` project, which itself depends on other projects. Installing dependencies with npm also installs all of their dependencies, so we don't have to worry about bringing everything in manually. All we need to be concerned with is referencing the direct dependencies of our own project.

Now we can run our build tasks by specifying the tasks to run at the command line. With our three tasks above, we would run them like so:

`gulp build:js build:html build:css`

This will run these three tasks for us, however it can be tedious typing out each task name every time we need to do a standard build. We can wrap up all of our various individual build tasks into a single build task by binding a list of other tasks to run to a new task (rather than binding a function to the task), like so:

``` javascript
gulp.task('build', ['build:js', 'build:html', 'build:css']);
```

We can then simply run our entire build like so:

`gulp build`

In this case, the buil task is __dependent upon__ build:js, build:html and build:css. We could make a full task dependent upon another as well. For example, we may wish to separate out our jshint into it's own task, as it is actually independent of 'building' the js. We may also wish to have jshint write out a report for us:

```javascript
gulp.task('validate:js', function() {
	return gulp
		.src(['src/*.js', 'src/**/*.js'])
		.pipe(jshint())
		.pipe(jshint.reporter('default'));
});

gulp.task('build:js' ['validate:js'], function() {
	...
});
```

In this case, the validate:js task must be run before build:js...much like our build task requires that the three other tasks be run before it (and, since build itself has no function, gulp will use it to simply kick off the other tasks).

### Default Task

Another note about tasks. Gulp allows us to define a default task to run, which is executed if we simply run 'gulp' at the command line. We define a default task like so:

``` javascript
gulp.task('default', ['build']);
```

### Task Concurrency

Gulp, in it's aim to be efficient, is also concurrent. Whenever possible, it will execute tasks with maximum concurrency. Each task is by nature an independent unit of work, and they do not generally depend on the output of other tasks. In complex scenarios, there are ways of creating dependencies between tasks, but doing so is not recommended because of Gulp's default nature to be concurrent whenever possible. 

In our original build task, we depend on build:js, build:html and build:css. Gulp will attemot to kick off all three of these tasks simultaneously, and will simply let each one run to completion, without care about which finishes first or anything else.

If our modified build:js task depended on more than one other task, gulp would also try to run all of those tasks concurrently if possible. Task serialization is not possible within dependencies, however if one were to chain dependencies off each other one at a time, that could technically enforce serial task execution. There are consequences of doing that, however. Not to mention the fact that it violates one of the primary goals of Gulp: efficiency. 

If ordered task execution is absolutely essential, it is best to use a framework that supports that in a gulpy way. One such project is run-sequence, which is actually used within the dependency list for a gulp task to define order dependent and order-independent groups of dependent tasks. It's a handy plugin...IF you need such a thing. For the most part, it is still best to try and work with gulp as much as possible than against it.

## Pipelines

In the above code examples, you will notice that we have called .pipe() multiple times in each task. This is how we build __streaming pipelines__ in gulp. Pipelines connect streams of Vynil (file descriptors) to stream processors (plugins). If we:

 `gulp.src('src/*.js').pipe(concat('temp.js')).pipe(uglify()).pipe(gulp.dest('./dist');` 

 in a task, the javascript files are "streamed" through the pipeline:

a.js \
b.js --> [{a.js},{b.js},{c.js}] --> concat('temp.js')   --> uglify()           --> ./dist/temp.min.js
c.js /                              \> [{temp.js}]      |   \> [{temp.min.js}] |

Notice here that the things moving through the pipeline get transformed along the way. We start out with three discrete javascript files. The first stage of the pipeline, our concatenation stage, merges those files together into a temp.js via simple concatenation. Our stream now consists of a single thing. The next stage of the pipeline "uglifies" the temp.js file (minifies and obfuscates), and it outputs a single thing into the stream. 

It should also be noted that the things in our pipeline really do "stream" through. The first stage of our pipeline gets a.js first, and concat will write the contents of that .js file to the temp.js file as it receives it. Then it will receive b.js, and write that, then it will receive c.js and write that. With streaming, you don't have to complete the reading of all source files first...they can be delivered as they are read. A more interesting pipeline from above is our build:html task. This actually works with multiple files as a stream for a couple of stages. We might have a state in our pipeline like this:

a.jade \
b.jade --> [...,...,{c.jade}] --> jade()                --> minhtml()                 --> ...
c.jade /                          \> [...,{b.html},...] |   \> [{a.min.html},...,...] |

In this case, c.jade was just read off disk, but b.jade has already been converted to b.html, and a.jade was already converted to a.html and minified to a.min.html. Pipelines **stream**, and as such, are very efficient.

### Pipeline Asynchrony

Furthermore, because Gulp tries to be as concurrent as possible, there can be multiple pipelines streaming data at the same time. Technically speaking, JavaScript runtimes are single-threaded, however JavaScript is also asynchronous by nature, and concurrent task execution will usually be interleaved on the same thread. In our build task from above, our build:js, build:html and build:css tasks will each have their own stream from the specified source files, and each one will be streaming those files through each of the stages of their own pipelines concurrently if possible. 

It is possible for a task to be synchronous, rather than asynchronous. In such a case, synchronous tasks will execute one after the other, in whatever order the Gulp orchestrator dedicdes to execute them, unless dependencies are specified for each task. Tasks will wait for their dependencies to run, and those dependencies will not necessarily execute in any particular order. The only order determinism in Gulp is that a task dependent upon others will wait for the others to complete. 

Pipelines can be efficient and concurrent because most of the time, they operate on files. I/O operations tend to be slow, and JavaScript runtimes like Node.js can execute other code while waiting for slow operations to complete, thus maximizing the efficiency of each concurrent pipeline.

## The Gulpfile

With Gulp, all of your tasks need to be defined in such a way that the gulp command line can find them. By default, gulp looks for the gulpfile. This is a file in your project directory called gulpfile.js. Some projects simply drop all their gulp code into this one file, the gulpfule, and leave it at that. In the case of our example code above, our gulpfile would look like this:

```javascript
var gulp = require('gulp'),
	jshint = require('gulp-jshint'),
	concat = require('gulp-concat'),
	uglify = require('gulp-uglify'),
	jade = require('gulp-jade'),
	less = require('gulp-less'),
	minhtml = require('gulp-minify-html')
	mincss = require('gulp-minify-css'),
	templatecache = require('gulp-angular-templatecache');

// Validation tasks
gulp.task('validate:js', function() {
	return gulp
		.src(['src/*.js', 'src/**/*.js'])
		.pipe(jshint())								// Validate js code
		.pipe(jshint.reporter('default'));			// Report on validation
});

// Build tasks
gulp.task('build:js', ['validate:js'], function() {
	return gulp
		.src(['src/*.js', 'src/**/*.js'])			// Source all js in ./src directory
		.pipe(concat()) 							// Join all js into one (output single Vynil)
		.pipe(uglify()) 							// Obfuscate and minify .js file
		.pipe(gulp.dest('./deploy/app.min.js')); 	// Copy final .js to deployment directory
});

gulp.task('build:html', function() {
	return gulp
		.src(['src/*.jade', 'src/**/*.jade'])		// Source all .jade templates
		.pipe(jade())								// Compile .jade into .html
		.pipe(minhtml())							// Minify each .html
		.pipe(templatecache('app.tmpls.js', {		// Generate an AngularJS template cache js
			module: 'app.tmpls',
			root: '/',
			standalone: 'true'
		}))
		.pipe(gulp.dest('./deploy/'));				// Copy app.tmpls.js to deployment directory
});

gulp.task('build:css', function() {
	return gulp
		.src('src/**/*.less')						// Source all .less templates
		.pipe(concat())								// Concatenate to single .less
		.pipe(less())								// Compile .less into .css
		.pipe(mincss())								// Minify .css file
		.pipe(gulp.dest('./deploy/content/css'));	// Copy .css to deployment directory
});

// Root tasks
gulp.task('build', ['build:js', 'build:html', 'build:css']);
gulp.task('default', ['build']);
```

Monolithic code files tend to be an anti-pattern, and with gulp it is no different. A gulpfile for a large project could become a monster, and be difficult to maintain...or even find a particular task in. I've seen such gulpfiles and they can be frustrating. 

As with any well-organized codebase, it is best to break things up into separate files. This is not actually supported "naturally" by gulp. Node has support for modules, but even that doesn't actually work with gulp. There is a handy third-party module for node that can help us get organized, however: require-dir. This module can bring in an entire directory of script files, basically like an include, into another script file. We can install it with npm:

`npm install require-dir --save-dev`

Once installed, this will allow us to break up our gulpfile into many separate files, put all those files in a directory, then require all the scripts in that directory in the gulpfile. Once separated out, say into a ./gulp directory, our scripts will look like this:

In ./gulp/validate-js.js:

```javascript
var gulp = require('gulp'),
	jshint = require('gulp-jshint');

gulp.task('validate:js', function() {
	return gulp
		.src(['src/*.js', 'src/**/*.js'])
		.pipe(jshint())								// Validate js code
		.pipe(jshint.reporter('default'));			// Report on validation
});
``` 
In ./gulp/build-js.js:

```javascript
var gulp = require('gulp'),
	concat = require('gulp-concat'),
	uglify = require('gulp-uglify');

gulp.task('build:js', ['validate:js'], function() {
	return gulp
		.src(['src/*.js', 'src/**/*.js'])			// Source all js in ./src directory
		.pipe(concat()) 							// Join all js into one (output single Vynil)
		.pipe(uglify()) 							// Obfuscate and minify .js file
		.pipe(gulp.dest('./deploy/app.min.js')); 	// Copy final .js to deployment directory
});
```

In ./gulp/build-html.js:

```javascript
var gulp = require('gulp'),
	jade = require('gulp-jade'),
	minhtml = require('gulp-minify-html')
	templatecache = require('gulp-angular-templatecache');

gulp.task('build:html', function() {
	return gulp
		.src(['src/*.jade', 'src/**/*.jade'])		// Source all .jade templates
		.pipe(jade())								// Compile .jade into .html
		.pipe(minhtml())							// Minify each .html
		.pipe(templatecache('app.tmpls.js', {		// Generate an AngularJS template cache js
			module: 'app.tmpls',
			root: '/',
			standalone: 'true'
		}))
		.pipe(gulp.dest('./deploy/'));				// Copy app.tmpls.js to deployment directory
});
```

In ./gulp/build-css.js:

```javascript
var gulp = require('gulp'),
	concat = require('gulp-concat'),
	less = require('gulp-less'),
	mincss = require('gulp-minify-css');

gulp.task('build:css', function() {
	return gulp
		.src('src/**/*.less')						// Source all .less templates
		.pipe(concat())								// Concatenate to single .less
		.pipe(less())								// Compile .less into .css
		.pipe(mincss())								// Minify .css file
		.pipe(gulp.dest('./deploy/content/css'));	// Copy .css to deployment directory
});
```

Our gulpfile is now reduced to the much simpler following file in ./gulpfile.js:

```javascript
var gulp = require('gulp'),
	requireDir = require('require-dir');

// Bring in all tasks
requireDir('./gulp');

// Root tasks
gulp.task('build', ['build:js', 'build:html', 'build:css']);
gulp.task('default', ['build']);
```

## More Complex Things

With the above, you should be able to get started accomplishing the most common tasks with gulp. Your "standard" build, compiling template languages like Jade and Less, packaging up and minifying code, and basic folder deployments. Gulp, with the right plugins, is far more powerful than this, however. It can also support the development process by providing web site hosting, live reloading, and more.

### Watching

The first thing to explore after getting a basic build going is watching. Watching, of code files for changes, is a built in feature of Gulp. A watch is passed a set of files for which some action, or actions, should be performed when any of those files changes. Normally, a set of gulp tasks will be specified to run when the watch triggers, but it is also possible to pass a callback function. Watches are frequently used to watch for changes in javascript, less, css, jade, html, and other "code" files, so they can be rebuilt and even redeployed automatically.

A simple watch for building javascript might look like this:

```javascript
gulp.task('watch:all', function() {
	gulp.watch('['src/*.js', 'src/**/*.js']', ['build:js']);
});
```

Whenever any javascript file in the src folder changes, it will trigger the build:js task. That's the same task we wrote before. That task will concatenate all the javascript files together, minify them, and copy them to a single app.min.js file in the deployment directory. We could also set up watches for all the other file types we build, and trigger their build tasks as well:

```javascript
gulp.task('watch:all', function() {
	gulp.watch(['src/*.js', 'src/**/*.js'], ['build:js']);
	gulp.watch(['src/*.jade', 'src/**/*.jade'], ['build:html']);
	gulp.watch('src/**/*.less', ['build:css']);
});
```

Now, any change to any of our code files will result in them being rebuilt and redeployed, automatically, without any intervention from the developer. When watches are present, the gulp command line will not exit, it will continue to run, so that it can keep an eye on the target files for changes, and execute the tasks. Issuing a CTRL-C at that command line can be used to kill gulp.

### Web Serving

Another common role that gulp plays in an javascript project is hosting a web server for development purposes. With modern platforms like AngularJS, web sites are often reduced to nothing more than client side javascript, assets, and some REST calls to an API. Hosting such a site generally only requires a simple web server. There are a number of plugins for Gulp that provide such functionality. Remember that Gulp is build on top of Node.js, and Node.js IS a web server. 

There are two common web server plugins for Gulp. One is gulp-connect, the other is gulp-webserver. The first, gulp-connect, wraps another npm module for node called Connect. Connect is a simple web server wrapper around the built-in node web server features, adding support for things like live reload, and very easy reloading of the server. Adding to our scripts above, we may have a run-site.js script:

```javascript
var gulp = require('gulp');
var connect = require('gulp-connect');

gulp.task('run:deploy', function() {
	connect.server({
		root: './deploy',
		livereload: true
	});
});

gulp.task('reload:deploy', function() {
	return gulp
		.src(['./deploy/*.html'])
		.pipe(connect.reload());
});

gulp.task('watch:deploy', function() {
	gulp.watch(['./deploy/*.js', './deploy/*.html'], ['reload:deploy']);
})
```

With the above gulp tasks, we can add more to our gulpfile and have a full development web server with livereload technology:

```javascript
var gulp = require('gulp'),
	requireDir = require('require-dir');

// Bring in all tasks
requireDir('./gulp');

// Root tasks
gulp.task('build', ['build:js', 'build:html', 'build:css']);
gulp.task('run', ['build', 'run:deploy', 'watch:all', 'watch:deploy']);
gulp.task('default', ['build']);
```

With our updated gulpfile, we now have a run task, which builds and deploys, starts the web server, than kicks off our watchers to rebuild code and copy it to our deployment directory on any changes. We also watch the deploy directory to trigger the connect.reload() functionality on html files if the deployed files change. 

There are additional options with connect that we can set when we call connect.server(). The object passed in contains options that control the server, such as what host and port it runs at, whether to run in https, as well as the ability to connect "middleware". I won't go into middleware now, but suffice it to say, that gives you a lot of lower level power with a simple server like connect. More than is necessary for most web projects.

Another option for hosting a development server with gulp is gulp-webserver. This plugin is a bit simpler and more strait forward than connect, and has built in proxying support, which is often necessary with development hosting. Any time your site makes REST calls, if they call out to another host, you will have to deal with CORS issues. That complicates things, and requires the management of http headers that have no value for the released product. With proxying, you can route all requests through the same host name and port as the web site, and reroute those calls to the necessary host and server on the server side, which avoids the CORS issues.

Setting up gulp-webserver is also very easy:

```javascript
var gulp = require('gulp');
var webserver = require('gulp-webserver');

gulp.task('run:deploy', function() {
	return gulp
		.src('./deploy')
		.pipe(webserver({
			livereload: true,
			directoryListing: false,
			open: true,
			proxies: [
				{ 
					source: '/api',
					target: 'http://dev.server.com/api/'
				}
			]
		});
});
```

With gulp-webserver, the livereload functionality is built in, and it automatically watches the deploy directory for changes, and automatically reloads the necessary pages in the browser. Our gulpfile then becomes:

```javascript
var gulp = require('gulp'),
	requireDir = require('require-dir');

// Bring in all tasks
requireDir('./gulp');

// Root tasks
gulp.task('build', ['build:js', 'build:html', 'build:css']);
gulp.task('run', ['build', 'run:deploy', 'watch:all']);
gulp.task('default', ['build']);
```

Like connect, webserver supports middleware, which gives us a LOT of power, more than we usually need. The additional proxy capabilities are a key bonus of using webserver. It is also a bit easier to watch multiple directories with webserver than with connect, which is sometimes a necessity. 

### Plumbing

One of the shortcomings of Gulp is how it's pipeline handles errors. Out of the box, when an error occurs, the pipeline shuts down. That ends your script, suddenly and without any proper error handling. For regular build tasks, this isn't much of an issue. However when it comes to watchers, it can be a problem, as an error would kill your build script, and the waters would stop watching. With modern editors that use auto-save by default (i.e. WebStorm), even an incomplete edit of code could cause the script to end. 

A plugin called gulp-plumber can help us here. With plumber, errors in the pipeline will be reported, but the script will not end. This prevents watchers from killing the script and ending automatic updating and reloading of web sites with livereload. Plumber is very simple and easy to use. In our tasks, we simply need to plug in plumber first:

```javascript
var gulp = require('gulp'),
	plumber = require('gulp-plumber'),
	concat = require('gulp-concat'),
	uglify = require('gulp-uglify');

gulp.task('build:js', ['validate:js'], function() {
	return gulp
		.src(['src/*.js', 'src/**/*.js'])			// Source all js in ./src directory
		.pipe(plumber())							// Plumb the lines!
		.pipe(concat()) 							// Join all js into one (output single Vynil)
		.pipe(uglify()) 							// Obfuscate and minify .js file
		.pipe(gulp.dest('./deploy/app.min.js')); 	// Copy final .js to deployment directory
});
```

You would add plumber to all scripts that need to be executed off a watch. Once plumber is in place, you can safely edit code that is watched and automatically rebuild/redeployed without fear that a typo or incomplete edits will kill your running gulp script. 

### Cleaning the Slate

Another common task with builds performed by gulp is to clean the destination or deployment directory. This is an important task that is often overlooked, and overlooking it can result in stale code or stale npm or bower dependencies sticking around in your deployment directory. That can lead to odd bugs or other issues that can be difficult to track down. So a 'clean' task is often created to wipe out everything in the deployment directory before a build. This is most commonly performed with the 'del' plugin. This is not actually a standard gulp plugin, however because gulp tasks are just regular old javascript code, we can still use this plugin like so:

```javascript
var gulp = require('gulp'),
	del = require('del');

gulp.task('clean', function() {
	del.sync('./deploy');
});
```

The del package is a convenient and **safe** way to delete files. It is safe, because it will never delete the current directory or any parent directories...only child directories. That prevents you from accidentally nuking your project. The .sync version will synchronously delete, which is a useful option when calling it within gulp, as gulp tasks are run concurrently (which has the potential to allow the build taks to start before the clean task is complete.)

In our gulpfile, we might do the following:

```javascript
var gulp = require('gulp'),
	requireDir = require('require-dir');

// Bring in all tasks
requireDir('./gulp');

// Root tasks
gulp.task('build', ['clean', build:js', 'build:html', 'build:css']);
gulp.task('run', ['build', 'run:deploy', 'watch:all']);
gulp.task('default', ['build']);
```

### Stream Management

As gulp uses streams, and tries to be as concurrent as possible, there are times when you may need to have more control over how those streams are managed. In particular, streams guarantee no order, which can be a problem in some cases, or at the very least undesirable in some cases. There are a number of npm packages out there that support better management of streams, concatenation of streams, and ordering of contents through streams. These plugins include merge, merge2, ordered-merge-stream, and others. Both merge2 and ordered-merge-stream offer odering capabilities, and merge2 somewhat supports concatenation of streams together. 

For very simple ordered streaming, the ordered-merge-stream package is the best option. It's as simple as specifying the streams you want to merge in order in an array passed to the merge function:

```javascript
var gulp = require('gulp'),
	merge = require('ordered-merge-stream');

gulp.task('dummy:merge', function() {
	var stream1 = gulp.src(...).pipe(...);
	var stream2 = gulp.src(...).pipe(...).pipe(...);
	var stream3 = gulp.src([..., ...]).pipe(...);

	return merge([stream2, stream1, stream3])
		.pipe(...)
		.pipe(...)
		...
		.pipe(gulp.dest(...));
});
```

For more capable stream management, merge2 is probably the best option. 

### Web Dependency Deployment

With modern web apps and bower, we can easily install and keep track of our client side dependencies. Bower is a package manager, much like npm for node, that works for client side javascript. Bower is actually an npm package itself, and installed like so:

`npm install -g bower`

Once bower is installed, we can install components much the same was as with npm:

`bower init
bower install angular -save`

This will bring up the interactive bower config file generator with `bower init`, then install the angular package and save it as a dependency in the bower config. From that point on, we can simply use `bower install` and other bower commands to manage the config file and install, update, or uninstall packages. Once we are using bower to manage our client side dependencies, we can leverage another gulp plugin, main-bower-files, to manage the deployment of only the important files from each bower package. I say important files, as usually most bower packages include their own bower config files, often npm package.json files, minified and non-minified versions, and possibly other content. We usually do not need all that content for our deployed projects...we just need the 'main' files from the package to be deployed. 