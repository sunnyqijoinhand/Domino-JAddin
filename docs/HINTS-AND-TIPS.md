[Homepage](README.md) | [Installation](INSTALLATION.md) | [Hints and Tips](HINTS-AND-TIPS.md) | [API Documentation](api/index.html) | [Open Source Code](https://github.com/AndyBrunner/Domino-JAddin) | [Download](DOWNLOAD.md)

### 1. Framework Architecture

**JAddin.class**

The JAddin class is loaded by the Domino RunJava task as the main Java thread. It executes under the control of RunJava and
shares the IBM Domino Java Virtual Machine (JVM) with RunJava.

- Initialize the JAddin framework
- Dynamically loads and starts the user add-in as a subclass of JAddinThread
- Monitors the Java heap space usage and calls the garbage collector if needed
- Acts on special framework commands (see command `Help!`)
- Calls user add-in methods addInXXX() (see below)

**JAddinThread.class**

This abstract class which must be implemented by the user add-in class. It runs as a separate thread to minimize any delays on the normal processing of the Domino server.

- Initialize the runtime environment
- Calls the user class thru addinStart()
- Includes several methods for accessing Domino objects and the server environment

**AddinName.class**

The user code runs in this subclass of JAddinThread and does all the processing of the application. Several methods are called from the framework which can be implemented by the user class. When the user class terminates, the framework will perform its cleanup and terminates the JAddin main thread.

**Method** | **Required** | **Description**
addinStart() | Yes | Starts the the application code
addinCommand() | No | Called for any console command entered
addinStop() | Yes | Called before termination
addinNextHour() | No | Called at each new hour
addinNextDay() | No | Called at each new day

There are many supporting methods provided by the superclass JAddinThread (see the documentation).

### 2. Console Commands

The framework supports a number of special commands:

```text
> Tell HelloWorld Help!
03.02.2019 09:34:28   JAddin: Quit!       Terminate the add-in thru the framework
03.02.2019 09:34:28   JAddin: GC!         Executes the Java Virtual Machine garbage collector
03.02.2019 09:34:28   JAddin: Debug!      Enable the debug logging to the console
03.02.2019 09:34:28   JAddin: NoDebug!    Disable the debug logging to the console
03.02.2019 09:34:28   JAddin: Heartbeat!  Manually start heartbeat processing (automatically done every 15 seconds)
03.02.2019 09:34:28   JAddin: Help!       Displays this help text
```

### 3. Domino Statistics

The JAddin framework sets and maintains a number of Domino statistic which are shown with the `Show Stat AddinName` command.

```text
> Show Stat HelloWorld
  HelloWorld.Domino.Version = Release 10.0.1|November 29, 2018 (Windows/64)
  HelloWorld.Domino.Platform = 6.2 (Windows 8)  
  HelloWorld.JAddin.StartedTime = 2019-02-03T08:31:47Z
  HelloWorld.JAddin.VersionDate = 2019-02-03
  HelloWorld.JAddin.VersionNumber = 2.1.0
  HelloWorld.JVM.GCCount = 0
  HelloWorld.JVM.HeapLimitKB = 131'072
  HelloWorld.JVM.HeapUsedKB = 20'489
  HelloWorld.JVM.Version = 1.8.0_181 (IBM Corporation)
```

### 4. Debugging

For problem determination, you may enable debugging with the special parameter `Debug!`. While active debugging adds a significant amount of data to the console log and to the log.nsf database, it can be helpful in finding the root of a problem.

**Command** | **Description**
`Load RunJava JAddin AddinName Debug!` | Start your add-in in debug mode
`Tell AddinName Debug!` | Enable debug mode while the add-in is running
`Tell AddinName NoDebug!` | Disable the debug mode while the add-in is running 

The debug output is written to the Domino console and includes the name of the Java method with the source line number issuing the message.

```text
> Load RunJava JAddin HelloWorld Debug!
03.02.2019 09:36:12   JVM: Java Virtual Machine initialized.
03.02.2019 09:36:12   RunJava: Started JAddin Java task.
03.02.2019 09:36:12   JAddin: Debug logging enabled - Enter 'Tell HelloWorld NoDebug!' to disable
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(147)                JAddin framework version 2.1.0
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(148)                HelloWorld will be called with parameters null
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(151)                Creating the Domino message queue
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(169)                Opening the Domino message queue
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(187)                Loading the user Java class HelloWorld
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(199)                User Java class HelloWorld successfully loaded
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(211)                => HelloWorld.addinInitialize()
03.02.2019 09:36:12   DEBUG: HelloWorld.addinInitialize(80)      -- addinInitialize()
03.02.2019 09:36:12   DEBUG: HelloWorld.addinInitialize(94)      Creating the Domino session
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(213)                <= HelloWorld.addinInitialize()
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(224)                => HelloWorld.start()
03.02.2019 09:36:12   DEBUG: JAddin.runNotes(226)                <= HelloWorld.start()
03.02.2019 09:36:12   DEBUG: HelloWorld.runNotes(117)            -- runNotes()
03.02.2019 09:36:12   DEBUG: HelloWorld.runNotes(130)            => HelloWorld.addinStart()
03.02.2019 09:36:12   HelloWorld: Started with parameters null
03.02.2019 09:36:12   HelloWorld: Running on Release 10.0.1 November 29, 2018
03.02.2019 09:36:12   HelloWorld: User code is executing...
> Tell HelloWorld Q
03.02.2019 09:36:27   HelloWorld: User code is executing...
03.02.2019 09:36:27   DEBUG: JAddin.getCommand(622)              User entered Quit, Exit or Domino shutdown is in progress
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(273)                JAddin termination in progress
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(277)                => HelloWorld.addinStop()
03.02.2019 09:36:27   HelloWorld: Termination in progress
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(279)                <= HelloWorld.addinStop()
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(290)                => JAddinThread.addinTerminate()
03.02.2019 09:36:27   DEBUG: HelloWorld.addinTerminate(158)      -- addinTerminate()
03.02.2019 09:36:27   DEBUG: HelloWorld.addinCleanup(190)        -- addinCleanup()
03.02.2019 09:36:27   DEBUG: JAddin.sendQuitCommand(649)         Sending Quit command to Domino message queue
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(292)                <= JAddinThread.addinTerminate()
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(303)                Sending interrupt to HelloWorld
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(308)                Waiting for HelloWorld termination
03.02.2019 09:36:27   DEBUG: JAddin.runNotes(313)                HelloWorld has terminated
03.02.2019 09:36:27   DEBUG: JAddin.jAddinCleanup(751)           -- jAddinCleanup()
03.02.2019 09:36:27   DEBUG: JAddin.jAddinCleanup(775)           Freeing the Domino resources
03.02.2019 09:36:28   DEBUG: JAddin.finalize(798)                -- finalize()
03.02.2019 09:36:28   RunJava: Finalized JAddin Java task.
03.02.2019 09:36:29   RunJava shutdown.
```

### 5. Common Error Messages

**Error Message** | **Possible Reason**
`RunJava: Can't find class JAddIn or lotus/notes/addins/jaddin/AddinName in the classpath.  Class names are case-sensitive.` | The RunJava task was unable to load the class. Make sure that it is written with exact upper and lower case characters and it can be found by the RunJava class loader.
`JAddin: Unable to load Java class AddinName` | The JAddin framework was unable to load the user class. Make sure that it is written with exact upper and lower case characters.
`RunJava: Can't find stopAddin method for class AddinName.` | The user class must be loaded thru the JAddin framework and not directly from RunJava. Use the command `Load RunJava JAddin AddinName` to start the user class.
`RunJava JVM: java.lang.NoClassDefFoundError: Addinname (wrong name: AddinName)` | The user class name in the command and the internal name do not match. Most likely you have not typed the name with correct upper and lower case characters.
`Out of memory` | All Java add-ins execute in a single Java Virtual Machine (JVM) in RunJava. The Domino Notes.Ini parameter `JavaMaxHeapSize=xxxxMB` may be used to increase the heap space.

### 6. Frequently Asked Questions

- Q: How do I develop my JAddin project in Eclipse?
- A: Make sure you include the two JAddin framework class files and the notes.jar file (installed with Notes and Domino) as external files in your project.

- Q: What is the heartbeart in JAddin?
- A: The main thread in JAddin gets triggered every 15 seconds to perform some internal housekeeping tasks. One of these checks makes sure that the Java heap space does not get filled up to avoid out-of-memory errors. If the free space falls below 10 percent, the Java virtual machine garbage collector is invoked and a message is written to the console.

- Q: I have copied a new version of my add-in to the server, but it does not get active during application startup.
- A: The RunJava task caches the Java classes in use. You must terminate all other RunJava tasks - and therefore terminate RunJava itself - to be able to force the reloading of your class file. 

