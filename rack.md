# Rack

The foundation of Rails and all other significant Ruby-based web frameworks is a simple HTTP adapter library called Rack.

Rack exists to standardise the interface between multiple Ruby Web frameworks and Ruby Web servers.

Rack abstracts away the handling of HTTP requests and responses into a single, simple call method.

Rails leverages Rack middlewares in a modular and extensible manner. Much of Action Controller is implemented as Rack middleware modules.
