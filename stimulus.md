# Stimulus

Stimulus is a JavaScript framework with modest ambitions. Unlike other front-end frameworks, Stimulus is designed to enhance server-rendered HTML by connecting JavaScript objects to elements on the page using simple annotations.

These JavaScript objects are called controllers, and Stimulus continuously monitors the page waiting for HTML `data-controller` attributes to appear. For each attribute, Stimulus looks at the attribute’s value to find a corresponding controller class, creates a new instance of that class, and connects it to the element.

You can think of it this way: just like the class attribute is a bridge connecting HTML to CSS, Stimulus’s `data-controller` attribute is a bridge connecting HTML to JavaScript.

Aside from controllers, the three other major Stimulus concepts are:

\- actions, which connect controller methods to DOM events using data-action attributes

\- targets, which locate elements of significance within a controller.

Targets Map Important Elements To Controller Properties.

\- values, which read, write, and observe data attributes on the controller’s element

Stimulus’s use of data attributes helps separate content from behaviour in the same way CSS separates content from presentation.

==Stimulus helps you build small, reusable controllers, giving you just enough structure to keep your code from devolving into “JavaScript soup.”==



