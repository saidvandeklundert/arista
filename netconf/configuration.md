## Configuring an Arista device using Netconf.


Before starting, the following configuration is required on the device:

```
leaf02#show running-config section management netconf
management api netconf
   transport ssh netconf
```

In these examples, a vEOS running 4.24.2.2F was used.

NETCONF is not very mature on EOS. There are still a lot of features lacking. Things like commit-checks or rollbacks are not implemented (yet).


## Configuration examples:

```python
# The Arista configuration paths can be browsed here:
# https://eos.arista.com/path-report/
# https://eos.arista.com/ncclient-example-with-eos/
# https://tools.ietf.org/html/rfc6241
#
from ncclient import manager

# Connecting
eos = manager.connect(host="192.168.0.24", port="830", timeout=30, username="admin", password="salt123", hostkey_verify=False)


# edit the running configuration
eos_cfg = '''
<config>
    <system>
        <config>
            <domain-name>ibm.yolo</domain-name>
        </config>
    </system>
</config>
'''

eos.edit_config(target = "running", config = eos_cfg)

# edit the candidate configuration:

eos_cfg = '''
<config>
    <system>
        <config>
            <hostname>veos</hostname>
        </config>
    </system>
</config>'''

eos.lock('candidate')
eos.edit_config(target = "candidate", config = eos_cfg)
eos.commit()
eos.unlock('candidate')

# to discard changes
eos.discard_changes()

# to merge (this is the default)
eos.edit_config(target = "running", default_operation="merge", config = eos_cfg)

# to replace (carefull with this!)
eos.edit_config(target = "running", default_operation="replace", config = eos_cfg)

# lock and unlock running configuration ( users can still 'conf t' on the device!):
eos.lock('running')
eos.unlock('running')

# Get configuration and state data:
eos.get()

# Get running configuration:
eos.get_config(source = "running")

# Examples getting information using subtrees
eos.get(filter=("subtree", "<network-instances/>"))
eos.get(filter=("subtree", "<acl/>"))
eos.get(filter=("subtree", "<system/>"))
eos.get(filter=("subtree", "<system><aaa></aaa></system>"))
eos.get(filter=("subtree", "<system><aaa><authentication></authentication></aaa></system>"))
eos.get(filter=("subtree", "<system><aaa><authentication><users></users></authentication></aaa></system>"))


eos_cfg_dns = '''
<config>
    <system>
        <dns>
            <config>
                <network-instance xmlns="http://arista.com/yang/openconfig/system/augments">default</network-instance>
            </config>
            <servers>
                <server>
                    <address>
                        8.8.8.8
                    </address>
                    <config>
                        <address>8.8.8.8</address>
                        <port>53</port>
                    </config>
                </server>
                <server>
                    <address>9.9.9.9</address>
                    <config>
                        <address>9.9.9.9</address>
                        <port>53</port>
                    </config>
                </server>
            </servers>
        </dns>
    </system>
</config>
'''
eos.edit_config(target = "running", config = eos_cfg_dns)

eos_cfg_hostname = '''
<config>
    <system>
        <config>
            <hostname>veos-1</hostname>
        </config>
    </system>
</config>
'''
eos.edit_config(target = "running", config = eos_cfg_hostname)


eos_cfg_domain = '''
<config>
    <system>
        <config>
            <domain-name>ibm.cloud</domain-name>
        </config>
    </system>
</config>
'''
eos.edit_config(target = "running", config = eos_cfg_domain)


create_vlan = """                            
    <config>                                                                         
      <network-instances>                                                            
        <network-instance>                                                           
          <name>default</name>                                                       
          <vlans>                                                                    
            <vlan>                                                                   
              <vlan-id>115</vlan-id>                                                  
              <config>                                                               
                <mac-learning xmlns="http://arista.com/yang/openconfig/network-instance/vlan/augments">true</mac-learning>
                <name>VLAN0115</name>                                                
                <vlan-id>115</vlan-id>                                                
              </config>                                                              
            </vlan>                                                                  
          </vlans>                                                                   
        </network-instance>                                                          
      </network-instances>                                                           
    </config>                                                                        
"""
eos.edit_config(target = "running", config = create_vlan)


eos_cfg_login_banner = '''
<config>
    <system>
        <config>
            <login-banner>Said pretends to know YANG.</login-banner>
        </config>
    </system>
</config>
'''
eos.edit_config(target = "running", config = eos_cfg_login_banner)


eos_cfg_login_username = '''
<config>  
  <system>  
    <aaa>  
      <authentication>  
        <users>
          <user>
          <username>said</username>
            <config>  
              <username>said</username>
              <password>said123</password>
              <role>network-admin</role>
            </config>
          </user>
        </users>
      </authentication>
    </aaa>
  </system>
</config>  
'''
eos.edit_config(target = "running", config = eos_cfg_login_username)

eos_cfg_login_username = '''
<config>  
  <system>  
    <aaa>  
      <authentication>  
        <users>
          <user>
          <username>said-2</username>
            <config>  
              <username>said-2</username>
              <password-hashed>$6$O73RCNz11Y7ERx.S$S/qlRN.1vYkIVy/825PHprzkyNGPcfRL2xTGdbmH2JzsLkjtQunIB9Ew3V6.COE2V6Q8SmrKzcXGnF0Pp8lAX1</password-hashed>
              <role>network-admin</role>
            </config>
          </user>
        </users>
      </authentication>
    </aaa>
  </system>
</config>  
'''
eos.edit_config(target = "running", default_operation="merge", config = eos_cfg_login_username)
```