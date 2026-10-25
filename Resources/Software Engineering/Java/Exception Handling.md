# Exception Handling

It's bad practice to catch the Exception class instead of Exception sub-classes because

- It makes the catch block catch runtime exceptions that shouldn't be caught
- Exceptions are designed to transmit problem reports to the handler, and doing this would prevent that and hide a serious error

Best Practices
At some point, exceptions **have** to be handled. So there hopefully is a place in an application that handles (catches) all exceptions. Usually, this is a place where unexpected exceptions are caught to prevent complete application crashes and instead provide some path of recovery, along with some excusing message to the user. This is usually provided by frameworks.

In general, in your classes, handle the expected failures and trust the contract your classes and methods provide to the caller. For example, in a method that fetches a record by an ID string from a database, do only catch the SQL exceptions, as your contract may clearly say that a null ID is not allowed. Throw back the bad input exceptions back to the caller, else you may end up validating user input at the wrong level.