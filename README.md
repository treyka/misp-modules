# MISP modules

[![Build Status](https://travis-ci.org/MISP/misp-modules.svg?branch=master)](https://travis-ci.org/MISP/misp-modules)

MISP modules are autonomous modules that can be used for expansion and other services in [MISP](https://github.com/MISP/MISP).

The modules are written in Python 3 following a simple API interface. The objective is to ease the extensions of MISP functionalities
without modifying core components. The API is available via a simple REST API which is independent from MISP installation or configuration.

MISP modules support is included in MISP starting from version 2.4.28.

For more information: [Extending MISP with Python modules](https://www.circl.lu/assets/files/misp-training/3.1-MISP-modules.pdf) slides from MISP training.

## Existing MISP modules

* [ASN History](misp_modules/modules/expansion/asn_history.py) - a hover and expansion module to expand an AS number with the ASN description and its history.
* [CIRCL Passive SSL](misp_modules/modules/expansion/circl_passivessl.py) - a hover and expansion module to expand IP addresses with the X.509 certificate seen.
* [CIRCL Passive DNS](misp_modules/modules/expansion/circl_passivedns.py) - a hover and expansion module to expand hostname and IP addresses with passive DNS information.
* [CVE](misp_modules/modules/expansion/cve.py) - a hover module to give more information about a vulnerability (CVE).
* [DNS](misp_modules/modules/expansion/dns.py) - a simple module to resolve MISP attributes like hostname and domain to expand IP addresses attributes.
* [EUPI](misp_modules/modules/expansion/eupi.py) - a hover and expansion module to get information about an URL from the [Phishing Initiative project](https://phishing-initiative.eu/?lang=en).
* [IPASN](misp_modules/modules/expansion/ipasn.py) - a hover and expansion to get the BGP ASN of an IP address.
* [passivetotal](misp_modules/modules/expansion/passivetotal.py) - a [passivetotal](https://www.passivetotal.org/) module that queries a number of different PassiveTotal datasets.
* [sourcecache](misp_modules/modules/expansion/sourcecache.py) - a module to cache a specific link from a MISP instance.

## How to install and start MISP modules?

~~~~bash
apt-get install python3-dev python3-pip libpq5
git clone https://github.com/MISP/misp-modules.git
cd misp-modules
pip3 install -r REQUIREMENTS
cd ..
python3 setup.py build
python3 setup.py install
python3 bin/misp-modules
~~~~

## How to add your own MISP modules?

Create your module in [misp_modules/modules/expansion/](misp_modules/modules/expansion/). The module should have at minimum three functions:

* **introspection** function that returns a dict of the supported attributes (input and output) by your expansion module.
* **handler** function which accepts a JSON document to expand the values and return a dictionary of the expanded values.
* **version** function that returns a dict with the version and the associated meta-data including potential configurations required of the module.

Don't forget to return an error key and value if an error is raised to propagate it to the MISP user-interface.

If your module requires additional configuration (to be exposed via the MISP user-interface), a config array is added to the meta-data output containing all the potential configuration values:

~~~
"meta": {
      "description": "PassiveTotal expansion service to expand values with multiple Passive DNS sources",
      "config": [
        "username",
        "password"
      ],
      "module-type": [
        "expansion",
        "hover"
      ],

...
~~~

### Module type

A MISP module can be of two types:

- **expansion** - service related to an attribute that can be used to extend and update an existing event.
- **hover** - service related to an attribute to provide additional information to the users without updating the event.

module-type is an array where the list of supported types can be added.

## Testing your modules?

MISP uses the **modules** function to discover the available MISP modules and their supported MISP attributes:

~~~
% curl -s http://127.0.0.1:6666/modules | jq .
[
  {
    "name": "passivetotal",
    "type": "expansion",
    "mispattributes": {
      "input": [
        "hostname",
        "domain",
        "ip-src",
        "ip-dst"
      ],
      "output": [
        "ip-src",
        "ip-dst",
        "hostname",
        "domain"
      ]
    },
    "meta": {
      "description": "PassiveTotal expansion service to expand values with multiple Passive DNS sources",
      "config": [
        "username",
        "password"
      ],
      "author": "Alexandre Dulaunoy",
      "version": "0.1"
    }
  },
  {
    "name": "sourcecache",
    "type": "expansion",
    "mispattributes": {
      "input": [
        "link"
      ],
      "output": [
        "link"
      ]
    },
    "meta": {
      "description": "Module to cache web pages of analysis reports, OSINT sources. The module returns a link of the cached page.",
      "author": "Alexandre Dulaunoy",
      "version": "0.1"
    }
  },
  {
    "name": "dns",
    "type": "expansion",
    "mispattributes": {
      "input": [
        "hostname",
        "domain"
      ],
      "output": [
        "ip-src",
        "ip-dst"
      ]
    },
    "meta": {
      "description": "Simple DNS expansion service to resolve IP address from MISP attributes",
      "author": "Alexandre Dulaunoy",
      "version": "0.1"
    }
  }
]

~~~

The MISP module service returns the available modules in a JSON array containing each module name along with their supported input attributes.

Based on this information, a query can be built in a JSON format and saved as body.json:

~~~json
{
  "hostname": "www.foo.be",
  "module": "dns"
}
~~~

Then you can POST this JSON format query towards the MISP object server:

~~~
curl -s http://127.0.0.1:6666/query -H "Content-Type: application/json" --data @body.json -X POST
~~~

The module should output the following JSON:

~~~json
{
  "results": [
    {
      "types": [
        "ip-src",
        "ip-dst"
      ],
      "values": [
        "188.65.217.78"
      ]
    }
  ]
}
~~~

## How to contribute your own module?

Fork the project, add your module, test it and make a pull-request. Modules can be also private as you can add a module in your own MISP installation.

