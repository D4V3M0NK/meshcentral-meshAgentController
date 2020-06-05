#MeshAgent Controller
This project is to be used in conjunction with the rather excellent [MeshCentral](https://github.com/Ylianst/MeshCentral) project and is designed to be a simple management BASH script to manage connections to remote systems.

####meshaction.txt
Every host on a MeshCentral system has a `meshaction.txt` file that's assigned to it. It's the hosts `meshaction.txt` files that will be used along with the `meshcmd` utility, both of which are available through the MeshCentral interface.

There needs to be a separate connection file for each host/port combination that you want access to. Additionally, each connection file will use a unique local port.

####Suggested Approach
1. Create a specific folder that will contain all your connection files. For simplicity, also place the `meshcmd` executable into this same folder.
1. Download the `meshaction.txt` file for that _specific_ host into the same folder. Each host within MeshCentral uses a specific ID that's listed in its `meshaction.txt` file, so you can't use one and expect it to work for other hosts. However, that being said, you _can_ use the same file and edit it slightly to connect to the same remote host, just on a different port (see below).
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
1. **NOTE1**: the password is optional at this point - if it's not provided the script below will prompt you for it before connecting.
1. Save the edited `meshaction.txt` as a different filename, **not** `meshaction.txt` (I use `Customer-Server-Port.txt`).
1. **NOTE2**: for different connections to the same host, you can now use the edited file as a template and just change the localPort, remotePort and remoteName (everything else stays the same)
1. Use the attached `fly` and `land` scripts to connect and disconnect.

####Using the `fly` script
Usage: ``` fly connectionFile.txt```

The `fly` script (which can be renamed to anything you choose!) is used to connect to a remote server. It will show you all the current connections, along with their PIDs that you've currently got running. (You'll need the PIDNumber to disconnect gracefully.)

####Using the `land` script
Usage: ``` land PIDNumber```

 The `land` script is used to disconnect from a remote server.

#####Example

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

######Connect
``` fly JohnDoeServicing-AD-80.txt```




Please don't shoot me for the rather poor code, I appreciate that I should assign variables using $() and not backticks, I'll get to that
[fly_land.zip](https://github.com/Ylianst/MeshCentral/files/4737929/fly_land.zip)

