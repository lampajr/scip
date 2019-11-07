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
<img src="invocation-table.png" width="550px" style="float: left; margin-right: 10px;"/>

### Live-Monitoring
<img src="subscribe-table.png" width="550px" style="float: left; margin-right: 10px;"/>





<img src="cancel-subscription-table.png" width="550px" style="float: left; margin-right: 10px;"/>

### Querying
<img src="querying-table.png" width="550px" style="float: left; margin-right: 10px;"/>


## JSON-RPC Binding
