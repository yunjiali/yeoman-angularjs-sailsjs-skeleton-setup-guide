<strong>This guide helps to setup a angularjs+sailsjs application from scratch</strong>
# Pre install

You need to install the following libaraies or software

* Nodejs
* npm
* Sailsjs
* Mysql database
* Redis (for shared sessions in production environment)
* Nginx latter version with support for socket.io

## Server setup in Ubuntu 14.04
### Pre-setup phrase
* Create ssh key and add new users follow [this tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
* Add a new user, for example ```demo``` and choose a password: ```adduser demo```
* Add your new user to sudo group: ```gpasswd -a demo sudo```
* Generate an ssh key: ```ssh-keygen``` and enter the file name you want to save the ssh key
* Add the key to newly created user ```demo```
	* In your machine, execute ```cat ~/.ssh/yourkeyname``` to get the string of the key
	* In the new VM, execute ```su - demo```
	* ```mkdir .ssh```
	* ```chmod 700 .ssh```
	* ```nano .ssh/authorized_keys```
	* copy and paste the string and save the file
	* ```chmod 600 .ssh/authorized_keys```
	* ```exit```
* Disallow the remote root login:
	* ```nano /etc/ssh/sshd_config```
	* Find ```PermitRootLogin yes``` and change it to ```no```
	* Save and exit
* Restart ssh service ```service ssh restart``` as root user

### Install software

#### Nodejs
* Suggest to follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server) to use nvm to install nodejs. In Ubuntu, nodejs' commandline tool is clashed with another node package, so sometimes, you need to use ```nodejs``` instead of ```node```
* Upgrade npm, because Ubuntu may have a legacy version: ```sudo npm install -g npm```
* Set the default nodejs version using nvm:

```nvm alias default 0.10.25```

<strong>Without setting the default version, the nodejs won't start when VM reboot. </strong>

* Install sailsjs, mocha and pm2 (optionally we can also start app using forever)

```
	sudo npm install sails -g
	sudo npm install pm2 -g
	sudo npm install forever -g
	sudo npm install mocha -g
```

* If ```sudo npm install``` throws ```/usr/bin/env: node: No such file or directory```, you can link the node you use to the sudo environment with the following command:

```sudo ln -s /home/demoapps/.nvm/v0.10.25/bin/node /usr/bin/node```

* For mocha testing, you can run:
* If you want to run just some test cases, run:

 ```NODE_ENV=test mocha test/bootstrap.test.js test/integration/**/*.test.js```

#### Install mysql
* ```sudo apt-get install mysql-server```
* Use ```mysql -u root -p``` to check if mysql is running.
* Change max_allowed_packet: sometimes, we need to write large amount of data into database, so we need to change this attribute to a larger number, 512MB for example:

	* Use ```mysql --help --verbose``` to list the possible positions for ```my.cnf``` (Look at the long text, it's somewhere in the middle). In Ubuntu, it will be ```/etc/mysql/my.cnf```
	* Locate the ```[mysqld]``` section and ```max_allowed_packet``` attribute, change it to 512MB
	* Restart mysql ```sudo /etc/init.d/mysql restart```



#### Install Redis
* Have a look at [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-a-redis-cluster-on-ubuntu-14-04)
* Execute ```sudo add-apt-repository ppa:chris-lea/redis-server```
* Then ```sudo apt-get update```
* Then ```sudo apt-get install redis-server```
* NOTE! We haven't setup password yet
* To test redis server is working, type ```redis-cli``` and then in the redis cli type ```ping```, you should get a ```PONG``` for response.


#### Install git
* ```sudo apt-get install git```
* You may need to setup ssh key to access your github repository
* Clone the repository ```git clone https://.....```

## Setup timezone
Sometimes the ubuntu server will be in another time zone. You can check the time using ```date``` command. But if the timezone is wrong, then you need to correct it. See [this tutorial](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)

* Execute ```sudo dpkg-reconfigure tzdata```
* Choose Europe->London
* Then run ```date``` again to see if the time is correct

## Some useful upgrade command:
* Update sailsjs
```sudo npm update -g sails```

## Purchasing SSL (123 Reg example)
There are many places you can buy SSL certificate and they are many types of certificate. Here we demonstrate how to purchase a wildcard ssl from 123 Reg and make it ready for install on nginx server.

* Purchase it (here)[https://www.123-reg.co.uk/ssl-certificates/wildcard-ssl-certificates.shtml]
* Generate the csr file using openssl. [Here is the tutorial](https://www.123-reg.co.uk/support/answers/SSL-Certificates/Generate-a-CSR/generate-a-csr-apache-open-ssl-634/)
* ```openssl genrsa -des3 -out www.mydomain.com.key 2048``` and enter your password
* ```openssl req -new -key www.mydomain.com.key -out 
* www.mydomain.com.csr``` **The Common Name should be *.example.com if it's a wildcard certificate**
* Verify your CSR: ```openssl req -noout -text -in www.mydomain.com.csr```
* Then, login to 123 reg and assign the certificate to a domain. Be sure to write ```*.example.com``` in the ```Domain``` field. And copy everything in the csr file, including the ```REQUESt BEGIN``` and ```REQUEST END``` lines.
* **Make sure you have setup one of the required emails to receive the certificate**. [See this for the list of possible emails](https://www.123-reg.co.uk/support/answers/SSL-Certificates/Installing-your-SSL/how-do-i-apply-an-ssl-certificate-4855/). 
* Then you will receive a email to accept the certificate.
* After that, you will receive the certificate in an email. However, that is not the certificate you are going to apply to nginx.
* Make sure you save both the certificate in the email and an intermediary certificate (which is public) in separate files.
* Copy the content in the intermediary certificate into your domain certificate file, and make sure the ```REQUEST BEGINS``` of the intermediary certificate is undernethe ```REQUEST END``` of the domain certificate.
* Copy the crt file after merge and the private key file (.key) to your vm, then you are ready to setup nginx for https.


#Init

## Initialise git
* Create two applications or folders named cient (for angularjs) and server (for sailsjs)
* git clone or git init both repository
* Copy .gitignore file for [client](https://github.com/yunjiali/yeoman-angularjs-sailsjs-skeleton-setup-guide/blob/master/gitignore-client) and [server](https://github.com/yunjiali/yeoman-angularjs-sailsjs-skeleton-setup-guide/blob/master/gitignore-server)
* Sometimes we have some folders to save temporary files. If you just want to keep the folder and ignore all the files in it, create a .gitignore file in that folder and add the following code:

```
	*
	!.gitignore
```	

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
* In the bower.json, change the yeoman default angularjs version indicator: ```^1.3.0``` to something more specific, such as ```~1.4.5```, this will avoid other bower package modifying the angularjs or it's related libraries' version (angular-animate, for example) and cause verion confliction problem.
* Remove the jshint javascript grammar check in watch/js:

```
	tasks: ['newer:jshint:all']
```
	  
* After the above steps are finished, please try:

```grunt serve```

(Optional) To see whether the webpage can be started. 
* Sometimes, the imagemin.js may cause problem. We need to install imagemin again:

```npm install grunt-contrib-imagemin --save-dev```

(Optional) if you want to disable search engine index your website, in app/robot.txt just add:

```
	User-agent: *
	Disallow: /
```

### Specific to INSPINA template v2.3

* Install grunt less support if you want to use less instead of css:

```npm install grunt-contrib-less```

and add "grunt-contrib-less@x.x.x" into package.json

* Add the following packages to bower.json

```
	"font-awesome":"~4.3.0",
	"pace":"~1.0.2",   
    "jquery-ui": "~1.11.1",               
    "angular-bootstrap": "~0.12.0",
    "metisMenu": "~2.0.2",
    "slimScroll": "https://github.com/rochal/jQuery-slimScroll.git#~1.3.6"
```    
<strong>Because of bower problem, we current use font-awesome 4.3.0. From fa 4.4.0, less is used, but grunt won't add it into the index.html automatically.</strong>
* Font-awesome use font to generate icons, but the fonts won't copy autoamtically to dist folder when we ```grunt:build```. To solve this problem, we need to add the following code to ```copy.dist.fils[]```

```
	{
          //for font-awesome
          expand: true,
          dot: true,
          cwd: 'bower_components/font-awesome',
          src: ['fonts/*.*'],
          dest: '<%= yeoman.dist %>'
        }
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

### Add http inspector
This function help us to handle http errors and display the right information.

* Execute ```yo angular:factory httpInspector```
* Add the following code after ```angular.module().```

```javascript
	.factory('httpInterceptor', ['$q', '$rootScope', '$location', '$filter',
  function ($q, $rootScope, $location, $filter) {
    //handle error messages
    //handle policy problem
    var $translate = $filter('translate');
    return {
      request:function(config){

        return config;
      },

      response: function (response) {
        // do something on success
        //console.log('success');
        //console.log('status', response.status);
        //return response;
        return response || $q.when(response);
      },

      responseError: function (response) {
        // do something on error

        //console.log('failure');
        //console.log('status', response.status)
        //return response;
        if(response.status === 0 || response.status === 503){
          //do something
        }
        else if(response.status === 400){
          //do something
        }

        return $q.reject(response);
      }
    }
}]);
```

### Add environment variables

Most of the time, we need to deploy angularjs app in different environment, such as development, test, production, staging, etc. For each environment, we will point to a different endpoint for http request, for example. So we need to setup the environment variables so that we can use ```grunt build:env``` to automatically persist the variables into the minified code. To do this, we need to install grunt-ng-constant package:

* Execute: ```npm install grunt-ng-constant --save-dev```. If we use --save-dev we don't need to add ```"grunt-ng-constant":"~1.1.0"``` to package.json
* In the initConfig section of Gruntfile.js, add the following code:

```javascript
	ngconstant: {
      options: {
        name: 'config',
        wrap: '"use strict";\n\n{%= __ngModule %}',
        space: '  '
      },
      development: {
        options: {
          dest: '<%= yeoman.app %>/scripts/config.js'
        },
        constants: {
          ENV: {
            name: 'development',
            apiEndpoint:'http://localhost:1337',
          }
        }
      },
      production: {
        options: {
          //should be yeoman.app here instead of yeoman.dist, or the new config.js won't be written to scripts.js'
          dest: '<%= yeoman.app %>/scripts/config.js'
        },
        constants: {
          ENV: {
            name:'production',
            apiEndpoint:'http://somedomain.demoapps.me',
          }
        }
      },
      //Other envrionments...
    },
```
* Add ```'ngconstant:development'``` to the ```serve``` task of grunt.task.run
* Add ```'ngconstant:'+target``` to the ```build``` task of grunt.task.run. Add a wrapper to build task if necessary:

```
	grunt.registerTask('build', 'Build for a specific server', function(target) {

    if (!target) {
      target = "production";
    }
    
    grunt.task.run
```

<strong>Please remember, if you add ```if(!target)...``` don't forget to add change ```grunt.registerTask``` to ```grunt.task.run``` after the ```if```. Something like this:</strong>

```
	grunt.task.run([
      'clean:dist',
      'less',
      'ngconstant:' + target,
      'wiredep',
      'useminPrepare',
      'concurrent:dist',
      'autoprefixer',
      'concat',
      'ngAnnotate',
      'copy:dist',
      'cdnify',
      'cssmin',
      'uglify',
      'filerev',
      'usemin',
      'htmlmin'
    ]);
```

* Add ```<script src="scripts/config.js"></script>``` into ```<!-- build:js({.tmp,app}) scripts/scripts.js -->``` section in index.html
<strong>NOTE!Please don't write as ```<script src="scripts/config.js"/>```</strong>
 
* Then inject config in our app: ```var app = angular.module('myApp', [ 'config' ]);```

* The ```ENV``` is the variable we can inject to controlers and we can access the env variable by ```ENV.apiEndpoint```

### Change name of the app
If you want to change the name of the app, you can refer to [this thread] (http://stackoverflow.com/questions/17168131/yeoman-yo-where-does-yo-scaffolding-pickup-the-name-of-the-angular-module)

Generally speaking, you need to:

* Rename the appname in bower.json and package.json
* Use WebStorm to find and place (Command+Shift+R) the string in the whole project ```oldClientApp``` with the new module name

### Automatic deployment using Grunt
TODO

## Init sailsjs
* Download the sailjs template from this [repository](https://github.com/yunjiali/sails-waterlock-jwt-template)
* Unzip the file and copy all the files to ```server``` folder
* Change application's name and version from package.json
* Go to ```server``` folder and execute:

```sudo npm install```

* After all dependencies are installed, try ```sails lift``` to see if the server can be successfully lifted.

### Change database
* Find connections.js under ```config``` folder
* Add or modify the mysql/mongodb connections
* Check ```development.js```, ```production.js``` and other configuration files in ```config/env``` folder and change the connection property in models to your newly defined connections.
* Login to mysql and create the related database and grant privilledge to the databases ```grant all on myapp_dev.* to 'test'@'localhost' identified by 'test';```

### Set CORS
* In config/cors.js allow ```allRoutes: true```, ```origin:'*'```

### Set Deployment configuration
* Look at ```config/env/production.js``` and change the port to the port that won't clash with other apps on the VM
* Also change database connections and log if necessary.
* Add/Change the session settings:
```session:{
    secret: 'find it in session.js',
    cookie: {
      maxAge: 30 * 24 * 60 * 60 * 1000
    },
    key:'yourapp',
    adapter: 'redis',
    prefix:'someprefix'
  }```

### Set Socket.io
* TODO: production use redis, change production.js
* ```npm install socket.io-redis --save```


### Remove default pages
If you want to copy client app using ```grunt build```, then you need to remove the default pages settings in ```config/routes.js```:

```
	//'/': {
  //  view: 'homepage'
  //}
```

## DEPRECATED Add login function 

### Client: Add Satellizer and authenticationService
We will use [Satellizer](https://github.com/sahat/satellizer) plus a authentication service with localStorage to enable the login function.

* Install satellizer:
```bower install satellizer --save-dev```

* Install angular-local-storage

```bower install angular-local-storage```

* Add ```LocalStorageModule``` into app.js
* Add ```satellizer``` into ```app.js```
* Add ```$authProvider``` and ```ENV``` into the app.config parameters
* Add the following code into config:

```
	$authProvider.baseUrl = ENV.apiEndpoint;
   $authProvider.loginUrl = '/auth/login';
   $authProvider.signupUrl = '/user/create';
   $authProvider.tokenPrefix = 'change to your appname';
```

For more details of the $authProvider configuration, please refer to [here](https://github.com/sahat/satellizer#configuration)

* In the terminal, using yo to add some views, controllers and services:

```
	yo angular:view login
	yo angular:controller login
	yo angular:controller header
	yo angular:factory authenticationService
```

* To control the views on clientside, we also need to create a policyService to control the view permissions for each page:

```yo angular:factory policyService```

* For authenticationService, we define the basic functions and storage for user authentication. Copy and paste the following code in to authenticationService.js:

```
	.factory('authenticationService', ["$http","$q", '$auth','ENV', 'localStorageService', function ($http, $q, $auth, ENV, localStorageService) {
    var userInfo;

    function login(email, password) {
      return $q(function(resolve, reject){
        $auth.login({email:email, password:password})
          .then(function (result) {
            console.log(result);
            userInfo = {
              username: result.data.user.username,
              firstname: result.data.user.firstname,
              lastname:result.data.user.lastname,
              email:result.data.user.email,
              role:result.data.user.role,
              id:result.data.user.id
            };

            localStorageService.set('userInfo', userInfo);
            resolve(userInfo);

          })
          .catch(function (error) {
            reject(error);
          });
      });
    }

    function logout() {
      return $q(function(resolve,reject){
        $auth.logout().then(function (result) {
          return $http({
            method: 'POST',
            url: ENV.apiEndpoint+'/auth/logout'
          });
        })
        .then(function(result){
            userInfo = null;
            localStorageService.set('userInfo', null);
            resolve(result);
        })
        .catch(function (error) {
            reject(error);
        });
      });
    }

    function getUserInfo() {
      return userInfo;
    }

    function getAccessToken() {
      return $auth.getToken();
    }

    function isLoggedIn() {
      //console.log(userInfo.tokenExpires);
      //console.log(Date.now());
      return $auth.isAuthenticated();
    }

    function init() {
      if (localStorageService.get('userInfo')) {
        userInfo = localStorageService.get('userInfo');
      }
    }
    init();

    return {
      login: login,
      logout: logout,
      getUserInfo: getUserInfo,
      getAccessToken: getAccessToken,
      isLoggedIn: isLoggedIn
    };
  }]);
```

* For policyService, copy and paste the following code:

```
	.factory('policyService', ['$location', '$q', 'authenticationService', function ($location, $q, authenticationService) {

    //require user login
    var loginRequired = function(){
      var deferred = $q.defer();

      if(!authenticationService.isLoggedIn()) {
        deferred.reject({ authenticated: false })
        $location.path('/login');
      } else {
        deferred.resolve()
      }

      return deferred.promise;
    }

    //user not logged in
    var notLoggedIn = function(){
      var deferred = $q.defer();

      if(!authenticationService.isLoggedIn()){
        deferred.resolve();
      }
      else{
        deferred.reject({authenticated: true });
        $location.path('/');
      }
    }

    var redirectIfAuthenticated = function(route){
      var deferred = $q.defer();

      if (authenticationService.isLoggedIn()) {
        deferred.reject({ authenticated: false });
        $location.path(route);
      } else {
        deferred.resolve()
      }

      return deferred.promise;
    }

    var isSameUser = function(userId){
      var deferred = $q.defer();
      var userInfo = authenticationService.getUserInfo();
      if(!userInfo) {
        deferred.reject({authenticated: false});
        $location.path('/');
      }

      if(userInfo.id !== parseInt(userId)) {
        deferred.reject({isSameUser: false});
        $location.path('/');
      }
      else
        deferred.resolve();

      return deferred.promise;
    }

    // Public API here
    return {
      loginRequired: loginRequired,
      notLoggedIn:  notLoggedIn,
      isSameUser: isSameUser,
      redirectIfAuthenticated: redirectIfAuthenticated
    };
  }]);
```

* Add login controller and template in app.js:

```
.when('/login', {
        templateUrl: 'views/login.html',
        controller: 'LoginCtrl',
        resolve:{
          notLoggedIn:function(policyService){
            return policyService.notLoggedIn();
          }
        }
      })
```

* Go back to ```app.js``` and add the resolver to each path, for example, login required:

```
resolve:{
          loginRequired:function(policyService){
            return policyService.loginRequired();
          }
        }
```

* Now we need to design the login page

```
<div class="row">
  <div class="col-md-offset-3 col-md-6 col-md-offset-3">
    <div class="ibox-content">
      <form class="m-t" ng-submit="login()" role="form" name="loginForm">
        <div class="form-group">
          <input type="email" class="form-control" ng-model="user.email" placeholder="Email" required autofocus>
        </div>
        <div class="form-group">
          <input type="password" class="form-control" ng-model="user.password" placeholder="Password" required>
        </div>
        <button type="submit" class="btn btn-primary block full-width m-b" ng-disabled="loginForm.$invalid">Login</button>
        <a href="#">
          <small>Forgot password?</small>
        </a>
        <p class="text-muted text-center">
          <small>Do not have an account?</small>
        </p>
        <a class="btn btn-sm btn-white btn-block" href="register.html">Create an account</a>
      </form>
    </div>
  </div>
</div>
```
**NOTE:Remove the create an account option if you don't want to have this function.** 

* In login.js (login controller), add the following code:

```
.controller('LoginCtrl', ['$scope','authenticationService','$location','toastr',function ($scope, authenticationService,$location, toastr) {
    $scope.user;
    $scope.login = function(){
      authenticationService.login($scope.user.email, $scope.user.password)
        .then(function(result) {
          toastr.info('You have successfully signed in!');
          $location.path('/');
        })
        .catch(function(error) {
          toastr.error(error, error.status);
        });
    }
  }]);
```
Here, login() function refers to the loginForm in login.html

* We need to change the index.html in order to hide and show the navbar items and buttons on header according to the login status.

	* Firstly, find the 'nav' tag with 'navbar-static-top' class and add ```ng-controller="HeaderCtrl"```
	* In header.js (HeaderCtrl) add the following code in order to provider some authentication services for header elements:

	```
	.controller('HeaderCtrl', ['$scope','$location','authenticationService', function ($scope,$location, authenticationService) {
    $scope.isLoggedIn = function(){
      return authenticationService.isLoggedIn();
    };

    $scope.logout = function(){
      authenticationService.logout();
      $location.path('/');
    };

    $scope.getUserInfo = function(){
      console.log(authenticationService.getUserInfo());
      return authenticationService.getUserInfo();
    }
  }]);
  ```
  
  * Add some information on navbar header:
  
  ```
  <ul class="nav navbar-top-links navbar-right">
            <li ng-if="isLoggedIn()">
              <span class="m-r-sm text-muted welcome-message">Welcome {{getUserInfo().username}}</span>
            </li>
            <li ng-if="isLoggedIn()">
              <a class="count-info" href="/messages" aria-haspopup="true" aria-expanded="false">
                <i class="fa fa-envelope"></i> <span class="label label-warning">16</span>
              </a>
            </li>
            <li ng-if="isLoggedIn()">
              <a ng-click="logout()">
                <i class="fa fa-sign-out"></i>Log out
              </a>
            </li>
            <li ng-if="!isLoggedIn()">
              <a href="#/login">
                <i class="fa fa-sign-in"></i> Log in
              </a>
            </li>
          </ul>
  ```
	* You can also use ```ng-if``` to hide some items from left nav bar if you like.

* By default, we have the following user in database:

```
	email: abc5@gmail.com
	password: hellowaterlock
```	


### Server: Enable Satellizer JWT login
[Satellizer](https://github.com/sahat/satellizer) is an authentication model designed for Angularjs. It has native support for jwt, but we just need to config the server side to make sure the login is success.

* In cors.js enable:

```credientials: true```
```headers:'origin, Content-Type, Authorization, accept'```

* In waterlock.js, change:

```
	jsonWebTokens:{

    // CHANGE THIS SECRET
    secret: 'change this secret',
    expiry:{
      unit: 'days',
      length: '7'
    },
    audience: 'change audience',
    subject: 'subject'
    ....
```

* Now after the client and server are all setup, you can test the login function. 

## Add reset password function
The waterlock has a build-in function to reset password through email. You need the following configurations before using it:

### Waterlock.js
***Firstly make sure that you have installed the latest waterlock and waterlock-local-auth model. The old version has problems to send reset email with old version of nodemailer.***

* In ```authMethod```, you need to set the ```tokens``` to ```true``` under waterlock-local-auth
* Then you need to select a mail service provider. You can use dummy mail server (i.e. remove all the seetings in ```mail:{}```, however, it's quite likely the email can't be sent because you are in dynamic IP.
* [Here] (https://github.com/waterlock/waterlock-local-auth/issues/7) is the link to explain the reset password workflow. And [here](http://stackoverflow.com/questions/35260067/sails-waterlock-password-reset-flow/35543141#35543141) is my explanation. 
* In short, you have to click the link in the reset password email first and then ```forwardUrl``` will point you to a form to input new password. The form will post to ```/auth/reset?password=newpassword```.


```
	{
      name:'waterlock-local-auth',
      passwordReset:{
        tokens: true,
        mail: {
          options:{
            service: 'SendGrid',
            //direct:true,
            auth: {
              user: 'myuser',
              pass: 'mypass'
            }
          },
          from: 'admin@somedomain.org',
          subject: 'Your Synote password reset!',
          forwardUrl: 'http://localhost:9000/#/password'
        },
        template:{
          file: '../views/email.jade',
          vars:{}
        }
      },
      createOnNotFound: false
    }

```

I use [SendGrid] (http://app.sendgrid.com) as it can hide the actual email address sending the reset password email. You can also use gmail and hotmail.

### Client side
You need to programme on the client-side to proivde:

* A programme to post to ```/auth/reset?email=emailaddress```
* A page containing reset password form
* A programme to post the reset password form to ```/auth/reset?password=newpassword```

# Deployment
* Login to your vm and git clone the repository you want to deploy. You can only clone the server side sailsjs code if necessary.
* Clone the repository can benefit us when we update the code on development machine and we don't need to copy all the code to deployment machine. We just need to use ```git pull``` to update the code. <strong>You need to make sure that the dependencies in node_modules do not change. Or you need to update the modules after git pull.</strong>
* In the cloned server app directory, execute ```npm install```
* Remove all the files in assets folder for the server side app
* Use ```grunt build:sometarget``` to build your client side app on your development machine.
* Copy all the files in ```dist``` to sails assets folder if you want to serve everything from there. Otherwise, you can copy them to nginx website folder if you want to separate the client website with server website.
* NOTE! If you copy everything to ```assets``` folder, you need to restart the sails service ```pm2 restart projectname``` to see the changes. The assets folder is not like nginx static htmls.

## Setup Nginx
This [tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-an-node-js-app-using-sails-js-on-an-ubuntu-vps) really help

* If you haven't installed nginx, use ```sudo apt-get install nginx```
* To create a new site: ```sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com```
* ```sudo nano /etc/nginx/sites-available/example.com```
* IF YOU PLAN TO HOST SAILSJS SEPARATELY FROM CLIENT: Add the following code:

```
	server {
        listen   80; ## listen for ipv4; this line is default and implied
        #listen   [::]:80 default ipv6only=on; ## listen for ipv6

        # Make site accessible from http://localhost/
        server_name example.com;
        
        location / {
    		proxy_pass         http://localhost:1337/;
    		proxy_redirect     off;
    		proxy_set_header   Host             $host;    			proxy_set_header   X-Real-IP        $remote_addr;
    		proxy_set_header   X-Forwarded-For  			$proxy_add_x_forwarded_for;
		}
}
```
* IF YOU PLAN TO COPY CLIENT TO SAILS' ASSETS FOLDER ADD THE FOLLOWING CODE:

```
		server {
	        listen   80; ## listen for ipv4; this line is default and implied
	        #listen   [::]:80 default ipv6only=on; ## listen for ipv6
	
	        # Make site accessible from http://localhost/
	        server_name example.com;

			location / {
				   proxy_pass http://localhost:1400;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
          }
      }
```

<strong>PLEASE NOTE: the proxy_pass should be localhost instead of 127.0.0.1</strong>

* If you want to setup https WITH http (allow both of them), use the followin configuration (see [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04)):

```
		server {
	        listen   80; ## listen for ipv4; this line is default and implied
	        listen 	443 ssl;
	        #listen   [::]:80 default ipv6only=on; ## listen for ipv6
	
	        # Make site accessible from http://localhost/
	        server_name example.com;
			ssl_certificate /etc/nginx/ssl/yourcert.crt;
        	ssl_certificate_key /etc/nginx/ssl/yourkey.key;
        	
			location / {
				   proxy_pass http://localhost:1400;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
          }
      }
```
* If you want to setup https and redirect http to https (allow only https), use the followin configuration (see [this answer use 301 redirect](http://serverfault.com/questions/250476/how-to-force-or-redirect-to-ssl-in-nginx)):
```
		server {
		    listen      80;
		    server_name example.com;
		    return 301 https://example.com$request_uri;
		}
		
		server {
	        listen 	443 ssl;
	        #listen   [::]:80 default ipv6only=on; ## listen for ipv6
	
	        # Make site accessible from http://localhost/
	        server_name example.com;
			ssl_certificate /etc/nginx/ssl/yourcert.crt;
        	ssl_certificate_key /etc/nginx/ssl/yourkey.key;
        	
			location / {
				   proxy_pass http://localhost:1400;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
          }
      }
```

* After saving the configuration file, add the new configuration to sites-enabled: ```sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com``` *BE CAREFUL!!! You have to use the full path here, not the relative path!* See this http://stackoverflow.com/questions/36366991/cannot-connect-to-aws-ec2-via-nginx-from-local-machine
* Remove the default website ```sudo rm /etc/nginx/sites-enabled/default```
* Restart Nginx ```sudo service nginx restart```
* (Optional) If anything fails, check the error log ```/var/log/nginx/nginx_error.log```

### Add username password to nginx server
* TODO: follow this tutorial https://www.digitalocean.com/community/tutorials/how-to-set-up-http-authentication-with-nginx-on-ubuntu-12-10
pm2 
## Start the app
* Use pm2 ```NODE_ENV=sometarget pm2 start app.js --name your_project_name --log-date-format "YYYY-MM-DD HH:mm:ss Z"```
* Or forever ```NODE_ENV=sometarget forever start app.js```

## Deploy new code
* We suggest to use git to keep sync with the development code. Everytime you want to update code, use ```git pull```
* From pm2, if you want to load the new code, just use ```pm2 restart yourproject```

## About log
It's better if we use rotate log:

```pm2 install pm2-logrotate```

This will automatically generate rotated log when you start pm2

#FAQ
## Server
## Client
### Angularjs version problem
I got this error on the angularjs webpage: ```Uncaught Error: [$injector:unpr] Unknown provider: $$asyncCallbackProvider <- $$asyncCallback <- $animate <- $compile```

This is usually because the version of angular-animate module is different from angularjs version. Use Check the bower.json file for angular-animate and use ```bower install angular-animate``` to get the correct version.

### Grunt ng-annotate version problem
I got this error when I run ```grunt build:sometarget```

```
Running "ngAnnotate:dist" (ngAnnotate) task
Warning: Cannot assign to read only property '$methodName' of false Use --force to continue.
```

This is because the grunt-ng-annotate model is too old. You need to change the version number in package.json and npm install the latest version.

### Grunt build stuck
Sometimes, the application build process will stuck in:

```
Running "uglify:generated" (uglify) task
File dist/scripts/oldieshim.js created: 118.72 kB → 30.09 kB
```

Just wait a little bit. It would be fine. I don't know why it takes so long sometimes.

### The minimised button and some other features depending on INSPINIA template js file doesn't work

This is because some of the javascript function sin INSPINIA is defined in ```js/vendor.js``` and this file by default is not included in the grunt build. So we need to include it in index.html. Write the following code after ```endbower``` and ```endbuild```:

```
<!-- endbower -->
    <!-- endbuild -->

    <!-- build:js({.tmp,app}) scripts/main.js -->
    <script src="js/vendor.js"></script>
    <!-- endbuild -->
```

###aProvider injection error after grunt minify

In the deployment server, you may get this error:

```Error: [$injector:unpr] Unknown provider: aProvider <- a```

But it never happens on development environment.

The problem is that the minified programme change the name of the variables, which cannot be recognised by angularjs. Check the following:

* In ```app.js```, anything in ```app.run```,```app.config```, etc. are wrapped as arrays instead of simply ```{attribute:function(variable){}}```, this will cause problem. Change it to ```{attribute:['variable', function(varible){}]```
* Check the same for all the services and controllers.
