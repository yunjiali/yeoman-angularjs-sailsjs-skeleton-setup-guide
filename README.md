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

And add font-awesome into bower.json

* Delete main.css in under styles folder, and remove the <link> tag in index.html:

```<link rel="stylesheet" href="styles/main.css">```

### Specific to Color-Admin template
* Copy template_content_html/assets/less/*.* to styles folder in angularjs project. The ```style.less``` file is the root less file referencing to everything else.
* Remove bootstrap folder just copied under styles. This is because bootstrap less has already been installed as bower component
* Remove bootstrap reference in style.less
* In index.html, add ```<link href="styles/style.less" rel="stylesheet/less" type="text/css" />``` under ```<!-- build:css(.tmp) styles/main.css -->```

## Fonts

## HTML




