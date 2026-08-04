# General Information
*Last updated for game version 0.3.1*

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
Used to populate the on-screen buttons in the interactive input mode thing. Read using [getCommands](#getcommands-array).

### lastCommand (String)
Internal, last command entered by user. Read using [getLastCommand](#getlastcommand-string).

>[!NOTE]
>This is basically the raw input from the user; it includes the arguments.
>Example: `"connect 127.0.0.0.1"`

### introText (String)
Displayed when first entering the computer.

### lastOutput (String)
Internal, last displayed text. Read using [getOutput](#getoutput-string).

### isIntro (bool)
Internal, makes the output return [*introText*](#introtext-string) if `true`.

### ended, endedFail (bool)
Internal, marks the computer as finished or not. *endedFail* counts as a failure.

>[!NOTE]
>It'd be better not to modify or read this value directly.  
>To modify, use [markFinished and markFinishedFail](#markfinished-markfinishedfail-void).  
>To read, use [hasEndedFailed, hasEnded, and hasEndedOrFailed](#hasended-hasendedfailed-hasendedorfailed-bool).  

### endedArgs (Array)
Arguments to pass to the **ComputerSimScene** containing this computer once it ends. Use [getEndedArgs](#getendedargs-array) to read.

>[!NOTE]
>When the scene ends, its final arguments array will look like this:  
>`[ failed (bool), endedArgs (Array) ]`,  
>where `failed` is `true` only when the computer is marked as finished with fail (see [ended variables](#ended-endedfail-bool), [the "has ended" functions](#hasended-hasendedfailed-hasendedorfailed-bool) and [the finish marking functions](#markfinished-markfinishedfail-void)).

## NetworkedComputerBase
### connectedTo (String)
The "IP address" of the current server. Essentially points to the [ID](#ip-string) of the [server](#networkedcomputerserver) that should run. `""` (empty string) acts as the default "local" server.

### servers (Dictionary[String, NetworkedComputerServer])
Stores the servers of the current computer network. For a simple, single-server computer, this would only have one entry, with a key equal to `""` (local server).

### localCmds (Dictionary[String, String])
Stores any new commands that this specific computer script provides. The keys are the command itself (what the player has to type), and the value is the description that's displayed by default when using the [`help`](#help-string) command.

>[!IMPORTANT]
>All new commands you make have to be registered here, even if they have an empty description, otherwise they won't work.

# Inner Classes
## NetworkedComputerBase
### NetworkedComputerServer
Contains its own commands, files, intro text when connecting, etc.

#### Variables
##### ip (String)
Essentially the ID of the server. Empty string (`""`) is always the local server, the default upon running the computer.

##### enterText (String)
Same as [*introText*](#introtext-string), displayed when the player connects to this server.

##### supportedCmds (Array[String])
Only these commands will be usable in this server. Includes [*defaultCmds*](#defaultcmds-array) by default.

##### adminCommands (Array[String])
Same as [*supportedCmds*](#supportedcmds-arraystring), but only usable when logged in as admin.

##### username, adminPassword (String)
Username and password for admin account. Password can be empty (`""`) to disable admin stuff.

##### loggedin (bool)
Whether or not the user is currently logged into the admin account of this server.

##### files, privateFiles (Array[ComputerFile])
Array of [**ComputerFile**](#computerfile)s for this server, private ones only show up when logged in. Automatically sorted.

##### data, disconnectData (Dictionary)
For per-server variables, such as transferring credits in the safe in Tavi's route.

Upon [disconnection](#disconnectcomputer-string), all of *data*'s keys that exist in *disconnectData* will be set to *disconnectData*'s values. This can be used to reset progress or something when the player disconnects.

##### nextAvailableId (int)
Internal, next ID available for file creation. Avoid modifying this as it messes with file sorting.

#### Functions
##### _init (NetworkedComputerServer)
( i:String (ip), cmds:Array[String] (supportedCmds), txt:String (enterText), defs:bool (include defaultCmds), d:Dictionary (data), dd:Dictionary (disconnectData), un:String (username), pw:String (adminPassword), adcmds:Array[String] (adminCommands) )

Constructor method. Only first parameter is required, admin stuff is disabled by default and only includes default commands.

##### saveData (Dictionary)
Creates and returns saved data.

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
Text displayed when this file is [`cat`](#cat-string)ed.

##### canDownload (bool)
Used to determine whether or not the file can be downloaded remotely. Currently only used in the `wget` local command in one of the vanilla computers.

##### id (int)
Internal, defaults to `-1` and is set automatically when needed. Used for sorting. Can set it to custom values but beware of conflicts.

##### method (String)
Function to call if the file is opened. Function should have the `localCmd_` prefix, for example a *method* of `doThings` would have to be defined as `func localCmd_doThings()` in the script.

### ComputerFileSorting
Internal class used for sorting files. Gonna stay undocumented for now, since you don't need to know anything about it to make your own computer.

# Functions
## ComputerBase
### learnCommand (void)
(command:String<sup>(untyped)</sup>)

Adds *command* to [*learnedCommands*](#learnedcommands-arraystring) if it's not already in it.

### getCommands (Array)
Returns [*learnedCommands*](#learnedcommands-arraystring).

### getTutorial (String)
Returns tutorial text that should be displayed. Returns empty string (`""`) by default. Put your tutorial display logic here, and tutorial progression logic in [progressTutorial](#progresstutorial-void).

### progressTutorial (void)
Called right before the output of a command is displayed. Progresses the computer's tutorial, does nothing by default. Use [getTutorial](#gettutorial-string) to return the actual text for the tutorial.

### getOutput (String)
Returns [*lastOutput*](#lastoutput-string) if [*isIntro*](#isintro-bool) is `false`, otherwise returns [*introText*](#introtext-string).

### getLastCommand (String)
Returns [*lastCommand*](#lastcommand-string).

### inputCommand (String)
(_commandString:String)

Called when the player enters a command. Sets [*isIntro*](#isintro-bool) to `false` and executes the command by calling [reactToCommand](#reacttocommand-string) after splitting the arguments into an array, before finally [progressing the tutorial](#progresstutorial-void) and returning the expected output.

### reactToCommand (String)
(_command:String, _args:Array, _commandStringRaw:String)

Returns final text to display after calling *_command* with the specified *_args*. *_commandStringRaw* is the raw input from the user. In **ComputerBase** this must be implemented in every computer.

### markFinished, markFinishedFail (void)
(theargs:Array<sup>(untyped)</sup>)

Marks the computer as ended, either as a fail or not depending on the method, and passes *theargs* to the [*endedArgs*](#endedargs-array) variable.

### hasEnded, hasEndedFailed, hasEndedOrFailed (bool)
Self-explanatory, returns `true` if the computer has ended in the appropriate context, otherwise returns `false`.

### getEndedArgs (Array)
Returns [*endedArgs*](#endedargs-array).

### saveData (Dictionary)
Saves the data of this computer. To add new data:
```
func saveData():
  var data = .saveData() # call parent function
  
  data["myProperty"] = "aa" # value
  return data
```

### loadData (void)
(_data:Dictionary<sup>(untyped)</sup>)

Loads data saved from [saveData](#savedata-dictionary). To load new data:
```
  func loadData(_data):
  .loadData(_data) # call parent function

  myProperty = SAVE.loadVar(_data, "myPropertyKey", null) # null is default, "myPropertyKey" is what was defined in saveData()
```

## NetworkedComputerBase
### newCompFile (ComputerFile)
(n:String, cat:String, f:String, down:bool, i:int<sup>(untyped)</sup>, meth:String)

Creates a new [**ComputerFile**](#computerfile) with a name of *n*, cat text of *cat* (with a font of *f*), downloadable if *down*, calls method *meth* when opened, and with an ID of *i*.

Only requires *n* to be specified, and defaults to a downloadable file with a generic description (`it's a file!`) and ID of `-1`.

>[!NOTE]
>This method does not assign a new ID if *i* is `-1`.

### addFileToServer (void)
(server:[NetworkedComputerServer](#networkedcomputerserver), file:[ComputerFile](#computerfile), private:bool)

Adds *file* to *server*'s public or private files based on *private*, and assigns a new ID if the *file's* ID is `-1`. Keeps the *server*'s files sorted.

### getServer (NetworkedComputerServer)
(ip:String)

Returns the server with the specified *ip* from the [*servers*](#servers-dictionarystring-networkedcomputerserver) variable. If such a server doesn't exist, silently returns `null`.

### login (String)
(_args:Array<sup>(untyped)</sup>)

Default command that handles logging into the current server as admin.

### ls (String)
(_args:Array)

Default command that lists all files for the current server, adding private files at the end if the user is logged in as admin.

### cat (String)
(_args:Array)

Default command that displays the [*catData*](#catdata-string) from a file (in its [specified font](#catfont-string)) with an ID equal to *_args* first element, if it exists.

### help (String)
(_args:Array)

Displays the description for the command equal to the first element of *_args* if it's not empty and only has one element, otherwise lists all commands (plus admin-only ones if the user is logged in).

If the specified command is admin-only, it will only be displayed if the user is logged in.

### connectComputer (String)
(_args:Array)

If *_args* only has one element, tries to connect to the server with that [*IP*](#ip-string) and displays its [*enterText*](#entertext-string).

### disconnectComputer (String)
(_args:Array)

Disconnects from the current server if its [*ip*](#ip-string) isn't local/empty (`""`) and *_args* is empty.

### reactToCommand (String)
(_command:String, _args:Array, _commandStringRaw:String)

Tries to run the specified command for the current server, checking default commands first and local commands later, checking filenames last. If the command doesn't exist, [learns](#learncommand-void) the [`help`](#help-string) command.

### getServersData (Dictionary)
Returns a Dictionary of all servers' data, keys are [IPs](#ip-string) and values are whatever the server's [saveData](#savedata-dictionary) function returns.

### saveData (Dictionary)
Saves the computer's data, adding its own specific data on top of its parent's [saveData](#savedata-dictionary-1) function.

### loadData (void)
(_data:Dictionary<sup>(untyped)</sup>)

Loads *_data*, calling its parent's [loadData](#loaddata-void-1) function first.
