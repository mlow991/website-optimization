# Website Optimization

## General Comments

* The folder "src" is the source folder that contains all un-minified CSS, javascript, and html files.
	* The index.html file in the source folder does not have inlined css for "above the fold" styles.  This is because this action is carried out by a grunt task utilizing the "grunt-critical" that ultimately puts the optimized file into the distribution folder.
	* The gruntfile and package.json are necessary for building the final files

* The folder "dist" contained all the optimized files produced by the grunt taskrunner.
	* grunt tasks included: grunt-critical, imagemin, responsive-images, htmlmin, cssmin, uglify
	* the "dist" folder is empty in this repository because the contents have been put in the main directory so they would be available for use in  githubpages


## Running Instructions

* All source files are located in the src folder
	* Within the src folder is a gruntfile.js and package.json
	* Use Grunt to build the proper node_modules folder
		* To install the dependencies needed to build the distribution version type "npm install" in the cmd line while in the root directory (the src folder)
	* Run the Grunt taskrunner by typing "grunt" in the cmd line while in the root directory
		* This will run the default grunt task which has all the tasks in proper order necessary for the final build
	* The published files will be created in the "dist" folder within the "src" folder
	* To have the published site viewable on github pages move all the contents of the "dist" folder outside the "src" folder and into the folder "%USERNAME%.github.io"
		* Where %USERNAME% is the git repository username that is hosting the page


## Optimizations made in views/js/main.js for pizza.html

* To increase the FPS to 60 for the moving pizzas the following changes were made:
	* To calls to .querySelector, .querySelectorAll
		* Replaced by some form of .getElements (.getElementsByClassName, .getElementById) depending on if it was a class or an ID
		* Done because the Web API call is faster than the former
		* If these calls were declared inside of a loop, then a variable was assigned to this call to the DOM and taken outside of the loop.  Within the loop the variable was replaced by the DOM call, effectively reducing the calls to the DOM.

	* To conditional statments of loops that access an unchanging variable
		* Declare a variable and assign it the value of that unchanging value
			* This is done outside the loop
		* The conditional statement now contains a local variable and does not have to contually measure a value each call to the method

	* To function updatePositions():
		* Phase variable moved out of loop and placed in a smaller loop that only assigns the 5 possible values for it (reduces uneeded calculations)
		* Document.body.scrollTop calculation assigned to a variable and taken out of the loop (this value only needs to be calculated once per update)
		* Offset variable added that takes into account the shift in pizzas after optimization
		* Items.style.left replaced by transform:translate3d (transform does not invoke layout or paint changes and is less expensive than left)
		* RequestAnimationFrame(updatePositions) added to end of update function for increased performance (allows the browser to choose when to animate frames)

	* To function called on DOMContentLoad
		* Decreased the amount of sliding pizzas when loading from 200 to 50 (trim the fat and eliminate exorbitant amount of pizza elements that would never bee rendered)
		* Assign all mover pizzas to the items array (so that this is only done once per load, instead of every update)
		* Declared serveral variables outside of the loop so that they would only be accessed once per call to document.addEventListener
			-variables: elem, movingPizzas

	* Added function containerSize
		* Takes into account media queries from CSS to determine the width of the container that holds the pizzas
		* Done to eliminate the need to access offsetWidth which invokes forced synchronous layout
		* If the media queries that control the width of the .container are changed then the values here need to be altered appropriately

	* Added function percentOrPixel
		* Reads the CSS width value and outputs a usable form of the value
		* Can deal with percent values as well as pixel values
		* Returns width of pizza div in pixels
		* Done to avoid the need to call offsetWidth which invokes forced synchronous layout

	* To function determineDx
		* Calls to offsetWidth are replaced with the newly created functions that do the same calculation (percentOrPixel and containerSize)
		* window.innerWidth fed to function containerSize (innerWidth gives the inner window value which corresponds to a media query that sets the width of the container class)

	* To changePizzaSizes
		* Variables dx and newwidth are taken out of the loop as they only need to be calculated once per size change
		* newwwidth calculated with percentOrPixel instead of offsetWidth to avoid forced synchronous layout
	
	* To css/style.css
		*  .randomPizzaContainer assigned a non-default width value of 100% to avoid trigger layout, painting, and compositing
		*  .mover class assigned value for "backface-visibilty: hidden" forcing each moving pizza to have its own composite layer, reducing paint time significantly