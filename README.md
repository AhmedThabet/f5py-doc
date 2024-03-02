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
The configugurations can be parsed from supported formats as well
Objects manipulation and JSON Output/Parsing are already supported
  
    * to_json()
    * to_yaml()
    * to_toml()
    * to_csv() <<-- Default implementation, works only on simple structs

    * from_json()
    * from_yaml()
    * from_toml()

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
    persist {
        /Common/EXAMPLE_PERSIST {
            default yes
        }
    }
    translate-address enabled
    translate-port enabled
}
ltm data-group internal /Common/abc_datagroup {
    description "test"
    records {
        0 {
            data "0"
        }
        1 {
            data 1
        }
    }
    type integer
}
""")

l
LTM at 0xe6231ed398, partitions: 1, virtual_servers: 1, pools: 1, snat_pools: 0, data_groups: 1
v = l.virtual_servers[0]
v
VS: 0xe6231ed578, name: /Common/www.ipvx.me , destination: /Common/35.156.107.63:443

v.to_json()
'{"name":"/Common/www.ipvx.me","destination":"/Common/35.156.107.63:443","ip-protocol":"tcp","profiles":[{"name":"/Common/clientssl","context":"clientside"},{"name":"/Common/serverssl","context":"serverside"},{"name":"/Common/http"},{"name":"/Common/tcp"}],"persist":{"name":"/Common/EXAMPLE_PERSIST","default":"yes"},"mask":"255.255.255.255","pool":"/Common/www.ipvx.me","source-address-translation":{"type":"automap"},"translate_address":"enabled","translate_port":"enabled","source":"0.0.0.0/0"}'

v.to_dict()

{'name': '/Common/www.ipvx.me',
 'destination': '/Common/35.156.107.63:443',
 'ip-protocol': 'tcp',
 'profiles': [{'name': '/Common/clientssl', 'context': 'clientside'},
  {'name': '/Common/serverssl', 'context': 'serverside'},
  {'name': '/Common/http'},
  {'name': '/Common/tcp'}],
 'persist': {'name': '/Common/EXAMPLE_PERSIST', 'default': 'yes'},
 'mask': '255.255.255.255',
 'pool': '/Common/www.ipvx.me',
 'source-address-translation': {'type': 'automap'},
 'translate_address': 'enabled',
 'translate_port': 'enabled',
 'source': '0.0.0.0/0'}

print(v.to_yaml())
name: /Common/www.ipvx.me
destination: /Common/35.156.107.63:443
ip-protocol: tcp
profiles:
- name: /Common/clientssl
  context: clientside
- name: /Common/serverssl
  context: serverside
- name: /Common/http
- name: /Common/tcp
persist:
  name: /Common/EXAMPLE_PERSIST
  default: yes
mask: 255.255.255.255
pool: /Common/www.ipvx.me
source-address-translation:
  type: automap
translate_address: enabled
translate_port: enabled
source: 0.0.0.0/0

v.destination.partition()
'Common'

v.destination.name()
'35.156.107.63'

v.destination.port()
443

v.destination.address()
'35.156.107.63'

v.to_toml()
'name = "/Common/www.ipvx.me"\ndestination = "/Common/35.156.107.63:443"\nip-protocol = "tcp"\nmask = "255.255.255.255"\npool = "/Common/www.ipvx.me"\ntranslate_address = "enabled"\ntranslate_port = "enabled"\nsource = "0.0.0.0/0"\n\n[[profiles]]\nname = "/Common/clientssl"\ncontext = "clientside"\n\n[[profiles]]\nname = "/Common/serverssl"\ncontext = "serverside"\n\n[[profiles]]\nname = "/Common/http"\n\n[[profiles]]\nname = "/Common/tcp"\n\n[persist]\nname = "/Common/EXAMPLE_PERSIST"\ndefault = "yes"\n\n[source-address-translation]\ntype = "automap"\n'

x = Virtual.from_toml(v.to_toml())
x
VS: 0x4de57ecd68, name: /Common/www.ipvx.me , destination: /Common/35.156.107.63:44

l.data_groups[0].to_json()
'{"name":"/Common/abc_datagroup","records":{"1":"1","0":"0"},"description":"test","type":"integer"}'

dir(l)

['__class__',
 '__delattr__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__getattribute__',
 '__getstate__',
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
 'data_groups',
 'from_json',
 'from_toml',
 'from_yaml',
 'new_default',
 'partitions',
 'pools',
 'snat_pools',
 'to_dict',
 'to_json',
 'to_toml',
 'to_yaml',
 'try_to_csv',
 'virtual_servers']



```

### Disclaimer

This library is not affiliated with F5 Networks in any way. It is a personal project and is not supported by F5 Networks.

by: <a href="https://www.linkedin.com/in/ahmedthabet/" target="_blank">Ahmed Thabet</a>