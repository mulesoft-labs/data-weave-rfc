# History of the DW Playground

A year ago, we decided to build a new playground to help our community learn our language. We already had a prototype that several people were using, but it was not supported and was not official. This version was so unstable that we had a script to restart it every few hours. The restart was very fast, so people did not suffer. 

We started designing and thinking about what we needed to have a stable playground that would not need to be restarted. It needs to be secure and fast. People like our playground because it gives instant feedback and does not require an extra step.

Our first decision was to choose the technology we wanted to use. Our first thought was to use Scala (the DW runtime is written in Scala), because it's the language our team is most familiar with and it was easy to integrate with our runtime. But then we took a more interesting turn. Could we do this in DW? A not unimportant detail: the old playground was already implemented in DW ;)

DW is a language designed for transformations, but the challenge was interesting and we decided to do a spike to see what gaps needed to be filled. And by analyzing the old Playground, we quickly realized:

- Fast HTTP capabilities
- Multi-Tenant runtime
- Monitoring

Like the missing language capabilities

# HTTP capabilities

For Http, we have developed an easy to use HTTP library. This library allows you to execute calls and serve HTTP requests. 

A simple example of running an http server that returns "hello world" when the endpoint "GET:/test"  is triggered. 

```
api( {port: 8082}, { "/test": { GET : ( request ) -> "Hello world" } })
```

We set a function called 'api', which we can use to start a server with a configuration and a handler method. The handler is given a path and the method to handle. 

This module can be found at [http-module](https://github.com/mulesoft/data-weave-io/tree/master/http-module). 

*Disclaimer:* This module is not available in Mule and there are no plans to include it.

## Multitenat Runtime

Our first task was to define what a multitenant runtime environment meant in terms of its capabilities. A multitenant runtime had to be able to execute all of the users' scripts in a secure, controlled, and fast manner. Speed was key to meeting users' XP expectations, as instant feedback is a very pleasant development experience. Security is the other critical point. Users have the freedom to write any script they want, and we need to be able to manage that.

Now that we have defined the minimum requirements for the multi-tenant, we have started designing the security aspects.

### Hosted runtimes

The first problem we had to solve was running dataweave inside dataweave. We needed some sort of 'eval' function. For this reason, we introduced the 'dw::Runtime::run' function.
A function that allows you to not only run a script with input, but also capture logs, errors, support modules and more. This function runs DW inside DW, meaning it creates a new engine that inherits some capabilities and properties from the parent runtime (the host).

### Security

Once the function was set up, we started working on the security aspects. The first thing we needed to do was define a 'SecurityManager'. A SecurityManager is a gatekeeper that allows or disallows certain operations based on a privilege model. It is simply a function that is given a 'privilege' (a label) and returns 'true' if it allows the current runtime, or 'false' if it does not.
There are several functions in the DW language that allow the user to do things that may not be safe in a multi-tenant runtime environment. For example, accessing environment variables on the running server. To support this, we introduce the annotation 'dw::Core::RuntimePrivilege'. This annotation allows a developer to mark a function with a required privilege. Before calling the annotated function, it ensures that the Runtime has been granted the required privilege.

For example, 'dw::System::envVars' requires the 'Environment' privilege to execute.

```
@RuntimePrivilege(requires = "Environment").
fun envVars(): dictionary = native("system::env")
```

If we run this script on the playground

```
%dw 2.0 output application/json
---
dw::System::envVars()
```

fails with the reason that the permission'Enviroment' privilege was not granted.

```
The specified required privilege: 'Environment' was not granted for this execution.

59| @Interceptor(interceptorFunction = "@native system::SecurityManagerCheckFunctionValue") ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Trace: at dw::Core::envVars ( line: 59, column: 36) at main::main ( line: 4, column: 1)

```

With the new feature, we have added a configuration to the 'run' function where we can specify what permissions are granted for this execution. This way we can grant different privileges to the HTTP server than to the script being executed.

This ensures that no user can access anything we do not want. But we are still vulnerable to DoS because we do not control how long a script can be executed. To address this, we needed to add a way to control the maximum execution time at the DW runtime level. We added a watchdog to monitor the time a script took to run, and a way to stop execution within a certain time. And we added a way to set the maximum execution time to the 'run' function.

Then a script could be executed like in the playground

```
%dw 2.0 output application/json
---
1 to 1000 map dw::Runtime::wait($,$)
```

fails with an exception like

```
The execution took: 2564ms, which exceeds the maximum time: 2500ms

1040| fun wait (value: T, timeout: Number): T = native("system::wait") ^^^^^^^^^^^^^^^^^^^^^^
```

This indicates that the script used more processing time than was allowed.

With a secured and managed runtime environment, we launched our first internal version of the playground.

## Monitoring

In addition to fixing some bugs, our first realisation was that it was difficult to reproduce and understand a problem once it occurred. So we needed a way to log data when a script was affecting our runtime so we could identify problems. To achieve this, we came up with the idea of implementing 'interceptors'. 

### Interceptors

This new experimental feature was a building block that allowed us to build monitoring policies to the functions we liked to monitor.

We have created a new annotation @Intereceptor. This annotation is used as a meta annotation, as the target of this annotation is another annotation, and receives as a parameter the interceptor function. We call interceptor function to the function being specify to the @Interceptor and interceptor annotation to the annotation being annotated with @Interceptor

The interceptor function needs to have a type like

```
(annotationArgs: {}, targetFunctionName: String, args: Array, callback: (args: Array) - > Any) -> Any
```

The first parameter are the arguments used passed to the interceptor annotation
The second parameter is the name of the function being annotated with the interceptor annotation
The third parameter are the arguments being sent to the intercepted function
The fourth parameter is the callback to the intercepted function


*An example:*

We are going to create an annotation that logs the function arguments based on a sample ratio.

```
/**
* Interceptor function for logging the arguments when the Sampling Ratio
**/
fun sampleArgs(arguments: {ratio: Number}, functionName: String, args: Array<Any>, callback: (Array<Any>) -> Any) : Any = do {
  var result = if(random() < arguments.ratio)
                callback(log( "[SAMPLING]" ++ functionName,args))
               else
                 callback(args)
  ---
  result
}

@Interceptor(interceptorFunction = sampleArgs)
annotation SampleArgs(ratio: Number)


@SampleArgs(ratio = 0.3)
fun myFunction(name: String) = name ++ “weaver”
```

On 30% of the invocations to *myFunction* the arguments are going to be logged

With the feature put in place, we were able to implement things like sampling based logging or error tracking. So that we can understand what is going on with the service and troubleshoot any problem.

## Conclusion

In order to develop the playground we made DW language and Runtime stronger and more powerful. We also found that some of the features were not mature enough so we introduced the concept of `Experimental` this new annotation is being used to mark any experimental function so that when it is used it warns the user. 

I think that the new tools can provide a rich set of use cases, like running dynamic scripts in a managed way or some dynamic scripting use cases. Please report any idea or use cases on the ideas portal. 

Here is a simplified version of our DW backend for you to understand how to use it.

```
import * from dw::core::Objects
import CORS from dw::io::http::Interceptors
import * from dw::io::http::Server

import run, ReaderInput,RunSuccess,ExecutionFailure from dw::Runtime
import dw::core::Binaries

var serverConfig: {host: String, port: Number} = { host: "localhost", port: 8081 }

fun runTransform(transformRequest) = do {
    var inputs: Dictionary<ReaderInput> = {
                    (
                        transformRequest.inputs mapObject {
                            ($$) : $ update {
                               case .value -> Binaries::fromBase64($)                               
                            }
                        }
                    )
                 }
                 

    ---
    run(transformRequest.main,transformRequest.fs, inputs)  match {
    		case is RunSuccess -> {
    		   success: true,
    		   result: {
    		        value: $.value as String {encoding: $.encoding default "UTF-8"},
    		        mimeType: $.mimeType,
    		        encoding: $.encoding default "UTF-8",
    		        logs: $.logs
    		   }
    		}
    		case is ExecutionFailure -> {
    		    success: false,
    		    error: {
    		        message: $.message,
    		        location: $.location,
    		        logs: $.logs
    		    }
    		}
    }
}

---
api(
    serverConfig mergeWith {interceptors: [CORS()]}, 
    {
      "/transform": {
          POST: (request) -> {
              body: runTransform(request.body)
          }
      },
        //TODO: need uri params to extract parts of the path
      "/": {
          GET: ((request) -> resourceResponse("index.html"))
      },
      "/.+": {
          GET: ((request) -> resourceResponse(request.path))
      }
  }
)
```



