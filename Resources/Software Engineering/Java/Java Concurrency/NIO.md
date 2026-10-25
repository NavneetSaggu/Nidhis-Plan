# NIO

# Java NIO vs IO

NIO is an advancement of conventional java IO API.

 The main enhancements are: 

- **Channels and Buffers:** In standard I/O API the character streams and byte streams are used. In NIO we work with channels and buffers. Data is always written from a buffer to a channel and read from a channel to a buffer.
- **Selectors:** Java NIO provides the concept of "selectors". It is an object that can be used for monitoring the multiple channels for events like data arrived, connection opened etc. Therefore single thread can monitor the multiple channels for data.
- **Non-blocking I/O:** Java NIO provides the feature of Non-blocking I/O. Here the application returns immediately whatever the data available and application should have pooling mechanism to find out when more data is ready.
- More details:
    - NIO uses buffer and channel oriented data flow model for IO which is faster and has better performance
    - NIO uses OS for conventional IO tasks
    - IO uses stream line data flow i.e one more byte at a time and relies on converting data objects into bytes and bytes into object; NIO deals with the data blocks (chunks of bytes)
    - In java IO stream objects are unidirectional while in NIO channels are bidirectional; can be used for both reading and writing data.
    - The streamline data flow in IO does not allow move forth and back in the data. If case need to move forth and back in the data read from a stream need to cache it in a buffer first. While in case of NIO we uses buffer oriented which allows to access data back and forth without need of caching.
    - NIO API also supports multi threading so that data can be read and written asynchronously in such as a way that while performing IO operations current thread is not blocked.This again make it more efficient than conventional java IO API.
    - Concept of multi threading is introduced with the introduction of **Selectors** in java NIO which allow to listen to multiple channels for IO events in asynchronous or non blocking way.
    - Multi threading in NIO make it Non blocking which means that thread is requested to read or write only when data is available otherwise thread can be used in other task for mean time.But this is not possible in case of conventional java IO as no multi threading is supported in it which make it as Blocking.
    - NIO allows to manage multiple channels using only a single thread,but the cost is that parsing the data might be somewhat more complicated than when reading data from a blocking stream in case of java IO.So in case fewer connections with very high bandwidth are required with sending a lot of data at a time,than in this case java IO API might be the best fit.
    
    [Quick guide.](https://www.tutorialspoint.com/java_nio/java_nio_quick_guide.htm)