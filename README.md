## Wordpress JSON API Auth Controller

Authentication add-on controller for the Wordpress JSON API plugin utilizing the Wordpress cookie validation and generation.

*Wordpress JSON API Plugin: https://github.com/dphiffer/wp-json-api*

I have modified Matt Berg's work a little, so that all wordpress data requires authorization to access (not just posting):
- Moved controllers to their own directory as the json-api suggests.
- Created a version of the core controller in which each function checks that a cookie has been created.

HOW TO USE (very rough and incomplete --- ask if you want me to write something more):
(Please note, I've commented OUT the nonce-check for core functions, e.g. info() or get_recent_posts(). 
The auth checker is in core_auth.php. It might be that this is a bad, insecure idea. If so, than all requests will look
like (B) below, rather than (A) below.)

Ok, all this nonce-nonsense it very confusing. So, here's the basic logic of how to call the API:

(A) To get some data:
1. get a nonce TO GET A COOKIE, i.e. with controller = auth, method = generate_auth_cookie
2. 
2. get an authentication cookie using the nonce
3. request your data using the cookie.

For greater security, creating a post requires a nonce, so it works like this:
1. get a nonce TO GET A COOKIE, i.e. with controller = auth, method = generate_auth_cookie
2. get an authentication cookie using the nonce
3. get a nonce TO CREATE A POST, with controller = posts_auth, method=create_post
4. create the post using the nonce + cookie + params for the post

Of course, each step is another call-back function, right, since it can't start until the previous step succeeds.


EXAMPLES:
See the "examples" folder for Alex Baker's javascript example. Do NOT use this example as is, because you will be exposing a real username and password for the world to see! (Thank you, Alex).



WORDPRESS INSTALLATION:

JSON-API:
Install the standard json-api.

Install the custom json-api-controllers outside the json-api folder in plugins. Create "json-api-controllers" in the plugins folder, and put our custom controllers there. Add these into the system by adding to the function.php of the theme we are using, as shown in the example below.

In Wordpress, activate the core_auth controller, but NOT the core controller.


(From the readme: 
= External controllers =

It is recommended that custom controllers are kept outside of `json-api/controllers` in order to avoid accidental deletion during upgrades or site migrations. To make your controller visible from an external plugin or theme directory you will need to use two filters: `json_api_controllers` and `json_api_[controller]_controller_path`. Move the `hello.php` file from the steps above into your theme's directory. Then add the following to your theme's `functions.php` file (if your theme doesn't have a file called `functions.php` you can create one).)

				/**
				 * Add external controllers for json-api plugin
				 *
				 */
				
				
				// core_auth controller
				function add_core_auth_controller($controllers) {
					$controllers[] = 'core_auth';
					return $controllers;
				}
				add_filter('json_api_controllers', 'add_core_auth_controller');
				
				function set_core_auth_controller_path() {
					return WP_PLUGIN_DIR."/json-api-controllers/core_auth.php";
				}
				add_filter('json_api_core_auth_controller_path', 'set_core_auth_controller_path');
				
				
				// auth controller
				function add_auth_controller($controllers) {
					$controllers[] = 'auth';
					return $controllers;
				}
				add_filter('json_api_controllers', 'add_auth_controller');
				
				function set_auth_controller_path() {
					return WP_PLUGIN_DIR."/json-api-controllers/auth.php";
				}
				add_filter('json_api_auth_controller_path', 'set_auth_controller_path');
				
				// auth posts controller
				function add_posts_auth_controller($controllers) {
					$controllers[] = 'posts_auth';
					return $controllers;
				}
				add_filter('json_api_controllers', 'add_posts_auth_controller');
				
				function set_posts_auth_controller_path() {
					return WP_PLUGIN_DIR."/json-api-controllers/posts_auth.php";
				}
				add_filter('json_api_posts_auth_controller_path', 'set_posts_auth_controller_path');
