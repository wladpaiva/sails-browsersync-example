# sails-browsersync-example

Example application with [Sails.js](http://sailsjs.org) + [Browsersync](https://www.browsersync.io)

#### References
* [Sails Tutorial](https://www.youtube.com/watch?v=uxojCaDSyZA)
* [Browsersync](https://www.browsersync.io)

## Run Instructions

#### Prerequisites
* git
* [nodejs](http://nodejs.org)
* [npm](http://npmjs.org)


```bash
git clone https://github.com/wlad/sails-browsersync-example.git
cd sails-browsersync-example
npm install
sails lift     # or node app.js
```

The browsersync will open a page using a different port from the sails process.
All changes on your `css` files will be reloaded by the browsersync page and all changes on your views will cause a page reload.
The only problem with this approach is that the changes on your `layout.ejs` won't cause the page reload once when you change a `*.css` or a `*.js` the `watch` task execute the `linkAssets` task, which means that if the `layout.ejs` have been included on autoreload, it would execute a page reload instead a simple css injection.

### Start from Scratch

1. Install Sails

   ```
   sudo npm -g install sails
   ```

2. Create new Sails project

   ```
   sails new sails-browsersync-example
   ```

3. Change directory to ```sails-browsersync-example```

4. Install the dependencies

   ```
   npm install sails-hook-autoreload connect-livereload grunt-browser-sync --save-dev
   ```

5. Create `tasks/config/browsersync.js`

   ```javascript
	module.exports = function (grunt) {

	  grunt.config.set('browserSync', {
	    dev: {
	      bsFiles: {
	        src: '.tmp/**/*'
	      },
	      options: {
	        // When your app also uses web sockets
	        // NOTE: requires 2.8.1 or above
	        proxy: {
	          target: "http://localhost:1337",
	          ws: true
	        },
	        watchTask: true,
	        reloadOnRestart: true
	      }
	    }
	  });

	  grunt.loadNpmTasks('grunt-browser-sync');
};

   ```

6. Edit ```tasks/config/watch.js``` to force the autoreload for the views changes

	```javascript
	module.exports = function(grunt) {

	  grunt.config.set('watch', {
	    assets: {
	      ...
	    },
	    views: {
	        files: ['views/**/*', '!views/layout.ejs'],
	        options: {
	          livereload: 54092
	        }
	    }
	  });

	  grunt.loadNpmTasks('grunt-contrib-watch');
	};
	```

7. Edit ```tasks/register/default.js``` and add the browsersync task before the watch task

	```javascript
	module.exports = function (grunt) {
	  grunt.registerTask('default', [
	    'compileAssets',
	    'linkAssets',
	    'browserSync',
	    'watch'
	  ]);
	};
	```

8. Edit ```config/http.js``` and add this middleware there to ensure that the autoreload will be inserted on the page
**PS:** Make sure you have added the ```connectLivereload``` into the order array.

	```javascript
	module.exports.http = {
	  middleware: {

	    connectLivereload: function (req, res, next) {
	      if (sails.config.environment !== 'production')
	        require('connect-livereload')({
	          port: 54092
	        })(req, res, next);
	      else return next();
	    },

	    order: [
	      'connectLivereload',
	      ...
	    ],
	    ...
	  },
	  ...
	};
	```


9. Enjoy it! :)
