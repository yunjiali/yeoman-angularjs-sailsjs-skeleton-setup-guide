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

## Init sailsjs
TBD

# Config Angularjs
This section will introduce how to apply a template (color-admin) to the skeleton angularjs project
## Javascript
## CSS
* install font-awesome
```bower install font-awesome```
And add font-awesome into bower.json:
```"font-awesome":"~x.x.x"```
## Fonts




