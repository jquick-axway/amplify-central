---
title: Customize access request and credentials request screens
linkTitle: Customize access request and credentials request screens
weight: 300
---
Amplify allows you to personalize the *access request* and *credential* screens by customizing consumer questions that return appropriate responses.

## Before you start

* You understand the concepts involved in the [API Server](/docs/integrate_with_central/api_server/).
* You know how to use [Amplify Central CLI](/docs/integrate_with_central/cli_central)

## Objectives

Learn how to customize the access request and credentials request screen in Marketplace using [react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form) framework.

## Use cases

* A provider may want to ask extra information from his consumer to be able to correctly provision his request. For example, when a consumer wants to access a resource or create credentials, the definition of the access or credentials may require extra parameters.
* A provider may want to send back extra information to the consumer. For example, credential information or public keys, etc.

In both cases, a schema definition based on **react-jsonchema-form** must be implemented. This schema is used to convey the information from the consumer to the provider (`schema`) and from provider to consumer (`provision`).

By default when using Discovery Agents, the extra information is integrated when the agent discovers APIs based on the security type of the API (APIKey / OAuth - internal / OAuth - external).

{{< alert title="Note" color="primary" >}}It is also possible to manually manage what you want to ask/send to your consumer.{{< /alert >}}

### Available components

Various simple components are available to use: string text, date, number, arrays, dropdown, file selector and more. It is also possible to define your own object by combining multiple simple components.

In addition, there are two special parameters `x-axway-order` and `x-axway-encrypted` that extend this framework:

* **x-axway-order** - use to determine the order in which the selected fields will be presented in the UI
* **x-axway-encrypted** - use to tell the system that the field must be encrypted for security purpose

Each item is described using json format. Below is a non-exhaustive list:

* type: string, integer, number, boolean
* format: date-time, date, time
* title: the title used to display the component in the UI
* description (optional): text that will appear below the title to help the user understand the field component
* default (optional): the default value
* enum: list of text to display
* minimum / maximum: value range for integer field
* minLength / maxLength: value length for string
* arrays: defining an array of items

You can use the [react-jsonschema-form playground](https://rjsf-team.github.io/react-jsonschema-form/) to try out the various combination.

{{< alert title="Note" color="primary" >}}
The playground allows you to change the UI components (UISchenma section of the playground); however, the Marketplace UI does not use these components. As a result, the Marketplace UI may render differently than what is reflected in the playground. For this reason, think of the playground as a validator of your schema using the playground JSONSchema section.
{{< /alert >}}

#### Text component

```json
{
  "properties": {
    "firstName": {
      "type": "string",
      "title": "First name",
      "description": "Enter your first name. Range between 3 and 25 characters",
      "minLength": 3,
      "maxLength": 25,
    }
  }
}
```

#### Integer and number components

```json
 {
  "properties": {
    "age": {
      "type": "integer",
      "title": "Age",
      "description": "Enter your age",
      "minimum": 0,
      "maximum": 130
    },
    "weight": {
      "type": "number",
      "title": "Weight",
      "description": "Enter your weight using the . as decimal separator",
      "minimum": 0
    }
  }
}
```

#### Dropdown component

```json
{
  "properties": {
    "blendMode": {
      "title": "Security",
      "description": "Choose you API security",
      "type": "string",
      "oneOf": [
        {
          "const": "api-key",
          "title": "API Key"
        },
        {
          "const": "OAuth-cli-id",
          "title": "OAuth clientID/clientSecret"
        },
        {
          "const": "pass-through",
          "title": "No security (not recommended)"
        }
      ]
    }
  }
}
```

#### File selector component

```json
{
  "properties": {
    "file": {
      "type": "string",
      "format": "data-url",
      "title": "Upload a single file"
    },
    "files": {
      "type": "array",
      "title": "Upload multiple files",
      "items": {
        "type": "string",
        "format": "data-url"
      }
    }
  }
}
```

#### Composite component

```json
{
  "title": "A registration form",
  "description": "This is a simple form to collect data from a user",
  "type": "object",
  "required": [
    "firstName",
    "lastName"
  ],
  "properties": {
    "firstName": {
      "type": "string",
      "title": "First name"
    },
    "lastName": {
      "type": "string",
      "title": "Last name"
    },
    "age": {
      "type": "integer",
      "title": "Age",
      "minimum": 0
    },
    "address": {
      "type": "string",
      "title": "Street #"
    },
    "city": {
      "type": "string",
      "title": "Town"
    },
    "zipcode": {
      "type": "integer",
      "title": "Zipcode",
      "minlength": 5,
      "maxlength": 5
    },
    "phone": {
      "type": "string",
      "title": "Telephone",
      "minLength": 10
    },
    "x-axway-order": [
        "firstName",
        "latName",
        "age",
        "phone",
        "address",
        "city",
        "zipcode"
    ]
  }
}
```

## Customize access request screen

To customize the access request screen, you need an `AccessRequestDefinition`. This object will contain the screen definition of the information required to provision access to a Service and the output the provider wants to return to the consumer. This object is scoped per environment, meaning that if you have multiple environments, you must duplicate the access request definition for each individual environment.

Name of the object: **AccessRequestDefinition**

Scope: environment

Object skeleton (json format):

```json
{
    "group": "management",
    "apiVersion": "v1alpha1",
    "kind": "AccessRequestDefinition",
    "name": "name-for-access-request-definition",
    "title": "title-for-access-request-definition",
    "metadata": {
        "scope": {
            "kind": "Environment",
            "name": "environment-name",
        },
    },
    "attributes": {},
    "finalizers": [],
    "tags": [],
    "spec": {
        "schema": {
          ...
        },
        "provision": {
          ...
        }
    }
}
```

`accessRequestDefinition` contains two optional schemas in its specification section:

* **schema** - to define the information that is required from the consumer and how it is displayed on the screen.
* **provision** - for sending information back to the consumer.

These above two schemas follow the component framework describe in the [Highlight](#highlights) section.

Once the access request is created, the consumer can see the supplied information as well as the provisioned information (if any) by opening the *access request detail* page and navigating to the **Schema** section. "Input from consumer" refers to the accessRequestDefinition schema and "Provisioned data from dataplane" refers to what the provider sent to the consumer.

### AccessRequestDefinition sample

Sample of an AccessRequestDefinition (json format) asking consumer to select a purpose in a dropdown:

```json
{
    "group": "management",
    "apiVersion": "v1alpha1",
    "kind": "AccessRequestDefinition",
    "name": "api-scopes",
    "title": "API Scopes",
    "metadata": {
        "scope": {
            "kind": "Environment",
            "name": "environment-name",
        },
    },
    "attributes": {},
    "finalizers": [],
    "tags": [],
    "spec": {
        "schema": {
            "type": "object",
            "$schema": "http://json-schema.org/draft-07/schema",
            "properties": {
                "scopes": {
                    "type": "object",
                    "title": "Why Do You Want Access?",
                    "properties": {
                        "Toggle": {
                          "title": "Select a value",
                          "type": "string",
                          "oneOf": [
                            {
                              "title": "Just trying things out",
                              "const": "tryout"
                            },
                            {
                              "title": "Would like to perform test",
                              "const": "testing"
                            },
                            {
                              "title": "Beginning developments",
                              "const": "developing"
                            }      ]
                        }
                    }
                }
            },
            "description": "Please choose the best answer",
            "additionalProperties": false
        }
    }
}
```

Once the AccessRequestDefinition object is created using the Axway Central CLI (`axway central apply -f ard-api-scopes.json`), it must be linked to an APIServiceInstance.

Sample of an APIServiceInstance (json format) using the previous accessRequestDefinition:

```json
{
    "group": "management",
    "apiVersion": "v1alpha1",
    "kind": "APIServiceInstance",
    "name": "api-service-instance",
    "title": "api-service-instance-title",
    "metadata": {
        "scope": {
            "kind": "Environment",
            "name": "environment-name",
        },
        "acl": [],
    },
    "attributes": {},
    "finalizers": [],
    "tags": [],
    "spec": {
        "endpoint": [
            {
                "host": "myhost.axway.com",
                "port": 8065,
                "routing": {
                    "basePath": "/api/v1/service1"
                },
                "protocol": "https"
            }
        ],
        "apiServiceRevision": "api-service-revision",
        "accessRequestDefinition": "api-scopes",
        "credentialRequestDefinitions": []
    },
}
```

If the APIService instance is already linked to an asset prior to adding the access request definition, you must create a new release of the asset and a new release of the product to take the modification into consideration in the Marketplace.

Otherwise, create an asset based on this service and then a product.

You will be able to see the purpose selector once you publish the product to the marketplace, subscribe to it to get an active subscription, and then request access.

![Access Request screen](/Images/central/integrate_with_central/AccessRequestDefinition.png)

## Customize credential request screen

To customize the credential request screen, you need a `CredentialRequestDefinition`. This object will contain the screen definition of the information required to provision access to a Service and the output the provider wants to return to his consumer. This object is scoped per environment, meaning that if you have multiple environments, you must duplicate the access request definition for each individual environment.

Name of the object: **CredentialRequestDefinition**

Object skeleton (json format):

```json
{
    "group": "management",
    "apiVersion": "v1alpha1",
    "kind": "CredentialRequestDefinition",
    "name": "name-for-credential-request-definition",
    "title": "title-for-credential-request-definition",
    "metadata": {
        "scope": {
            "kind": "Environment",
            "name": "environment-name",
        },
    },
    "attributes": {},
    "finalizers": [],
    "tags": [],
    "spec": {
        "schema": {
          ...
        },
        "provision": {
          ...
        }
    }
}
```

The CredentialRequestDefinition contains two optional schemas in its specification section:

* **schema** - to define the information that is required from the consumer and how it is displayed on the screen.
* **provision** - for sending information back to the consumer.

Those above two schemas follow the component framework describe in the [Highlight](#highlights) section.

Once the credential is created, the consumer can see the supplied information as well as the provisioned information (if any) by opening the *credential detail* page and navigating to the **Credential value** section.

### CredentialRequestDefinition samples

Sample of the consumer giving the Javascript origin values and the provider returning the information provisioned (APIKey credential) in an encrypted value:

```json
{
        "group": "management",
        "apiVersion": "v1alpha1",
        "kind": "CredentialRequestDefinition",
        "name": "api-key",
        "title": "API Key",
        "metadata": {
            "scope": {
                "kind": "Environment",
                "name": "environment-name",
            },
        },
        "attributes": {},
        "finalizers": [],
        "tags": [],        "spec": {
            "schema": {
                "type": "object",
                "$schema": "http://json-schema.org/draft-07/schema#",
                "properties": {
                    "cors": {
                        "type": "array",
                        "items": {
                            "anyOf": [
                                {
                                    "type": "string",
                                    "title": ""
                                }
                            ]
                        },
                        "title": "Javascript Origins"
                    }
                },
                "description": "",
                "x-axway-order": [
                    "cors"
                ]
            },
            "provision": {
                "schema": {
                    "type": "object",
                    "$schema": "http://json-schema.org/draft-07/schema#",
                    "required": [
                        "apiKey"
                    ],
                    "properties": {
                        "apiKey": {
                            "type": "string",
                            "title": "API Key",
                            "x-axway-encrypted": true
                        }
                    },
                    "description": "",
                    "x-axway-order": [
                        "apiKey"
                    ]
                }
            }
        }
}
```

Sample of no information supplied by the consumer but information provisioned (OAuth credential) sent back to consumer:

```json
{
        "group": "management",
        "apiVersion": "v1alpha1",
        "kind": "CredentialRequestDefinition",
        "name": "oauth-client-credential",
        "title": "OAuth Client credential",
        "metadata": {
            "scope": {
                "kind": "Environment",
                "name": "environment-name",
            },
        },
        "attributes": {},
        "finalizers": [],
        "tags": [],
        "spec": {
            "schema": {
                "type": "object",
                "$schema": "http://json-schema.org/draft-07/schema#",
                "properties": {},
                "description": ""
            },
            "provision": {
                "schema": {
                    "type": "object",
                    "$schema": "http://json-schema.org/draft-07/schema#",
                    "required": [
                        "clientId",
                        "clientSecret"
                    ],
                    "properties": {
                        "clientId": {
                            "type": "string",
                            "title": "Client ID",
                            "description": ""
                        },
                        "clientSecret": {
                            "type": "string",
                            "title": "Client Secret",
                            "description": "",
                            "x-axway-encrypted": true
                        }
                    },
                    "description": ""
                }
            }
        }
}
```

Sample asking the consumer for a PEM public key file and returning oauth clientID:

```json
{
        "group": "management",
        "apiVersion": "v1alpha1",
        "kind": "CredentialRequestDefinition",
        "name": "oauth-client-certificate",
        "title": "OAuth Client Certificate",
        "metadata": {
            "scope": {
                "kind": "Environment",
                "name": "environment-name",
            },
        },
        "attributes": {},
        "finalizers": [],
        "tags": [],
        "spec": {
            "schema": {
                "type": "object",
                "$schema": "http://json-schema.org/draft-07/schema#",
                "required": [
                    "publicKey"
                ],
                "properties": {
                    "notes": {
                        "type": "string",
                        "title": "Notes",
                        "default": "Register the public key required for mTLS from the client to the server.",
                        "readOnly": true
                    },
                    "publicKey": {
                        "type": "string",
                        "title": "Public Key in PEM format",
                        "format": "data-url"
                    }
                }
            },
            "provision": {
                "schema": {
                    "type": "object",
                    "$schema": "http://json-schema.org/draft-07/schema#",
                    "required": [
                        "clientId"
                    ],
                    "properties": {
                        "clientId": {
                            "type": "string",
                            "description": "Client ID"
                        }
                    },
                    "description": "OAuth provisioning credentials"
                }
            }
        }
    }
```

Once the CredentialRequestDefinition object is created using the Axway central CLI (`axway central apply -f crd-pem-key.json`), you must link it to the corresponding APIServiceInstance.

Sample of an APIServiceInstance (json format) using the previous AccessRequestDefinition and a CredentialRequestDefinition:

```json
{
    "group": "management",
    "apiVersion": "v1alpha1",
    "kind": "APIServiceInstance",
    "name": "api-service-instance",
    "title": "api-service-instance-title",
    "metadata": {
        "scope": {
            "kind": "Environment",
            "name": "environment-name",
        },
        "acl": [],
    },
    "attributes": {},
    "finalizers": [],
    "tags": [],
    "spec": {
        "endpoint": [
            {
                "host": "myhost.axway.com",
                "port": 8065,
                "routing": {
                    "basePath": "/api/v1/service1"
                },
                "protocol": "https"
            }
        ],
        "apiServiceRevision": "api-service-revision",
        "accessRequestDefinition": "api-scopes",
        "credentialRequestDefinitions": [
          "oauth-client-certificate"
        ]
    },
}
```

If the APIService instance is already linked to an asset prior to adding the credential request definition, you must create a new release of the asset and a new release of the product to take the modification into consideration in the Marketplace.

Otherwise, create an asset based on this service and then a product.

You will be able to see the purpose selector once you publish the product to the Marketplace, subscribe to it to get an active subscription, and then request access.

![Access Request screen](/Images/central/integrate_with_central/AccessRequestDefinition.png)

Once your access is granted, you can ask for credential. You should be able to see the file selector:

![Credential Request screen](/Images/central/integrate_with_central/CredentialRequestDefinition.png)
