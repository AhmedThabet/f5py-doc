# f5py

Is a Python library written in Rust for the F5 BIG-IP Configurations parsing.

The idea is to provide a simple and easy to use library to parse, manipulate, construct and generate F5 BIG-IP configurations in multiple output formats

Starting with JSON for facilitating the automation of F5 python scripts written by network and security engineers.

Then the plan is to support generating to the following formats

    * LTM commands
    * F5 REST API
    * F5 AS3

Although this library is not a replacement for the F5 SDK, it is a good starting point for those who want to automate F5 BIG-IP LTM configurations in a simpler way

## Actual State

F5 TMM Configurations generated within the scf files (LTM Commands) are parsed to generate the native rust objects that represent the configurations.
Objects manipulation and JSON Output are already supported

## Roadmap
    
    * CSV output, having Virtual Servers as a reference. 
    * Generating LTM commands
    * Parsing and generating F5 iControl REST API
    * Parsing and generating F5 AS3
  
## Useful Links

    * Github Issues Repository: `https://github.com/AhmedThabet/f5py-doc`
    
    * The same library is used in this WASM frontend, so it's possible to parse the configs without writing a simple line of code!
        `https://ipvx.me/f5`
    
## Usage

All the objects can be initialize by passing the raw config text to the class's initializer
or using the new_default() function which will create a new object with default values

```python
from f5py import *

l = LTM("""
ltm pool /Common/www.ipvx.me {
    members {
        /Common/10.1.2.3:8888 {
            address 10.1.2.3
        }
    }
    monitor /Common/TCP-8888
}
ltm virtual /Common/www.ipvx.me {
    destination /Common/35.156.107.63:443
    ip-protocol tcp
    mask 255.255.255.255
    pool /Common/www.ipvx.me
    profiles {
        /Common/clientssl {
            context clientside
        }
        /Common/serverssl {
            context serverside
        }
        /Common/http { }
        /Common/tcp { }
    }
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
}
""")

l
LTM at 0x7fff57e086b0, partitions: 1, virtual_servers: 1, pools: 1, snat_p
v = l.virtual_servers[0]
v
VS: 0x7fff57e086c8, name: /Common/www.ipvx.me , destination: /Common/35.156.107

v.to_json()
'{"name":"/Common/www.ipvx.me","destination":"/Common/35.156.107.63:443","ip-protocol":"tcp","profiles":[{"name":"/Common/clientssl","context":"clientside"},{"name":"/Common/serverssl","context":"serverside"},{"name":"/Common/http"},{"name":"/Common/tcp"}],"mask":"255.255.255.255","pool":"/Common/www.ipvx.me","source-address-translation":{"type":"automap"},"translate_address":"enabled","translate_port":"enabled","source":"0.0.0.0/0"}'

v.to_dict()

{'name': '/Common/www.ipvx.me',
 'destination': '/Common/35.156.107.63:443',
 'ip-protocol': 'tcp',
 'profiles': [{'name': '/Common/clientssl', 'context': 'clientside'},
  {'name': '/Common/serverssl', 'context': 'serverside'},
  {'name': '/Common/http'},
  {'name': '/Common/tcp'}],
 'mask': '255.255.255.255',
 'pool': '/Common/www.ipvx.me',
 'source-address-translation': {'type': 'automap'},
 'translate_address': 'enabled',
 'translate_port': 'enabled',
 'source': '0.0.0.0/0'}

v.destination.get_partition()
'Common'

v.destination.get_name()
'35.156.107.63'

v.destination.get_port()
443

v.destination.get_address()
'35.156.107.63'

v.destination.get_full_path()
'/Common/35.156.107.63'

dir(l)

['__class__',
 '__delattr__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__getattribute__',
 '__gt__',
 '__hash__',
 '__init__',
 '__init_subclass__',
 '__le__',
 '__lt__',
 '__module__',
 '__ne__',
 '__new__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__',
 'new_default',
 'partitions',
 'pools',
 'snat_pools',
 'to_dict',
 'to_json',
 'virtual_servers']

```

### Disclaimer

This library is not affiliated with F5 Networks in any way. It is a personal project and is not supported by F5 Networks.
