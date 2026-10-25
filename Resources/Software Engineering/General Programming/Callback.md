# Callback

A callback is an executable code passed as an argument to a function which is called when a specific event happens. It may be implemented in different forms in different programming languages, such as function pointers, anonymous functions and listeners/observers (object-oriented paradigm). 

It *often* refers to a single executable code, but that is not required. A counterexample: the SurfaceHolder interface in Android SDK (API level 29) allows the registration of multiple callbacks using the addCallback() method.

### Compared to Listener/Observer

- Listener / Observer: in object-oriented paradigm, it is an object with a method which is passed as an argument to a function which is called when a specific event happens. It is one possible implementation of multiple callbacks.
- Observer pattern: an object oriented software design pattern which recommends using observer/listeners in order to decouple the observer class from the observed class.