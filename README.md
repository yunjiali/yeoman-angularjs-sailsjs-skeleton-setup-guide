This guide helps to setup a angularjs+sailsjs application from scratch
# Pre install

You need to install the following libaraies or software
* Nodejs
* npm
* Sailsjs
* Yeoman
* Mysql database
## Some useful upgrade command:
* Update sailsjs
```sudo npm update -g sails```

#Init

## Initialise git
* Create two applications or folders named cient (for angularjs) and server (for sailsjs)
* git clone or git init both repository
* Copy .gitignore file for [client](https://github.com/yunjiali/yeoman-angularjs-sailsjs-skeleton-setup-guide/blob/master/gitignore-client) and [server](https://github.com/yunjiali/yeoman-angularjs-sailsjs-skeleton-setup-guide/blob/master/gitignore-server)

## Init Angularjs client side
* Go to client folder and type in:
```
yo angular [appname]
```
* Would you like to use Sass (with Compass): No
* Would you like to include Bootstrap: Yes
* Which modules would yo ulike to include: all
This may take a few minutes
* When you are asked which angularjs version you want to install, just choose the latest one
	  
* After the above steps are finished, please try:

```grunt serve```

To see whether the webpage can be started. 
* Sometimes, the imagemin.js may cause problem. We need to install imagemin again:

```npm install grunt-contrib-imagemin --save-dev```

### Specific to INSPINA template v2.3

* Install grunt less support if you want to use less instead of css:

```npm install grunt-contrib-less```

and add "grunt-contrib-less@x.x.x" into package.json

* Add the following packages to bower.json

```
	"font-awesome":"~4.4.0",
	"pace":"~1.0.2",   
    "jquery-ui": "~1.11.1",               
    "angular-bootstrap": "~0.12.0",
    "metisMenu": "~2.0.2",
    "slimScroll": "https://github.com/rochal/jQuery-slimScroll.git#~1.3.6"
```    
* Execute: ```bower install```

* In Angular_Seed_Project_Grunt, copy app/styles/ animation.css, style.css, patterns folder into project's app/styles
* Add the following lines under: ```<!-- build:css(.tmp) styles/main.css -->```

```
<link href="http://fonts.googleapis.com/css?family=Open+Sans:300,400,600,700" rel="stylesheet">
<link href="styles/animate.css" type="text/css" rel="stylesheet"/>
<link href="styles/style.css" type="text/css" rel="stylesheet"/>
```
* Copy In Angular_Seed_Project_Grunt, copy app/less folder to project's app/less

* Config grunt:
	* Add 'less' task after 'watch' task:
	
	```
	// Compile less to css
		less: {
	      development: {
	        options: {
	          compress: true,
	          optimization: 2
	        },
	        files: {
	          '<%= yeoman.app %>/styles/style.css': '<%= yeoman.app %>/less/style.less'
	        }
	      }
	    }
    ```
    * Replace old styles in watch task as:
    
    ```
    styles: {
        files: ['<%= yeoman.app %>/styles/{,*/}*.css','<%= yeoman.app %>/less/**/*.less'],
        tasks: ['less','newer:copy:styles', 'autoprefixer'],
        options: {
          nospawn: true,
          livereload: '<%= connect.options.livereload %>'
        }
      }
    ```
    * Add 'less' into grunt.registerTask('build'...) under grunt.task.run([])
	* Add 'less' into grunt.registerTask('test',[]);

### HTML
* In <html> add lang=en
* Add title in <head>
* Open Static_Seed_Project/index.html, copy the 
```<div id="wrapper">...</div>``` into project's index.html page
* Locate:

	```<div class="wrapper wrapper-content animated fadeInRight">```

	and change it to:

	```<div class="wrapper wrapper-content animated fadeInRight" ng-view=""></div>```

### Modify Controllers
* Add a new controller for navigation bar ```<nav>```:

```yo angular:controller nav```

* Add ```ng-controller="NavCtrl"``` to ```<nav>``` tag
* Add ```['$scope','$location',function($scope, $location){...}])``` to nav.js in app/scripts/controlers
* Add the following code to nav.js

```javascript
	$scope.isActive = function (viewLocation) {
      return viewLocation === $location.path();
    };
```

* Change the navbar list to:

```
		<li ng-class="{active:isActive('#/')}">
            <a href="#/"><i class="fa fa-th-large"></i> <span class="nav-label">Main view</span></a>
          </li>
          <li ng-class="{active:isActive('/about')}">
            <a href="#/about"><i class="fa fa-th-large"></i> <span class="nav-label">About</span> </a>
          </li>
```
This makes sure the selected nav bar item will be highlighted as active

* Find inspinia.js under Inspinia's folder Static_Seed_Project/js/inspinia.js, and copy all the code to project's app/js/vendor.js
* To make sure or angular-non-related javascript work, add ```<script src="js/vendor.js"></script>``` to index.html right after ```<!--bower:js--> ... <!--endbower-->``` before ```<!-- endbuild -->```


## Init sailsjs
TBD



# Some deprecated content
## Add grunt-less support
* Install grunt less support if you want to use less instead of css:

```npm install grunt-contrib-less```

and add "grunt-contrib-less@x.x.x" into package.json

* (Optional) See [this link](https://github.com/twbs/bootstrap/issues/16663) about an issue in bootstrap's bower. The new version of bootstrap's bower may miss the bootstrap css file in "main". So...
* Add less task in Gruntfile.js:
	* Add the following code into watch:{}
	```
		less:{
        files:['<%=yeoman.app%>/less/**/*.less'],
        tasks:['less']
      }
  ```
  	* Add the following code under grunt.initConfig after watch:{}
  	```
  	less: {
      dist: {
        files: {
          '<%= yeoman.app %>/styles/main.css': ['<%= yeoman.app %>/less/style.less']
        },
        options: {
          sourceMap: true,
          sourceMapFilename: '<%= yeoman.app %>/styles/main.css.map',
          sourceMapBasepath: '<%= yeoman.app %>/',
          sourceMapRootpath: '/'
        }
      }
    },
    ```
	* add 'less' into concurrent:{dist:[]}
	* add 'less' into grunt.registerTask('serve'...) under grunt.task.run([])
	* add 'less' into grunt.registerTask('test',[]);




