---
layout: page
title: Scala.js Tutorial
---
{% include JB/setup %}

This step-by-step tutorial where we start with the setup of a Scala.js sbt project and end up having some user interaction and unit testing. The code created in this tutorial is available with one commit per step in the [scalajs-tutorial](https://github.com/scala-js/scalajs-tutorial) repository on GitHub.

## <a name="prerequisites"></a> Step 0: Prerequisites

To go through this tutorial, you will need to [download & install sbt](http://www.scala-sbt.org/0.13/tutorial/Setup.html) (>= 0.13.0). Note that no prior sbt knowledge (only a working installation) is required to follow the tutorial.

## <a name="setup"></a> Step 1: Setup

First create a new folder where your sbt project will go.

### sbt Setup

To setup Scala.js in a new sbt project, we need to do two things:

1. Add the Scala.js sbt plugin
2. Apply the Scala.js specific settings to the project

Adding the Scala.js sbt plugin is a one-liner in `project/build.sbt`:

    addSbtPlugin("org.scala-lang.modules.scalajs" % "scalajs-sbt-plugin" % "0.5.0")

We also setup basic project settings and Scala.js settings in the sbt build file (`build.sbt`):

    scalaJSSettings

    name := "Scala.js Tutorial"

    scalaVersion := "2.11.1"

Last, we need a `build.properties` to specify the sbt version:

    sbt.version=0.13.0

That is all we need to configure the build.

If at this point you prefer to use Eclipse or IDEA as your IDE, you may use [sbteclipse](https://github.com/typesafehub/sbteclipse/wiki/Using-sbteclipse) or [idea-eclipse](https://github.com/mpeltonen/sbt-idea) project files for the IDE. Note that for compiling and running your application, you will still need to use sbt from the command line.

### HelloWorld application

For starters, we add a very simple `TutorialApp` in the `tutorial.webapp` package. Create the file `src/main/scala/tutorial/webapp/TutorialApp.scala`:

{% highlight scala %}
package tutorial.webapp

import scala.scalajs.js.JSApp

object TutorialApp extends JSApp {
  def main(): Unit = {
    println("Hello world!")
  }
}
{% endhighlight %}

As you expect, this will simply print "HelloWorld" when run. To run this, simply launch `sbt` and invoke the `run` task:

    $ sbt
    > run
    [info] Compiling 1 Scala source to (...)/scala-js-tutorial/target/scala-2.11/classes...
    [info] Running tutorial.webapp.TutorialApp
    Hello world!
    [success] (...)

Congratulations! You have successfully compiled and run your first Scala.js application. The code is actually run by a JavaScript interpreter. If you do not believe this (it happens to us occasionally), you can use sbt's `last`:

    > last
    (...)
    [info] Running tutorial.webapp.TutorialApp
    [debug] with JSEnv of type class scala.scalajs.sbtplugin.env.rhino.RhinoJSEnv
    [debug] with classpath of type class scala.scalajs.tools.classpath.CompleteIRClasspath
    [success] (...)

So your code has actually been executed by the [Rhino](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino) JavaScript interpreter.

## <a name="integrating-html"></a> Step 2: Integrating with HTML

Now that we have a simple JavaScript application, we would like to use it in an HTML page. To do this, we need two steps:

1. Generate a single JavaScript file out of our compiled code
2. Create an HTML page which includes that file and calls the application

### Generate JavaScript

To generate a single JavaScript file using sbt, just use the `fastOptJS` task:

    > fastOptJS
    [info] Fast optimizing (...)/scala-js-tutorial/target/scala-2.11/scala-js-tutorial-fastopt.js
    [success] (...)

This will perform some dead-code elimination and generate the `scala-js-tutorial-fastopt.js` file containing reachable JavaScript code.

### Create the HTML Page

The only thing which is specific to Scala.js, is how we can call the Scala code from JavaScript. First have a look at our HTML page (`scalajs-tutorial-fastopt.html` in the project root), we will go in the details right after.

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>The Scala.js Tutorial</title>
  </head>
  <body>
    <!-- Include Scala.js compiled code -->
    <script type="text/javascript" src="./target/scala-2.11/scala-js-tutorial-fastopt.js"></script>
    <!-- Run tutorial.webapp.TutorialApp -->
    <script type="text/javascript">
      tutorial.webapp.TutorialApp().main();
    </script>
  </body>
</html>
{% endhighlight %}

The first script tag simply includes the generated code (attention, you might need to adapt the Scala version from `2.11` to `2.10` here if you are using Scala 2.10.x instead of 2.11.x).

In the second script tag, we first get the `TutorialApp` object. Note the `()`: `TutorialApp` is a function in JavaScript, since potential object initialization code needs to be run upon the first access to the object. We then simply call the `main` method on the `TutorialApp` object.

Since `TutorialApp` extends `JSApp`, the object itself and its `main` method are automatically made available to JavaScript. This is not true in general. Continue reading this tutorial or have a look at the [Export Scala.js API to JavaScript](./export-to-javascript.html) guide for details.

If you now open the newly created HTML page in your favorite browser, you will see ... nothing. The `println` in the `main` method goes right to the JavaScript console, which is not shown by default in a browser. However, if you open the JavaScript console (e.g. in Chrome: right click -> Inspect Element -> Console) you can see the HelloWorld message.

## <a name="using-dom"></a> Step 3: Using the DOM

As the last step has shown, running JavaScript inside a HTML page is not particularly useful if you cannot interact with the page. JavaScript provides the DOM API to interact with the page.

### Adding the DOM Library

To use the JavaScript DOM library in Scala.js, it is best to use the Scala.js DOM library which provides types for the DOM. To add it to your sbt project, add the following line to your `build.sbt`:

    libraryDependencies += "org.scala-lang.modules.scalajs" %%% "scalajs-dom" % "0.6"

sbt-savvy folks will notice the `%%%` instead of the usual `%%`. It means we are using a Scala.js library and not a normal Scala library. Have a look at the [Depending on Libraries](./sbt/depending.html) guide for details. Don't forget to reload the build file if sbt is still running:

    > reload
    [info] Loading global plugins from (...)
    [info] Loading project definition from (...)/scala-js-tutorial/project
    [info] Set current project to Scala.js Tutorial (in build (...)/scala-js-tutorial/)

If you are using an IDE plugin, you will also have to regenerate the project files for autocompletion to work.

### Using the DOM Library

Now that we added the DOM library, let's adapt our HelloWorld example to add a `<p>` tag to the body of the page, rather than printing to the console.

First of all, we import a couple of things:

{% highlight scala %}
import org.scalajs.dom
import dom.document
{% endhighlight %}

`dom` is the root of the JavaScript DOM and corresponds to the `window` object in JavaScript. We additionally import `document` (which corresponds to `document` in JavaScript) for convenience.

We now create a method that allows us to append a `<p>` tag with a given text to a given node:

{% highlight scala %}
def appendPar(targetNode: dom.Node, text: String): Unit = {
  val parNode = document.createElement("p")
  val textNode = document.createTextNode(text)
  parNode.appendChild(textNode)
  targetNode.appendChild(parNode)
}
{% endhighlight %}

Replace the call to `println` with a call to `appendPar` in the `main` method:

{% highlight scala %}
def main(): Unit = {
  appendPar(document.body, "Hello World")
}
{% endhighlight %}

### Rebuild the JavaScript

To rebuild the JavaScript, simply invoke `fastOptJS` again:

    > fastOptJS
    [info] Compiling 1 Scala source to (...)/scala-js-tutorial/target/scala-2.11/classes...
    [info] Fast optimizing (...)/scala-js-tutorial/target/scala-2.11/scala-js-tutorial-fastopt.js
    [success] (...)

As you can see from the log, sbt automatically detects that the sources must be recompiled before fast optimizing.

You can now reload the HTML in your browser and you should see a nice "Hello World" message.

Re-typing `fastOptJS` each time you change your source file is cumbersome. Luckily sbt is able to watch your files and recompile as needed:

    > ~fastOptJS
    [success] (...)
    1. Waiting for source changes... (press enter to interrupt)

From this point in the tutorial we assume you have an sbt with this command running, so we don't need to bother with rebuilding each time.

## <a name="js-export"></a> Step 4: Reacting on User Input

This step shows how you can add a button and react to events on it by still just using the DOM (we will use jQuery in the next step). We want to add a button that adds another `<p>` tag to the body when it is clicked.

We start by adding a method to `TutorialApp` which will be called when the button is clicked:

{% highlight scala %}
@JSExport
def addClickedMessage(): Unit = {
  appendPar(document.body, "You clicked the button!")
}
{% endhighlight %}

You will notice the `@JSExport` annotation. It tells the Scala.js compiler to make a method callable from JavaScript. We must also import this annotation:

{% highlight scala %}
import scala.scalajs.js.annotation.JSExport
{% endhighlight %}

To find out more about how to call Scala.js methods from JavaScript, have a look at the [Export Scala.js API to JavaScript](./export-to-javascript.html) guide.

Since we now have a method that is callable from JavaScript, all we have to do is add a button to our HTML and set its `onclick` attribute (make sure to add the button *before* the `<script>` tags):

{% highlight html %}
<button id="click-me-button" type="button" onclick="tutorial.webapp.TutorialApp().addClickedMessage()">Click me!</button>
{% endhighlight %}

Reload your HTML page (remember, sbt compiles your code automatically) and try to click the button. It should add a new paragraph saying "You clicked the button!" each time you click it.

## <a name="using-jquery"></a> Step 5: Using jQuery

Larger web applications have a tendency to set up reactions to events in JavaScript rather than specifying attributes. We will transform our current mini-application to use this paradigm with the help of jQuery. Also we will replace all usages of the DOM API with jQuery.

### Depending on jQuery

Just like for the DOM, there is a typed library for jQuery, especially packaged for Scala.js. Replace the `libraryDependencies += ...` line in your `build.sbt` by:

    libraryDependencies += "org.scala-lang.modules.scalajs" %%% "scalajs-jquery" % "0.6"

Since we won't be using the DOM directly, we don't need the old library anymore. Note that the jQuery library internally depends on the DOM, but we don't have to care about this. sbt takes care of it automatically.

Don't forget to reload the sbt configuration now:

1. Hit enter to abort the `~fastOptJS` command
2. Type `reload`
3. Start `~fastOptJS` again

Again, make sure to update your IDE project files if you are using a plugin.

### Using jQuery

In `TutorialApp.scala`, remove the imports for the DOM, and add the import for jQuery:

{% highlight scala %}
import org.scalajs.jquery.jQuery
{% endhighlight %}

This allows you to easily access the `jQuery` object (usually referred to as `$` in JavaScript) in your code.

We can now remove `appendPar` and replace all calls to it by the simple:

{% highlight scala %}
jQuery("body").append("<p>[message]</p>")
{% endhighlight %}

Where `[message]` is the string originally passed to `appendPar`.

### Adding JavaScript libraries

You can now try and reload your webpage. You will observe that - although it seems we have done everything right - you don't see a "Hello World" message. Some browsers might give you a `TypeError`. The problem is that we haven't included the jQuery library itself, which is a plain JavaScript library.

An option is to include `jquery.js` from an external source, such as [jsDelivr](http://www.jsdelivr.com/)

{% highlight html %}
<script type="text/javascript" src="http://cdn.jsdelivr.net/jquery/2.1.1/jquery.js"></script>
{% endhighlight %}

This can easily become very cumbersome, if you depend on multiple libraries. The Scala.js sbt plugin provides a mechanism for libraries to declare the plain JavaScript libraries they depend on and bundle them in single file. All you have to do is activate this and then include the file.

In your `build.sbt`, set:

    skip in ScalaJSKeys.packageJSDependencies := false

After reloading and rerunning `fastOptJS`, this will create `scala-js-tutorial-jsdeps.js` containing all JavaScript libraries next to the main JavaScript file. We can then simply include this file and don't need to worry about JavaScript libraries anymore:

{% highlight html %}
<!-- Include JavaScript dependencies -->
<script type="text/javascript" src="./target/scala-2.11/scala-js-tutorial-jsdeps.js"></script>
{% endhighlight %}

### Setup UI in Scala.js

We still want to get rid of the `onclick` attribute of our `<button>`. After removing the attribute, we add the `setupUI` method, in which we use jQuery to add an event handler to the button. We also move the "Hello World" message into this function.

{% highlight scala %}
def setupUI(): Unit = {
  jQuery("#click-me-button").click(addClickedMessage _)
  jQuery("body").append("<p>Hello World</p>")
}
{% endhighlight %}

Since we do not call `addClickedMessage` from plain JavaScript anymore, we can remove the `@JSExport` annotation (and the corresponding import).

Finally, we add a last call to `jQuery` in the main method, in order to execute `setupUI`, once the DOM is loaded:

{% highlight scala %}
def main(): Unit = {
  jQuery(setupUI _)
}
{% endhighlight %}

Again, since we are not calling `setupUI` directly from plain JavaScript, we do not need to export it (even though jQuery will call it).

We now have an application whose UI is completely setup from within Scala.js. The next step will show how we can test this application.

## <a name="testing"></a> Step 6: Testing

Coming soon... (relies on external library not yet published for Scala.js 0.5.0)

## <a name="optimizing"></a> Step 7: Optimizing for Production

Here we show a couple of things you might want to do when you promote your application to production.

### Full Optimization

Size is critical for JavaScript code on the web. To compress the compiled code even further, the Scala.js sbt plugin uses the advanced optimizations of the [Google Closure Compiler](http://developers.google.com/closure/compiler/). To run full optimizations, simply use the `fullOptJS` task:

    > fullOptJS
    [info] Optimizing (...)/scala-js-tutorial/target/scala-2.11/scala-js-tutorial-opt.js
    [info] Closure: 0 error(s), 0 warning(s)
    [success] (...)

Note that this can take a while on a larger project (tens of seconds) we do therefore not recommend to use `fullOptJS` during development.

### Automatically Creating a Launcher

Before creating another HTML file which includes the fully optimized JavaScript, we are going to introduce another feature of the sbt plugin. Since the sbt plugin is able to detect the `JSApp` object of the application, there is no need to repeat this in the HTML file. If you add the following setting to your `build.sbt`, sbt will create a `scala-js-tutorial-launcher.js` file which calls the main method:

    ScalaJSKeys.persistLauncher in Compile := true

We can now include this file in our HTML page instead of the manual launcher:

{% highlight html %}
<!-- Run JSApp -->
<script type="text/javascript" src="./target/scala-2.11/scala-js-tutorial-launcher.js"></script>
{% endhighlight %}

If we rename our `JSApp` object, we need not change our HTML at all anymore. Note that the launcher generation only works if you have a single `JSApp` object. If you happen to have multiple `JSApp` objects but still would like to generate a launcher, you can set the `JSApp` object explicitly. Have a look at the [Compiling, Running, Linking, Optimizing](./sbt/run.html) guide for more details.

### Putting it all Together

We can now create our final production HTML file `scalajs-tutorial.html` which includes the fully optimized code:

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>The Scala.js Tutorial</title>
  </head>
  <body>
    <button id="click-me-button" type="button">Click me!</button>

    <!-- Include JavaScript dependencies -->
    <script type="text/javascript" src="./target/scala-2.11/scala-js-tutorial-jsdeps.js"></script>
    <!-- Include Scala.js compiled code -->
    <script type="text/javascript" src="./target/scala-2.11/scala-js-tutorial-opt.js"></script>
    <!-- Run main object -->
    <script type="text/javascript" src="./target/scala-2.11/scala-js-tutorial-launcher.js"></script>
  </body>
</html>
{% endhighlight %}

This completes the Scala.js tutorial. Refer to our [documentation page](./index.html) for deeper insights into various aspects of Scala.js.