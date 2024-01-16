%%%
title = "XML to JSON conversion rules for RESTful EPP"
abbrev = "XML to JSON for RESTful EPP"
ipr = "trust200902"
area = "Internet"
workgroup = "Network Working Group"
submissiontype = "IETF"
keyword = [""]
TocDepth = 4

[seriesInfo]
name = "Internet-Draft"
value = "draft-wullink-restful-epp-json-00"
stream = "IETF"
status = "standard"

[[author]]
initials="M."
surname="Wullink"
fullname="Maarten Wullink"
abbrev = ""
organization = "SIDN Labs"
  [author.address]
  email = "maarten.wullink@sidn.nl"
  uri = "https://sidn.nl/"

[[author]]
initials="M."
surname="Davids"
fullname="Marco Davids"
abbrev = ""
organization = "SIDN Labs"
  [author.address]
  email = "marco.davids@sidn.nl"
  uri = "https://sidn.nl/"
%%%

.# Abstract

This document describes the rules for converting an EPP [@!RFC5730] XML message to a JSON [@!RFC8259] message for use with RESTful EPP [REF-TO-REPP-HERE].

{mainmatter}

# Introduction

The Extensible Provisioning Protocol (EPP) [@!RFC5730] uses an on XML based protocol. This protocol is wel defined, using XML Schema Definition (XSD) for validation of XML messages, the XSDs are published as part of the EPP RFCs. This document describes rules for converting valid EPP XML messages to the JavaScript Object Notation (JSON) Data Interchange Format [@!RFC8259], for use with RESTful EPP (REPP).

# Terminology

In this document the following terminology is used.

EPP RFCs - This is a reference to the EPP version 1.0
specifications [@!RFC5730], [@!RFC5731], [@!RFC5732] and [@!RFC5733].

RESTful EPP or REPP - The RESTful transport for EPP described in
this document.


# Conventions Used in This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [@!RFC2119].

JSON is case sensitive. Unless stated otherwise, JSON specifications and examples provided in this document MUST be interpreted in the
character case presented. The examples in this document assume that request and response messages
are properly formatted JSON documents. Indentation and white space in examples are provided only to illustrate element relationships and for improving readability, and are not REQUIRED features of the protocol.


# Conversion Rules

A XML element may use one of 7 possible forms, the sections below describe how these forms MUST be translated to valid JSON.

1. Empty
2. Pure text content
3. Attributes only
4. Pure text content and attributes
5. Child elements with different names
6. Child elements with identical names
7. Child element(s) and contiguous text

## Empty

An empty XML element MUST be mapped to to a key matching the name of the element and a null value.

XML:
```xml
<hello/>
```

JSON:
```json
{
    "hello": null
}
```

##  Pure text content
 
An XML element containing text only MUST be mapped to a key matching the name of the element and the text MUST be used for the value

XML:
```xml
<lang>en</lang>
```

JSON:
```json
{
    "lang": "en"
}
```

## Attributes only
 
An XML element containing one or more atributes only, MUST be mapped to a JSON object matching the name of the element. Each XML attribute, prefixed using the `@` character, MUST be added as a key-value pair to the object.

XML:
```xml
<msgQ count="5" id="12345"/>
```

JSON:
```json
{
    "msgQ": {
        "@count": "5",
        "@id": "12345"
    }
}
```

## Pure text content and attributes
 
An XML element containing one or more atributes and text content only, MUST be mapped to a JSON object matching the name of the element. The text content MUST, prefixed using the string `#text`, MUST be added as a key-value pair to the object.

XML:
```xml
<msg lang="en">Command completed successfully</msg>
```

JSON:
```json
{
    "msg": {
        "@lang": "en",
        "#text": "Command completed successfully"
    }
}
```

## Child elements with different names
 
An XML element containing one or more child elements, where each child uses an unique name, MUST be mapped to a JSON object matching the name of the element. Each child element MUST be added as a key-value pair to the parent object.

XML:
```xml
<trID>
    <clTRID>ABC-12345</clTRID>
    <svTRID>54321-XYZ</svTRID>
</trID>
```

JSON:
```json
{
    "trID": {
        "clTRID": "ABC-12345",
        "svTRID": "54321-XYZ"
    }
}
```

## Child elements with identical names
 
An XML element containing multiple child elements, where multiple child elements use the same name, MUST be mapped to a JSON object containing an array. The name of the array MUST match the name of the non-unique children, each child element MUST be converted to JSON and added to the array.

XML:
```xml
<host>
    <addr>192.0.2.1</addr>
    <addr>192.0.2.2</addr>
</host>
```

JSON:
```json
{
    "host": {
        "addr": [
            "192.0.2.1",
            "192.0.2.2"
        ]
    }
}
```

## Child elements and contiguous text
 
An XML element containing one or more child elements and contiguous text, MUST be mapped to a JSON object containing a key-value entry for each child element, the text value MUST result in a key named `#text`. 

XML:
```xml
<msg lang="en">
    Credit balance low.
    <limit>100</limit>
    <bal>5</bal>
</msg>
```

JSON:
```json
{
    "msg": {
        "@lang": "en",
        "limit": 100,
        "bal": 5,
        "#text": "Credit balance low."
    }
}
```

When child elements are mixed with multiple text segments, the resulting `#text` key-value entry MUST be an array, containing all text segments.

XML:
```xml
<msg lang="en">
    Credit balance low.
    <limit>100</limit>
    <bal>5</bal>
    Please increase balance.
</msg>
```

JSON:
```json
{
    "msg": {
        "@lang": "en",
        "limit": 100,
        "bal": 5,
        "#text": ["Credit balance low.", "Please increase balance asap."]
    }
}
```

The rules above are based on the conversion approach found on [@?XMLCOM-WEB]


# Examples

This section lists examples for every EPP command supported by REPP, the examples  

## Hello

The Hello request message does not exist in the context of REPP.

Example XML response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <greeting>
        <svID>Example EPP server epp.example.com</svID>
        <svDate>2000-06-08T22:00:00.0Z</svDate>
        <svcMenu>
            <version>1.0</version>
            <lang>en</lang>
            <lang>fr</lang>
            <objURI>urn:ietf:params:xml:ns:obj1</objURI>
            <objURI>urn:ietf:params:xml:ns:obj2</objURI>
            <objURI>urn:ietf:params:xml:ns:obj3</objURI>
            <svcExtension>
                <extURI>http://custom/obj1ext-1.0</extURI>
            </svcExtension>
        </svcMenu>
        <dcp>
            <access>
                <all/>
            </access>
            <statement>
                <purpose>
                    <admin/>
                    <prov/>
                </purpose>
                <recipient>
                    <ours/>
                    <public/>
                </recipient>
                <retention>
                    <stated/>
                </retention>
            </statement>
        </dcp>
    </greeting>
</epp>
```
Example JSON response:

XML namespaces are not converted to JSON and are ignored.

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "greeting": {
            "svID": "Example REPP server v1.0",
            "svDate": "2000-06-08T22:00:00.0Z",
            "svcMenu": {
                "version": "1.0",
                "lang": [
                    "en",
                    "fr"
                ]
            },
            "dcp": {
                "access": {
                    "all": null
                },
                "statement": {
                    "purpose": {
                        "admin": null,
                        "prov": null
                    },
                    "recipient": {
                        "ours": null,
                        "public": null
                    },
                    "retention": {
                        "stated": null
                    }
                }
            }
        }
    }
}
```

##  Login

The Login request and response message are not used for REPP.

##  Logout

The Logout request and response message are not used for REPP.

## Check

The Check request and responses messages are not used for REPP.

## Info

The Info request message is not used for REPP.

Example XML Domain Info response:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <resData>
            <domain:infData
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:roid>EXAMPLE1-REP</domain:roid>
                <domain:status s="ok"/>
                <domain:registrant>jd1234</domain:registrant>
                <domain:contact type="admin">sh8013</domain:contact>
                <domain:contact type="tech">sh8013</domain:contact>
                <domain:ns>
                    <domain:hostObj>ns1.example.com</domain:hostObj>
                    <domain:hostObj>ns1.example.net</domain:hostObj>
                </domain:ns>
                <domain:host>ns1.example.com</domain:host>
                <domain:host>ns2.example.com</domain:host>
                <domain:clID>ClientX</domain:clID>
                <domain:crID>ClientY</domain:crID>
                <domain:crDate>1999-04-03T22:00:00.0Z</domain:crDate>
                <domain:upID>ClientX</domain:upID>
                <domain:upDate>1999-12-03T09:00:00.0Z</domain:upDate>
                <domain:exDate>2005-04-03T22:00:00.0Z</domain:exDate>
                <domain:trDate>2000-04-08T09:00:00.0Z</domain:trDate>
                <domain:authInfo>
                    <domain:pw>2fooBAR</domain:pw>
                </domain:authInfo>
            </domain:infData>
        </resData>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54322-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Info response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "resData": {
                "domain:infData": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:roid": "EXAMPLE1-REP",
                    "domain:status": {
                        "@s": "ok"
                    },
                    "domain:registrant": "jd1234",
                    "domain:contact": [
                        {
                            "@type": "admin",
                            "#text": "sh8013"
                        },
                        {
                            "@type": "tech",
                            "#text": "sh8013"
                        }
                    ],
                    "domain:ns": {
                        "domain:hostObj": [
                            "ns1.example.com",
                            "ns1.example.net"
                        ]
                    },
                    "domain:host": [
                        "ns1.example.com",
                        "ns2.example.com"
                    ],
                    "domain:clID": "ClientX",
                    "domain:crID": "ClientY",
                    "domain:crDate": "1999-04-03T22:00:00.0Z",
                    "domain:upID": "ClientX",
                    "domain:upDate": "1999-12-03T09:00:00.0Z",
                    "domain:exDate": "2005-04-03T22:00:00.0Z",
                    "domain:trDate": "2000-04-08T09:00:00.0Z",
                    "domain:authInfo": {
                        "domain:pw": "2fooBAR"
                    }
                }
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "54322-XYZ"
            }
        }
    }
}
```

## Poll

The Poll request message is not used for REPP.

Example XML response:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1301">
            <msg>Command completed successfully; ack to dequeue</msg>
        </result>
        <msgQ count="4" id="12346">
            <qDate>2000-06-08T22:10:00.0Z</qDate>
            <msg lang="en">Credit balance low.
                <limit>100</limit>
                <bal>5</bal>
            </msg>
        </msgQ>
        <trID>
            <clTRID>ABC-12346</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1301",
                "msg": "Command completed successfully; ack to dequeue"
            },
            "msgQ": {
                "@count": "4",
                "@id": "12346",
                "qDate": "2024-01-15T22:10:00.0Z",
                "msg": {
                    "@lang": "en",
                    "limit": "100",
                    "bal": "5",
                    "#text": "Credit balance low."
                }
            },
            "trID": {
                "clTRID": "ABC-12346",
                "svTRID": "54321-XYZ"
            }
        }
    }
}
```

## Poll Ack

The Poll Ack request message is not used for REPP.

Example XML response:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <msgQ count="0" id="12345"/>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>XYZ-12345</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "msgQ": {
                "@count": "0",
                "@id": "12345"
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "XYZ-12345"
            }
        }
    }
}
```


## Transfer Query

The Domain Transfer Query request message is not used for REPP.

Example XML  Domain Transfer Query response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <resData>
            <domain:trnData
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:trStatus>pending</domain:trStatus>
                <domain:reID>ClientX</domain:reID>
                <domain:reDate>2000-06-06T22:00:00.0Z</domain:reDate>
                <domain:acID>ClientY</domain:acID>
                <domain:acDate>2000-06-11T22:00:00.0Z</domain:acDate>
                <domain:exDate>2002-09-08T22:00:00.0Z</domain:exDate>
            </domain:trnData>
        </resData>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54322-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Transfer Query response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "resData": {
                "domain:trnData": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:trStatus": "pending",
                    "domain:reID": "ClientX",
                    "domain:reDate": "2000-06-06T22:00:00.0Z",
                    "domain:acID": "ClientY",
                    "domain:acDate": "2000-06-11T22:00:00.0Z",
                    "domain:exDate": "2002-09-08T22:00:00.0Z"
                }
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "54322-XYZ"
            }
        }
    }
}
```

## Create

Example XML Domain Create request:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <command>
        <create>
            <domain:create
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:period unit="y">2</domain:period>
                <domain:ns>
                    <domain:hostObj>ns1.example.net</domain:hostObj>
                    <domain:hostObj>ns2.example.net</domain:hostObj>
                </domain:ns>
                <domain:registrant>jd1234</domain:registrant>
                <domain:contact type="admin">sh8013</domain:contact>
                <domain:contact type="tech">sh8013</domain:contact>
                <domain:authInfo>
                    <domain:pw>2fooBAR</domain:pw>
                </domain:authInfo>
            </domain:create>
        </create>
        <clTRID>ABC-12345</clTRID>
    </command>
</epp>
```

Example JSON Domain Create request:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "command": {
            "create": {
                "domain:create": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:period": {
                        "@unit": "y",
                        "#text": "2"
                    },
                    "domain:ns": {
                        "domain:hostObj": [
                            "ns1.example.net",
                            "ns2.example.net"
                        ]
                    },
                    "domain:registrant": "jd1234",
                    "domain:contact": [
                        {
                            "@type": "admin",
                            "#text": "sh8013"
                        },
                        {
                            "@type": "tech",
                            "#text": "sh8013"
                        }
                    ],
                    "domain:authInfo": {
                        "domain:pw": "2fooBAR"
                    }
                }
            },
            "clTRID": "ABC-12345"
        }
    }
}
```


Example XML Domain Create response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <resData>
            <domain:creData
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:crDate>1999-04-03T22:00:00.0Z</domain:crDate>
                <domain:exDate>2001-04-03T22:00:00.0Z</domain:exDate>
            </domain:creData>
        </resData>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Create response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "resData": {
                "domain:creData": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:crDate": "1999-04-03T22:00:00.0Z",
                    "domain:exDate": "2001-04-03T22:00:00.0Z"
                }
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "54321-XYZ"
            }
        }
    }
}
```

## Delete

The Delete request message is not used for REPP.

Example XML Domain Delete response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Delete response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "54321-XYZ"
            }
        }
    }
}
```

## Renew

The Renew request message is not used for REPP.

Example XML Domain Renew response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <resData>
            <domain:renData
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:exDate>2005-04-03T22:00:00.0Z</domain:exDate>
            </domain:renData>
        </resData>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54322-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Renew response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "resData": {
                "domain:renData": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:exDate": "2005-04-03T22:00:00.0Z"
                }
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "54322-XYZ"
            }
        }
    }
}
```

## Transfer Request

The Transfer request message is not used for REPP.


Example XML Domain Transfer response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1001">
            <msg>Command completed successfully; action pending</msg>
        </result>
        <resData>
            <domain:trnData
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:trStatus>pending</domain:trStatus>
                <domain:reID>ClientX</domain:reID>
                <domain:reDate>2000-06-08T22:00:00.0Z</domain:reDate>
                <domain:acID>ClientY</domain:acID>
                <domain:acDate>2000-06-13T22:00:00.0Z</domain:acDate>
                <domain:exDate>2002-09-08T22:00:00.0Z</domain:exDate>
            </domain:trnData>
        </resData>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54322-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Transfer response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1001",
                "msg": "Command completed successfully; action pending"
            },
            "resData": {
                "domain:trnData": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:trStatus": "pending",
                    "domain:reID": "ClientX",
                    "domain:reDate": "2000-06-08T22:00:00.0Z",
                    "domain:acID": "ClientY",
                    "domain:acDate": "2000-06-13T22:00:00.0Z",
                    "domain:exDate": "2002-09-08T22:00:00.0Z"
                }
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "54322-XYZ"
            }
        }
    }
}
```



## Transfer Cancel

The Transfer Cancel request message is not used for REPP.


Example XML Domain Cancel Transfer response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>XYZ-12345</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Cancel Transfer response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "XYZ-12345"
            }
        }
    }
}
```

## Transfer Reject

The Transfer Reject request message is not used for REPP and the response message is the same as for the Transfer Cancel command.

## Transfer Approve

The Transfer Approve request message is not used for REPP and the response message is the same as for the Transfer Cancel command.

## Update

Example XML Domain Update request:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <command>
        <update>
            <domain:update
                xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
                <domain:name>example.com</domain:name>
                <domain:chg>
                    <domain:registrant>sh8013</domain:registrant>
                    <domain:authInfo>
                        <domain:pw>2BARfoo</domain:pw>
                    </domain:authInfo>
                </domain:chg>
            </domain:update>
        </update>
        <clTRID>ABC-12345</clTRID>
    </command>
</epp>
```

Example JSON Domain Update request:


```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "command": {
            "update": {
                "domain:update": {
                    "@xmlns:domain": "urn:ietf:params:xml:ns:domain-1.0",
                    "domain:name": "example.com",
                    "domain:chg": {
                        "domain:registrant": "sh8013",
                        "domain:authInfo": {
                            "domain:pw": "2BARfoo"
                        }
                    }
                }
            },
            "clTRID": "ABC-12345"
        }
    }
}
```

Example XML Domain Update response:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp
    xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="1000">
            <msg>Command completed successfully</msg>
        </result>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>XYZ-12345</svTRID>
        </trID>
    </response>
</epp>
```

Example JSON Domain Update response:

```json
{
    "epp": {
        "@xmlns": "urn:ietf:params:xml:ns:epp-1.0",
        "response": {
            "result": {
                "@code": "1000",
                "msg": "Command completed successfully"
            },
            "trID": {
                "clTRID": "ABC-12345",
                "svTRID": "XYZ-12345"
            }
        }
    }
}
```

# IANA Considerations

The new `application/epp+json` MIME media type is used in this document, the registration template is included in Appendix A.

# Internationalization Considerations

TODO

# Security Considerations

TODO

# Acknowledgments

TODO

{backmatter}

# Appendix A.  Media Type Registration: application/epp+json

   MIME media type name: application

   MIME subtype name: epp+json

   Required parameters: none

   Optional parameters: Same as the charset parameter of application/json
   as specified in [@!RFC8259].

   Encoding considerations: Same as the encoding considerations of
   application/xml as specified in [@!RFC8259].

   Security considerations: This type has all of the security
   considerations described in [@!RFC8259] plus the considerations
   specified in the Security Considerations section of this document.

   Published specification: This document.

   Applications that use this media type: RESTful EPP client and server implementations.
   
   Additional information: None

   Magic number(s): None.

   File extension(s): .json

   Macintosh file type code(s): "TEXT"

   Person & email address for further information: See the "Author's
   Address" section of this document.

   Intended usage: COMMON

   Author/Change controller: IETF

<reference anchor="XMLCOM-WEB" target="https://www.xml.com/pub/a/2006/05/31/converting-between-xml-and-json.html">
  <front>
    <title>Converting Between XML and JSON</title>
    <author>
      <organization>XML.com</organization>
    </author>
    <date year="2006"/>
  </front>
</reference>