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
This document specifies the Smart Contract Invocation Protocol (SCIP) natural evolvement of the Smart Contract Locator ([SCL](https://bit.ly/2WRXvCZ)), intended to provide a protocol specification in the context of blockchains integration that allows external consumer applications to invoke smart contract functions in a uniform manner regardless of the underlying blockchain technology. Moreover it provides the capability to monitor smart contracts at runtime. The protocol assumes that the underlying blockchain technology is compatible with the Smart Contract Description Language ([SCDL](https://bit.ly/2JZUA5S))

## Protocol Specification
The protocol defines a total of six methods that can be grouped into three categories:
* *Invocation of smart contracts function.*
* *Live-monitoring of function invocation and event occurrences.*
* *Querying of past invocations and events*

### Function Invocation

**The *InvokeFunction* Method**: This method is used to allow an external application to invoke a specific smart contract function. Figure 1 shows the steps taken by client application and gateway when this method is triggered: The client formulates an *InvokeFunction* request message (i) in according to the structure defined in table 1.

<figure>
    <img src="invoke-steps-fig.png" width="600px">
    <figcaption><strong>Figure 1</strong>  -  Steps performed by client and gateway during the execution of *InvokeFunction* method</figcaption>
</figure>

Once the message is formulated the client has to sign it using the algorithm  "SHA256withECDSA" and the normative curve "secp256k1", and then send it to the SCIP gateway. At this point the gateway formulates a blockchain (technology-specific) transaction out of the request message (using the **identifier**, and **params** fields), and signs it on behalf of the client application. Afterwords it stored the pair defined by the signed transaction (Tx) and the Signed Request Message (SRM) for the purpose of non-repudiation (ii) . The SRM exchange is mandatory whether the client and the gateway are managed by two different entities, otherwise it is not yet necessary. Once the transaction is formulated and signed, the gateway sends it to a blockchain node using its API (iii). The node, then, validates it, by executing the target smart contract function locally, and start the consensus process by announcing it  to the network of nodes (iv). Afterwords, it assigns a unique identifier to the transaction, and informs the gateway about it along the potential output values (v). Afterwords, the gateway informs the client application about the successful submission of the transaction (synchronous response to the original client request (i)) (vi), and at the same time, starts querying the blockchain node about the status of the transaction (vii). If the transaction receives enough confidence, in according to the Degree of Confidence provided by the client,  the gateway sends an asynchronous message to the address specified in **callback** field containing the execution results (viii).


<figure>
    <img src="invocation-table.png" width="600px">
    <figcaption><strong>Table 1</strong>  -  The structure of the <i>InvokeFunction</i> request and response message</figcaption>
</figure>



### Live-Monitoring
<img src="subscribe-table.png" width="550px" style="float: left; margin-right: 10px;"/>





<img src="cancel-subscription-table.png" width="550px" style="float: left; margin-right: 10px;"/>

### Querying
<img src="querying-table.png" width="550px" style="float: left; margin-right: 10px;"/>


## JSON-RPC Binding



