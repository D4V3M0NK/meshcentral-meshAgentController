# MeshAgent Controller
This project is to be used in conjunction with the rather excellent [MeshCentral](https://github.com/Ylianst/MeshCentral) project and is designed to be a simple management BASH script to manage connections to remote systems.

## meshaction.txt
Every host on a MeshCentral system has a `meshaction.txt` file that's assigned to it. It's the hosts `meshaction.txt` files that will be used along with the `meshcmd` utility, both of which are available through the MeshCentral interface.

There needs to be a separate connection file for each host/port combination that you want access to. Additionally, each connection file will use a unique local port.

## Latest changes
- Added MFA capability 
## Prior changes
- Ability to place connection files in their own folder
- Process check when using `land`

## Suggested Approach
1. Create a specific folder that will contain all your connection files. ~~For simplicity, also place the `meshcmd` executable into this same folder.~~
1. Download the `meshaction.txt` file for that _specific_ host and place it into the folder that will contain all your connection files
   1. Each host within MeshCentral uses a specific ID that's listed in its `meshaction.txt` file, so you can't use one and expect it to work for other hosts. 
   1. That being said, you _can_ use the same file and edit it slightly to connect to the same remote host, just on a different port (see below).
1. Edit the downloaded meshaction.txt as per the changes shown below:
    ```
    {
        "localPort": UNIQUE_LOCAL_PORT_NUMBER,
        "remotePort": REMOTE_PORT_NUMBER,
        "remoteName": "CustomerName - Port/Application",
        "username": "YOUR_MESHCENTRAL_USERNAME",
        "password": "YOUR_MESHCENTRAL_PASSWORD",
    }
    ```
1. Explanation of changes:
   1. `localPort`
      1. the port that runs locally on your system that you'll point an application to, in order to route to a remote server. 
      1. takes an integer value, without surrounding quotes.
   1. `remotePort`
      1. the port on the remote server that you want to (ultimately) connect to.
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
1. Save the edited `meshaction.txt` as a different filename, **not** `meshaction.txt` (I use `Customer-Server-Port.txt`).
   1. **Note 2**: for different connections to the _**same**_ host, you can now use the edited file as a template and just change the `localPort`, `remotePort` and `remoteName` (everything else stays the same)
1. Use the attached `fly` and `land` scripts to connect and disconnect.

### Using the `fly` script
Usage: ` fly connectionFile.txt`

The `fly` script (which can be renamed to anything you choose!) is used to connect to a remote server. It will show you all the current connections, along with their PIDs that you've currently got running. (You'll need the PIDNumber to disconnect gracefully.)

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
"serverId": "YRNC3498RYCN203487CG2N3F8C2G3NFC98GFCN3847FGN2C304H2CNO89WF9FWN4837FWGCN87FGC8F7GCN8F7CGWN87GNCW",
"serverHttpsHash": "C2BYO59Y8XN2547HY27042G9C04G8CY4T7P0W9UHNWAC348YNC3X9M8Z398X4RNVL34NFWC98YNCO589YX98X-09UP03QJ0R",
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
"serverId": "YRNC3498RYCN203487CG2N3F8C2G3NFC98GFCN3847FGN2C304H2CNO89WF9FWN4837FWGCN87FGC8F7GCN8F7CGWN87GNCW",
"serverHttpsHash": "C2BYO59Y8XN2547HY27042G9C04G8CY4T7P0W9UHNWAC348YNC3X9M8Z398X4RNVL34NFWC98YNCO589YX98X-09UP03QJ0R",
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

We're back to a single connection. Let's close those connection down too:

``` ./land 17588```

![](https://github.com/D4V3M0NK/meshcentral-meshAgentController/blob/master/img/m4.png "No more connections")

No more connections.

