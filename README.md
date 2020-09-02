## The MeshAgent Router Controller
The MeshAgent Router Controller ("MARC") is to be used in conjunction with the rather excellent [MeshCentral](https://github.com/Ylianst/MeshCentral) project and is designed to be a simple BASH script to manage one or more connections to remote devices configured to connect to MeshCentral.

#### Requirements
- MeshCentral 2.0 ("MC2") up and running
- meshcmd (available from the MC2 interface)
- meshaction.txt files (available from the MC2 interface)
- [jq](https://stedolan.github.io/jq/) (`apt install -y jq`)
- sed (`apt install -y sed`)

##### meshcmd
Known as the MeshCentral Router ([user guide](http://info.meshcentral.com/downloads/MeshCentral2/MeshCentral2RouterUserGuide-0.0.2.pdf)), the `meshcmd` utility is downloaded from the MeshCentral server, a link for which is included with every device that's listed on the server.  

MARC is designed to work exclusively with the Linux command line utility: the Windows version of the utlity is self explanatory and works really well without additional assistance.

##### meshaction.txt
Every device configured to work with a MeshCentral server has an associated `meshaction.txt` file and it's that file, along with the `meshcmd` utility, that will permit you to set up local ports on your workstation that will route traffic through to the remote device. Both the `meshaction.txt` and the `meshcmd` files are downloadable through the MeshCentral web server, but once downloaded, you don't need to log into the web server for various tasks, they can all be managed through the router and MARC.

For each remote device/port combination, there needs to be a separate connection file which will be configured to use a unique local port on your workstation in order for you to access that remote device. This is explained below.

##### jq
Wonderfully powerful utility that enables command line utilities to parse JSON files.

## Latest changes
- server hash update utility
- use of environment variable file
## Prior changes
- Added MFA capability 
- Ability to place connection files in their own folder
- Process check when using `land`

## Suggested Approach
1. On your workstation, create a specific folder that will contain all your connection files. ~~For simplicity, also place the `meshcmd` executable into this same folder.~~ We'll call this folder the _"connection file folder_".
1. Download the `meshaction.txt` file for a _specific_ host and place it into the _connection file folder_.
   1. Each host within MeshCentral uses a specific ID that's listed in its `meshaction.txt` file, so you can't use one and expect it to work for _other_ hosts. 
   1. That being said, you _can_ use the same file and edit it slightly to connect to the __same__ remote host, just on a __different__ port (this is explained below).
1. Edit the downloaded meshaction.txt and make changes just to the entries  shown below:
    ```
    {
        "localPort": UNIQUE_LOCAL_PORT_NUMBER,
        "remotePort": REMOTE_PORT_NUMBER,
        "remoteName": "CustomerName - Port/Application",
        "username": "YOUR_MESHCENTRAL_USERNAME",
        "password": "YOUR_MESHCENTRAL_PASSWORD",
    }
    ```
1. Explanation of changes required:
   1. `localPort`
      1. your workstation's local port that will route traffic through to a remote server. 
      1. takes an integer value, without surrounding quotes.
   1. `remotePort`
      1. remote device port that you want to (ultimately) connect to.
      1. takes an integer value, without surrounding quotes.
   1. `remoteName`
      1. a description that will be shown on the review screen of the connection.
      1. takes a string value within surrounding quotes.
         1. eg "Customer ABC - Web server - MySQL connection"
   1. `username`
      1. your username used to log into the MeshCentral server.
      1. takes a string value within surrounding quotes.
   1. `password` 
      1. your password used to log into the MeshCentral server.
      1. takes a string value within surrounding quotes.
      1. **Note 1**: the password is optional at this point - if it's not provided the script below will prompt you for it before connecting. There are obvious security issues with either approach:
         1. Having the password in the .txt file makes it visible to anyone who has access to that file.
         1. Providing the password at the point of running the command will make that password visible by anyone running `ps -ef | grep meshcmd`
1. __Rename__ the now edited `meshaction.txt` as a different filename, **not** `meshaction.txt` (I use `Customer-Device-Port.txt`).
   1. **Note 2**: for different connections to the _**same**_ host, you can now use the edited file as a template and just change the `localPort`, `remotePort` and `remoteName` to point to a different port on the remote device. Everything else stays the same in the file. Keep saving the file (using different filenames) as you see fit.
1. Use the attached `fly` and `land` scripts to connect and disconnect to the remote devices.

### The environment variable file  `.env`
From Sep 2, 2020, MARC uses an environment variable file named `.env` and is used by various scripts. The idea is to use the default `.env_template` file within this project, rename it to `.env`, edit it accordingly and save it in the same folder as the `fly`, `land`, `serverHttpHash` and other utilities that use it. 

The options within the `.env` file that can be set, are outlined below.

---

__`connectionsFolder`__ (optional, default = BLANK)

The folder that contains all the connection files. When set, you can initiate `fly` with just the name of the connection file, as opposed to the full path to that connection file. The setting can either reference the absolute folder path, or relative folder path to the `meshcmd` file:
* `connectionsFolder=/home/dave/files/remoteDevices` (absolute referencing)
* `connectionsFolder=./files/remoteDevices` (relative referencing)


For example, assume the following folder structure:
```
├── home
│   ├── dave
│   │   ├── README.md
│   │   ├── fly
│   │   ├── land
│   │   ├── meshcmd
│   │   ├── files
│   │   │   ├── remoteDevices
│   │   │   │   ├── custabc-www-http.txt   <-- connect to this host/port
│   │   │   │   ├── custabc-www-ssh.txt
│   │   │   │   ├── custabc-www-sql.txt
```

With `connectionsFolder` set to _an_empty_string_ (the default) then `fly` needs to be invoked as:
* `fly /home/dave/files/remoteDevices/custabc-www-http.txt` (absolute), or
* `fly ./files/remoteDevices/custabc-www-http.txt` (relative)
   
With `connectionsFolder` set to `'/home/dave/files/remoteDevices/'` (absolute) or `'./files/remoteDevices/` (relative) then `fly` can be invoked simply with:
* `fly custabc-www-http.txt`

---

__`mfa`__ (optional, default = 0)
    
`mfa` takes an integer value and relates whether (`mfa=1`) or not (`mfa=0`) multi-factor authentication is enabled on the MeshCentral server. If set, `fly` will prompt the user for their MFA pin number in order to connect to MeshCentral and (ultimately) to the remote device. 

__Note__ : At the moment this is a global setting for MARC, irrespective of whether or not the user listed in the connection file has MFA enabled. If you have connection files for different users and some of those users have MFA enabled within MC2 and others haven't, it's best to set MARC's `mfa` setting to `1`: this means that it will prompt everyone for a PIN number, MC2 users that aren't set up with MFA can enter any number, they'll be authenticated upon their username and password alone.

---

### Using the `fly` script
Usage: ` fly [ path/to/ ] connectionFile.txt`

Placed within the same folder as the `meshcmd` executable, the `fly` script is used to connect to a remote server. It will show you all the current connections, along with their PIDs that you've currently got running. (You'll need the PIDNumber to disconnect gracefully.)

---



### Using the `land` script
Usage: ` land [ all | PIDNumber ]`

 The `land` script is used to disconnect from a remote server. If supplied with the `all` parameter, then all connections to all remote servers will be terminated. If a `PIDNumber` is supplied, then the connection to the specific server associated with that process ID will be terminated.

## Example

Assume the following file for an HTTP connection to an example remote server:

`JohnDoeServicing-ADServer-80.txt`:

```
{
"action": "route",
"localPort": 50040,
"remoteName": "JohnDoeServicing - ADServer - HTTP",
"remoteNodeId": "node//cn347yn23782yx2n48r7xy24897y24n389743ynx8947ryxn3847yxn87yx14F2A",
"remotePort": 80,
"username": "meshcentral.userCredentials@example.org",
"password": "SuperSecretPasswordGoesHere!",
"serverId": "777989898049678376764981685781805666748049558985858353726883855678727273789072797881758576485565",
"serverHttpsHash": "685467486957565180867055737551577774767588567054827973678373527888665085675589867078666657877477R",
"debugLevel": 0,
"serverUrl": "wss://remoteservices.example.org:443/meshrelay.ashx"
}
```

We also need to connect to the same server, but with RDP, so we can copy `JohnDoeServicing-ADServer-80.txt` to `JohnDoeServicing-ADServer-3389.txt` and just change the `remoteName`, `localPort` and `remotePort`:

```
{
"action": "route",
"localPort": 50041,
"remoteName": "JohnDoeServicing - ADServer - RDP",
"remoteNodeId": "node//cn347yn23782yx2n48r7xy24897y24n389743ynx8947ryxn3847yxn87yx14F2A",
"remotePort": 3389,
"username": "meshcentral.userCredentials@example.org",
"password": "SuperSecretPasswordGoesHere!",
"serverId": "777989898049678376764981685781805666748049558985858353726883855678727273789072797881758576485565",
"serverHttpsHash": "685467486957565180867055737551577774767588567054827973678373527888665085675589867078666657877477R",
"debugLevel": 0,
"serverUrl": "wss://remoteservices.example.org:443/meshrelay.ashx"
}
```

**Connecting**

``` ./fly JohnDoeServicing-AD-80.txt```

![](https://github.com/D4V3M0NK/meshcentral-meshAgentController/blob/master/img/m1.png "Initial connection to HTTP")

Here we see that the initial connection has been made, with a PID of 17588. Let's now connect to the RDP port of 3389:

``` ./fly JohnDoeServicing-AD-3389.txt```

![](https://github.com/D4V3M0NK/meshcentral-meshAgentController/blob/master/img/m2.png "Connection to HTTP and RDP")

The second connection has been made, with a PID of 17610, and we can see that we still have our previous HTTP connection still open.

Now let's disconnect from our RDP connection. From the list we can see that we're using that PID of 17610, so all we need to do is land that connection:

``` ./land 17610```

![](https://github.com/D4V3M0NK/meshcentral-meshAgentController/blob/master/img/m3.png "Disconnect from RDP")

We're back to a single connection. Let's close that connection down too: we can use either ` ./land 17588` or `./land all` (the latter will close this and all other connections, had we had any).

![](https://github.com/D4V3M0NK/meshcentral-meshAgentController/blob/master/img/m4.png "No more connections")

No more connections.

---

### Using the `serverHashUpdate` script
Usage: ` serverHashUpdate`
If you're using Let's Encrypt (certbot) to maintain the TLS certificates on MC2, then those certificates will expire every 90 days. When they do, all of your connection files will need to be updated with a new hash in order for you to be able to connect. You'll probably not know about this (unless you're being notified by the Lets Encrypt registration site, or another method) until you _try_ connecting, to be told that `No or invalid "serverHttpsHash" specified.` It's at this point that you'll need to know the new server hash.

At the moment, there's no automatic way of requesting the new server hash, therefore what you'll need to do is obtain a connection file directly from the MC2 server:

1. Log into the MC2 server
1. Select a remote server - don't connect to it, just select it
   1. This will typically take you to the page that shows the 7 Day Power State of the remote server. 
1. Just underneath the 7 Day Power State display, are two links:
   1. Interface
   1. __MeshCmd__ 
1. Click on __MeshCmd__
1. A window appears with a Operating System drop down list. Just ignore this, but click on the Action File link __MeshAction (.txt)__
1. This downloads the current connection file for this server. It contains the current server hash.
1. Open the __MeshAction.txt__ file
1. Copy the `serverHttpsHash` value
```
{
 ...
 "serverHttpsHash": "835449528056555351754866525254754973759084877483788668887776737688527050537068655375878687887469",
 ...
}
```

The `serverHashUpdate` script is used to update all the connection files with a new server hash. It will backup all connection files (as a folder) before updating the files, allowing you to keep those backups afterwards (default).

1. Run the `serverHashUpdate` script
   1. The script shows (if set) the connection file location
1. You're prompted as to whether or not you want to keep a backup of your existing configuration files folder (the default is "Y")
1. Copy the new server hash (the `serverHttpsHash` as outlined above)
1. The script runs through all the files, showing
   1. The current file being updated
   1. The old server hash
   1. The new server hash
1. The script finishes showing whether or not a backup was kept.