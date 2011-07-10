=== Custom Metadata Manager ===
Contributors: batmoo
Donate link: http://digitalize.ca/donate
Tags: custom metadata, custom metadata manager metadata, postmeta, post meta, user meta, custom post types, custom fields
Requires at least: 3.0
Tested up to: 3.2
Stable tag: 0.3

An easy way to add custom fields to your object types (post, pages, custom post types, users)

== Description ==

An easy way to add custom fields to your object types (post, pages, custom post types, users).

The goal of this plugin is to help you rapidly build familiar, intuitive interfaces for your users in a very WordPress-native way. 

The custom field panel is nice, but not quite the easiest thing for users to work with. Adding your own metaboxes and fields involves a lot of time and repetitive code that could be better used elsewhere.

This plugin handles all that heavy-lifting for you behind-the-scenes, so that you can focus on more on building out and connecting your data rather than all the minor details. This single piece of code `x_add_metadata_field( 'my-field-name', 'post' );` generates a metabox with a text field inside it, with the necessary hooks to save the entered values.

The API is similar to that used for registering custom post types and taxonomies so it should be familiar territory.

Like what you see? Want more field types and features added? [Get in touch](mailto:batmoo@gmail.com)

> *See "Other Notes" section for usage information*

== Installation ==

1. Install through the WordPress admin or upload the plugin folder to your `/wp-content/plugins/` directory
1. Activate the plugin through the 'Plugins' menu in WordPress
1. Add the necessary code to register your custom groups and fields to your functions.php or plugin.
1. Enjoy!

== Frequently Asked Questions ==

= Why a code-based approach instead of a UI? =

Because the UI thing has [been](http://wordpress.org/extend/plugins/verve-meta-boxes/) [done](http://wordpress.org/extend/plugins/fresh-page/) [before](http://wordpress.org/extend/plugins/pods/). And this more closely aligns with the existing WordPress approach of registering new types of content (post types, taxonomies, etc.)

This is also a developer feature, aimed towards site builders. And real developers don't need UIs ;)

(But really, though, the main benefit of this fact comes into play when you're working with multiple environments, i.e. development/local, qa/staging, production. This approach makes it easy to replicate UIs and features without having to worry about database synchronization and other crazy things.)

For another really well-done, really powerful code-based plugin for managing custom fields, check out [Easy Custom Fields](http://wordpress.org/extend/plugins/easy-custom-fields/).

= Why isn't the function just `add_metdata_field`? Do you really need the stupid `x_`? =

I'm being good and ["namespacing" my public functions](http://andrewnacin.com/2010/05/11/in-wordpress-prefix-everything/). You should too.

== Screenshots ==

1. Write easy, intuitive and WordPress-like code to add new fields.
2. Custom Metadata Manager supports basic field types with an way to render your own.
3. An example of what's possible with only 40 lines of code.
4. Adding custom columns is also easy. You can go with a default display, or specify your own output callback

== Changelog ==

= 0.3 =
* Can now limit or exclude fields or groups from specific ids
* Added updated screenshots and new code samples!
* Bug fix: the custom display examples weren't working well
* Bug fix: fields not showing on "Add New" page. Thanks Jan Fabry!
* Bug fix: fields not showing on "My Profile" page. Thanks Mike Tew!

= 0.2 =
* Added a textarea field type
* Added support for comments (you can now specify comments as an object type)
* Added basic styling for fields so that they look nice

= 0.1 =
* Initial release

== Usage ==

= Object Types =

The main idea behind this plugin is to have a single API to work with regardless of the object type. Currently, Custom Metadata Manager works with `user`, `comment` and any built-in or custom post types, e.g. `post`, `page`, etc.


= Registering your fields = 

For the sake of performance (and to avoid potential race conditions), always register your custom fields in the `admin_init` hook. This way your front-end doesn't get bogged down with unnecessary processing and you can be sure that your fields will be registered safely. Here's a code sample:

`
add_action( 'admin_init', 'my_theme_init_custom_fields' );

function my_theme_ainit_custom_fields() {
	if( function_exists( 'x_add_metadata_field' ) && function_exists( 'x_add_metadata_group' ) ) {
		x_add_metadata_field( 'my_field', array( 'user', 'post' ) );
	}
}
`


= Getting the data = 

You can get the data as you normally would using the `get_metadata` function. Custom Metadata manager stores all data using the WordPress metadata APIs using the slug name you provide. That way, even if you decide to deactivate this wonderful plugin, your data is safe and accessible.

Example:
`
$value = get_metadata( 'post', get_the_ID(), 'featured', true ); // Returns post metadata value for the field 'featured'
`

= Adding Metadata Groups =

A group is essentially a metabox that groups together multiple fields. Register the group before any fields

`
x_add_metadata_group( $slug, $object_types, $args );
`


**Parameters**

* `$slug` (string) The key under which the metadata will be stored.
* `$object_types` (string|array) The object types to which this field should be added. Supported: post, page, any custom post type, user, comment.


**Options and Overrides**
`
$args = array(
	'label' => $group_slug // Label for the group
	, 'context' => 'normal' // (post only)
	, 'priority' => 'default' // (post only)
	, 'autosave' => false // (post only) Should the group be saved in autosave? NOT IMPLEMENTED YET!
	, 'exclude' => '' // see below for details
	, 'include' => '' // see below for details
);
`


= Adding Metadata Fields =

`x_add_metadata_field( $slug, $object_types, $args );`


**Parameters**

* `$slug` (string) The key under which the metadata will be stored. For post_types, prefix the slug with an underscore (e.g. `_hidden`) to hide it from the the Custom Fields box.
* `$object_types` (string|array) The object types to which this field should be added. Supported: post, page, any custom post type, user, comment.


**Options and Overrides**
`
$args = array( 
	'group' => '' // The slug of group the field should be added to. This needs to be registered with x_add_metadata_group first.
	, 'field_type' => 'text' // The type of field; possible values: text, textarea, checkbox, radio, select
	, 'label' => '' // Label for the field
	, 'description' => '' // Description of the field, displayed below the input
	, 'values' => array() // Values for select and radio buttons. Associative array
	, 'display_callback' => '' // Callback to custom render the field
	, 'sanitize_callback' => '' // Callback to sanitize data before it's saved
	, 'display_column' => false // Add the field to the columns when viewing all posts
	, 'display_column_callback' => '' // Callback to render output for the custom column
	, 'required_cap' => '' // The cap required to view and edit the field
	, 'exclude' => '' // see below for details
	, 'include' => '' // see below for details
);
`

= Include / Exclude =

With v0.3, you can exclude fields and groups from specific object. For example, with the following, field-1 will show up for all posts except post #123:

`
$args = array( 
	'exclude' => 123
);
x_add_metadata_field( 'field-1', 'post', $args );
`

Alternatively, you can limit ("include") fields and groups to specific objects. The following will ''only'' show group-1 to post #456:

`
$args = array( 
	'include' => 123
);
x_add_metadata_group( 'group-1', 'post', $args );
`

You can pass in an array of IDs:

`
$args = array( 
	'include' => array( 123, 456, 789 );
);

With multiple object types, you can pass in an associative array:

`
$args = array( 
	'exclude' => array(
		'post' => 123
		, 'user' => array( 123, 456, 789 )
	)
);


= Examples =
 
For examples, please see the [custom_metadata_examples.php](http://svn.wp-plugins.org/custom-metadata/trunk/custom_metadata_examples.php) file included with the plugin. Add a constant to your wp-config.php called `CUSTOM_METADATA_MANAGER_DEBUG` with a value of `true` to see it in action:

`define( 'CUSTOM_METADATA_MANAGER_DEBUG', true );`

= TODOs =

Stuff I have planned for the future:

* Improved styling of rendered fields
* Ability Pass in attributes for built-in fields (e.g. class, data-*, etc.)
* Additional field types (multi-select, upload, rich-text, etc.)
* Limit or exclude groups and fields using a custom callback
* Autosave support for fields on post types
* Option to enqueue scripts and styles
* Client- and server-side validation support
* Add groups and fields to Quick Edit

