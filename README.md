# Onion Omega USB Webcam Snapshot ubus script
Tested on ONION OMEGA v1 

## Requirements
- Expansion dock
- Compatible USB webcam 

## Instructions
- Install `fswebcam`: `opkg update && opkg install fswebcam`
- Copy `webcam` into `/usr/libexec/rpcd`.
- Make it executable: `chmod +x /usr/libexec/rpcd/webcam`
- Restart `rpcd`: `/etc/init.d/rpcd restart`

## Verify installation
```bash
ubus list -v webcam
```
You should get something like this (the `<id>` varies): 
```
'webcam' @<id>
        "snapshot":{"use_light":"String", "resolution":"String"}
```
If you get `Command failed: Not found`, run through the #Instructions 
steps and make sure you have the right filename, permissions and you 
have restarted `rpcd`.  

## Command parameters
- `use_light` defaults to "0". Set this to "1" to use the optional LED (see below)
- `resolution` defaults to "640x480". Set this to any other value *you are sure* the camera supports.

## GPIO LED (optional)
You can plug a (white?) LED into GPIO `26` and `GND` and have the command 
light it while taking the snapshot. If you need to use a different GPIO,
make sure to update the script and restart `rpcd`:

```bash
# snip #

LIGHT_GPIO=13

# snip #
``` 


## Test it without parameters (no light, default resolution):
```bash
ubus call webcam snapshot
```

You should get:
```json
{
        "path": "\/tmp\/snapshot-2018-01-10_234214.jpg"
}

```

## Test it with light:
```bash
ubus call webcam snapshot '{"use_light":"1"}'
```

You should get:
```json
{
        "path": "\/tmp\/snapshot-2018-01-10_234214.jpg"
}

```

## Test it with light and custom resolution:
```bash
ubus call webcam snapshot '{"use_light":"1","resolution":"1280x720"}'
```

You should get:
```json
{
        "path": "\/tmp\/snapshot-2018-01-10_234214.jpg"
}

```

## Setting up ubus ACL for HTTP requests

Edit `/usr/share/rpcd/acl.d/onion-console.json` and add a section for `webcam` under `ubus` in both *read* and *write* sections, then restart `rpcd`.
```json
{                                                                                    
        "onion-console": {                                                           
                "description": "Permissions to allow onion-console to work properly",
                "read": {                             
                        "ubus": {  
                                "webcam": [           
                                        "*"           
                                ],                   
                                "file": [             
                                        "list",       
                                        "read",       
                                        "exec",       
                                        "write",      
                                        "stat"        
                                ]
                                
                                // Content omitted
                                }}}}
```
```json
{ // Content omitted
                "write": {                            
                        "ubus": {
                                "webcam": [           
                                        "*"           
                                ],                    
                                "onion": [            
                                        "wifi-scan",  
                                        "wifi-setup", 
                                        "oupgrade",   
                                        "omega-led",  
                                        "gpio"        
                                ]
                                
                                // Content omitted
                                }}}
```

Example curl test
```bash
# Get ubus rpc session id
curl -d '{ "jsonrpc": "2.0", "id": 1, "method": "call", "params": [ "00000000000000000000000000000000", "session", "login", { "username": "root", "password": "secret"  } ] }'  http://your.server.ip/ubus
# Call command with obtained session id (example given: 1234567890)
curl -d '{ "jsonrpc": "2.0", "id": 1, "method": "call", "params": [ "1234567890", "webcam", "snapshot", { "use_light": "1", "resolution": "640x480"  } ] }'  http://your.server.ip/ubus
```
