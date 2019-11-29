# Smart Contract Invocation Protocol (SCIP)

**Version**: 1.0.0

**Date**: November 7, 2019

**Author**:  
  Ghareeb Falazi  
  Uwe Breitenbücher  
  Florian Daniel  
  Andrea Lamparelli  
  Frank Leymann  

## Introduction
This document specifies the Smart Contract Invocation Protocol (SCIP) as natural evolvement of the Smart Contract Locator ([SCL](https://bit.ly/2WRXvCZ)), intended to provide a protocol specification in the context of blockchains integration that allows external consumer applications to invoke smart contract functions in a uniform manner regardless of the underlying blockchain technology. Moreover it provides the capability to monitor smart contracts at runtime. The protocol assumes that the underlying blockchain technology is compatible with the Smart Contract Description Language ([SCDL](https://bit.ly/2JZUA5S)). This core of the interface consists of a set of *methods* that can be used by blockchain-external consumer applications to interact with *smart contracts*. The methods are provided by an entity, called *gateway* (actual SCIP endpoint), which mediates between two or more different network technologies: the Internet and the blockchain networks. This gateway is reachable using the aforementioned *Smart Contract Locator* ([SCL](https://bit.ly/2WRXvCZ)), which uniquely identifies a smart contract outside the blockchain. We assume that client applications authenticate themselves with the gateway using OAuth 2.0, and that attacks like the Man-In-The-Middle (MITM) are thus prevented.

<br/>

## Protocol Specification
The protocol define a total of four different allowed methods:

* The *invocation* of a smart contract function
* The *subscription* to notifications regarding function invocations or event occurrences
* The *unsubscription* from live monitoring
* The *querying* of past invocations or events

Every method is associated to a specific request message, all of them can be found in **Figure 1**, which shows the metamodel of all four request and two response messages. . The bold boxes represent the SCIP messages, whereas the regular ones represent the message content fields. For readability, the metamodel is split into related sub-models.

<br/><br/>

<figure>
    <img src="images/messages-metamodel.png" width="800px">
    <figcaption><strong>Figure 1</strong>  -  The metamodel of SCIP messages.</figcaption>
</figure>

<br/><br/>

All methods return a *synchronous* response message indicating the success or failure of the request, and some of them additionally return one or more *asynchronous* responses or errors. **Table 1** provides a detailed description of all call constructs defined in the previous metamodel.

<figure>
    <img src="images/fields-table.png" width="700px">
    <figcaption><strong>Table 1</strong>  -  Description of fields used in SCIP protocol.</figcaption>
</figure>

<br/><br/>

Some methods may require a point in time at which an event or a function took place, in this context the *time* refers to the UTC timestamp of the transaction that triggered the event or invoked the function. In particular the time is represented using the ISO 8601-1:2019 combined date and time representation. Certain other methods have a parameter called *degree of confidence* (DoC), which refers to the likelihood that a transaction included in a block will remain persistently stored on the blockchain. A value close to 1 means that the client application wants to receive the result only after ruling out the possibility that the block - including the transaction - may eventually be dropped from the blockchain, whereas a value close to 0 means that the client wants to receive the result as soon as it is available. 

The parameter structure is exactly the same defined in the *Smart Contract Description Language* specification ([SCDL](https://bit.ly/2JZUA5S)). Note that for the type we have used a technology-agnostic, abstract format using JSON Schema, this because blockchains support different encoding and types for parameters passed to or returned from smart contract functions or events, hence thanks to this proposed abstract format, developers can uniquely and abstractly describe native blockchain data types, and client applications can formulate function inputs in text-based JSON, without having to understand native data types. A complete table of abstract format types definition for different blockchains can be found [here](https://github.com/floriandanielit/scdl/blob/master/README.md#data-encoding).

<br/>

### Invocation

The invocation request is performed through the **Invoke** method, which allows an external application to invoke a specific smart contract function. The structure of the *Invocation* request message, as well as the asynchronous *Callback* message are explained in the **Figure 1a**. In particular this request message contains the name of the function and the list of input parameters, which allow to uniquely identify the specific function, a callback URL to which the gateway will send the asynchronous responses, a base64 encoding of the request message, and finally some optional fields like degree of confidence, correlation identifier, timeout and a list of output parameters that that specifies which parameters the client wants to receive as result from the function invocation (must be a subset of the actual return parameters of the function). A complete list of *Invocation Request* fields can be found in **Table 2**. A deeper description of how this request is handled by the gateway can be found in [invocation example](#step-by-step-invocation) section.

<br/>

<figure>
    <img src="images/invocation-table.png" width="650px">
    <figcaption><strong>Table 2</strong>  -  The structure of the <i>Invoke</i> request and response message</figcaption>
</figure>  

<br/><br/>

### Subscription

This request is executed using the **Subscribe** method, in particular this request facilitates the live monitoring of smart contracts by allowing a client application to receive asynchronous notifications about event occurrences or smart contract function invocations. The structure of the message is described in **Figure 1b**, notice that exactly one field, between function identifier and event identifier, must be included. When receiving this specific request, the gateway identifies the designated event/function using the appropriate *identifier*, the field *parameters* further helps the gateway to differentiate between overloads. Then the gateway starts monitoring the event or function and whenever an occurrence is detected, the associated event outputs / function inputs are used to populate and evaluate the Boolean expression specified in the filter. If the expression returns *true* and the transaction causing the occurrence has reached the specified *degree of confidence*, a *Callback* (**Figure 1a**) message to the address specified in *callback URL* is issued. If a new subscription request is made by a client application with a correlation identifier already in
use, then the old subscription is cancelled and replaced with the new one. This could happen, e.g., if a client application wants to change the expression of the filter, the value of the degree of confidence, or the callback URL of an existing subscription.

<br/>

<figure>
    <img src="images/subscribe-table.png" width="650px">
    <figcaption><strong>Table 3</strong>  -  The structure of the <i>Subscribe</i> request and response messages</figcaption>
</figure>  

<br/><br/>

### Unsubscription

This request, in particular the **Unsubscribe** method, is used to explicitly cancel subscriptions of a client, previously generated using invocations of the *Subscribe* method. The structure of this message is exlpained in **Figure 1b**. It has four optional fields, which can be used in three ways: (i) if only either function identifier
or event identifier plus parameters are present, then all respective subscriptions that belong to the target smart contract are cancelled; (ii) if only the correlation identifier is provided, then only the subscription corresponding to the identifier is cancelled; (iii) if none of the parameters is provided, then all subscriptions to the target smart contract are cancelled. All other combinations are invalid. This method does not have any asynchronous notifications, but only a synchronous one that indicates the success or not of the invocation.

<br/>

<figure>
    <img src="images/cancel-subscription-table.png" width="550px">
    <figcaption><strong>Table 4</strong>  -  The structure of the <i>Unsubscribe</i> request message.</figcaption>
</figure>  

<br/><br/>

### Querying

This request is performed through the **Query** method, its purpose is to allow client application to query previous occurrences of event or function invocations. It structure, as well as the synchronous response (**Query Result**) message are described in **Figure 1c**. When receiving a *Query* request message, the gateway scans the history of the blockchain and searches for event occurrences / function invocations with a prototype that matches the provided *event identifier/function identifier* and *parameters*. Furthermore the *timeframe* specifies the time frame in which the search results should be considered (by default the start time is the one of the genesis block, and the end time is the one of the latest block). Thanks to the *filter* field the client can specify a Boolean expression over the event/function parameters in order to select only those occurrences that satisfy it.

<br/>

<figure>
    <img src="images/querying-table.png" width="550px">
    <figcaption><strong>Table 5</strong>  -  The structure of the <i>Query</i> request and response messages.</figcaption>
</figure> 

<br/><br/>

### Step-by-Step Function Invocation

This section provides a step-by-step description of the **Invoke** method invocation. **Figure 2** shows the flowchart explaining the actions performed by both client application and gateway in case of a valid function invocation request id triggered. 

The client starts by formulating an *Invoke* request message (i) in according to the structure defined in the [invocation](#invocation) section.  Once the message is formulated the client has to sign it using the algorithm  "SHA256withECDSA" and the normative curve "secp256k1", and then send it to the SCIP gateway. At this point the gateway formulates a blockchain (technology-specific) transaction out of the request message (using the **identifier**, and **params** fields), and signs it on behalf of the client application. Afterwords it stored the pair defined by the signed transaction (Tx) and the Signed Request Message (SRM) for the purpose of non-repudiation (ii) .

<br/>

<figure>
    <img src="images/invoke-steps-fig.png" width="600px">
    <figcaption><strong>Figure 2</strong>  -  Steps performed by client and gateway during the execution of *Invoke* method</figcaption>
</figure>

<br/><br/>

The SRM exchange is mandatory whether the client and the gateway are managed by two different entities, otherwise it is not yet necessary. Once the transaction is formulated and signed, the gateway sends it to a blockchain node using its API (iii). The node, then, validates it, by executing the target smart contract function locally, and start the consensus process by announcing it  to the network of nodes (iv). Afterwords, it assigns a unique identifier to the transaction, and informs the gateway about it along the potential output values (v). Afterwords, the gateway informs the client application about the successful submission of the transaction (synchronous response to the original client request (i)) (vi), and at the same time, starts querying the blockchain node about the status of the transaction (vii). If the transaction receives enough confidence, in according to the **degree of confidence** field of the request message before the **timeout** is reached,  the gateway sends an asynchronous message to the address specified in **callback** field containing the execution results (viii).  Note that the gateway is allowed to have its own internal timeout for such requests, which may differ from the one provided by the client. Therefore, clients should expect an asynchronous *timeout error*. To facilitate the correlation between the request message and the response message by the client application, the callback contains a copy of the Correlation identifier provided in the request message.

<br/>

## JSON-RPC Binding

SCIP does not force to use a specific protocol for carrying all these messages, hence different bindings could be used. Here, we propose a JSON-RPC binding for SCIP, which is a stateless transport-agnostic remote procedure call protocol that uses JSON as its data format. JSON-RPC supports two Message Exchange Patterns (MEP) between a client and a server,  ***request-response*** and ***notification***. Since all our methods produce a synchronous response, we embed the initial message exchange in a request-response MEP,  where the client application plays the role of JSON-RPC client whereas the gateway plays the role of JSON-RPC server. Two methods (i.e., *Invoke* and *Subscribe*) also have one or more asynchronous responses, which we embed in notification MEP sent from the gateway to the client application, here the gateway and the client application switch their JSON-RPC roles.  

### Request

A JSON-RPC request message in a client-server MEP has the following template:

```
{
    "jsonrpc": "2.0",
    "method": <name>,
    "params": ...,
    "id": <id>
}
```

We embed a SCIP method invocation within this template in the following way: (i) we assign the method name to the ```"method"```  member; (ii) we formulate the request message into a JSON object and assign it to the  ```"params"```  member; (iii) we set an unused random identifier to the ```"id"``` member.

### Response

A JSON-RPC response message in a client-server MEP is a *synchronous* message that is sent by the server to the client, in response to a specific request made by the client. The response is used to provide a specific result if the request succeed, or an error message if the request failed for some reasons. In according to the nature of the response, the JSON-RPC provides two different templates.

#### Success Response

A JSON-RPC response message, when no *synchronous errors* are detected, i.e., errors that can be detected during the initial request-response MEP, has the following template:

```
{
    "jsonrpc": "2.0",
    "result": <value>,
    "id": <id>
}
```

We embed the synchronous successful response of SCIP methods within this template as follows: (i) we assign the JSON representation of the synchronous result of the method to the ```"result"``` member, and (ii) we assign the same id used in the request message to the ```"id"``` member. The synchronous result of an SCIP method could be represented either as an array of JSON objects, in the case of *querying methods*, or as the value ```"OK"```.

#### Error Response

Instead, when an error is detected, the synchronous response that indicates it has the following template:

```
{
    "jsonrpc": "2.0",
    "error": {
    	"code": <code>,
    	"message": <msg>,
    	"data": <data>
    },
    "id": <id>
}
```

As before, ```"id"``` is assigned to the same *id* used in the request message, whereas the ```"code"``` member is assigned to a suitable error code as indicated by Table 5.  Finally the ```"message"``` member is assigned a suitable description of the error and an optional primitive or structured ```data``` member can be added in order to provide additional information about the error. Note that, JSON-RPC-specific errors are also reported using such response messages.

<figure>
    <img src="images/errors-table.png" width="600px">
    <figcaption><strong>Table 5</strong>  -  Synchronous and Asynchronous SCIP errors and their codes.</figcaption>
</figure> 

### Notification

A JSON-RPC notification message has the same template as the request message but without the ```"id"``` member. The gateway uses such messages to return asynchronous responses to the client application, or to indicate that a *Timeout* error has occurred. In this scenario we have assumed the existence of a JSON-RPC method called **ReceiveCallback** at the client application side, which is used to receive these asynchronous responses. Note that in this case the gateway plays the role of the JSON-RPC client and the client application plays the role of the JSON-RPC server.



## Examples

A simple example of JSON-RPC messages exchange for the **Subscribe** to event SCIP method is the following (Here, ```-->``` indicates a message from the client application to the gateway, whereas ```<--``` indicates a message in the other direction):

```
// Client request
--> {
	"jsonrpc": "2.0", 
	"method": "Subscribe", 					        // Name of the request
	"id": 1,
	"params": {
		"eventId": "priceChanged",				// Name of the event
		"params": [{
			"name": "newPrice",				// Name of the parameter
			"type": {				        // Parameter type
				"type": "integer",
				"minimum": 0,
				"maximum": 65535
			}
		}],
		"doc": 98.9,						// Degree of Confidence 
		"corrId": "abcdefg12345",
		"callback": "https://my-domain.com/callbacks",
		"filter": "newPrice <= 500"
	}
}
	
// Synchronous response
<-- {
	"jsonrpc": "2.0", 
	"result": "OK",
	"id": 1,
}
	
// Asynchronous response
<-- {
	"jsonrpc": "2.0", 
	"method": "ReceiveCallback",
	"params": {
		"eventId": "priceChanged",				// Name of the event
		"params": [{
			"name": "newPrice",				// Name of the parameter
			"value": 410,    				// Value of the parameter
			...
		}],
		"corrId": "abcdefg12345",
		"timestamp": "2019-11-06T17:08:00Z"
	}
}
```
