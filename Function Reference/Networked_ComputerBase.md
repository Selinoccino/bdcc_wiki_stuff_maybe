# General Information
*Last updated for game version 0.2.5*

The **ComputerBase** class and its subclasses (**NetworkedCimputerBase**) are used by and ran through the **ComputerSimScene** script, which is itself a [**Scene**](/Your-first-scene.md).

## ComputerBase
Class for creating computers and handling their input. Extends [**Reference**](https://docs.godotengine.org/en/3.6/classes/class_reference.html).

## NetworkedComputerBase
Class for easily creating computers in a modular way. Extends **ComputerBase**. This page contains information that sometimes applies to both, but focuses on **NetworkedComputerBase**.


# Signals
These classes have no signals.

# Variables

## ComputerBase
### id (String)
ID of the computer, used to run it and differentiate from other computers.

### learnedCommands (Array[String])
Used to populate the on-screen buttons in the interactive input mode thing.

### lastCommand (String)
Internal, last command entered by user.

>[!NOTE]
>This is basically the raw input from the user; it includes the arguments.
>Example: `"connect 127.0.0.0.1"`

### introText (String)
Displays when first entering the computer.

### lastOutput (String)
Internal, last displayed text.

### isIntro (bool)
Internal, makes the output return [*introText*](#introtext-string) if `true`.

### ended, endedFail (bool)
Internal, marks the computer as finished. *endedFail* counts as a failure.

>[!NOTE]
>It'd be better not to modify or read this value directly.  
>To modify, use [markFinished and markFinishedFail]().  
>To read, use [hasEndedFailed, hasEnded, and hasEndedOrFailed]().  

### endedArgs (Array)
Arguments to pass to the **ComputerSimScene** containing this computer once it ends.

>[!NOTE]
>When the scene ends, its final arguments array will look like this:  
>`[ failed (bool), endedArgs (Array) ]`,  
>where `failed` is `true` only when the computer is marked as finished with fail (see [ended variables](#ended-endedfail-bool) and [the finish marking functions]())

## NetworkedComputerBase
### connectedTo (String)
The "IP address" of the current server. Essentially points to the ID of the server that should run. `""` (empty string) acts as the default "local" server.

### servers (Dictionary[String, NetworkedComputerServer])
Stores the servers of the current computer network. For a simple, single-server computer, this would only have one entry, with a key equal to `""` (local server).

### localCmds (Dictionary[String, String])
Stores any new commands that this specific computer script provides. The keys are the command itself (what the player has to type), and the value is the description that's displayed by default when using the `help` command.

>[!IMPORTANT]
>All new commands you make have to be registered here, even if they have an empty description.

# Inner Classes
## NetworkedComputerBase
### NetworkedComputerServer
Contains its own commands, files, intro text when connecting, etc.

#### Variables
##### ip (String)
Essentially the ID of the server. Empty string (`""`) is always local server, default upon running the computer.

##### enterText (String)
Same as [*introText*](#introtext-string), displayed when the player connects to this server.

##### supportedCmds (Array[String])
Only these commands will be usable in this server. Includes *defaultCmds* by default.

##### adminCommands (Array[String])
Same as [*supportedCmds*](#supportedcmds-arraystring), but only usable when logged in as admin.

##### username, adminPassword (String)
Username and password for admin account. Password can be empty to disable admin stuff.

##### loggedin (bool)
Whether or not the user is currently logged into the admin account of this server.

##### files, privateFiles (Array[ComputerFile])
Array of [**ComputerFile**]()s for this server, private ones only show up when logged in. Automatically sorted.

##### data, disconnectData (Dictionary)
For per-server variables, such as transferring credits in the safe in Tavi's route.

Upon disconnection, all of *data*'s keys that exist in *disconnectData* will be set to *disconnectData*'s values. This can be used to reset progress or something when the player disconnects.

##### nextAvailableId (int)
Internal, next ID available for file creation. Avoid modifying this as it messes with file sorting.

#### Functions
##### _init (NetworkedComputerServer)
( i:String (ip), cmds:Array[String] (supportedCmds), txt:String (enterText), defs:bool (include defaultCmds), d:Dictionary (data), dd:Dictionary (disconnectData), un:String (username), pw:String (adminPassword), adcmds:Array[String] (adminCommands) )

Constructor method. Only first parameter is required, admin stuff is disabled by default and only includes default commands.

##### saveData (Dictionary)
Returns saved data.

##### loadData (void)
(d:Dictionary)

Loads saved data.

##### hasFileWithId (bool)
(id:int, private:bool)

Returns `true` if a file with the specified ID exists in the files (or admin-only files with *private* set to `true`), otherwise returns `false`.

##### addFile, addPrivateFile (void)
(f:ComputerFile)

Adds a file to either the public or the admin-only file list. If there's already a file where this one would be placed, silently aborts and does nothing.

##### getFileFromName (ComputerFile)
(n:String)

Looks for a file of *n* name, respecting current player admin status (aka looks in private files first if player is logged in, and then public files). Returns the file if found, otherwise returns `null`.

##### getFileFromId (ComputerFile)
(i:int)

Returns the file with the specified ID if it exists, otherwise returns `null`. Looks in private files first if the player is logged in.

### ComputerFile
Basic file for use in the servers.

#### Variables
##### name (String)
Display name of the file, including extension.

##### catData (String)
Text displayed when this file is `cat`ed.

##### canDownload (bool)
Used to determine whether or not the file can be downloaded remotely. Currently only used in the `wget` local command in one of the vanilla computers.

##### id (int)
Internal, defaults to `-1` and is set automatically when needed. Used for sorting. Can set it to custom values but beware of conflicts.

##### method (String)
Method to call if the file is opened. Method should have the `localCmd_` prefix, for example a *method* of `doThings` would have to be defined as `func localCmd_doThings()` in the script.

### ComputerFileSorting
Internal class used for sorting files. Gonna stay undocumented for now.

