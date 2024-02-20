**Actors in Concurrent Programming: A Comprehensive Overview**

*Introduction:*
Concurrent programming has advanced beyond traditional synchronization mechanisms, introducing higher-level abstractions to simplify development. One notable model is the Actor Model, providing an even more abstract and effective approach to concurrency control. In this summary, we delve into the core concepts of actors, their structure, and the principles of message passing.
The fundamental limitation with locks and isolated sections is that even with careful implementation, a single thread accessing a shared object directly can introduce subtle and challenging-to-detect concurrency errors. The Actor Model addresses this by mandating that all accesses to an object be isolated by default. Objects are encapsulated within actors, and no other actor can access them directly.

*1. Actor Structure:*
Actors, in the context of concurrent programming, encapsulate state and behavior, ensuring that no external entity can directly access an actor's state. The fundamental components of an actor include a Mailbox, a set of Methods, and Local State. To illustrate this, let's create a simple Java actor class.

```java
import akka.actor.AbstractActor;
import akka.actor.Props;

public class MyActor extends AbstractActor {
    private int localState = 0;

    @Override
    public Receive createReceive() {
        return receiveBuilder()
                .match(Increment.class, increment -> {
                    localState += increment.value;
                    System.out.println("Actor: Incremented by " + increment.value + ". Local State: " + localState);
                })
                .build();
    }

    public static Props props() {
        return Props.create(MyActor.class);
    }
}
```

*2. Message Passing:*
The Actor Model relies on message passing for communication between actors. Messages are the only means by which actors interact. Let's define a message class, `Increment`, and demonstrate message passing in Java.

```java
public class Increment {
    public final int value;

    public Increment(int value) {
        this.value = value;
    }
}

// Sending a message to the actor
myActor.tell(new Increment(5), ActorRef.noSender());
```

*3. Actor System and Initialization:*
Actors operate within an Actor System, providing the runtime environment for their execution. Creating an instance of an actor involves using the Actor System to spawn actors.

```java
import akka.actor.ActorRef;
import akka.actor.ActorSystem;

// Create an Actor System
ActorSystem system = ActorSystem.create("MyActorSystem");

// Create an instance of the actor
ActorRef myActor = system.actorOf(MyActor.props(), "myActor");
```

*4. Reactive Behavior:*
Actors are inherently reactive, responding to messages asynchronously. They execute methods in response to incoming messages, updating their local state accordingly. The reactive nature of actors ensures a controlled and isolated execution flow.

## Akka Example

Download the Akka Quickstart Guide Maven Project to set up the Java Runtime correctly.

This is an example of how we can use the Akka Actor library to create Actor objects that always provide mutual exclusion. This paradigm also enables event driven programming with ease, while providing thread safety.

```java
package com.example;

import akka.actor.AbstractActor;
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;

// Main application to demonstrate the actor pipeline
public class ActorPipeline {

    public static class FirstActor extends AbstractActor {
        private final ActorRef nextActor;

        public FirstActor(ActorRef nextActor) {
            this.nextActor = nextActor;
        }

        @Override
        public Receive createReceive() {
            return receiveBuilder()
                    .match(String.class, message -> {
                        if (message.equals(message.toLowerCase())) {
                            System.out.println("Forwarding message: " + message);
                            nextActor.tell(message, getSelf());
                        }
                    })
                    .build();
        }
    }

    // Define the SecondActor
    public static class SecondActor extends AbstractActor {
        @Override
        public Receive createReceive() {
            return receiveBuilder()
                    .match(String.class, message -> {
                        if (message.length() % 2==0) {
                            // Process even-length messages
                            System.out.println("Processed message: " + message);
                        }
                    })
                    .build();
        }
    }

    public static void main(String[] args) {
        // Create an Actor System
        ActorSystem system = ActorSystem.create("ActorPipelineSystem");

        // Create instances of the actors
        ActorRef secondActor = system.actorOf(Props.create(SecondActor.class), "secondActor");
        ActorRef firstActor = system.actorOf(Props.create(FirstActor.class, secondActor), "firstActor");

        // Send messages to the first actor
        firstActor.tell("hello", ActorRef.noSender());
        firstActor.tell("World", ActorRef.noSender());
        firstActor.tell("akka", ActorRef.noSender());
        firstActor.tell("Actors", ActorRef.noSender());

        // Shutdown the system when done
        system.terminate();
    }
}



```

#### Logs

```perl
[2024-02-19 18:09:21,992] [INFO] [akka.event.slf4j.Slf4jLogger] [ActorPipelineSystem-akka.actor.default-dispatcher-4] [] - Slf4jLogger started
Forwarding message: hello
Forwarding message: akka
Processed message: akka
[2024-02-19 18:09:22,082] [INFO] [akka.actor.CoordinatedShutdown] [ActorPipelineSystem-akka.actor.default-dispatcher-5] [CoordinatedShutdown(akka://ActorPipelineSystem)] - Running CoordinatedShutdown with reason [ActorSystemTerminateReason]
```
