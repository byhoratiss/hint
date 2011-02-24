# [Hint Module](https://github.com/goyote/hint) for the Kohana Framework

Hint is a light version of the MSG module. Some of the big changes are:

- Support for multiple drivers is dropped (as well as the cookie driver.) Messages will now be stored in the session by default (which is the common scenario, if you need cookie support use the MSG module.)
- Hint no longer needs the singleton pattern; it has evolved into a fully static class (which was my original intention for MSG.) 

This allows leaner code:

Compare:

    MSG::instance()->set(MSG::SUCCESS, 'Your account has been deleted');

Versus:

    Hint::set(Hint::SUCCESS, 'Your account has been deleted');

---
## New Features

*The complete set of changes are documented in the CHANGELOG.*

Hint supports both `sprintf` a `strtr` for embedding values into the message.

    // Embed some values with sprintf
    Hint::set(Hint::ERROR, '%s is not writable', array($file));

    // Embed some values with strtr
    Hint::set(Hint::ERROR, ':file is not writable', 
        array(':file' => $file));

A `__callStatic` magic method (PHP 5.3) was added to allow you to set messages in a clearer fashion.

    Hint::error('%s is not writable', array($file));

Compare

    Hint::set(Hint::SUCCESS, 'Your account has been deleted');

Versus

    Hint::success('Your account has been deleted');

---
## Installation

Hint installs like any other module. If you're familiar with git, you can fire up a terminal, and add the module with the following command:

    git submodule add git://github.com/goyote/hint.git modules/hint

[!!] **Note:** For a better description on how to add ko modules, please read [http://kohanaframework.org/guide/kohana/tutorials/git](http://kohanaframework.org/guide/kohana/tutorials/git).

Alternatively, you can install Hint manually. To do so, first download the zip file from [http://github.com/goyote/hint](http://github.com/goyote/hint) and extract the `hint` folder into your `modules` directory. Then open the bootstrap file, and modify the `Kohana::modules` function call:

    Kohana::modules(array(
        ...
        'hint' => MODPATH.'hint', // Short messages

---
## Usage

### set()

Use `set()` to store a new message.

    Hint::set(Hint::SUCCESS, 'Unsubscribed.');

You're allowed to store multiple messages; they'll be rendered in the order in which they were introduced.

To rapidly set multiple messages of the same type, you can pass an array as the second argument.

    $post = new Validation($_POST);
    ...
    
	$messages = Arr::map(array('HTML', 'chars), array_values($post->errors()));
    Hint::set(Hint::ERROR, $messages);

You can embed values into the message with `sprintf` or `strtr`.

    // Embed some values with sprintf
    Hint::set(Hint::ERROR, '%s is not writable', array($file));

    // Embed some values with strtr
    Hint::set(Hint::ERROR, ':file is not writable', 
        array(':file' => $file));

You can also set new messages using the `__callStatic` magic method (PHP 5.3)

    Hint::error(array_values($post->errors()));

If you need to attach additional data to the message, you can use the fourth parameter (it accepts data of any type.)

    Hint::notice('Your profile is empty', NULL, array('help' => 'Did you forget to...'));

#### Strategies

If you have many messages scattered over several files, one strategy is to group them into a single file for easier maintenance.

e.g. `messages/hint.php`

    <?php defined('SYSPATH') or die('No direct script access.');
    
    return array(
        'auth' => array(
            login => array(
                'success' => 'Welcome back, :username',
                'error' => 'You are already logged in',
            ),
            'logout => array(
                'success' => 'See ya, come back soon',
            ),
            'register' => array(
                'success' => 'You have successfully registered a new account',
                'error' => 'Please logout before creating a new account',
            ),
            'some_other_action' => ...
        ),
        'some_other_controller' => ...
    );

Then in your logic, you can use the message file:

    Hint::error(__(Kohana::message('hint', 'auth.login.error')));
    
    Hint::set(Hint::SUCCESS, Kohana::message('hint', 'auth.login.success'),
		array(':username' => $this->user->username));

### get()

If no arguments are passed, `get()` will return the whole stack of messages.

    // Get messages
    $messages = Hint::get();

If you only want messages of a certain type (e.g. ERROR) then simply pass in the constant holding the type.

    // Retrieve error messages
    $error_messages = Hint::get(Hint::ERROR);

You can also pass an array of types.

    // Retrieve alerts and warnings
    $messages = Hint::get(array(Hint::ALERT, Hint::WARNING));

Or get everything except a certain type.

    // Get messages of any type except errors and alerts
    $messages = Hint::get(array(1 => array(Hint::ERROR, Hint::ALERT)));

If no messages are found, `get()` will return `NULL`. To override this default value, pass a second argument with your custom value.

    // Override the default return value (NULL) with a custom string
    $messages = Hint::get(Hint::NOTICE, 'no messages found');
    
    // $messages === 'no messages found'

### get_once()

`get_once()` behaves exactly like `get()`, but the only difference is, `get_once()` deletes the messages after retrieval.
    
    // Get messages
    $messages = Hint->get_once();
    
    Hint->get(); // Returns NULL
    
    // `get_once` also retrieves by type
    $alert_messages = Hint->get_once(Hint::ALERT);

### delete()

    // Delete messages
    Hint::delete();
    
    // Delete warning messages
    Hint::delete(Hint::WARNING);

### render()

Finally, you'll want to render the messages in the easiest way possible. To do that, all you have to do is call `echo Hint::render()` anywhere in your view.

    &lt;div id="wrapper"&gt;
        ...
        &lt;?php echo Hint::render() ?&gt;

Or, you can render only a specific type of message, e.g.

    &lt;!-- Render all the messages except the errors at the top of the page --&gt;
    &lt;?php echo Hint::render(array(1 =&gt; array(Hint::ERROR))) ?&gt;
    &lt;form ...&gt;
        &lt;!-- Now render the error messages --&gt;
        &lt;?php echo Hint::render(Hint::ERROR) ?&gt;

Render the messages in a custom view:

    echo Hint::render(NULL, TRUE, 'hint/widget');

`render()` returns a string.

You can modify the default view `hint/default` by copy-pasting it into your `application/views` directory, and making your modifications there.

---
## Types of messages (constants)

	Hint::ERROR
	Hint::SUCCESS
	Hint::NOTICE
	Hint::ALERT
	Hint::ACCESS
	Hint::WARNING
