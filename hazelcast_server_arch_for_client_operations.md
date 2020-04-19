# Hazelcast Core Objects

Packet - Binary Data structure exchanged between Members.

ClientMessage - Binary Data structure exchanged between  clients and hazelcast server.

Operation - Hazelcast calls this as a Runnable. Base class for each action performed on a Distributed object. For example, put() operation on a map, get() operation on list, etc will be converted into `Operation` objects eventually on server. That `Operation` instance will have the corresponding logic to perform the action on the distributed object.

![Operation hierarchy](operation-hierarchy.png)

MessageTask - Interface for all the client messages that needs to be handled at hazelcast server.  For example, to put key/value for a Map operation, `ClientMessage` from client contains bytes for the map name and the key/value pair to be added. `MapPutMessageTask` (implementation of `MessageTask`) decodes the bytes, and creates `MapPutOperation`(implementation of `Operation`)



# hazelcast-architecture-analysis-for-client-operations

The following is an analysis of Hazelcast architecture for client operations on data structures.

 

Following is architecture view of the hazelcast server, when it is performing client operations.

![Hazelcast architecture for client operations](hazelcast_arch_flow_client_operations.png)

Hazelcast client sends the operations to be performed on data structures as `ClientMessage` to the server, using Open Binary Control Protocol.

Networking layer on receiving the message from client, decodes the bytes frame into `ClientMessage` and gives it to the Client Engine (`ClientMessage` consumer).

If the type of message is to perform an operation on data structure, client engine builds the appropriate `MessageTask` for the operation received, and will invoke operation service with the `MessageTask` for executing the operation.

MessageTask is a Runnable. Operation Service proxies the `MessageTask` to Operation Executor to execute the `MessageTask` in its own Partition thread.

Each Instance of a partition thread is responsible for a set of Partitions, and for each responsible partition,  partition threads holds Operation Runner. Operation Runner  eventually prepares the `Operation` to be executed. `Operation` is executed on the local objects and using BackupHandler & PartitionService, send the operations to remote members (partition replicas).