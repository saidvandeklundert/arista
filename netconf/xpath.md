
```python
from ncclient import manager
from lxml import etree

eos = manager.connect(host="192.168.0.24", port="830", timeout=30, username="admin", password="salt123", hostkey_verify=False)

# Next parts require the following function to remove namespace information from the returned XML:
def strip_ns(root: etree.Element) -> etree.Element:
    """
    This function removes all namespace information from an XML Element tree
    so that a Caller can then use the `xpath` function without having
    to deal with the complexities of namespaces.
    
    Courtesy of Jeremy Schulman who is awesome:
    https://github.com/jeremyschulman/xml-tutorial/blob/master/strip-namespaces.md
    """
    # first we visit each node in the tree and set the tag name to its localname
    # value; thus removing its namespace prefix
    for elem in root.getiterator():
        elem.tag = etree.QName(elem).localname
    # at this point there are no tags with namespaces, so we run the cleanup
    # process to remove the namespace definitions from within the tree.
    etree.cleanup_namespaces(root)
    return root

# Searching through the configuration using XPATH:
#
# first we retrieve the configuration and load it as <class 'lxml.etree._Element'>
# after this we strip the namespace information
# then we use xpath to find the hostname configuration and domain name configuration
xml_obj = etree.fromstring(eos.get_config(source = "running").xml)

xml_obj_no_ns = strip_ns(xml_obj)
print(etree.tostring(xml_obj_no_ns[0], pretty_print=True))

xml_obj_no_ns.xpath('.//hostname')[0].text
xml_obj_no_ns.xpath('.//domain-name')[0].text

# Searching through the configuration and state information using XPATH:
#
# first we retrieve all information and load it as <class 'lxml.etree._Element'>
# after this we strip the namespace information
# then we use xpath to find the hostname configuration and domain name configuration

xml_obj = etree.fromstring(eos.get().xml)
xml_obj_no_ns = strip_ns(xml_obj)

print(etree.tostring(xml_obj_no_ns[0], pretty_print=True))

xml_obj_no_ns.xpath('.//hostname')[0].text
xml_obj_no_ns.xpath('.//domain-name')[0].text
xml_obj_no_ns.xpath('.//neighbors')
xml_obj_no_ns.xpath('.//neighbor-address')
xml_obj_no_ns.xpath('.//neighbor-address')[0].text
```