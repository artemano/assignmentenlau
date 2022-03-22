# Nodo de Procesamiento
Da cubrimiento a todas las necesidades transaccionales a través de la red de blockchain.

- **JSON-RPC.** Implementa el API JSON RPC o capa que permite acceder al blockchain. El stack provee los mecanismos de interacción con clientes de la red que usan JSON-RPC que debe implementarse sobre servidores JSON-RPC con WSS (Web Socket Connection). 
Implementa la interface con el el modulo de blockchain a través de:
    ```
    jsonrpc/blockchain.go
    ---
    type blockchainInterface interface {
        // Header returns the current header of the chain (genesis if empty)
        Header() *types.Header

        // GetReceiptsByHash returns the receipts for a hash
        GetReceiptsByHash(hash types.Hash) ([]*types.Receipt, error)

        // Subscribe subscribes for chain head events
        SubscribeEvents() blockchain.Subscription

        // GetHeaderByNumber returns the header by number
        GetHeaderByNumber(block uint64) (*types.Header, bool)

        // GetAvgGasPrice returns the average gas price
        GetAvgGasPrice() *big.Int

        // AddTx adds a new transaction to the tx pool
        AddTx(tx *types.Transaction) error

        // State returns a reference to the state
        State() state.State

        // BeginTxn starts a transition object
        BeginTxn(parentRoot types.Hash, header *types.Header) (*state.Transition, error)

        // GetBlockByHash gets a block using the provided hash
        GetBlockByHash(hash types.Hash, full bool) (*types.Block, bool)

        // ApplyTxn applies a transaction object to the blockchain
        ApplyTxn(header *types.Header, txn *types.Transaction) ([]byte, bool, error)

        stateHelperInterface
    }
    ```

- **Blockchain**. Es uno de los módulos principales que se encarga de las reorganizaciones de los bloques. Esto significa que se encarga de toda la lógica que ocurre cuando un nuevo bloque es añadido a la cadena de bloques. Aquí en este módulo STATE representa el objeto de transición y se encarga de todos los cambios en el estado cuando un nuevo bloque se añade. Gestiona:
  - Transacciones en ejecución
  - Ejecución del la EVM (Ethereum Virtual Machine)
  - Cambiar/Gestionar los arboles Merkle

    Entre las operaciones mas importantes realiza la escritura de bloques (WriteBlocks)y gestiona las subscripciones a través de eventos usando la interfaz Subscription
    
    ```
    blockchain/subscription.go
    type Subscription interface {
        // Returns a Blockchain Event channel
        GetEventCh() chan *Event
        
        // Returns the latest event (blocking)
        GetEvent() *Event
        
        // Closes the subscription
        Close()
    }
    ```
- **Txn Pool**. Estos reciben las solicitudes desde los clientes como agregadores para agilizar el proceso de validación en una sola transacción. Este móduo representa la implementación del pool de transacciones donde las transacciones son adicionas desde diferentes partes del sistema. Este módulo también expone varios características úitiles para operaciones de nodos. La interfaz se expone a través de los comandos *Status, AddTxn, y Subscribe*
    ```
    service TxnPoolOperator {
        // Status returns the current status of the pool
        rpc Status(google.protobuf.Empty) returns (TxnPoolStatusResp);

        // AddTxn adds a local transaction to the pool
        rpc AddTxn(AddTxnReq) returns (google.protobuf.Empty);

        // Subscribe subscribes for new events in the txpool
        rpc Subscribe(google.protobuf.Empty) returns (stream TxPoolEvent);
    }

    ```

- **Consensus**. Las transacciones son sometidas al proceso de validación en este componente que implementa el mecanimos via alguno de los plugis que implementan los diferentes métodos de concenso. Aquí se realiza el proceso de sincronización entre los nodos que participan en la prueba para luego ajuntar un nuevo bloque a la cadena de bloques en el módulo de blockchain. Actualmente soporta los siguientes motores de concenso:
  - IBFT PoA
  - IBFT PoS

    La interfaz hacia los mecanismos de concenso son:
    ```
    consensus/consensus.go
    // Consensus is the interface for consensus
    type Consensus interface {
        // VerifyHeader verifies the header is correct
        VerifyHeader(parent, header *types.Header) error

        // Start starts the consensus
        Start() error

        // Close closes the connection
        Close() error
    }
    ```
- **Libp2p** Este componente provee toda la capa de red P2P
- Otros Componentes no menos importantes:
  - Minimal. funciona como un hub de interconexión entre todos los módulos de polygon.
  - Storage. Utiliza LevelDB para almancenamiento de Datos y almacen de datos en memoria. Esta es una capa de abstracción que permita realizar consultas y persistenca a nviel de BD.

# **Lenguaje de Implementación** 
Go en versiones ~1.16 

### Instalación plataforma para desarrolllo
```
go install github.com/0xPolygon/polygon-edge@develop
```