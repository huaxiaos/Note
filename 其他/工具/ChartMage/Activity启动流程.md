participant ZygoteInit
participant ActivityThread
participant Instrumentation
participant Activity
participant MainActivity
ZygoteInit -->> ZygoteInit: MethodAndArgsCaller run
ZygoteInit -->> ActivityThread: MethodAndArgsCaller invoke
ActivityThread ->> ActivityThread: Looper loop
ActivityThread ->> ActivityThread: Handler dispatchMessage
ActivityThread ->> ActivityThread: ActivityThread handleMessage
ActivityThread ->> ActivityThread: handleLaunchActivity
ActivityThread ->> Instrumentation: : performLaunchActivity
Instrumentation ->> Activity: callActivityOnCreate
Activity ->> MainActivity: performCreate
MainActivity ->> MainActivity: onCreate