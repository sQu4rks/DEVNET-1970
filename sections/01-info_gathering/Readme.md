# Information Gathering

ToDo(mneiding): Insert demo video

In this demo, we are retrieving information from a device. pyATS provides functionalities to connect to devices and then issue a command to the device as if you were typing them after connecting via SSH manually. A special feature of pyATS is it's *parsing* functionality. Instead of just returning you the textual output of a `show` command we get structured data (in the form of a python dictionary) back that we can then use going forward. 

## Step-by-step instructions

1. Create a new folder called `01_info_gathering` and in it create a file called `testbed.yaml`. This file will contain the connection details to our device. In this example we are connecting to a sandbox device using ssh.
```yaml
---
testbed:
  name: alwaysonsbxs
  credentials: 
    default:
      username: "developer"
      password: "C1sco12345"
      enable: "C1sco12345"

devices:
  csr1000v-1:
    os: iosxe
    type: iosxe
    connections:
      defaults:
        class: unicon.Unicon
      ssh:
        protocol: ssh
        ip: "sandbox-iosxe-latest-1.cisco.com"
        port: "22"
```
2. Create another file called `facts.py`. This file is going to contain the logic of loading our previously created testbed file, connect to all devices in the testbed, issue the show version command and print back the device name as well as the version we rerieved. Your directory structure should now look like this:
```
- 01_info_gathering
  |- testbed.yaml
  |- facts.py
```
3. In the `facts.py` file, we will first import the load function from genie and load the testbed from the previously created file. `genie` is part of the `pyATS` ecosystem and is the part that offers the parsers as well as the functionality to load testbed files. For the sake of simplicity we will keep referring to the entire framework as `pyATS` while it truely is `pyATS` and `genie`.
```python
from genie.testbed import load

testbed = load('testbed.yaml')
```
4. Next, we are iterating over all the devices in the testbed. The `testbed.devices` property is a dict-like object that contains a name and a device object for each of our network devices we have listed in the `testbed.yaml` file.
```python
for name, dev in testbed.devices.items():
```
5. Inside of the for-loop, connect to each device using the `connect()` method provided by the device object. We pass the `log_stdout=False` flag to disable logging. Otherwise pyATS would show us all the textual output it receives (and parses) from the device. 
```python
   dev.connect(log_stdout=False)
```
6. Still in the for-loop and after connecting to the device we use the `parse()` method to issue the `show version` command, parse the text into a python dictionary and safe that dictionary under the name `out`. Finally, we print the name of our device and the short version of the currently running Cisco IOS version.
```python
   out = dev.parse('show version')
   print(f"{name}: {out['version']['short_version']}")
```
7. In order to see all the available information that were parsed from the `show version` command you can also print the entire dictionary. 
```python
   print(out)
```

The entire `facts.py` file then looks like this:

```python
from genie.testbed import load

testbed = load('testbed.yaml')

for name, dev in testbed.devices.items():
   dev.connect(log_stdout=False)

   out = dev.parse('show version')
   print(f"{name}: {out['version']['short_version']}")

   # Optional if you include step 7
   print(out)
```

<div align="right">
   
   [Back to Overview](../../Readme.md) - [Next](../02-config_diff/)
</div>