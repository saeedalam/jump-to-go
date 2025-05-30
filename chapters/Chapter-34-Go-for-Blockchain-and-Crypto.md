# **Chapter 34: Go for Blockchain and Cryptocurrency**

## **34.1 Introduction to Blockchain Development with Go**

Go has become one of the predominant languages in the blockchain space, powering some of the most popular blockchain platforms and tools. Its combination of performance, simplicity, and strong concurrency support makes it particularly well-suited for blockchain development.

### Go's Role in Blockchain Technology

Go's advantages for blockchain development include:

1. **Performance**: Blockchain nodes need to process transactions efficiently, and Go's compilation to native code with minimal runtime overhead helps achieve this.

2. **Concurrency**: Go's goroutines and channels are ideal for handling multiple concurrent operations in a blockchain network, such as transaction processing, consensus, and network communication.

3. **Simplicity**: Go's straightforward syntax and strong typing help build reliable blockchain systems with fewer bugs.

4. **Cross-platform Support**: Go applications can be easily compiled for different platforms, enabling blockchain nodes to run across diverse environments.

5. **Memory Safety**: Go's garbage collection and memory safety features help prevent memory-related vulnerabilities that could be exploited in a blockchain context.

### History of Go in Cryptocurrency Projects

Go's journey in the blockchain space began with smaller projects but quickly gained prominence:

- **2013-2014**: Early adoption in cryptocurrency tools and smaller blockchain projects
- **2015**: Ethereum introduced a Go implementation (go-ethereum or Geth)
- **2016-2017**: The ICO boom led to numerous Go-based blockchain projects
- **2018-present**: Mainstream adoption with multiple major blockchain platforms choosing Go

### Key Blockchain Projects Written in Go

Several prominent blockchain platforms and tools are written in Go:

#### Ethereum (go-ethereum/Geth)

Geth is the official Go implementation of the Ethereum protocol, making it one of the most widely deployed blockchain clients:

```go
// Example of using the go-ethereum library to interact with the Ethereum blockchain
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/ethclient"
)

func main() {
	// Connect to an Ethereum node
	client, err := ethclient.Dial("https://mainnet.infura.io/v3/YOUR-PROJECT-ID")
	if err != nil {
		log.Fatalf("Failed to connect to the Ethereum client: %v", err)
	}

	// Get the latest block number
	header, err := client.HeaderByNumber(context.Background(), nil)
	if err != nil {
		log.Fatalf("Failed to get latest header: %v", err)
	}
	blockNumber := header.Number

	// Get block information
	block, err := client.BlockByNumber(context.Background(), blockNumber)
	if err != nil {
		log.Fatalf("Failed to get block: %v", err)
	}

	// Print block details
	fmt.Printf("Block Number: %s\n", blockNumber.String())
	fmt.Printf("Block Time: %v\n", block.Time())
	fmt.Printf("Block Hash: %s\n", block.Hash().Hex())
	fmt.Printf("Number of Transactions: %d\n", len(block.Transactions()))
}
```

#### Hyperledger Fabric

Hyperledger Fabric is an enterprise-grade permissioned blockchain framework that uses Go for its core implementation:

```go
// Example of a simple Hyperledger Fabric chaincode (smart contract) in Go
package main

import (
	"fmt"
	"encoding/json"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// AssetTransfer defines the smart contract for managing assets
type AssetTransfer struct {
	contractapi.Contract
}

// Asset represents a general asset that can be tracked
type Asset struct {
	ID             string `json:"ID"`
	Owner          string `json:"Owner"`
	Value          int    `json:"Value"`
	LastModifiedBy string `json:"LastModifiedBy"`
}

// InitLedger adds a base set of assets to the ledger
func (a *AssetTransfer) InitLedger(ctx contractapi.TransactionContextInterface) error {
	assets := []Asset{
		{ID: "asset1", Owner: "Alice", Value: 100, LastModifiedBy: "initLedger"},
		{ID: "asset2", Owner: "Bob", Value: 200, LastModifiedBy: "initLedger"},
	}

	for _, asset := range assets {
		assetJSON, err := json.Marshal(asset)
		if err != nil {
			return fmt.Errorf("failed to marshal asset: %v", err)
		}

		err = ctx.GetStub().PutState(asset.ID, assetJSON)
		if err != nil {
			return fmt.Errorf("failed to put asset on ledger: %v", err)
		}
	}

	return nil
}

// CreateAsset creates a new asset on the ledger
func (a *AssetTransfer) CreateAsset(ctx contractapi.TransactionContextInterface, id string, owner string, value int) error {
	// Check if asset already exists
	exists, err := a.AssetExists(ctx, id)
	if err != nil {
		return fmt.Errorf("failed to check if asset exists: %v", err)
	}
	if exists {
		return fmt.Errorf("asset %s already exists", id)
	}

	// Get transaction submitter's identity
	clientID, err := ctx.GetClientIdentity().GetID()
	if err != nil {
		return fmt.Errorf("failed to get client identity: %v", err)
	}

	// Create the asset
	asset := Asset{
		ID:             id,
		Owner:          owner,
		Value:          value,
		LastModifiedBy: clientID,
	}

	// Store the asset on the ledger
	assetJSON, err := json.Marshal(asset)
	if err != nil {
		return fmt.Errorf("failed to marshal asset: %v", err)
	}

	return ctx.GetStub().PutState(id, assetJSON)
}

// AssetExists returns true if the asset with the given ID exists
func (a *AssetTransfer) AssetExists(ctx contractapi.TransactionContextInterface, id string) (bool, error) {
	assetJSON, err := ctx.GetStub().GetState(id)
	if err != nil {
		return false, fmt.Errorf("failed to read from world state: %v", err)
	}
	return assetJSON != nil, nil
}

// TransferAsset transfers an asset to a new owner
func (a *AssetTransfer) TransferAsset(ctx contractapi.TransactionContextInterface, id string, newOwner string) error {
	// Get the asset
	assetJSON, err := ctx.GetStub().GetState(id)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if assetJSON == nil {
		return fmt.Errorf("asset %s does not exist", id)
	}

	var asset Asset
	err = json.Unmarshal(assetJSON, &asset)
	if err != nil {
		return fmt.Errorf("failed to unmarshal asset: %v", err)
	}

	// Get transaction submitter's identity
	clientID, err := ctx.GetClientIdentity().GetID()
	if err != nil {
		return fmt.Errorf("failed to get client identity: %v", err)
	}

	// Update the asset
	asset.Owner = newOwner
	asset.LastModifiedBy = clientID

	// Store the updated asset
	assetJSON, err = json.Marshal(asset)
	if err != nil {
		return fmt.Errorf("failed to marshal asset: %v", err)
	}

	return ctx.GetStub().PutState(id, assetJSON)
}

func main() {
	assetChaincode, err := contractapi.NewChaincode(&AssetTransfer{})
	if err != nil {
		fmt.Printf("Error creating asset chaincode: %v\n", err)
		return
	}

	if err := assetChaincode.Start(); err != nil {
		fmt.Printf("Error starting asset chaincode: %v\n", err)
	}
}
```

#### Tendermint/Cosmos

Tendermint, a core component of the Cosmos ecosystem, is a consensus engine and blockchain application platform written in Go:

```go
// Example of a simple Cosmos SDK application module in Go
package bank

import (
	"encoding/json"
	"fmt"

	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/module"
)

// Module implements the cosmos-sdk Module interface
type Module struct {
	keeper Keeper
}

// NewModule creates a new bank Module
func NewModule(keeper Keeper) Module {
	return Module{
		keeper: keeper,
	}
}

// Name returns the module's name
func (m Module) Name() string {
	return "bank"
}

// RegisterInvariants registers the module's invariants
func (m Module) RegisterInvariants(ir sdk.InvariantRegistry) {
	ir.RegisterRoute(m.Name(), "total-supply", TotalSupply(m.keeper))
}

// TotalSupply checks that the total supply matches the sum of all account balances
func TotalSupply(keeper Keeper) sdk.Invariant {
	return func(ctx sdk.Context) (string, bool) {
		totalSupply := keeper.GetTotalSupply(ctx)
		totalAccountsBalance := keeper.GetTotalAccountsBalance(ctx)

		broken := !totalSupply.IsEqual(totalAccountsBalance)
		return sdk.FormatInvariant(
			"bank", "total-supply",
			fmt.Sprintf(
				"total supply %s != sum of accounts balances %s",
				totalSupply, totalAccountsBalance,
			),
		), broken
	}
}

// HandleMsg handles all the messages for the bank module
func (m Module) HandleMsg(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
	switch msg := msg.(type) {
	case MsgSend:
		return handleMsgSend(ctx, m.keeper, msg)
	default:
		return nil, sdk.ErrUnknownRequest(fmt.Sprintf("unrecognized bank message type: %T", msg))
	}
}

// MsgSend defines a message to send coins from one account to another
type MsgSend struct {
	FromAddress sdk.AccAddress `json:"from_address"`
	ToAddress   sdk.AccAddress `json:"to_address"`
	Amount      sdk.Coins      `json:"amount"`
}

// handleMsgSend handles the MsgSend message
func handleMsgSend(ctx sdk.Context, keeper Keeper, msg MsgSend) (*sdk.Result, error) {
	// Check if the sender has enough coins
	if !keeper.HasCoins(ctx, msg.FromAddress, msg.Amount) {
		return nil, sdk.ErrInsufficientFunds("insufficient funds")
	}

	// Transfer coins from sender to receiver
	err := keeper.SendCoins(ctx, msg.FromAddress, msg.ToAddress, msg.Amount)
	if err != nil {
		return nil, err
	}

	// Emit an event for the transfer
	ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			sdk.EventTypeMessage,
			sdk.NewAttribute(sdk.AttributeKeyModule, "bank"),
			sdk.NewAttribute(sdk.AttributeKeySender, msg.FromAddress.String()),
			sdk.NewAttribute(sdk.AttributeKeyRecipient, msg.ToAddress.String()),
			sdk.NewAttribute(sdk.AttributeKeyAmount, msg.Amount.String()),
		),
	)

	return &sdk.Result{
		Events: ctx.EventManager().ABCIEvents(),
	}, nil
}
```

#### Other Notable Go Blockchain Projects

- **Solana**: Uses Go for various tools and infrastructure components
- **Avalanche**: Uses Go for its core implementation
- **Polkadot**: Uses Go for some of its tools and infrastructure
- **Algorand**: Core node implementation is in Go
- **Filecoin**: Implements its blockchain in Go

Go's dominance in the blockchain space is a testament to its suitability for building secure, high-performance distributed systems. In the following sections, we'll explore how to build blockchain applications in Go, from cryptographic primitives to smart contract integration and beyond.

## **34.2 Cryptographic Fundamentals in Go**

Cryptography is the cornerstone of blockchain technology, and Go provides robust cryptographic capabilities through its standard library and ecosystem. This section explores the essential cryptographic primitives and how to implement them in Go.

### Cryptographic Primitives in the Standard Library

Go's standard library includes the `crypto` package and its subpackages, which provide implementations of various cryptographic algorithms:

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"io"
	"log"
)

func main() {
	// Generate a hash
	data := []byte("Hello, Blockchain!")
	hash := sha256.Sum256(data)
	fmt.Printf("SHA-256: %x\n", hash)

	// Symmetric encryption with AES
	plaintext := []byte("Secret blockchain data")

	// Generate a random key
	key := make([]byte, 32) // AES-256
	if _, err := io.ReadFull(rand.Reader, key); err != nil {
		log.Fatal(err)
	}

	// Create cipher block
	block, err := aes.NewCipher(key)
	if err != nil {
		log.Fatal(err)
	}

	// Create GCM mode
	gcm, err := cipher.NewGCM(block)
	if err != nil {
		log.Fatal(err)
	}

	// Generate a nonce
	nonce := make([]byte, gcm.NonceSize())
	if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
		log.Fatal(err)
	}

	// Encrypt
	ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
	fmt.Printf("Encrypted: %x\n", ciphertext)

	// Decrypt
	nonceSize := gcm.NonceSize()
	if len(ciphertext) < nonceSize {
		log.Fatal("Ciphertext too short")
	}

	nonce, ciphertext = ciphertext[:nonceSize], ciphertext[nonceSize:]
	decrypted, err := gcm.Open(nil, nonce, ciphertext, nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Decrypted: %s\n", decrypted)
}
```

Go's crypto package includes several important subpackages for blockchain development:

| Package         | Description                                       | Common Blockchain Use                     |
| --------------- | ------------------------------------------------- | ----------------------------------------- |
| `crypto/sha256` | SHA-256 hash algorithm                            | Transaction and block hashing             |
| `crypto/sha512` | SHA-512 hash algorithm                            | Enhanced security hashing                 |
| `crypto/ecdsa`  | Elliptic Curve Digital Signature Algorithm        | Transaction signing and verification      |
| `crypto/aes`    | Advanced Encryption Standard                      | Symmetric encryption for wallet security  |
| `crypto/rand`   | Cryptographically secure random number generation | Key generation, nonce creation            |
| `crypto/rsa`    | RSA encryption and signatures                     | Alternative signature schemes             |
| `crypto/hmac`   | Hash-based Message Authentication Codes           | Authentication and integrity verification |

### Hash Functions and Digital Signatures

Hash functions and digital signatures are fundamental to blockchain security. Here's how to implement them in Go:

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"fmt"
	"log"
	"math/big"
)

func main() {
	// Create a message to sign
	message := []byte("Transfer 1.5 BTC to address 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa")

	// Hash the message (single round)
	hash1 := sha256.Sum256(message)
	fmt.Printf("SHA-256: %x\n", hash1)

	// Double SHA-256 (used in Bitcoin)
	hash2 := sha256.Sum256(hash1[:])
	fmt.Printf("Double SHA-256: %x\n", hash2)

	// Generate ECDSA key pair
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		log.Fatal(err)
	}

	// Sign the hash
	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash1[:])
	if err != nil {
		log.Fatal(err)
	}

	// Verify the signature
	valid := ecdsa.Verify(&privateKey.PublicKey, hash1[:], r, s)
	fmt.Printf("Signature valid: %v\n", valid)

	// Export the signature components
	fmt.Printf("Signature r: %x\n", r.Bytes())
	fmt.Printf("Signature s: %x\n", s.Bytes())
}
```

#### Blockchain Hashing in Practice

In blockchain systems, hashing is used in various ways:

1. **Transaction Hashing**: Each transaction is hashed to create a unique identifier
2. **Merkle Trees**: Transactions are organized in Merkle trees for efficient verification
3. **Block Hashing**: The block header is hashed to create a unique block identifier
4. **Proof of Work**: Miners repeatedly hash block headers with different nonces to find a hash below a target difficulty

Here's a simple implementation of a Merkle tree, which is fundamental to blockchain data structures:

```go
package main

import (
	"crypto/sha256"
	"fmt"
)

// MerkleNode represents a node in a Merkle tree
type MerkleNode struct {
	Left  *MerkleNode
	Right *MerkleNode
	Data  []byte
}

// NewMerkleNode creates a new Merkle tree node
func NewMerkleNode(left, right *MerkleNode, data []byte) *MerkleNode {
	node := MerkleNode{}

	if left == nil && right == nil {
		// Leaf node - hash the data
		hash := sha256.Sum256(data)
		node.Data = hash[:]
	} else {
		// Internal node - hash the concatenation of children
		prevHashes := append(left.Data, right.Data...)
		hash := sha256.Sum256(prevHashes)
		node.Data = hash[:]
	}

	node.Left = left
	node.Right = right

	return &node
}

// NewMerkleTree creates a new Merkle tree from a list of data
func NewMerkleTree(data [][]byte) *MerkleNode {
	var nodes []*MerkleNode

	// Create leaf nodes
	for _, datum := range data {
		node := NewMerkleNode(nil, nil, datum)
		nodes = append(nodes, node)
	}

	// If we have an odd number of leaves, duplicate the last one
	if len(nodes)%2 != 0 {
		nodes = append(nodes, nodes[len(nodes)-1])
	}

	// Build the tree bottom-up
	for len(nodes) > 1 {
		var level []*MerkleNode

		for i := 0; i < len(nodes); i += 2 {
			node := NewMerkleNode(nodes[i], nodes[i+1], nil)
			level = append(level, node)
		}

		// If we have an odd number of nodes at this level, duplicate the last one
		if len(level)%2 != 0 && len(level) > 1 {
			level = append(level, level[len(level)-1])
		}

		nodes = level
	}

	// Return the root node
	return nodes[0]
}

func main() {
	// Example transactions
	data1 := []byte("Transaction 1: Alice sends 1 BTC to Bob")
	data2 := []byte("Transaction 2: Bob sends 0.5 BTC to Charlie")
	data3 := []byte("Transaction 3: Charlie sends 0.25 BTC to Dave")
	data4 := []byte("Transaction 4: Dave sends 0.1 BTC to Eve")

	// Create a Merkle tree from transactions
	tree := NewMerkleTree([][]byte{data1, data2, data3, data4})

	// Print the Merkle root
	fmt.Printf("Merkle Root: %x\n", tree.Data)

	// Create a second tree with the same data to verify
	tree2 := NewMerkleTree([][]byte{data1, data2, data3, data4})
	fmt.Printf("Second Merkle Root: %x\n", tree2.Data)
	fmt.Printf("Roots match: %v\n", string(tree.Data) == string(tree2.Data))

	// Create a tree with modified data to show how it changes the root
	data3Modified := []byte("Transaction 3: Charlie sends 1.25 BTC to Dave")
	treeModified := NewMerkleTree([][]byte{data1, data2, data3Modified, data4})
	fmt.Printf("Modified Merkle Root: %x\n", treeModified.Data)
	fmt.Printf("Original and modified roots match: %v\n", string(tree.Data) == string(treeModified.Data))
}
```

### Key Generation and Management

Proper key management is crucial for blockchain applications. Here's how to implement secure key handling in Go:

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"fmt"
	"log"
)

// Generate a new ECDSA key pair
func generateKeyPair() (*ecdsa.PrivateKey, error) {
	return ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
}

// Derive an address from a public key (simplified)
func deriveAddress(publicKey *ecdsa.PublicKey) string {
	// Serialize the public key
	x := publicKey.X.Bytes()
	y := publicKey.Y.Bytes()
	pubKeyBytes := append(x, y...)

	// Hash the public key
	hash := sha256.Sum256(pubKeyBytes)

	// In real blockchain systems, additional steps like RIPEMD-160
	// and checksums would be applied

	// Return first 20 bytes as hex (similar to Ethereum)
	return hex.EncodeToString(hash[:20])
}

// Sign a message with a private key
func signMessage(privateKey *ecdsa.PrivateKey, message []byte) ([]byte, error) {
	// Hash the message
	hash := sha256.Sum256(message)

	// Sign the hash
	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:])
	if err != nil {
		return nil, err
	}

	// Combine r and s into a single signature
	rBytes := r.Bytes()
	sBytes := s.Bytes()
	signature := append(rBytes, sBytes...)

	return signature, nil
}

// Verify a signature against a public key
func verifySignature(publicKey *ecdsa.PublicKey, message, signature []byte) bool {
	// Hash the message
	hash := sha256.Sum256(message)

	// Split signature into r and s components
	// This is simplified; in practice, parsing would be more robust
	sigLen := len(signature)
	r := new(big.Int).SetBytes(signature[:sigLen/2])
	s := new(big.Int).SetBytes(signature[sigLen/2:])

	// Verify the signature
	return ecdsa.Verify(publicKey, hash[:], r, s)
}

func main() {
	// Generate a key pair
	privateKey, err := generateKeyPair()
	if err != nil {
		log.Fatal(err)
	}

	// Derive address from public key
	address := deriveAddress(&privateKey.PublicKey)
	fmt.Printf("Derived address: %s\n", address)

	// Create a message to sign
	message := []byte("Send 5 BTC to address 3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy")

	// Sign the message
	signature, err := signMessage(privateKey, message)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Signature: %x\n", signature)

	// Verify the signature
	valid := verifySignature(&privateKey.PublicKey, message, signature)
	fmt.Printf("Signature valid: %v\n", valid)

	// Try to verify with a tampered message
	tamperedMessage := []byte("Send 50 BTC to address 3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy")
	validTampered := verifySignature(&privateKey.PublicKey, tamperedMessage, signature)
	fmt.Printf("Tampered message signature valid: %v\n", validTampered)
}
```

### Secure Random Number Generation

Secure random number generation is essential for cryptographic operations. Go provides the `crypto/rand` package for this purpose:

```go
package main

import (
	"crypto/rand"
	"encoding/hex"
	"fmt"
	"log"
	"math/big"
)

func main() {
	// Generate random bytes (e.g., for a private key)
	privateKeyBytes := make([]byte, 32)
	_, err := rand.Read(privateKeyBytes)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Random private key: %x\n", privateKeyBytes)

	// Generate a random number between 0 and 2^256-1
	max := new(big.Int)
	max.Exp(big.NewInt(2), big.NewInt(256), nil)
	max.Sub(max, big.NewInt(1))

	n, err := rand.Int(rand.Reader, max)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Random number: %s\n", n.String())

	// Generate a random nonce for mining
	miningNonce, err := rand.Int(rand.Reader, big.NewInt(4294967295)) // 2^32 - 1
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Mining nonce: %d\n", miningNonce)
}
```

### Building a Simplified Transaction Signing System

Let's combine these concepts to build a simplified transaction signing system:

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"log"
	"math/big"
	"time"
)

// Transaction represents a basic blockchain transaction
type Transaction struct {
	Sender    string `json:"sender"`
	Recipient string `json:"recipient"`
	Amount    string `json:"amount"`
	Nonce     uint64 `json:"nonce"`
	Timestamp int64  `json:"timestamp"`
	Signature string `json:"signature,omitempty"`
}

// NewTransaction creates a new unsigned transaction
func NewTransaction(sender, recipient, amount string, nonce uint64) *Transaction {
	return &Transaction{
		Sender:    sender,
		Recipient: recipient,
		Amount:    amount,
		Nonce:     nonce,
		Timestamp: time.Now().Unix(),
	}
}

// Hash calculates the hash of the transaction
func (tx *Transaction) Hash() []byte {
	// Create a copy without the signature
	txCopy := *tx
	txCopy.Signature = ""

	// Marshal to JSON
	jsonData, err := json.Marshal(txCopy)
	if err != nil {
		log.Fatalf("Failed to marshal transaction: %v", err)
	}

	// Return the SHA-256 hash
	hash := sha256.Sum256(jsonData)
	return hash[:]
}

// Sign signs the transaction with a private key
func (tx *Transaction) Sign(privateKey *ecdsa.PrivateKey) error {
	// Get the transaction hash
	hash := tx.Hash()

	// Sign the hash
	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash)
	if err != nil {
		return err
	}

	// Combine r and s into a signature
	rBytes := r.Bytes()
	sBytes := s.Bytes()
	signature := append(rBytes, sBytes...)

	// Store the signature as hex
	tx.Signature = hex.EncodeToString(signature)
	return nil
}

// Verify verifies the transaction signature
func (tx *Transaction) Verify(publicKey *ecdsa.PublicKey) bool {
	// Cannot verify if no signature
	if tx.Signature == "" {
		return false
	}

	// Get the transaction hash
	hash := tx.Hash()

	// Decode the signature
	signatureBytes, err := hex.DecodeString(tx.Signature)
	if err != nil {
		return false
	}

	// Split signature into r and s
	// This is simplified; in practice, parsing would be more robust
	sigLen := len(signatureBytes)
	r := new(big.Int).SetBytes(signatureBytes[:sigLen/2])
	s := new(big.Int).SetBytes(signatureBytes[sigLen/2:])

	// Verify the signature
	return ecdsa.Verify(publicKey, hash, r, s)
}

func main() {
	// Generate a key pair for the sender
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		log.Fatal(err)
	}

	// Derive address from public key (simplified)
	pubKeyBytes := elliptic.Marshal(elliptic.P256(), privateKey.PublicKey.X, privateKey.PublicKey.Y)
	pubKeyHash := sha256.Sum256(pubKeyBytes)
	senderAddress := hex.EncodeToString(pubKeyHash[:20])

	// Create a recipient address
	recipientAddress := "3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy"

	// Create a new transaction
	tx := NewTransaction(senderAddress, recipientAddress, "1.5 BTC", 1)

	// Sign the transaction
	if err := tx.Sign(privateKey); err != nil {
		log.Fatal(err)
	}

	// Verify the signature
	valid := tx.Verify(&privateKey.PublicKey)

	// Print transaction details
	fmt.Printf("Transaction:\n")
	fmt.Printf("  Sender:    %s\n", tx.Sender)
	fmt.Printf("  Recipient: %s\n", tx.Recipient)
	fmt.Printf("  Amount:    %s\n", tx.Amount)
	fmt.Printf("  Nonce:     %d\n", tx.Nonce)
	fmt.Printf("  Timestamp: %d\n", tx.Timestamp)
	fmt.Printf("  Signature: %s\n", tx.Signature)
	fmt.Printf("  Valid:     %v\n", valid)

	// Demonstrate tamper detection
	tamperedTx := *tx
	tamperedTx.Amount = "10.0 BTC"
	tamperedValid := tamperedTx.Verify(&privateKey.PublicKey)
	fmt.Printf("\nTampered transaction valid: %v\n", tamperedValid)
}
```

### Secure Key Storage Patterns

Securely storing private keys is crucial for blockchain applications. Here's a simple example of encrypted key storage:

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"golang.org/x/crypto/scrypt"
	"io"
	"io/ioutil"
	"log"
	"os"
	"path/filepath"
	"time"
)

// EncryptedKeystore represents an encrypted private key
type EncryptedKeystore struct {
	IV        string `json:"iv"`
	Ciphertext string `json:"ciphertext"`
	Salt       string `json:"salt"`
	KDF        string `json:"kdf"`
	KDFParams  struct {
		N      int    `json:"n"`
		R      int    `json:"r"`
		P      int    `json:"p"`
		DKLen  int    `json:"dklen"`
	} `json:"kdfparams"`
	MAC        string `json:"mac"`
}

// WalletData represents the wallet data to be encrypted
type WalletData struct {
	Mnemonic   string            `json:"mnemonic"`
	PrivateKey string            `json:"private_key"`
	Addresses  map[string]string `json:"addresses"`
}

// encryptWallet encrypts wallet data with a password
func encryptWallet(data *WalletData, password string) (*EncryptedKeystore, error) {
	// Convert wallet data to JSON
	dataJSON, err := json.Marshal(data)
	if err != nil {
		return nil, fmt.Errorf("failed to marshal wallet data: %w", err)
	}

	// Generate random salt
	salt := make([]byte, 32)
	if _, err := io.ReadFull(rand.Reader, salt); err != nil {
		return nil, err
	}

	// Derive key from password using scrypt
	derivedKey, err := scrypt.Key([]byte(password), salt, 1<<18, 8, 1, 32)
	if err != nil {
		return nil, err
	}

	// Generate random IV
	iv := make([]byte, 16)
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return nil, err
	}

	// Create AES cipher
	block, err := aes.NewCipher(derivedKey[:16])
	if err != nil {
		return nil, err
	}

	// Encrypt data
	ciphertext := make([]byte, len(dataJSON))
	stream := cipher.NewCTR(block, iv)
	stream.XORKeyStream(ciphertext, dataJSON)

	// Calculate MAC
	mac := sha256.Sum256(append(derivedKey[16:], ciphertext...))

	// Create encrypted wallet
	encrypted := &EncryptedKeystore{
		IV:         hex.EncodeToString(iv),
		Ciphertext: hex.EncodeToString(ciphertext),
		Salt:       hex.EncodeToString(salt),
		KDF:        "scrypt",
		MAC:        hex.EncodeToString(mac[:]),
	}
	encrypted.KDFParams.N = scryptParams.N
	encrypted.KDFParams.R = scryptParams.R
	encrypted.KDFParams.P = scryptParams.P
	encrypted.KDFParams.DKLen = scryptParams.DKLen

	return encrypted, nil
}

// decryptWallet decrypts an encrypted wallet with a password
func decryptWallet(encrypted *EncryptedKeystore, password string) (*WalletData, error) {
	// Decode hex strings
	iv, err := hex.DecodeString(encrypted.IV)
	if err != nil {
		return nil, fmt.Errorf("failed to decode IV: %w", err)
	}

	ciphertext, err := hex.DecodeString(encrypted.Ciphertext)
	if err != nil {
		return nil, fmt.Errorf("failed to decode ciphertext: %w", err)
	}

	salt, err := hex.DecodeString(encrypted.Salt)
	if err != nil {
		return nil, fmt.Errorf("failed to decode salt: %w", err)
	}

	mac, err := hex.DecodeString(encrypted.MAC)
	if err != nil {
		return nil, fmt.Errorf("failed to decode MAC: %w", err)
	}

	// Derive key from password
	derivedKey, err := scrypt.Key(
		[]byte(password),
		salt,
		encrypted.KDFParams.N,
		encrypted.KDFParams.R,
		encrypted.KDFParams.P,
		encrypted.KDFParams.DKLen,
	)
	if err != nil {
		return nil, fmt.Errorf("failed to derive key: %w", err)
	}

	// Verify MAC
	calculatedMAC := sha256.Sum256(append(derivedKey[16:], ciphertext...))
	if !hmac.Equal(calculatedMAC[:], mac) {
		return nil, fmt.Errorf("invalid password (MAC mismatch)")
	}

	// Decrypt data
	block, err := aes.NewCipher(derivedKey[:16])
	if err != nil {
		return nil, fmt.Errorf("failed to create cipher: %w", err)
	}

	plaintext := make([]byte, len(ciphertext))
	stream := cipher.NewCTR(block, iv)
	stream.XORKeyStream(plaintext, ciphertext)

	// Unmarshal wallet data
	var walletData WalletData
	if err := json.Unmarshal(plaintext, &walletData); err != nil {
		return nil, fmt.Errorf("failed to unmarshal wallet data: %w", err)
	}

	return &walletData, nil
}

// saveWalletToFile saves an encrypted wallet to a file
func saveWalletToFile(encrypted *EncryptedKeystore, filePath string) error {
	// Create directory if it doesn't exist
	if err := os.MkdirAll(filepath.Dir(filePath), 0700); err != nil {
		return fmt.Errorf("failed to create directory: %w", err)
	}

	// Marshal to JSON
	data, err := json.MarshalIndent(encrypted, "", "  ")
	if err != nil {
		return fmt.Errorf("failed to marshal wallet: %w", err)
	}

	// Write to file
	if err := ioutil.WriteFile(filePath, data, 0600); err != nil {
		return fmt.Errorf("failed to write file: %w", err)
	}

	return nil
}

// loadWalletFromFile loads an encrypted wallet from a file
func loadWalletFromFile(filePath string) (*EncryptedKeystore, error) {
	// Read file
	data, err := ioutil.ReadFile(filePath)
	if err != nil {
		return nil, fmt.Errorf("failed to read file: %w", err)
	}

	// Unmarshal JSON
	var encrypted EncryptedKeystore
	if err := json.Unmarshal(data, &encrypted); err != nil {
		return nil, fmt.Errorf("failed to unmarshal wallet: %w", err)
	}

	return &encrypted, nil
}

func main() {
	// Sample wallet data
	walletData := &WalletData{
		Mnemonic:   "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about",
		PrivateKey: "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
		Addresses: map[string]string{
			"eth": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
			"btc": "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa",
		},
	}

	// Encrypt wallet
	password := "supersecurepassword"
	encryptedWallet, err := encryptWallet(walletData, password)
	if err != nil {
		log.Fatalf("Failed to encrypt wallet: %v", err)
	}

	// Save to file
	filePath := "./wallet.json"
	if err := saveWalletToFile(encryptedWallet, filePath); err != nil {
		log.Fatalf("Failed to save wallet: %v", err)
	}

	fmt.Printf("Saved encrypted wallet to: %s\n", filePath)
}
```

These examples demonstrate the key components of a blockchain wallet implementation in Go:

1. **Key Generation**: Creating secure cryptographic key pairs
2. **Transaction Signing**: Preparing and signing blockchain transactions
3. **Mnemonic Phrases**: Implementing BIP-39 for seed generation from words
4. **HD Wallet Derivation**: Using BIP-44 to derive multiple accounts from a single seed
5. **Secure Storage**: Encrypting and storing wallet data securely

In a production environment, you would typically use established libraries like go-ethereum's accounts package or btcutil for these operations, but understanding the underlying concepts is essential for blockchain developers.

## **34.3 Building a Simple Blockchain**

Now that we understand the cryptographic fundamentals, let's build a simple blockchain from scratch in Go. This implementation will cover the core concepts: blocks, chains, proof of work, and transaction validation.

### Core Blockchain Data Structures

The fundamental data structure in a blockchain is the block. Each block contains a header with metadata and a list of transactions:

```go
package main

import (
	"bytes"
	"crypto/sha256"
	"encoding/gob"
	"encoding/hex"
	"fmt"
	"log"
	"math"
	"math/big"
	"time"
)

// Block represents a block in the blockchain
type Block struct {
	Timestamp     int64
	Transactions  []*Transaction
	PrevBlockHash []byte
	Hash          []byte
	Nonce         int
	Difficulty    int
}

// NewBlock creates and returns a new block
func NewBlock(transactions []*Transaction, prevBlockHash []byte, difficulty int) *Block {
	block := &Block{
		Timestamp:     time.Now().Unix(),
		Transactions:  transactions,
		PrevBlockHash: prevBlockHash,
		Hash:          []byte{},
		Nonce:         0,
		Difficulty:    difficulty,
	}

	block.MineBlock()

	return block
}

// NewGenesisBlock creates the genesis block
func NewGenesisBlock(coinbase *Transaction, difficulty int) *Block {
	return NewBlock([]*Transaction{coinbase}, []byte{}, difficulty)
}

// HashTransactions creates a hash of all transactions in the block
func (b *Block) HashTransactions() []byte {
	var txHashes [][]byte

	for _, tx := range b.Transactions {
		txHashes = append(txHashes, tx.Hash())
	}

	// Create a merkle tree from transaction hashes
	tree := NewMerkleTree(txHashes)

	return tree.Data
}

// Serialize serializes the block
func (b *Block) Serialize() []byte {
	var result bytes.Buffer
	encoder := gob.NewEncoder(&result)

	err := encoder.Encode(b)
	if err != nil {
		log.Panic(err)
	}

	return result.Bytes()
}

// DeserializeBlock deserializes a block
func DeserializeBlock(d []byte) *Block {
	var block Block

	decoder := gob.NewDecoder(bytes.NewReader(d))
	err := decoder.Decode(&block)
	if err != nil {
		log.Panic(err)
	}

	return &block
}

// Blockchain represents a chain of blocks
type Blockchain struct {
	Blocks       []*Block
	Difficulty   int
	MiningReward int
}

// NewBlockchain creates a new blockchain with a genesis block
func NewBlockchain(address string, difficulty, miningReward int) *Blockchain {
	// Create coinbase transaction for the genesis block
	coinbaseTx := NewCoinbaseTX(address, "Genesis Block")
	genesis := NewGenesisBlock(coinbaseTx, difficulty)

	return &Blockchain{
		Blocks:       []*Block{genesis},
		Difficulty:   difficulty,
		MiningReward: miningReward,
	}
}

// AddBlock adds a new block to the blockchain
func (bc *Blockchain) AddBlock(transactions []*Transaction) {
	// Validate transactions
	var validTxs []*Transaction
	for _, tx := range transactions {
		if bc.VerifyTransaction(tx) {
			validTxs = append(validTxs, tx)
		}
	}

	// Get the last block to find its hash
	prevBlock := bc.Blocks[len(bc.Blocks)-1]

	// Create a new block with the transactions
	newBlock := NewBlock(validTxs, prevBlock.Hash, bc.Difficulty)

	// Add the new block to the chain
	bc.Blocks = append(bc.Blocks, newBlock)
}

// FindUnspentTransactions finds all unspent transactions
func (bc *Blockchain) FindUnspentTransactions(address string) []*Transaction {
	var unspentTXs []*Transaction
	spentTXOs := make(map[string][]int)

	// Iterate over blocks in reverse order (newest first)
	for i := len(bc.Blocks) - 1; i >= 0; i-- {
		block := bc.Blocks[i]

		// Iterate over transactions in the block
		for _, tx := range block.Transactions {
			txID := hex.EncodeToString(tx.ID)

			// Check outputs
			for outIdx, out := range tx.Outputs {
				// Check if output is spent
				if spentTXOs[txID] != nil {
					for _, spentOut := range spentTXOs[txID] {
						if spentOut == outIdx {
							continue
						}
					}
				}

				// If output belongs to the address, add to unspent
				if out.CanBeUnlockedWith(address) {
					unspentTXs = append(unspentTXs, tx)
				}
			}

			// Add inputs to spent outputs
			if !tx.IsCoinbase() {
				for _, in := range tx.Inputs {
					if in.CanUnlockOutputWith(address) {
						inTxID := hex.EncodeToString(in.TxID)
						spentTXOs[inTxID] = append(spentTXOs[inTxID], in.OutputIndex)
					}
				}
			}
		}
	}

	return unspentTXs
}

// FindUTXO finds all unspent transaction outputs for an address
func (bc *Blockchain) FindUTXO(address string) []TXOutput {
	var UTXOs []TXOutput
	unspentTXs := bc.FindUnspentTransactions(address)

	for _, tx := range unspentTXs {
		for _, out := range tx.Outputs {
			if out.CanBeUnlockedWith(address) {
				UTXOs = append(UTXOs, out)
			}
		}
	}

	return UTXOs
}

// GetBalance returns the balance of an address
func (bc *Blockchain) GetBalance(address string) int {
	balance := 0
	UTXOs := bc.FindUTXO(address)

	for _, out := range UTXOs {
		balance += out.Value
	}

	return balance
}

// VerifyTransaction verifies a transaction
func (bc *Blockchain) VerifyTransaction(tx *Transaction) bool {
	if tx.IsCoinbase() {
		return true
	}

	// Verify each input
	for _, input := range tx.Inputs {
		// Find the referenced transaction
		refTx := bc.FindTransaction(input.TxID)
		if refTx == nil {
			return false
		}

		// Check if the output index is valid
		if input.OutputIndex >= len(refTx.Outputs) {
			return false
		}

		// Verify the signature
		// Note: In a real implementation, this would use proper signature verification
		if !input.CanUnlockOutputWith(refTx.Outputs[input.OutputIndex].ScriptPubKey) {
			return false
		}
	}

	return true
}

// FindTransaction finds a transaction by ID
func (bc *Blockchain) FindTransaction(ID []byte) *Transaction {
	for _, block := range bc.Blocks {
		for _, tx := range block.Transactions {
			if bytes.Equal(tx.ID, ID) {
				return tx
			}
		}
	}

	return nil
}

// FindSpendableOutputs finds spendable outputs for an address up to an amount
func (bc *Blockchain) FindSpendableOutputs(address string, amount int) (int, map[string][]int) {
	unspentOutputs := make(map[string][]int)
	unspentTXs := bc.FindUnspentTransactions(address)
	accumulated := 0

Work:
	for _, tx := range unspentTXs {
		txID := hex.EncodeToString(tx.ID)

		for outIdx, out := range tx.Outputs {
			if out.CanBeUnlockedWith(address) && accumulated < amount {
				accumulated += out.Value
				unspentOutputs[txID] = append(unspentOutputs[txID], outIdx)

				if accumulated >= amount {
					break Work
				}
			}
		}
	}

	return accumulated, unspentOutputs
}
```

### Implementing Proof of Work

Proof of Work (PoW) is the consensus mechanism used by Bitcoin and many other blockchains. It requires miners to find a value (nonce) that, when hashed with the block data, produces a hash with a specific number of leading zeros:

```go
// MineBlock mines a new block with the provided transactions
func (b *Block) MineBlock() {
	target := big.NewInt(1)
	target.Lsh(target, uint(256-b.Difficulty))

	var hashInt big.Int
	var hash [32]byte
	nonce := 0

	fmt.Printf("Mining a new block")
	for nonce < math.MaxInt64 {
		data := b.prepareData(nonce)
		hash = sha256.Sum256(data)
		fmt.Printf("\r%x", hash)

		hashInt.SetBytes(hash[:])
		if hashInt.Cmp(target) == -1 {
			break
		}
		nonce++
	}
	fmt.Println()

	b.Hash = hash[:]
	b.Nonce = nonce
}

// prepareData prepares data for hashing
func (b *Block) prepareData(nonce int) []byte {
	data := bytes.Join(
		[][]byte{
			b.PrevBlockHash,
			b.HashTransactions(),
			[]byte(fmt.Sprintf("%d", b.Timestamp)),
			[]byte(fmt.Sprintf("%d", b.Difficulty)),
			[]byte(fmt.Sprintf("%d", nonce)),
		},
		[]byte{},
	)

	return data
}

// ValidatePoW validates the Proof of Work
func (b *Block) ValidatePoW() bool {
	var hashInt big.Int

	target := big.NewInt(1)
	target.Lsh(target, uint(256-b.Difficulty))

	data := b.prepareData(b.Nonce)
	hash := sha256.Sum256(data)
	hashInt.SetBytes(hash[:])

	return hashInt.Cmp(target) == -1
}
```

### Transaction Structure and Validation

In a blockchain, transactions represent the transfer of value. Here's a simplified transaction model:

```go
// TXInput represents a transaction input
type TXInput struct {
	TxID        []byte // Reference to the transaction containing the output
	OutputIndex int    // Which output of that transaction
	ScriptSig   string // Data to unlock the output (simplified)
}

// TXOutput represents a transaction output
type TXOutput struct {
	Value        int    // Amount
	ScriptPubKey string // Conditions to unlock this output (simplified)
}

// CanUnlockOutputWith checks if the input can unlock an output
func (in *TXInput) CanUnlockOutputWith(unlockingData string) bool {
	return in.ScriptSig == unlockingData
}

// CanBeUnlockedWith checks if the output can be unlocked
func (out *TXOutput) CanBeUnlockedWith(unlockingData string) bool {
	return out.ScriptPubKey == unlockingData
}

// Transaction represents a blockchain transaction
type Transaction struct {
	ID      []byte
	Inputs  []TXInput
	Outputs []TXOutput
}

// SetID sets the ID of a transaction
func (tx *Transaction) SetID() {
	var encoded bytes.Buffer
	enc := gob.NewEncoder(&encoded)
	err := enc.Encode(tx)
	if err != nil {
		log.Panic(err)
	}

	hash := sha256.Sum256(encoded.Bytes())
	tx.ID = hash[:]
}

// IsCoinbase checks if a transaction is a coinbase
func (tx *Transaction) IsCoinbase() bool {
	return len(tx.Inputs) == 1 && len(tx.Inputs[0].TxID) == 0 && tx.Inputs[0].OutputIndex == -1
}

// NewCoinbaseTX creates a new coinbase transaction
func NewCoinbaseTX(to, data string) *Transaction {
	if data == "" {
		data = fmt.Sprintf("Reward to %s", to)
	}

	txin := TXInput{[]byte{}, -1, data}
	txout := TXOutput{50, to} // 50 is the reward

	tx := Transaction{nil, []TXInput{txin}, []TXOutput{txout}}
	tx.SetID()

	return &tx
}

// NewUTXOTransaction creates a new transaction
func NewUTXOTransaction(from, to string, amount int, bc *Blockchain) *Transaction {
	var inputs []TXInput
	var outputs []TXOutput

	// Find spendable outputs
	acc, validOutputs := bc.FindSpendableOutputs(from, amount)
	if acc < amount {
		log.Panic("Not enough funds")
	}

	// Build inputs
	for txid, outs := range validOutputs {
		txID, err := hex.DecodeString(txid)
		if err != nil {
			log.Panic(err)
		}

		for _, out := range outs {
			input := TXInput{txID, out, from}
			inputs = append(inputs, input)
		}
	}

	// Build outputs
	outputs = append(outputs, TXOutput{amount, to})
	if acc > amount {
		outputs = append(outputs, TXOutput{acc - amount, from}) // Change
	}

	tx := Transaction{nil, inputs, outputs}
	tx.SetID()

	return &tx
}

// Hash returns the hash of the transaction
func (tx *Transaction) Hash() []byte {
	var hash [32]byte
	txCopy := *tx
	txCopy.ID = []byte{}

	var encoded bytes.Buffer
	enc := gob.NewEncoder(&encoded)
	err := enc.Encode(txCopy)
	if err != nil {
		log.Panic(err)
	}

	hash = sha256.Sum256(encoded.Bytes())
	return hash[:]
}
```

### Putting It All Together

Now let's implement a simple CLI to interact with our blockchain:

```go
func main() {
	// Initialize blockchain
	bc := NewBlockchain("miner-address", 4, 50) // Difficulty 4, reward 50

	// Create some transactions
	tx1 := NewUTXOTransaction("miner-address", "alice", 10, bc)
	tx2 := NewUTXOTransaction("miner-address", "bob", 15, bc)

	// Add a block with these transactions
	bc.AddBlock([]*Transaction{tx1, tx2})

	// Mine another block with a transaction between alice and bob
	tx3 := NewUTXOTransaction("alice", "bob", 5, bc)
	bc.AddBlock([]*Transaction{tx3})

	// Print blockchain state
	fmt.Println("Blockchain state:")
	for i, block := range bc.Blocks {
		fmt.Printf("Block %d:\n", i)
		fmt.Printf("  Timestamp: %d\n", block.Timestamp)
		fmt.Printf("  Hash: %x\n", block.Hash)
		fmt.Printf("  PrevHash: %x\n", block.PrevBlockHash)
		fmt.Printf("  Nonce: %d\n", block.Nonce)
		fmt.Printf("  Transactions: %d\n", len(block.Transactions))
		fmt.Println()
	}

	// Check balances
	fmt.Printf("Balance of miner: %d\n", bc.GetBalance("miner-address"))
	fmt.Printf("Balance of alice: %d\n", bc.GetBalance("alice"))
	fmt.Printf("Balance of bob: %d\n", bc.GetBalance("bob"))
}
```

This implementation covers the fundamental concepts of a blockchain:

1. **Blocks**: Data structures containing transactions and metadata
2. **Chain**: A sequence of blocks where each references the previous one
3. **Proof of Work**: A consensus mechanism to prevent spam and attacks
4. **Transactions**: A system for transferring value between addresses
5. **UTXO Model**: Tracking unspent transaction outputs for balance management

In a production blockchain, you would need to add:

1. **Networking**: P2P communication between nodes
2. **Persistence**: Storing the blockchain in a database
3. **Wallet Management**: Proper key management and address generation
4. **Scripting**: More sophisticated transaction conditions
5. **Consensus Rules**: Handling forks and chain reorganizations

The example above uses simplified addressing and signature verification. In a real blockchain implementation, you would use proper cryptographic signatures and address derivation as shown in the previous section.

## **34.4 Smart Contract Integration**

Smart contracts are self-executing programs that run on blockchain networks. In this section, we'll explore how to interact with smart contracts on Ethereum and other blockchain platforms using Go.

### Connecting to Ethereum Networks

Go-Ethereum (geth) provides a comprehensive client for connecting to Ethereum networks. Here's how to connect to various Ethereum networks:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/ethclient"
)

func main() {
	// Connect to the Ethereum mainnet
	mainnetClient, err := ethclient.Dial("https://mainnet.infura.io/v3/YOUR-PROJECT-ID")
	if err != nil {
		log.Fatalf("Failed to connect to Ethereum mainnet: %v", err)
	}
	defer mainnetClient.Close()

	// Connect to the Goerli testnet
	goerliClient, err := ethclient.Dial("https://goerli.infura.io/v3/YOUR-PROJECT-ID")
	if err != nil {
		log.Fatalf("Failed to connect to Goerli testnet: %v", err)
	}
	defer goerliClient.Close()

	// Connect to a local node
	localClient, err := ethclient.Dial("http://localhost:8545")
	if err != nil {
		log.Fatalf("Failed to connect to local node: %v", err)
	}
	defer localClient.Close()

	// Get the latest block number from each network
	ctx := context.Background()

	mainnetBlock, err := mainnetClient.BlockByNumber(ctx, nil)
	if err != nil {
		log.Fatalf("Failed to get mainnet block: %v", err)
	}

	goerliBlock, err := goerliClient.BlockByNumber(ctx, nil)
	if err != nil {
		log.Fatalf("Failed to get goerli block: %v", err)
	}

	// Print block numbers
	fmt.Printf("Ethereum Mainnet Block: %s\n", mainnetBlock.Number().String())
	fmt.Printf("Goerli Testnet Block: %s\n", goerliBlock.Number().String())
}
```

### ABI Encoding and Decoding

To interact with smart contracts, you need to work with the Application Binary Interface (ABI). The ABI defines how to encode function calls and decode return values:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"math/big"
	"strings"

	"github.com/ethereum/go-ethereum/accounts/abi"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/crypto"
)

// Simple ERC-20 ABI with just the balanceOf function
const erc20ABI = `[
	{
		"constant": true,
		"inputs": [{"name": "_owner", "type": "address"}],
		"name": "balanceOf",
		"outputs": [{"name": "balance", "type": "uint256"}],
		"type": "function"
	}
]`

// This is a simple example of how to use the ABI package to encode function calls
func main() {
	// Parse the ABI
	parsedABI, err := abi.JSON(strings.NewReader(erc20ABI))
	if err != nil {
		log.Fatalf("Failed to parse ABI: %v", err)
	}

	// Define the address for which we want to check the balance
	address := common.HexToAddress("0x742d35Cc6634C0532925a3b844Bc454e4438f44e")

	// Encode the function call to balanceOf(address)
	data, err := parsedABI.Pack("balanceOf", address)
	if err != nil {
		log.Fatalf("Failed to pack data: %v", err)
	}

	fmt.Printf("Function signature + encoded parameters: %x\n", data)

	// The first 4 bytes are the function signature (selector)
	funcSignature := data[:4]
	fmt.Printf("Function signature: %x\n", funcSignature)

	// The rest are the encoded parameters
	encodedParams := data[4:]
	fmt.Printf("Encoded parameters: %x\n", encodedParams)

	// Manually calculate the function selector to verify
	// keccak256("balanceOf(address)") and take the first 4 bytes
	hash := crypto.Keccak256([]byte("balanceOf(address)"))
	manualSelector := hash[:4]
	fmt.Printf("Manually calculated selector: %x\n", manualSelector)

	// Simulate receiving a response and unpacking it
	// In a real scenario, this would come from an eth_call RPC
	mockResponseHex := "0x000000000000000000000000000000000000000000000001a055690d9db80000" // 30 ETH
	mockResponse, err := hex.DecodeString(strings.TrimPrefix(mockResponseHex, "0x"))
	if err != nil {
		log.Fatalf("Failed to decode hex: %v", err)
	}

	// Unpack the result
	var balance *big.Int
	err = parsedABI.UnpackIntoInterface(&balance, "balanceOf", mockResponse)
	if err != nil {
		log.Fatalf("Failed to unpack result: %v", err)
	}

	fmt.Printf("Balance: %s wei (%s ETH)\n",
		balance.String(),
		new(big.Float).Quo(
			new(big.Float).SetInt(balance),
			new(big.Float).SetInt(big.NewInt(1e18)),
		).String(),
	)
}
```

### Smart Contract Deployment and Interaction

Let's look at how to deploy and interact with a smart contract:

```go
package main

import (
	"context"
	"crypto/ecdsa"
	"fmt"
	"log"
	"math/big"
	"strings"

	"github.com/ethereum/go-ethereum/accounts/abi"
	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
)

// SimpleToken ABI and bytecode
const simpleTokenABI = `[
	{
		"inputs": [{"name": "initialSupply", "type": "uint256"}],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"inputs": [{"name": "account", "type": "address"}],
		"name": "balanceOf",
		"outputs": [{"name": "", "type": "uint256"}],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{"name": "recipient", "type": "address"},
			{"name": "amount", "type": "uint256"}
		],
		"name": "transfer",
		"outputs": [{"name": "", "type": "bool"}],
		"stateMutability": "nonpayable",
		"type": "function"
	}
]`

// Compiled SimpleToken bytecode (truncated for example)
const simpleTokenBytecode = "0x608060405234801561001057600080fd5b5060405161047f38038061047f83398181016040528101906100329190610099565b806000803373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002081905550506100c6565b600080fd5b6000819050919050565b61007681610063565b811461008157600080fd5b50565b6000815190506100938161006d565b92915050565b6000602082840312156100af576100ae61005e565b5b60006100bd84828501610084565b91505092915050565b6103aa806100d56000396000f3fe608060405260043610610046576000357c01000000000000000000000000000000000000000000000000000000009004806370a082311461004a578063a9059cbb1461008a575b5f80fd5b34801561005557600080fd5b50610074600480360381019061006f9190610209565b6100bd565b6040516100819190610251565b60405180910390f35b34801561009557600080fd5b506100a0610104565b604051610058919061027c565b5f80fd5b5f8082815260200190815260200160002054905090565b5f8082815260200190815260200160002060008282546101239190610298565b925050819055506001905090565b5f80fd5b5f73ffffffffffffffffffffffffffffffffffffffff82169050919050565b5f61015c82610135565b9050919050565b61016c81610151565b811461017757600080fd5b50565b5f8135905061018981610163565b92915050565b5f819050919050565b6101a28161018f565b81146101ad57600080fd5b50565b5f813590506101bf81610199565b92915050565b5f80604083850312156101db576101da610130565b5b5f6101e88582860161017a565b92505060206101f9858286016101b0565b9150509250929050565b5f6020828403121561021f5761021e610130565b5b5f61022c8482850161017a565b91505092915050565b61023e8161018f565b82525050565b5f8115159050919050565b5f61025b82610244565b9050919050565b61026b8161024c565b82525050565b5f6020820190506102865f830184610235565b92915050565b5f6020820190506102a15f830184610260565b92915050565b5f80828401905083811015610240578482600052809450505050509291505054"

func main() {
	// Connect to a local Ethereum node
	client, err := ethclient.Dial("http://localhost:8545")
	if err != nil {
		log.Fatalf("Failed to connect to the Ethereum client: %v", err)
	}
	defer client.Close()

	// Generate a private key for testing (don't do this in production)
	privateKey, err := crypto.GenerateKey()
	if err != nil {
		log.Fatalf("Failed to generate private key: %v", err)
	}

	// Get the public address
	from := crypto.PubkeyToAddress(privateKey.PublicKey)
	fmt.Printf("From address: %s\n", from.Hex())

	// Get the nonce for the account
	ctx := context.Background()
	nonce, err := client.PendingNonceAt(ctx, from)
	if err != nil {
		log.Fatalf("Failed to get nonce: %v", err)
	}

	// Set gas price and limit
	gasPrice, err := client.SuggestGasPrice(ctx)
	if err != nil {
		log.Fatalf("Failed to suggest gas price: %v", err)
	}

	// Create auth for the transaction
	chainID, err := client.NetworkID(ctx)
	if err != nil {
		log.Fatalf("Failed to get network ID: %v", err)
	}
	auth, err := bind.NewKeyedTransactorWithChainID(privateKey, chainID)
	if err != nil {
		log.Fatalf("Failed to create authorized transactor: %v", err)
	}
	auth.Nonce = big.NewInt(int64(nonce))
	auth.Value = big.NewInt(0)     // No ETH sent
	auth.GasLimit = uint64(3000000) // Gas limit
	auth.GasPrice = gasPrice

	// Parse the ABI
	parsedABI, err := abi.JSON(strings.NewReader(simpleTokenABI))
	if err != nil {
		log.Fatalf("Failed to parse ABI: %v", err)
	}

	// Initial supply: 1,000,000 tokens (with 18 decimals)
	initialSupply := new(big.Int)
	initialSupply.Exp(big.NewInt(10), big.NewInt(18+6), nil) // 10^(18+6) = 10^24

	// Pack constructor arguments
	input, err := parsedABI.Pack("", initialSupply)
	if err != nil {
		log.Fatalf("Failed to pack constructor arguments: %v", err)
	}

	// Append constructor arguments to bytecode
	bytecode := common.FromHex(simpleTokenBytecode)
	bytecode = append(bytecode, input...)

	// Deploy the contract
	tx := types.NewContractCreation(nonce, big.NewInt(0), auth.GasLimit, gasPrice, bytecode)
	signedTx, err := types.SignTx(tx, types.NewEIP155Signer(chainID), privateKey)
	if err != nil {
		log.Fatalf("Failed to sign transaction: %v", err)
	}

	// Send the transaction
	err = client.SendTransaction(ctx, signedTx)
	if err != nil {
		log.Fatalf("Failed to send transaction: %v", err)
	}

	fmt.Printf("Transaction sent: %s\n", signedTx.Hash().Hex())
	fmt.Println("Waiting for transaction to be mined...")

	// Wait for transaction receipt
	receipt, err := bind.WaitMined(ctx, client, signedTx)
	if err != nil {
		log.Fatalf("Failed to get transaction receipt: %v", err)
	}

	contractAddress := receipt.ContractAddress
	fmt.Printf("Contract deployed at: %s\n", contractAddress.Hex())
}
```

### Event Monitoring and Filtering

Smart contracts emit events that you can monitor:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"
	"strings"

	"github.com/ethereum/go-ethereum"
	"github.com/ethereum/go-ethereum/accounts/abi"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/ethclient"
)

// ERC-20 Transfer event ABI
const erc20TransferEventABI = `[{
	"anonymous": false,
	"inputs": [
		{"indexed": true, "name": "from", "type": "address"},
		{"indexed": true, "name": "to", "type": "address"},
		{"indexed": false, "name": "value", "type": "uint256"}
	],
	"name": "Transfer",
	"type": "event"
}]`

// Transfer event structure
type Transfer struct {
	From  common.Address
	To    common.Address
	Value *big.Int
}

func main() {
	// Connect to an Ethereum node
	client, err := ethclient.Dial("wss://mainnet.infura.io/ws/v3/YOUR-PROJECT-ID")
	if err != nil {
		log.Fatalf("Failed to connect to the Ethereum client: %v", err)
	}
	defer client.Close()

	// Parse the ERC-20 ABI to get event signature
	parsedABI, err := abi.JSON(strings.NewReader(erc20TransferEventABI))
	if err != nil {
		log.Fatalf("Failed to parse ABI: %v", err)
	}

	// Get the Transfer event signature
	transferEventSig := parsedABI.Events["Transfer"].ID

	// USDC contract address
	usdcAddress := common.HexToAddress("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")

	// Set up the query for Transfer events
	query := ethereum.FilterQuery{
		Addresses: []common.Address{usdcAddress},
		Topics: [][]common.Hash{
			{transferEventSig}, // Transfer event signature
		},
	}

	// Create a channel to receive event logs
	logs := make(chan types.Log)

	// Subscribe to logs matching the filter
	sub, err := client.SubscribeFilterLogs(context.Background(), query, logs)
	if err != nil {
		log.Fatalf("Failed to subscribe to logs: %v", err)
	}

	fmt.Println("Watching for USDC transfers...")

	// Process incoming logs
	for {
		select {
		case err := <-sub.Err():
			log.Fatalf("Subscription error: %v", err)
		case vLog := <-logs:
			// Parse the log data into our Transfer struct
			var transfer Transfer
			err := parsedABI.UnpackIntoInterface(&transfer, "Transfer", vLog.Data)
			if err != nil {
				log.Printf("Failed to unpack event data: %v", err)
				continue
			}

			// The first two indexed parameters (from, to) are in the Topics
			transfer.From = common.BytesToAddress(vLog.Topics[1].Bytes())
			transfer.To = common.BytesToAddress(vLog.Topics[2].Bytes())

			// Print transfer details
			fmt.Printf("Transfer: %s -> %s, Value: %s USDC\n",
				transfer.From.Hex(),
				transfer.To.Hex(),
				new(big.Float).Quo(
					new(big.Float).SetInt(transfer.Value),
					new(big.Float).SetInt(big.NewInt(1e6)), // USDC has 6 decimals
				).String(),
			)
		}
	}
}
```

### Using Go Bindings for Smart Contracts

The go-ethereum library includes a tool called `abigen` that generates type-safe Go bindings for Solidity smart contracts:

```bash
# Generate Go bindings from a Solidity smart contract ABI
abigen --abi=./Token.abi --pkg=token --out=Token.go
```

Once generated, you can use these bindings to interact with the contract:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/ethclient"

	"your-project/token" // Generated bindings
)

func main() {
	// Connect to an Ethereum node
	client, err := ethclient.Dial("https://mainnet.infura.io/v3/YOUR-PROJECT-ID")
	if err != nil {
		log.Fatalf("Failed to connect to Ethereum client: %v", err)
	}
	defer client.Close()

	// USDC contract address
	tokenAddress := common.HexToAddress("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")

	// Create a new instance of the token contract
	tokenContract, err := token.NewToken(tokenAddress, client)
	if err != nil {
		log.Fatalf("Failed to instantiate token contract: %v", err)
	}

	// Get token name and symbol
	name, err := tokenContract.Name(&bind.CallOpts{})
	if err != nil {
		log.Fatalf("Failed to get token name: %v", err)
	}

	symbol, err := tokenContract.Symbol(&bind.CallOpts{})
	if err != nil {
		log.Fatalf("Failed to get token symbol: %v", err)
	}

	decimals, err := tokenContract.Decimals(&bind.CallOpts{})
	if err != nil {
		log.Fatalf("Failed to get token decimals: %v", err)
	}

	fmt.Printf("Token: %s (%s), Decimals: %d\n", name, symbol, decimals)

	// Get balance of an address
	address := common.HexToAddress("0x742d35Cc6634C0532925a3b844Bc454e4438f44e")
	balance, err := tokenContract.BalanceOf(&bind.CallOpts{}, address)
	if err != nil {
		log.Fatalf("Failed to get balance: %v", err)
	}

	// Calculate human-readable balance
	divisor := new(big.Int).Exp(big.NewInt(10), big.NewInt(int64(decimals)), nil)
	readable := new(big.Float).Quo(
		new(big.Float).SetInt(balance),
		new(big.Float).SetInt(divisor),
	)

	fmt.Printf("Balance of %s: %s %s\n", address.Hex(), readable.String(), symbol)
}
```

These examples demonstrate how to deploy, interact with, and monitor smart contracts using Go. In the next section, we'll cover wallet implementation and key management.

## **34.5 Wallet Implementation**

Wallets are a critical component of blockchain applications, allowing users to securely store and manage their private keys and interact with blockchain networks. In this section, we'll explore how to implement wallet functionality in Go.

### Creating and Managing Keypairs

Let's start with the basics of creating and managing cryptographic key pairs:

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"encoding/hex"
	"fmt"
	"log"

	"github.com/ethereum/go-ethereum/common/hexutil"
	"github.com/ethereum/go-ethereum/crypto"
)

// Wallet represents a simple cryptocurrency wallet
type Wallet struct {
	PrivateKey *ecdsa.PrivateKey
	PublicKey  *ecdsa.PublicKey
	Address    string
}

// NewWallet creates a new wallet with a random private key
func NewWallet() (*Wallet, error) {
	// Generate a new ECDSA private key
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		return nil, fmt.Errorf("failed to generate private key: %w", err)
	}

	// Get the public key
	publicKey := &privateKey.PublicKey

	// Create a new wallet
	wallet := &Wallet{
		PrivateKey: privateKey,
		PublicKey:  publicKey,
		Address:    deriveAddress(publicKey),
	}

	return wallet, nil
}

// NewWalletFromPrivateKey creates a wallet from an existing private key
func NewWalletFromPrivateKey(privateKeyHex string) (*Wallet, error) {
	// Decode the private key
	privateKeyBytes, err := hex.DecodeString(privateKeyHex)
	if err != nil {
		return nil, fmt.Errorf("failed to decode private key: %w", err)
	}

	// Convert bytes to ECDSA private key
	privateKey, err := crypto.ToECDSA(privateKeyBytes)
	if err != nil {
		return nil, fmt.Errorf("invalid private key: %w", err)
	}

	// Get the public key
	publicKey := &privateKey.PublicKey

	// Create a new wallet
	wallet := &Wallet{
		PrivateKey: privateKey,
		PublicKey:  publicKey,
		Address:    deriveAddress(publicKey),
	}

	return wallet, nil
}

// deriveAddress derives an Ethereum-style address from a public key
func deriveAddress(publicKey *ecdsa.PublicKey) string {
	// Convert public key to bytes
	publicKeyBytes := elliptic.Marshal(publicKey.Curve, publicKey.X, publicKey.Y)

	// Hash the public key using Keccak-256
	hash := crypto.Keccak256(publicKeyBytes[1:]) // Skip the first byte (compression flag)

	// Take the last 20 bytes of the hash to get the address
	address := hash[12:]

	// Return as hex string with 0x prefix
	return hexutil.Encode(address)
}

func main() {
	// Create a new wallet
	wallet, err := NewWallet()
	if err != nil {
		log.Fatalf("Failed to create wallet: %v", err)
	}

	// Display wallet information
	fmt.Printf("New Wallet:\n")
	fmt.Printf("  Address:     %s\n", wallet.Address)
	fmt.Printf("  Private Key: %x\n", crypto.FromECDSA(wallet.PrivateKey))
	fmt.Printf("  Public Key:  %x\n", crypto.FromECDSAPub(wallet.PublicKey))

	// Create a wallet from an existing private key
	privateKeyHex := hex.EncodeToString(crypto.FromECDSA(wallet.PrivateKey))
	importedWallet, err := NewWalletFromPrivateKey(privateKeyHex)
	if err != nil {
		log.Fatalf("Failed to import wallet: %v", err)
	}

	fmt.Printf("\nImported Wallet:\n")
	fmt.Printf("  Address:     %s\n", importedWallet.Address)
	fmt.Printf("  Private Key: %x\n", crypto.FromECDSA(importedWallet.PrivateKey))
	fmt.Printf("  Public Key:  %x\n", crypto.FromECDSAPub(importedWallet.PublicKey))
}
```

### Transaction Signing

Let's implement transaction signing for Ethereum using Go:

```go
package main

import (
	"context"
	"crypto/ecdsa"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
)

// signTransaction signs an Ethereum transaction
func signTransaction(client *ethclient.Client, privateKey *ecdsa.PrivateKey, to common.Address, amount *big.Int) (*types.Transaction, error) {
	// Get the sender address from the private key
	publicKey := privateKey.Public()
	publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)
	if !ok {
		return nil, fmt.Errorf("error casting public key to ECDSA")
	}
	from := crypto.PubkeyToAddress(*publicKeyECDSA)

	// Get the nonce for the sender account
	nonce, err := client.PendingNonceAt(context.Background(), from)
	if err != nil {
		return nil, fmt.Errorf("failed to get nonce: %w", err)
	}

	// Get gas price
	gasPrice, err := client.SuggestGasPrice(context.Background())
	if err != nil {
		return nil, fmt.Errorf("failed to suggest gas price: %w", err)
	}

	// Get chain ID
	chainID, err := client.NetworkID(context.Background())
	if err != nil {
		return nil, fmt.Errorf("failed to get network ID: %w", err)
	}
	auth, err := bind.NewKeyedTransactorWithChainID(privateKey, chainID)
	if err != nil {
		return nil, fmt.Errorf("failed to create authorized transactor: %w", err)
	}
	auth.Nonce = big.NewInt(int64(nonce))
	auth.Value = big.NewInt(0)     // No ETH sent
	auth.GasLimit = uint64(3000000) // Gas limit
	auth.GasPrice = gasPrice

	// Create transaction
	tx := types.NewTransaction(nonce, to, amount, 21000, gasPrice, nil)

	// Sign the transaction
	signedTx, err := types.SignTx(tx, types.NewEIP155Signer(chainID), privateKey)
	if err != nil {
		return nil, fmt.Errorf("failed to sign transaction: %w", err)
	}

	return signedTx, nil
}

// sendTransaction sends a signed transaction to the network
func sendTransaction(client *ethclient.Client, signedTx *types.Transaction) (common.Hash, error) {
	// Send the transaction
	err := client.SendTransaction(context.Background(), signedTx)
	if err != nil {
		return common.Hash{}, fmt.Errorf("failed to send transaction: %w", err)
	}

	return signedTx.Hash(), nil
}

func main() {
	// Connect to an Ethereum node
	client, err := ethclient.Dial("https://goerli.infura.io/v3/YOUR-PROJECT-ID")
	if err != nil {
		log.Fatalf("Failed to connect to the Ethereum client: %v", err)
	}
	defer client.Close()

	// Load or generate a private key
	privateKey, err := crypto.GenerateKey()
	if err != nil {
		log.Fatalf("Failed to generate private key: %v", err)
	}

	// Get the sender address
	from := crypto.PubkeyToAddress(privateKey.PublicKey)
	fmt.Printf("From address: %s\n", from.Hex())

	// Recipient address
	to := common.HexToAddress("0x742d35Cc6634C0532925a3b844Bc454e4438f44e")

	// Amount to send: 0.01 ETH
	amount := new(big.Int)
	amount.Exp(big.NewInt(10), big.NewInt(16), nil) // 10^16 wei = 0.01 ETH

	// Sign the transaction
	signedTx, err := signTransaction(client, privateKey, to, amount)
	if err != nil {
		log.Fatalf("Failed to sign transaction: %v", err)
	}

	// Send the transaction
	txHash, err := sendTransaction(client, signedTx)
	if err != nil {
		log.Fatalf("Failed to send transaction: %v", err)
	}

	fmt.Printf("Transaction sent: %s\n", txHash.Hex())
}
```

### BIP-39 Mnemonic Phrases

BIP-39 defines a way to generate mnemonic phrases (a sequence of words) that can be used to derive deterministic keys. Let's implement BIP-39 support in our wallet:

```go
package main

import (
	"fmt"
	"log"

	"github.com/ethereum/go-ethereum/accounts"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/tyler-smith/go-bip39"
)

// generateMnemonic generates a new BIP-39 mnemonic phrase
func generateMnemonic() (string, error) {
	// Generate entropy (128 bits = 12 words, 256 bits = 24 words)
	entropy, err := bip39.NewEntropy(256)
	if err != nil {
		return "", fmt.Errorf("failed to generate entropy: %w", err)
	}

	// Generate mnemonic from entropy
	mnemonic, err := bip39.NewMnemonic(entropy)
	if err != nil {
		return "", fmt.Errorf("failed to generate mnemonic: %w", err)
	}

	return mnemonic, nil
}

// mnemonicToSeed converts a mnemonic phrase to a seed
func mnemonicToSeed(mnemonic, passphrase string) []byte {
	return bip39.NewSeed(mnemonic, passphrase)
}

// validateMnemonic checks if a mnemonic phrase is valid
func validateMnemonic(mnemonic string) bool {
	return bip39.IsMnemonicValid(mnemonic)
}

func main() {
	// Generate a new mnemonic
	mnemonic, err := generateMnemonic()
	if err != nil {
		log.Fatalf("Failed to generate mnemonic: %v", err)
	}

	fmt.Printf("Mnemonic (24 words):\n%s\n\n", mnemonic)

	// Validate the mnemonic
	isValid := validateMnemonic(mnemonic)
	fmt.Printf("Mnemonic is valid: %v\n\n", isValid)

	// Convert mnemonic to seed
	// Optional passphrase adds extra security
	seed := mnemonicToSeed(mnemonic, "optional passphrase")
	fmt.Printf("Seed: %x\n\n", seed)

	// Generate a private key from the seed
	// This is a simplified approach; in a real wallet, you'd use HD wallet derivation
	privateKey, err := crypto.ToECDSA(crypto.Keccak256(seed)[:32])
	if err != nil {
		log.Fatalf("Failed to generate private key: %v", err)
	}

	// Get the address
	address := crypto.PubkeyToAddress(privateKey.PublicKey)
	fmt.Printf("Derived address: %s\n", address.Hex())
}
```

### HD Wallet Derivation Paths

Hierarchical Deterministic (HD) wallets allow the generation of multiple keypairs from a single seed. Let's implement BIP-44 derivation:

```go
package main

import (
	"fmt"
	"log"

	"github.com/btcsuite/btcd/btcec"
	"github.com/btcsuite/btcd/chaincfg"
	"github.com/btcsuite/btcutil/hdkeychain"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/tyler-smith/go-bip39"
)

// HDWallet represents a hierarchical deterministic wallet
type HDWallet struct {
	Mnemonic  string
	Seed      []byte
	MasterKey *hdkeychain.ExtendedKey
}

// NewHDWallet creates a new HD wallet from a mnemonic
func NewHDWallet(mnemonic string, passphrase string) (*HDWallet, error) {
	if !bip39.IsMnemonicValid(mnemonic) {
		return nil, fmt.Errorf("invalid mnemonic")
	}

	seed := bip39.NewSeed(mnemonic, passphrase)
	masterKey, err := hdkeychain.NewMaster(seed, &chaincfg.MainNetParams)
	if err != nil {
		return nil, fmt.Errorf("failed to create master key: %w", err)
	}

	return &HDWallet{
		Mnemonic:  mnemonic,
		Seed:      seed,
		MasterKey: masterKey,
	}, nil
}

// NewRandomHDWallet creates a new random HD wallet
func NewRandomHDWallet() (*HDWallet, error) {
	// Generate a new mnemonic
	entropy, err := bip39.NewEntropy(256)
	if err != nil {
		return nil, fmt.Errorf("failed to generate entropy: %w", err)
	}

	mnemonic, err := bip39.NewMnemonic(entropy)
	if err != nil {
		return nil, fmt.Errorf("failed to generate mnemonic: %w", err)
	}

	return NewHDWallet(mnemonic, "")
}

// DeriveEthereumAccount derives an Ethereum account at the specified index
func (w *HDWallet) DeriveEthereumAccount(index uint32) (common.Address, *btcec.PrivateKey, error) {
	// BIP-44 derivation path for Ethereum: m/44'/60'/0'/0/index
	// Purpose: 44'
	purpose, err := w.MasterKey.Derive(hdkeychain.HardenedKeyStart + 44)
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to derive purpose: %w", err)
	}

	// Coin type: 60' (Ethereum)
	coinType, err := purpose.Derive(hdkeychain.HardenedKeyStart + 60)
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to derive coin type: %w", err)
	}

	// Account: 0'
	account, err := coinType.Derive(hdkeychain.HardenedKeyStart + 0)
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to derive account: %w", err)
	}

	// Change: 0 (external chain)
	change, err := account.Derive(0)
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to derive change: %w", err)
	}

	// Address index
	addressKey, err := change.Derive(index)
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to derive address: %w", err)
	}

	// Get the private key
	privateKey, err := addressKey.ECPrivKey()
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to get private key: %w", err)
	}

	// Convert to Ethereum address
	privateKeyBytes := privateKey.Serialize()
	ethPrivateKey, err := crypto.ToECDSA(privateKeyBytes)
	if err != nil {
		return common.Address{}, nil, fmt.Errorf("failed to convert to ECDSA: %w", err)
	}

	address := crypto.PubkeyToAddress(ethPrivateKey.PublicKey)
	return address, privateKey, nil
}

func main() {
	// Create a new random HD wallet
	wallet, err := NewRandomHDWallet()
	if err != nil {
		log.Fatalf("Failed to create wallet: %v", err)
	}

	fmt.Printf("Mnemonic:\n%s\n\n", wallet.Mnemonic)
	fmt.Printf("Seed: %x\n\n", wallet.Seed)

	// Derive multiple Ethereum accounts
	fmt.Println("Derived Ethereum accounts:")
	for i := uint32(0); i < 5; i++ {
		address, _, err := wallet.DeriveEthereumAccount(i)
		if err != nil {
			log.Fatalf("Failed to derive account %d: %v", i, err)
		}
		fmt.Printf("Account %d: %s\n", i, address.Hex())
	}

	// Use an existing mnemonic
	existingMnemonic := "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about"
	existingWallet, err := NewHDWallet(existingMnemonic, "")
	if err != nil {
		log.Fatalf("Failed to create wallet from existing mnemonic: %v", err)
	}

	fmt.Printf("\nDerived accounts from test mnemonic:\n")
	for i := uint32(0); i < 5; i++ {
		address, _, err := existingWallet.DeriveEthereumAccount(i)
		if err != nil {
			log.Fatalf("Failed to derive account %d: %v", i, err)
		}
		fmt.Printf("Account %d: %s\n", i, address.Hex())
	}
}
```

### Storing Wallet Securely

Securely storing private keys is crucial for blockchain applications. Here's a simple example of encrypted key storage:

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"golang.org/x/crypto/scrypt"
	"io"
	"io/ioutil"
	"log"
	"os"
	"path/filepath"
	"time"
)

// EncryptedKeystore represents an encrypted private key
type EncryptedKeystore struct {
	IV        string `json:"iv"`
	Ciphertext string `json:"ciphertext"`
	Salt       string `json:"salt"`
	KDF        string `json:"kdf"`
	KDFParams  struct {
		N      int    `json:"n"`
		R      int    `json:"r"`
		P      int    `json:"p"`
		DKLen  int    `json:"dklen"`
	} `json:"kdfparams"`
	MAC        string `json:"mac"`
}

// WalletData represents the wallet data to be encrypted
type WalletData struct {
	Mnemonic   string            `json:"mnemonic"`
	PrivateKey string            `json:"private_key"`
	Addresses  map[string]string `json:"addresses"`
}

// encryptWallet encrypts wallet data with a password
func encryptWallet(data *WalletData, password string) (*EncryptedKeystore, error) {
	// Convert wallet data to JSON
	dataJSON, err := json.Marshal(data)
	if err != nil {
		return nil, fmt.Errorf("failed to marshal wallet data: %w", err)
	}

	// Generate random salt
	salt := make([]byte, 32)
	if _, err := io.ReadFull(rand.Reader, salt); err != nil {
		return nil, err
	}

	// Derive key from password using scrypt
	derivedKey, err := scrypt.Key([]byte(password), salt, 1<<18, 8, 1, 32)
	if err != nil {
		return nil, err
	}

	// Generate random IV
	iv := make([]byte, 16)
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return nil, err
	}

	// Create AES cipher
	block, err := aes.NewCipher(derivedKey[:16])
	if err != nil {
		return nil, err
	}

	// Encrypt data
	ciphertext := make([]byte, len(dataJSON))
	stream := cipher.NewCTR(block, iv)
	stream.XORKeyStream(ciphertext, dataJSON)

	// Calculate MAC
	mac := sha256.Sum256(append(derivedKey[16:], ciphertext...))

	// Create encrypted wallet
	encrypted := &EncryptedKeystore{
		IV:         hex.EncodeToString(iv),
		Ciphertext: hex.EncodeToString(ciphertext),
		Salt:       hex.EncodeToString(salt),
		KDF:        "scrypt",
		MAC:        hex.EncodeToString(mac[:]),
	}
	encrypted.KDFParams.N = scryptParams.N
	encrypted.KDFParams.R = scryptParams.R
	encrypted.KDFParams.P = scryptParams.P
	encrypted.KDFParams.DKLen = scryptParams.DKLen

	return encrypted, nil
}

// decryptWallet decrypts an encrypted wallet with a password
func decryptWallet(encrypted *EncryptedKeystore, password string) (*WalletData, error) {
	// Decode hex strings
	iv, err := hex.DecodeString(encrypted.IV)
	if err != nil {
		return nil, fmt.Errorf("failed to decode IV: %w", err)
	}

	ciphertext, err := hex.DecodeString(encrypted.Ciphertext)
	if err != nil {
		return nil, fmt.Errorf("failed to decode ciphertext: %w", err)
	}

	salt, err := hex.DecodeString(encrypted.Salt)
	if err != nil {
		return nil, fmt.Errorf("failed to decode salt: %w", err)
	}

	mac, err := hex.DecodeString(encrypted.MAC)
	if err != nil {
		return nil, fmt.Errorf("failed to decode MAC: %w", err)
	}

	// Derive key from password
	derivedKey, err := scrypt.Key(
		[]byte(password),
		salt,
		encrypted.KDFParams.N,
		encrypted.KDFParams.R,
		encrypted.KDFParams.P,
		encrypted.KDFParams.DKLen,
	)
	if err != nil {
		return nil, fmt.Errorf("failed to derive key: %w", err)
	}

	// Verify MAC
	calculatedMAC := sha256.Sum256(append(derivedKey[16:], ciphertext...))
	if !hmac.Equal(calculatedMAC[:], mac) {
		return nil, fmt.Errorf("invalid password (MAC mismatch)")
	}

	// Decrypt data
	block, err := aes.NewCipher(derivedKey[:16])
	if err != nil {
		return nil, fmt.Errorf("failed to create cipher: %w", err)
	}

	plaintext := make([]byte, len(ciphertext))
	stream := cipher.NewCTR(block, iv)
	stream.XORKeyStream(plaintext, ciphertext)

	// Unmarshal wallet data
	var walletData WalletData
	if err := json.Unmarshal(plaintext, &walletData); err != nil {
		return nil, fmt.Errorf("failed to unmarshal wallet data: %w", err)
	}

	return &walletData, nil
}

// saveWalletToFile saves an encrypted wallet to a file
func saveWalletToFile(encrypted *EncryptedKeystore, filePath string) error {
	// Create directory if it doesn't exist
	if err := os.MkdirAll(filepath.Dir(filePath), 0700); err != nil {
		return fmt.Errorf("failed to create directory: %w", err)
	}

	// Marshal to JSON
	data, err := json.MarshalIndent(encrypted, "", "  ")
	if err != nil {
		return fmt.Errorf("failed to marshal wallet: %w", err)
	}

	// Write to file
	if err := ioutil.WriteFile(filePath, data, 0600); err != nil {
		return fmt.Errorf("failed to write file: %w", err)
	}

	return nil
}

// loadWalletFromFile loads an encrypted wallet from a file
func loadWalletFromFile(filePath string) (*EncryptedKeystore, error) {
	// Read file
	data, err := ioutil.ReadFile(filePath)
	if err != nil {
		return nil, fmt.Errorf("failed to read file: %w", err)
	}

	// Unmarshal JSON
	var encrypted EncryptedKeystore
	if err := json.Unmarshal(data, &encrypted); err != nil {
		return nil, fmt.Errorf("failed to unmarshal wallet: %w", err)
	}

	return &encrypted, nil
}

func main() {
	// Sample wallet data
	walletData := &WalletData{
		Mnemonic:   "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about",
		PrivateKey: "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
		Addresses: map[string]string{
			"eth": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
			"btc": "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa",
		},
	}

	// Encrypt wallet
	password := "supersecurepassword"
	encryptedWallet, err := encryptWallet(walletData, password)
	if err != nil {
		log.Fatalf("Failed to encrypt wallet: %v", err)
	}

	// Save to file
	filePath := "./wallet.json"
	if err := saveWalletToFile(encryptedWallet, filePath); err != nil {
		log.Fatalf("Failed to save wallet: %v", err)
	}

	fmt.Printf("Saved encrypted wallet to: %s\n", filePath)
}
```

These examples demonstrate the key components of a blockchain wallet implementation in Go:

1. **Key Generation**: Creating secure cryptographic key pairs
2. **Transaction Signing**: Preparing and signing blockchain transactions
3. **Mnemonic Phrases**: Implementing BIP-39 for seed generation from words
4. **HD Wallet Derivation**: Using BIP-44 to derive multiple accounts from a single seed
5. **Secure Storage**: Encrypting and storing wallet data securely

In a production environment, you would typically use established libraries like go-ethereum's accounts package or btcutil for these operations, but understanding the underlying concepts is essential for blockchain developers.

## **34.6 Interacting with Blockchain Networks**

- Ethereum client implementation
- Bitcoin integration
- Solana RPC clients
- Cross-chain interactions

## **34.7 Building Decentralized Applications (dApps)**

- Architecture patterns for Web3 applications
- Backend services for dApps
- Authentication with wallets
- Handling blockchain events

## **34.8 Layer 2 Solutions and Scaling**

- Implementing state channels
- Optimistic rollups
- Zero-knowledge proofs
- Sidechains and cross-chain bridges

## **34.9 Security Best Practices**

- Common vulnerabilities in blockchain applications
- Secure key management
- Transaction validation and replay protection
- Auditing and formal verification

## **34.10 Real-world Applications and Case Studies**

- Decentralized finance (DeFi) implementations
- NFT marketplaces and platforms
- Supply chain traceability
- Identity management systems

## **34.11 Regulatory Compliance and Legal Considerations**

- KYC/AML integration
- Transaction monitoring
- Compliance reporting
- Privacy-preserving techniques

## **34.12 Conclusion**

- Future of Go in blockchain development
- Emerging trends in cryptocurrency
- Building a career in blockchain with Go
