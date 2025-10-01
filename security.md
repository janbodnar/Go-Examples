# Go Security Best Practices

Security is a critical aspect of software development that must be considered  
from the earliest stages of application design. In Go, security encompasses  
protecting sensitive data, implementing proper authentication and authorization,  
securing network communications, and preventing common vulnerabilities like  
injection attacks, cross-site scripting, and data breaches.  

Go provides excellent built-in support for security through its standard library,  
which includes robust packages for cryptography, TLS/SSL, secure random number  
generation, and various encoding schemes. The language's strong type system,  
explicit error handling, and memory safety features make it inherently more  
secure than languages like C or C++, eliminating entire classes of vulnerabilities  
related to buffer overflows and memory corruption.  

Modern applications handle increasingly sensitive data including user credentials,  
payment information, health records, and confidential business data. Protecting  
this data requires implementing defense-in-depth strategies that combine multiple  
security layers: secure data storage, encrypted communications, strong authentication,  
proper authorization, input validation, and comprehensive audit logging.  

Go's crypto packages implement industry-standard algorithms and protocols,  
providing developers with battle-tested implementations rather than requiring  
them to create custom security solutions. The `crypto` package family includes  
support for hashing (SHA-256, SHA-512), symmetric encryption (AES), asymmetric  
encryption (RSA, ECDSA), message authentication codes (HMAC), and more.  

TLS (Transport Layer Security) is essential for securing network communications  
in modern applications. Go's `crypto/tls` package provides comprehensive support  
for both client and server TLS implementations, including mutual TLS authentication,  
certificate validation, and modern cipher suite selection. The package makes it  
straightforward to implement secure HTTPS servers and clients.  

JSON Web Tokens (JWTs) have become the de facto standard for authentication in  
modern web applications and APIs. JWTs provide a compact, URL-safe method for  
representing claims between parties. They enable stateless authentication, making  
them ideal for microservices architectures and distributed systems. Implementing  
JWTs correctly requires understanding token signing, verification, expiration,  
and secure storage patterns.  

Security is not just about implementing algorithms correctly—it's also about  
avoiding common pitfalls like hardcoding secrets, improper error handling that  
leaks information, timing attacks in comparisons, and insufficient input validation.  
This guide covers both the technical implementation of security features and the  
best practices that prevent vulnerabilities.  

This comprehensive guide provides 60 production-ready Go code examples covering  
security best practices. Each example includes detailed explanations of security  
concepts, implementation details, and potential pitfalls to avoid. The examples  
progress from foundational concepts to advanced patterns used in production  
systems.  


## Handling secrets from environment variables

Environment variables provide a secure way to pass sensitive configuration to  
applications without hardcoding secrets in source code.  

```go
package main

import (
    "fmt"
    "log"
    "os"
)

type Config struct {
    DatabaseURL    string
    APIKey         string
    JWTSecret      string
    EncryptionKey  string
}

func LoadConfig() (*Config, error) {
    // Check required environment variables
    required := []string{"DATABASE_URL", "API_KEY", "JWT_SECRET", "ENCRYPTION_KEY"}
    for _, key := range required {
        if os.Getenv(key) == "" {
            return nil, fmt.Errorf("required environment variable %s is not set", key)
        }
    }
    
    return &Config{
        DatabaseURL:   os.Getenv("DATABASE_URL"),
        APIKey:        os.Getenv("API_KEY"),
        JWTSecret:     os.Getenv("JWT_SECRET"),
        EncryptionKey: os.Getenv("ENCRYPTION_KEY"),
    }, nil
}

func (c *Config) String() string {
    // Never log actual secrets - mask them
    return fmt.Sprintf("Config{DatabaseURL: %s, APIKey: %s, JWTSecret: %s, EncryptionKey: %s}",
        maskSecret(c.DatabaseURL),
        maskSecret(c.APIKey),
        maskSecret(c.JWTSecret),
        maskSecret(c.EncryptionKey))
}

func maskSecret(secret string) string {
    if len(secret) <= 4 {
        return "****"
    }
    return secret[:4] + "****"
}

func main() {
    config, err := LoadConfig()
    if err != nil {
        log.Fatalf("Failed to load configuration: %v", err)
    }
    
    fmt.Println("Configuration loaded successfully")
    fmt.Println(config)
}
```

This example demonstrates secure configuration loading from environment variables.  
The code validates that all required secrets are present before proceeding,  
preventing runtime failures. The `String()` method masks sensitive values when  
logging, preventing accidental secret exposure in logs. Always validate environment  
variables at startup and fail fast if configuration is incomplete.  



## Loading secrets from .env files

Using .env files during development provides a convenient way to manage secrets  
without exposing them in version control.  

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

type SecretManager struct {
    secrets map[string]string
}

func NewSecretManager() *SecretManager {
    return &SecretManager{
        secrets: make(map[string]string),
    }
}

func (sm *SecretManager) LoadFromFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("failed to open env file: %w", err)
    }
    defer file.Close()
    
    scanner := bufio.NewScanner(file)
    lineNum := 0
    
    for scanner.Scan() {
        lineNum++
        line := strings.TrimSpace(scanner.Text())
        
        // Skip empty lines and comments
        if line == "" || strings.HasPrefix(line, "#") {
            continue
        }
        
        parts := strings.SplitN(line, "=", 2)
        if len(parts) != 2 {
            return fmt.Errorf("invalid format at line %d: %s", lineNum, line)
        }
        
        key := strings.TrimSpace(parts[0])
        value := strings.TrimSpace(parts[1])
        
        // Remove quotes if present
        value = strings.Trim(value, "\"'")
        
        sm.secrets[key] = value
        // Set as environment variable
        os.Setenv(key, value)
    }
    
    if err := scanner.Err(); err != nil {
        return fmt.Errorf("error reading env file: %w", err)
    }
    
    return nil
}

func (sm *SecretManager) Get(key string) (string, bool) {
    value, exists := sm.secrets[key]
    return value, exists
}

func (sm *SecretManager) MustGet(key string) string {
    value, exists := sm.secrets[key]
    if !exists {
        panic(fmt.Sprintf("required secret %s not found", key))
    }
    return value
}

func main() {
    sm := NewSecretManager()
    
    // Load .env file (never commit this file to version control)
    if err := sm.LoadFromFile(".env"); err != nil {
        fmt.Printf("Warning: could not load .env file: %v\n", err)
    }
    
    // Access secrets with safety checks
    if dbURL, exists := sm.Get("DATABASE_URL"); exists {
        fmt.Printf("Database URL loaded (length: %d)\n", len(dbURL))
    }
    
    // Or use MustGet for required secrets
    apiKey := sm.MustGet("API_KEY")
    fmt.Printf("API Key loaded (length: %d)\n", len(apiKey))
}
```

This example implements a simple .env file parser for development environments.  
The parser handles comments, empty lines, and quoted values. It loads secrets  
into both a local map and environment variables, providing flexible access  
patterns. Always add .env files to .gitignore to prevent committing secrets.  
For production, use proper secret management services instead of files.  

## Secure random token generation

Cryptographically secure random tokens are essential for session IDs, API keys,  
and other security-critical identifiers.  

```go
package main

import (
    "crypto/rand"
    "encoding/base64"
    "encoding/hex"
    "fmt"
    "log"
)

type TokenGenerator struct{}

func NewTokenGenerator() *TokenGenerator {
    return &TokenGenerator{}
}

// GenerateBytes generates n random bytes using crypto/rand
func (tg *TokenGenerator) GenerateBytes(n int) ([]byte, error) {
    b := make([]byte, n)
    _, err := rand.Read(b)
    if err != nil {
        return nil, fmt.Errorf("failed to generate random bytes: %w", err)
    }
    return b, nil
}

// GenerateHex generates a random token as a hex string
func (tg *TokenGenerator) GenerateHex(byteLength int) (string, error) {
    b, err := tg.GenerateBytes(byteLength)
    if err != nil {
        return "", err
    }
    return hex.EncodeToString(b), nil
}

// GenerateBase64 generates a random token as a base64 URL-safe string
func (tg *TokenGenerator) GenerateBase64(byteLength int) (string, error) {
    b, err := tg.GenerateBytes(byteLength)
    if err != nil {
        return "", err
    }
    return base64.URLEncoding.EncodeToString(b), nil
}

// GenerateSessionToken creates a secure session token
func (tg *TokenGenerator) GenerateSessionToken() (string, error) {
    return tg.GenerateBase64(32) // 32 bytes = 256 bits
}

// GenerateAPIKey creates a secure API key
func (tg *TokenGenerator) GenerateAPIKey() (string, error) {
    return tg.GenerateHex(32) // 64 character hex string
}

func main() {
    tg := NewTokenGenerator()
    
    // Generate session token
    sessionToken, err := tg.GenerateSessionToken()
    if err != nil {
        log.Fatalf("Failed to generate session token: %v", err)
    }
    fmt.Printf("Session Token: %s\n", sessionToken)
    
    // Generate API key
    apiKey, err := tg.GenerateAPIKey()
    if err != nil {
        log.Fatalf("Failed to generate API key: %v", err)
    }
    fmt.Printf("API Key: %s\n", apiKey)
    
    // Generate random bytes for encryption keys
    encryptionKey, err := tg.GenerateBytes(32)
    if err != nil {
        log.Fatalf("Failed to generate encryption key: %v", err)
    }
    fmt.Printf("Encryption Key (hex): %s\n", hex.EncodeToString(encryptionKey))
}
```

This example demonstrates generating cryptographically secure random tokens  
using `crypto/rand`, which provides access to the operating system's secure  
random number generator. Never use `math/rand` for security-sensitive tokens  
as it is deterministic and predictable. The example provides multiple encoding  
options: hex for human-readable tokens and base64 for compact representation.  
Always use at least 128 bits (16 bytes) of randomness for security tokens.  



## Password hashing with bcrypt

Password hashing protects user credentials by making it computationally infeasible  
to reverse the hash and obtain the original password.  

```go
package main

import (
    "fmt"
    "log"
    
    "golang.org/x/crypto/bcrypt"
)

type PasswordHasher struct {
    cost int
}

func NewPasswordHasher(cost int) *PasswordHasher {
    // Cost between 10-14 is recommended. Higher is more secure but slower
    if cost < bcrypt.MinCost || cost > bcrypt.MaxCost {
        cost = bcrypt.DefaultCost
    }
    return &PasswordHasher{cost: cost}
}

func (ph *PasswordHasher) Hash(password string) (string, error) {
    if len(password) == 0 {
        return "", fmt.Errorf("password cannot be empty")
    }
    
    if len(password) > 72 {
        return "", fmt.Errorf("password too long (max 72 bytes for bcrypt)")
    }
    
    hash, err := bcrypt.GenerateFromPassword([]byte(password), ph.cost)
    if err != nil {
        return "", fmt.Errorf("failed to hash password: %w", err)
    }
    
    return string(hash), nil
}

func (ph *PasswordHasher) Verify(password, hash string) error {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    if err != nil {
        if err == bcrypt.ErrMismatchedHashAndPassword {
            return fmt.Errorf("invalid password")
        }
        return fmt.Errorf("failed to verify password: %w", err)
    }
    return nil
}

func (ph *PasswordHasher) NeedsRehash(hash string) bool {
    cost, err := bcrypt.Cost([]byte(hash))
    if err != nil {
        return true
    }
    return cost < ph.cost
}

func main() {
    hasher := NewPasswordHasher(12) // Cost factor of 12
    
    password := "SecureP@ssw0rd123"
    
    // Hash the password
    hash, err := hasher.Hash(password)
    if err != nil {
        log.Fatalf("Failed to hash password: %v", err)
    }
    
    fmt.Printf("Original password: %s\n", password)
    fmt.Printf("Hashed password: %s\n", hash)
    
    // Verify correct password
    if err := hasher.Verify(password, hash); err != nil {
        fmt.Printf("Verification failed: %v\n", err)
    } else {
        fmt.Println("Password verified successfully!")
    }
    
    // Verify incorrect password
    wrongPassword := "WrongPassword"
    if err := hasher.Verify(wrongPassword, hash); err != nil {
        fmt.Printf("Wrong password correctly rejected: %v\n", err)
    }
    
    // Check if hash needs updating
    if hasher.NeedsRehash(hash) {
        fmt.Println("Password hash should be regenerated with higher cost")
    } else {
        fmt.Println("Password hash is up to date")
    }
}
```

This example implements secure password hashing using bcrypt, which includes  
a salt and is designed to be slow to compute, making brute-force attacks  
impractical. The cost factor determines how many iterations are performed—each  
increment doubles the time required. Cost 12 is a good balance for modern  
systems. The `NeedsRehash` function allows upgrading password security when  
cost factors are increased. Never store passwords in plain text or use fast  
hashing algorithms like MD5 or SHA-1 for passwords.  


## SHA-256 hashing for data integrity

SHA-256 provides cryptographic hashing for verifying data integrity and creating  
content-based identifiers.  

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "io"
    "os"
)

type HashUtility struct{}

func NewHashUtility() *HashUtility {
    return &HashUtility{}
}

// HashString computes SHA-256 hash of a string
func (hu *HashUtility) HashString(data string) string {
    hash := sha256.Sum256([]byte(data))
    return hex.EncodeToString(hash[:])
}

// HashBytes computes SHA-256 hash of byte slice
func (hu *HashUtility) HashBytes(data []byte) string {
    hash := sha256.Sum256(data)
    return hex.EncodeToString(hash[:])
}

// HashFile computes SHA-256 hash of a file
func (hu *HashUtility) HashFile(filename string) (string, error) {
    file, err := os.Open(filename)
    if err != nil {
        return "", fmt.Errorf("failed to open file: %w", err)
    }
    defer file.Close()
    
    hasher := sha256.New()
    if _, err := io.Copy(hasher, file); err != nil {
        return "", fmt.Errorf("failed to hash file: %w", err)
    }
    
    return hex.EncodeToString(hasher.Sum(nil)), nil
}

// VerifyIntegrity checks if data matches expected hash
func (hu *HashUtility) VerifyIntegrity(data []byte, expectedHash string) bool {
    actualHash := hu.HashBytes(data)
    return actualHash == expectedHash
}

// CreateChecksum creates a checksum for data
func (hu *HashUtility) CreateChecksum(data []byte) ([]byte, string) {
    hash := sha256.Sum256(data)
    hashStr := hex.EncodeToString(hash[:])
    return hash[:], hashStr
}

func main() {
    hu := NewHashUtility()
    
    // Hash a string
    message := "Hello there, this is a secret message"
    hash := hu.HashString(message)
    fmt.Printf("Message: %s\n", message)
    fmt.Printf("SHA-256 Hash: %s\n", hash)
    
    // Hash bytes
    data := []byte("Sensitive data that needs integrity verification")
    dataHash := hu.HashBytes(data)
    fmt.Printf("\nData Hash: %s\n", dataHash)
    
    // Verify integrity
    if hu.VerifyIntegrity(data, dataHash) {
        fmt.Println("Data integrity verified successfully!")
    }
    
    // Demonstrate collision resistance
    similar1 := "Password123"
    similar2 := "Password124"
    hash1 := hu.HashString(similar1)
    hash2 := hu.HashString(similar2)
    fmt.Printf("\n'%s' hash: %s\n", similar1, hash1)
    fmt.Printf("'%s' hash: %s\n", similar2, hash2)
    fmt.Println("Even slight changes produce completely different hashes")
    
    // Create checksum
    payload := []byte("Important payload data")
    checksum, checksumStr := hu.CreateChecksum(payload)
    fmt.Printf("\nPayload checksum: %s\n", checksumStr)
    fmt.Printf("Checksum bytes: %d\n", len(checksum))
}
```

This example demonstrates SHA-256 hashing for data integrity verification.  
SHA-256 is a one-way cryptographic hash function that produces a unique 256-bit  
hash for any input. Even tiny changes in input produce completely different  
hashes. Note that SHA-256 is fast and should NOT be used for password hashing—use  
bcrypt or argon2 instead. SHA-256 is ideal for checksums, content-addressable  
storage, digital signatures, and verifying downloads haven't been tampered with.  


## HMAC for message authentication

HMAC (Hash-based Message Authentication Code) ensures message integrity and  
authenticity using a shared secret key.  

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

type HMACUtility struct {
    key []byte
}

func NewHMACUtility(key []byte) *HMACUtility {
    if len(key) < 32 {
        panic("HMAC key should be at least 32 bytes")
    }
    return &HMACUtility{key: key}
}

// GenerateMAC creates a message authentication code
func (hu *HMACUtility) GenerateMAC(message []byte) string {
    mac := hmac.New(sha256.New, hu.key)
    mac.Write(message)
    return hex.EncodeToString(mac.Sum(nil))
}

// VerifyMAC checks if MAC is valid for message
func (hu *HMACUtility) VerifyMAC(message []byte, messageMAC string) bool {
    expectedMAC := hu.GenerateMAC(message)
    // Use constant-time comparison to prevent timing attacks
    return hmac.Equal([]byte(expectedMAC), []byte(messageMAC))
}

// Sign message with MAC
func (hu *HMACUtility) Sign(message []byte) ([]byte, string) {
    mac := hu.GenerateMAC(message)
    return message, mac
}

func main() {
    // Generate a secure key (in practice, load from secure storage)
    key := []byte("my-32-byte-secret-key-for-hmac!!")
    
    hm := NewHMACUtility(key)
    
    // Generate MAC for a message
    message := []byte("Transfer $1000 to account 12345")
    mac := hm.GenerateMAC(message)
    
    fmt.Printf("Message: %s\n", string(message))
    fmt.Printf("MAC: %s\n", mac)
    
    // Verify the MAC
    if hm.VerifyMAC(message, mac) {
        fmt.Println("\nMAC verification: SUCCESS")
        fmt.Println("Message is authentic and hasn't been tampered with")
    } else {
        fmt.Println("\nMAC verification: FAILED")
    }
    
    // Try to tamper with the message
    tamperedMessage := []byte("Transfer $9000 to account 12345")
    fmt.Printf("\nTampered message: %s\n", string(tamperedMessage))
    if hm.VerifyMAC(tamperedMessage, mac) {
        fmt.Println("MAC verification: SUCCESS (should not happen!)")
    } else {
        fmt.Println("MAC verification: FAILED (expected - tampering detected!)")
    }
    
    // Sign a message
    data, signature := hm.Sign([]byte("Important data"))
    fmt.Printf("\nSigned data: %s\n", string(data))
    fmt.Printf("Signature: %s\n", signature)
}
```

This example demonstrates HMAC for ensuring message integrity and authenticity.  
HMAC combines a cryptographic hash function with a secret key to produce a MAC  
that can verify both that a message hasn't been modified and that it came from  
someone who knows the secret key. Always use constant-time comparison (`hmac.Equal`)  
when verifying MACs to prevent timing attacks. HMAC is widely used in API  
authentication, webhook verification, and secure cookie signing.  


## Argon2 password hashing

Argon2 is a modern password hashing algorithm that won the Password Hashing  
Competition and provides better security than bcrypt for new applications.  

```go
package main

import (
    "crypto/rand"
    "encoding/base64"
    "fmt"
    "log"
    "strings"
    
    "golang.org/x/crypto/argon2"
)

type Argon2Hasher struct {
    time    uint32
    memory  uint32
    threads uint8
    keyLen  uint32
}

func NewArgon2Hasher() *Argon2Hasher {
    return &Argon2Hasher{
        time:    1,         // Number of iterations
        memory:  64 * 1024, // 64 MB
        threads: 4,         // Number of parallel threads
        keyLen:  32,        // 32 bytes = 256 bits
    }
}

func (ah *Argon2Hasher) Hash(password string) (string, error) {
    // Generate a random salt
    salt := make([]byte, 16)
    if _, err := rand.Read(salt); err != nil {
        return "", fmt.Errorf("failed to generate salt: %w", err)
    }
    
    // Generate the hash
    hash := argon2.IDKey([]byte(password), salt, ah.time, ah.memory, ah.threads, ah.keyLen)
    
    // Encode hash and salt together
    b64Salt := base64.RawStdEncoding.EncodeToString(salt)
    b64Hash := base64.RawStdEncoding.EncodeToString(hash)
    
    // Format: $argon2id$v=19$m=65536,t=1,p=4$salt$hash
    encoded := fmt.Sprintf("$argon2id$v=19$m=%d,t=%d,p=%d$%s$%s",
        ah.memory, ah.time, ah.threads, b64Salt, b64Hash)
    
    return encoded, nil
}

func (ah *Argon2Hasher) Verify(password, encodedHash string) (bool, error) {
    // Parse the encoded hash
    parts := strings.Split(encodedHash, "$")
    if len(parts) != 6 {
        return false, fmt.Errorf("invalid hash format")
    }
    
    var memory, time uint32
    var threads uint8
    _, err := fmt.Sscanf(parts[3], "m=%d,t=%d,p=%d", &memory, &time, &threads)
    if err != nil {
        return false, fmt.Errorf("failed to parse parameters: %w", err)
    }
    
    salt, err := base64.RawStdEncoding.DecodeString(parts[4])
    if err != nil {
        return false, fmt.Errorf("failed to decode salt: %w", err)
    }
    
    hash, err := base64.RawStdEncoding.DecodeString(parts[5])
    if err != nil {
        return false, fmt.Errorf("failed to decode hash: %w", err)
    }
    
    // Compute hash with same parameters
    computedHash := argon2.IDKey([]byte(password), salt, time, memory, threads, uint32(len(hash)))
    
    // Constant-time comparison
    return compareHashes(hash, computedHash), nil
}

func compareHashes(a, b []byte) bool {
    if len(a) != len(b) {
        return false
    }
    var result byte
    for i := 0; i < len(a); i++ {
        result |= a[i] ^ b[i]
    }
    return result == 0
}

func main() {
    hasher := NewArgon2Hasher()
    
    password := "MySecureP@ssw0rd!"
    
    // Hash password
    hash, err := hasher.Hash(password)
    if err != nil {
        log.Fatalf("Failed to hash password: %v", err)
    }
    
    fmt.Printf("Password: %s\n", password)
    fmt.Printf("Argon2 Hash: %s\n", hash)
    
    // Verify correct password
    valid, err := hasher.Verify(password, hash)
    if err != nil {
        log.Fatalf("Verification error: %v", err)
    }
    
    if valid {
        fmt.Println("\nPassword verification: SUCCESS")
    } else {
        fmt.Println("\nPassword verification: FAILED")
    }
    
    // Verify wrong password
    wrongPassword := "WrongPassword"
    valid, err = hasher.Verify(wrongPassword, hash)
    if err != nil {
        log.Printf("Verification error: %v", err)
    }
    
    if !valid {
        fmt.Println("Wrong password correctly rejected")
    }
}
```

This example implements Argon2id, the recommended variant of Argon2 that provides  
resistance to both side-channel and GPU attacks. Argon2 is memory-hard, making  
it expensive to attack with specialized hardware. The parameters control time cost,  
memory usage, and parallelism. For new applications, prefer Argon2 over bcrypt  
due to its superior resistance to hardware-accelerated attacks. The hash format  
includes all parameters, allowing for gradual migration when security requirements  
change.  


## AES-GCM encryption

AES-GCM provides authenticated encryption combining confidentiality and integrity  
in a single operation.  

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "io"
    "log"
)

type AESEncryptor struct {
    key []byte
}

func NewAESEncryptor(key []byte) (*AESEncryptor, error) {
    if len(key) != 16 && len(key) != 24 && len(key) != 32 {
        return nil, fmt.Errorf("key must be 16, 24, or 32 bytes")
    }
    return &AESEncryptor{key: key}, nil
}

// Encrypt encrypts plaintext using AES-GCM
func (ae *AESEncryptor) Encrypt(plaintext []byte) ([]byte, error) {
    block, err := aes.NewCipher(ae.key)
    if err != nil {
        return nil, fmt.Errorf("failed to create cipher: %w", err)
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, fmt.Errorf("failed to create GCM: %w", err)
    }
    
    // Create a nonce. GCM requires a unique nonce for each encryption
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return nil, fmt.Errorf("failed to generate nonce: %w", err)
    }
    
    // Seal encrypts and authenticates plaintext
    // Prepend nonce to ciphertext for storage/transmission
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}

// Decrypt decrypts ciphertext using AES-GCM
func (ae *AESEncryptor) Decrypt(ciphertext []byte) ([]byte, error) {
    block, err := aes.NewCipher(ae.key)
    if err != nil {
        return nil, fmt.Errorf("failed to create cipher: %w", err)
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, fmt.Errorf("failed to create GCM: %w", err)
    }
    
    nonceSize := gcm.NonceSize()
    if len(ciphertext) < nonceSize {
        return nil, fmt.Errorf("ciphertext too short")
    }
    
    // Extract nonce and ciphertext
    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    
    // Open decrypts and authenticates ciphertext
    plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, fmt.Errorf("decryption failed: %w", err)
    }
    
    return plaintext, nil
}

func main() {
    // Generate a 32-byte (256-bit) key
    key := make([]byte, 32)
    if _, err := rand.Read(key); err != nil {
        log.Fatalf("Failed to generate key: %v", err)
    }
    
    encryptor, err := NewAESEncryptor(key)
    if err != nil {
        log.Fatalf("Failed to create encryptor: %v", err)
    }
    
    // Encrypt data
    plaintext := []byte("Sensitive information that needs protection")
    fmt.Printf("Plaintext: %s\n", string(plaintext))
    
    ciphertext, err := encryptor.Encrypt(plaintext)
    if err != nil {
        log.Fatalf("Encryption failed: %v", err)
    }
    fmt.Printf("Ciphertext (hex): %s\n", hex.EncodeToString(ciphertext))
    
    // Decrypt data
    decrypted, err := encryptor.Decrypt(ciphertext)
    if err != nil {
        log.Fatalf("Decryption failed: %v", err)
    }
    fmt.Printf("Decrypted: %s\n", string(decrypted))
    
    // Verify decryption
    if string(plaintext) == string(decrypted) {
        fmt.Println("\nEncryption/Decryption successful!")
    }
}
```

This example demonstrates AES-GCM (Galois/Counter Mode), which provides both  
encryption and authentication. GCM is an AEAD (Authenticated Encryption with  
Associated Data) mode that detects tampering automatically. Always use a unique  
nonce for each encryption operation—reusing nonces breaks security completely.  
The nonce is prepended to the ciphertext for convenience. AES-GCM is widely  
used in TLS 1.3 and modern security protocols due to its excellent performance  
and security properties.  


## ChaCha20-Poly1305 encryption

ChaCha20-Poly1305 is a modern authenticated encryption algorithm that provides
excellent performance on systems without AES hardware acceleration.  

```go
package main

import (
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "log"
    
    "golang.org/x/crypto/chacha20poly1305"
)

type ChaChaEncryptor struct {
    aead cipher.AEAD
}

func NewChaChaEncryptor(key []byte) (*ChaChaEncryptor, error) {
    if len(key) != chacha20poly1305.KeySize {
        return nil, fmt.Errorf("key must be %d bytes", chacha20poly1305.KeySize)
    }
    
    aead, err := chacha20poly1305.New(key)
    if err != nil {
        return nil, fmt.Errorf("failed to create AEAD: %w", err)
    }
    
    return &ChaChaEncryptor{aead: aead}, nil
}

func (ce *ChaChaEncryptor) Encrypt(plaintext []byte) ([]byte, error) {
    nonce := make([]byte, ce.aead.NonceSize())
    if _, err := rand.Read(nonce); err != nil {
        return nil, fmt.Errorf("failed to generate nonce: %w", err)
    }
    
    ciphertext := ce.aead.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}

func (ce *ChaChaEncryptor) Decrypt(ciphertext []byte) ([]byte, error) {
    if len(ciphertext) < ce.aead.NonceSize() {
        return nil, fmt.Errorf("ciphertext too short")
    }
    
    nonce := ciphertext[:ce.aead.NonceSize()]
    ciphertext = ciphertext[ce.aead.NonceSize():]
    
    plaintext, err := ce.aead.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, fmt.Errorf("decryption failed: %w", err)
    }
    
    return plaintext, nil
}

func main() {
    key := make([]byte, chacha20poly1305.KeySize)
    if _, err := rand.Read(key); err != nil {
        log.Fatalf("Failed to generate key: %v", err)
    }
    
    encryptor, err := NewChaChaEncryptor(key)
    if err != nil {
        log.Fatalf("Failed to create encryptor: %v", err)
    }
    
    plaintext := []byte("Confidential message")
    fmt.Printf("Plaintext: %s\n", string(plaintext))
    
    ciphertext, err := encryptor.Encrypt(plaintext)
    if err != nil {
        log.Fatalf("Encryption failed: %v", err)
    }
    fmt.Printf("Ciphertext (hex): %s\n", hex.EncodeToString(ciphertext))
    
    decrypted, err := encryptor.Decrypt(ciphertext)
    if err != nil {
        log.Fatalf("Decryption failed: %v", err)
    }
    fmt.Printf("Decrypted: %s\n", string(decrypted))
}
```

This example demonstrates ChaCha20-Poly1305, a modern AEAD cipher that's often
faster than AES on devices without hardware AES support. It's used in TLS 1.3
and WireGuard VPN. The algorithm provides the same security guarantees as
AES-GCM but with better software performance. Like GCM, never reuse nonces.  


## ChaCha20-Poly1305 encryption (corrected)

ChaCha20-Poly1305 is a modern authenticated encryption algorithm that provides  
excellent performance on systems without AES hardware acceleration.  

```go
package main

import (
    "crypto/cipher"
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "log"
    
    "golang.org/x/crypto/chacha20poly1305"
)

type ChaChaEncryptor struct {
    aead cipher.AEAD
}

func NewChaChaEncryptor(key []byte) (*ChaChaEncryptor, error) {
    if len(key) != chacha20poly1305.KeySize {
        return nil, fmt.Errorf("key must be %d bytes", chacha20poly1305.KeySize)
    }
    
    aead, err := chacha20poly1305.New(key)
    if err != nil {
        return nil, fmt.Errorf("failed to create AEAD: %w", err)
    }
    
    return &ChaChaEncryptor{aead: aead}, nil
}

func (ce *ChaChaEncryptor) Encrypt(plaintext []byte) ([]byte, error) {
    nonce := make([]byte, ce.aead.NonceSize())
    if _, err := rand.Read(nonce); err != nil {
        return nil, fmt.Errorf("failed to generate nonce: %w", err)
    }
    
    ciphertext := ce.aead.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}

func (ce *ChaChaEncryptor) Decrypt(ciphertext []byte) ([]byte, error) {
    if len(ciphertext) < ce.aead.NonceSize() {
        return nil, fmt.Errorf("ciphertext too short")
    }
    
    nonce := ciphertext[:ce.aead.NonceSize()]
    ciphertext = ciphertext[ce.aead.NonceSize():]
    
    plaintext, err := ce.aead.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, fmt.Errorf("decryption failed: %w", err)
    }
    
    return plaintext, nil
}

func main() {
    key := make([]byte, chacha20poly1305.KeySize)
    if _, err := rand.Read(key); err != nil {
        log.Fatalf("Failed to generate key: %v", err)
    }
    
    encryptor, err := NewChaChaEncryptor(key)
    if err != nil {
        log.Fatalf("Failed to create encryptor: %v", err)
    }
    
    plaintext := []byte("Confidential message")
    fmt.Printf("Plaintext: %s\n", string(plaintext))
    
    ciphertext, err := encryptor.Encrypt(plaintext)
    if err != nil {
        log.Fatalf("Encryption failed: %v", err)
    }
    fmt.Printf("Ciphertext (hex): %s\n", hex.EncodeToString(ciphertext))
    
    decrypted, err := encryptor.Decrypt(ciphertext)
    if err != nil {
        log.Fatalf("Decryption failed: %v", err)
    }
    fmt.Printf("Decrypted: %s\n", string(decrypted))
}
```

This example demonstrates ChaCha20-Poly1305, a modern AEAD cipher that's often  
faster than AES on devices without hardware AES support. It's used in TLS 1.3  
and WireGuard VPN. The algorithm provides the same security guarantees as  
AES-GCM but with better software performance. Like GCM, never reuse nonces.  


## RSA key generation

RSA key generation creates asymmetric key pairs for encryption and digital  
signatures.  

```go
package main

import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/x509"
    "encoding/pem"
    "fmt"
    "log"
    "os"
)

type RSAKeyManager struct {
    privateKey *rsa.PrivateKey
    publicKey  *rsa.PublicKey
}

func NewRSAKeyManager(bits int) (*RSAKeyManager, error) {
    if bits < 2048 {
        return nil, fmt.Errorf("key size must be at least 2048 bits")
    }
    
    privateKey, err := rsa.GenerateKey(rand.Reader, bits)
    if err != nil {
        return nil, fmt.Errorf("failed to generate key: %w", err)
    }
    
    return &RSAKeyManager{
        privateKey: privateKey,
        publicKey:  &privateKey.PublicKey,
    }, nil
}

// SavePrivateKey saves private key to PEM file
func (km *RSAKeyManager) SavePrivateKey(filename string) error {
    privateKeyBytes := x509.MarshalPKCS1PrivateKey(km.privateKey)
    
    privateKeyPEM := &pem.Block{
        Type:  "RSA PRIVATE KEY",
        Bytes: privateKeyBytes,
    }
    
    file, err := os.Create(filename)
    if err != nil {
        return fmt.Errorf("failed to create file: %w", err)
    }
    defer file.Close()
    
    if err := pem.Encode(file, privateKeyPEM); err != nil {
        return fmt.Errorf("failed to encode private key: %w", err)
    }
    
    return nil
}

// SavePublicKey saves public key to PEM file
func (km *RSAKeyManager) SavePublicKey(filename string) error {
    publicKeyBytes, err := x509.MarshalPKIXPublicKey(km.publicKey)
    if err != nil {
        return fmt.Errorf("failed to marshal public key: %w", err)
    }
    
    publicKeyPEM := &pem.Block{
        Type:  "PUBLIC KEY",
        Bytes: publicKeyBytes,
    }
    
    file, err := os.Create(filename)
    if err != nil {
        return fmt.Errorf("failed to create file: %w", err)
    }
    defer file.Close()
    
    if err := pem.Encode(file, publicKeyPEM); err != nil {
        return fmt.Errorf("failed to encode public key: %w", err)
    }
    
    return nil
}

// LoadPrivateKey loads private key from PEM file
func LoadPrivateKey(filename string) (*rsa.PrivateKey, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read file: %w", err)
    }
    
    block, _ := pem.Decode(data)
    if block == nil {
        return nil, fmt.Errorf("failed to decode PEM block")
    }
    
    privateKey, err := x509.ParsePKCS1PrivateKey(block.Bytes)
    if err != nil {
        return nil, fmt.Errorf("failed to parse private key: %w", err)
    }
    
    return privateKey, nil
}

func main() {
    // Generate 2048-bit RSA key pair
    keyManager, err := NewRSAKeyManager(2048)
    if err != nil {
        log.Fatalf("Failed to generate keys: %v", err)
    }
    
    fmt.Println("RSA key pair generated successfully")
    fmt.Printf("Key size: %d bits\n", keyManager.privateKey.N.BitLen())
    
    // Save keys to files
    if err := keyManager.SavePrivateKey("private.pem"); err != nil {
        log.Fatalf("Failed to save private key: %v", err)
    }
    fmt.Println("Private key saved to private.pem")
    
    if err := keyManager.SavePublicKey("public.pem"); err != nil {
        log.Fatalf("Failed to save public key: %v", err)
    }
    fmt.Println("Public key saved to public.pem")
    
    // Load private key back
    loadedKey, err := LoadPrivateKey("private.pem")
    if err != nil {
        log.Fatalf("Failed to load private key: %v", err)
    }
    fmt.Println("\nPrivate key loaded successfully")
    fmt.Printf("Loaded key size: %d bits\n", loadedKey.N.BitLen())
    
    // Clean up
    os.Remove("private.pem")
    os.Remove("public.pem")
}
```

This example demonstrates RSA key pair generation and management. RSA requires  
at least 2048-bit keys for security (3072-bit or 4096-bit for long-term protection).  
The private key must be kept secret and protected, while the public key can be  
freely distributed. PEM format is the standard for storing and transmitting keys.  
Always generate keys using `crypto/rand` for cryptographic security. In production,  
store private keys in secure key management systems, not in files.  


## RSA encryption and decryption

RSA encryption enables secure data transmission using asymmetric key pairs.  

```go
package main

import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "log"
)

type RSAEncryption struct {
    privateKey *rsa.PrivateKey
    publicKey  *rsa.PublicKey
}

func NewRSAEncryption(bits int) (*RSAEncryption, error) {
    privateKey, err := rsa.GenerateKey(rand.Reader, bits)
    if err != nil {
        return nil, fmt.Errorf("failed to generate key: %w", err)
    }
    
    return &RSAEncryption{
        privateKey: privateKey,
        publicKey:  &privateKey.PublicKey,
    }, nil
}

// Encrypt encrypts data with public key using OAEP padding
func (re *RSAEncryption) Encrypt(plaintext []byte) ([]byte, error) {
    ciphertext, err := rsa.EncryptOAEP(
        sha256.New(),
        rand.Reader,
        re.publicKey,
        plaintext,
        nil,
    )
    if err != nil {
        return nil, fmt.Errorf("encryption failed: %w", err)
    }
    return ciphertext, nil
}

// Decrypt decrypts data with private key
func (re *RSAEncryption) Decrypt(ciphertext []byte) ([]byte, error) {
    plaintext, err := rsa.DecryptOAEP(
        sha256.New(),
        rand.Reader,
        re.privateKey,
        ciphertext,
        nil,
    )
    if err != nil {
        return nil, fmt.Errorf("decryption failed: %w", err)
    }
    return plaintext, nil
}

func main() {
    rse, err := NewRSAEncryption(2048)
    if err != nil {
        log.Fatalf("Failed to create RSA encryption: %v", err)
    }
    
    message := []byte("Secret message for RSA encryption")
    fmt.Printf("Original message: %s\n", string(message))
    
    // Encrypt with public key
    ciphertext, err := rse.Encrypt(message)
    if err != nil {
        log.Fatalf("Encryption failed: %v", err)
    }
    fmt.Printf("Ciphertext (hex): %s\n", hex.EncodeToString(ciphertext))
    
    // Decrypt with private key
    decrypted, err := rse.Decrypt(ciphertext)
    if err != nil {
        log.Fatalf("Decryption failed: %v", err)
    }
    fmt.Printf("Decrypted message: %s\n", string(decrypted))
    
    if string(message) == string(decrypted) {
        fmt.Println("\nRSA encryption/decryption successful!")
    }
}
```

This example demonstrates RSA encryption using OAEP (Optimal Asymmetric Encryption  
Padding) which provides semantic security. RSA can only encrypt data smaller than  
the key size minus padding overhead (e.g., 2048-bit key ≈ 190 bytes max). For  
larger data, use hybrid encryption: encrypt data with AES, then encrypt the AES  
key with RSA. Never use RSA without proper padding—OAEP is the recommended scheme.  

## Digital signatures with RSA-PSS

Digital signatures prove authenticity and non-repudiation of messages.  

```go
package main

import (
    "crypto"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "log"
)

type RSASigner struct {
    privateKey *rsa.PrivateKey
    publicKey  *rsa.PublicKey
}

func NewRSASigner(bits int) (*RSASigner, error) {
    privateKey, err := rsa.GenerateKey(rand.Reader, bits)
    if err != nil {
        return nil, fmt.Errorf("failed to generate key: %w", err)
    }
    
    return &RSASigner{
        privateKey: privateKey,
        publicKey:  &privateKey.PublicKey,
    }, nil
}

// Sign creates a digital signature using RSA-PSS
func (rs *RSASigner) Sign(message []byte) ([]byte, error) {
    hashed := sha256.Sum256(message)
    
    signature, err := rsa.SignPSS(
        rand.Reader,
        rs.privateKey,
        crypto.SHA256,
        hashed[:],
        nil,
    )
    if err != nil {
        return nil, fmt.Errorf("signing failed: %w", err)
    }
    
    return signature, nil
}

// Verify checks if signature is valid for message
func (rs *RSASigner) Verify(message, signature []byte) error {
    hashed := sha256.Sum256(message)
    
    err := rsa.VerifyPSS(
        rs.publicKey,
        crypto.SHA256,
        hashed[:],
        signature,
        nil,
    )
    if err != nil {
        return fmt.Errorf("signature verification failed: %w", err)
    }
    
    return nil
}

func main() {
    signer, err := NewRSASigner(2048)
    if err != nil {
        log.Fatalf("Failed to create signer: %v", err)
    }
    
    message := []byte("This document requires a digital signature")
    fmt.Printf("Message: %s\n", string(message))
    
    // Sign the message
    signature, err := signer.Sign(message)
    if err != nil {
        log.Fatalf("Signing failed: %v", err)
    }
    fmt.Printf("Signature (hex): %s\n", hex.EncodeToString(signature))
    
    // Verify the signature
    if err := signer.Verify(message, signature); err != nil {
        fmt.Printf("Verification failed: %v\n", err)
    } else {
        fmt.Println("\nSignature verified successfully!")
        fmt.Println("Message is authentic and hasn't been modified")
    }
    
    // Try to verify tampered message
    tamperedMessage := []byte("This document has been tampered with")
    fmt.Printf("\nTampered message: %s\n", string(tamperedMessage))
    if err := signer.Verify(tamperedMessage, signature); err != nil {
        fmt.Println("Verification correctly failed for tampered message")
    }
}
```

This example demonstrates digital signatures using RSA-PSS (Probabilistic Signature  
Scheme), which provides provable security. Signatures prove that a message was  
created by the holder of the private key and hasn't been modified. RSA-PSS is  
preferred over PKCS#1 v1.5 signatures due to better security properties. Digital  
signatures are essential for software distribution, document signing, and API  
authentication where non-repudiation is required.  

## ECDSA key generation and signing

ECDSA provides smaller keys with equivalent security compared to RSA.  

```go
package main

import (
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/sha256"
    "encoding/asn1"
    "encoding/hex"
    "fmt"
    "log"
    "math/big"
)

type ECDSASigner struct {
    privateKey *ecdsa.PrivateKey
    publicKey  *ecdsa.PublicKey
}

type ECDSASignature struct {
    R, S *big.Int
}

func NewECDSASigner() (*ECDSASigner, error) {
    // Use P-256 curve (also known as secp256r1 or prime256v1)
    privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        return nil, fmt.Errorf("failed to generate key: %w", err)
    }
    
    return &ECDSASigner{
        privateKey: privateKey,
        publicKey:  &privateKey.PublicKey,
    }, nil
}

// Sign creates an ECDSA signature
func (es *ECDSASigner) Sign(message []byte) ([]byte, error) {
    hashed := sha256.Sum256(message)
    
    r, s, err := ecdsa.Sign(rand.Reader, es.privateKey, hashed[:])
    if err != nil {
        return nil, fmt.Errorf("signing failed: %w", err)
    }
    
    // Encode signature as ASN.1
    signature := ECDSASignature{R: r, S: s}
    signatureBytes, err := asn1.Marshal(signature)
    if err != nil {
        return nil, fmt.Errorf("failed to marshal signature: %w", err)
    }
    
    return signatureBytes, nil
}

// Verify checks ECDSA signature
func (es *ECDSASigner) Verify(message, signatureBytes []byte) (bool, error) {
    hashed := sha256.Sum256(message)
    
    var signature ECDSASignature
    if _, err := asn1.Unmarshal(signatureBytes, &signature); err != nil {
        return false, fmt.Errorf("failed to unmarshal signature: %w", err)
    }
    
    valid := ecdsa.Verify(es.publicKey, hashed[:], signature.R, signature.S)
    return valid, nil
}

func main() {
    signer, err := NewECDSASigner()
    if err != nil {
        log.Fatalf("Failed to create ECDSA signer: %v", err)
    }
    
    fmt.Println("ECDSA P-256 key pair generated")
    
    message := []byte("Sign this important message")
    fmt.Printf("Message: %s\n", string(message))
    
    // Sign the message
    signature, err := signer.Sign(message)
    if err != nil {
        log.Fatalf("Signing failed: %v", err)
    }
    fmt.Printf("Signature (hex): %s\n", hex.EncodeToString(signature))
    fmt.Printf("Signature size: %d bytes\n", len(signature))
    
    // Verify the signature
    valid, err := signer.Verify(message, signature)
    if err != nil {
        log.Fatalf("Verification error: %v", err)
    }
    
    if valid {
        fmt.Println("\nSignature verified successfully!")
    } else {
        fmt.Println("\nSignature verification failed!")
    }
    
    // Test with wrong message
    wrongMessage := []byte("Different message")
    valid, _ = signer.Verify(wrongMessage, signature)
    if !valid {
        fmt.Println("Signature correctly rejected for different message")
    }
}
```

This example demonstrates ECDSA (Elliptic Curve Digital Signature Algorithm)  
using the P-256 curve. ECDSA provides the same security as RSA with much smaller  
keys: a 256-bit ECDSA key provides equivalent security to a 3072-bit RSA key.  
This makes ECDSA ideal for constrained environments and reduces bandwidth  
requirements. ECDSA is widely used in TLS, Bitcoin, and modern authentication  
systems. Always use a secure curve like P-256 or P-384.  


## PBKDF2 key derivation

PBKDF2 derives cryptographic keys from passwords using repeated hashing.  

```go
package main

import (
    "crypto/rand"
    "crypto/sha256"
    "encoding/base64"
    "fmt"
    "log"
    
    "golang.org/x/crypto/pbkdf2"
)

type PBKDF2Deriver struct {
    iterations int
    keyLength  int
}

func NewPBKDF2Deriver() *PBKDF2Deriver {
    return &PBKDF2Deriver{
        iterations: 100000, // NIST recommends at least 100,000 iterations
        keyLength:  32,     // 256 bits
    }
}

// DeriveKey derives a key from password and salt
func (pd *PBKDF2Deriver) DeriveKey(password string, salt []byte) []byte {
    return pbkdf2.Key([]byte(password), salt, pd.iterations, pd.keyLength, sha256.New)
}

// GenerateSalt creates a random salt
func (pd *PBKDF2Deriver) GenerateSalt() ([]byte, error) {
    salt := make([]byte, 16) // 128 bits
    if _, err := rand.Read(salt); err != nil {
        return nil, fmt.Errorf("failed to generate salt: %w", err)
    }
    return salt, nil
}

// CreateStoredKey generates salt and derives key for storage
func (pd *PBKDF2Deriver) CreateStoredKey(password string) (string, error) {
    salt, err := pd.GenerateSalt()
    if err != nil {
        return "", err
    }
    
    key := pd.DeriveKey(password, salt)
    
    // Encode salt and key together
    encoded := fmt.Sprintf("%s:%s",
        base64.StdEncoding.EncodeToString(salt),
        base64.StdEncoding.EncodeToString(key))
    
    return encoded, nil
}

func main() {
    deriver := NewPBKDF2Deriver()
    
    password := "UserPassword123!"
    
    // Generate salt
    salt, err := deriver.GenerateSalt()
    if err != nil {
        log.Fatalf("Failed to generate salt: %v", err)
    }
    
    fmt.Printf("Password: %s\n", password)
    fmt.Printf("Salt (base64): %s\n", base64.StdEncoding.EncodeToString(salt))
    
    // Derive key
    key := deriver.DeriveKey(password, salt)
    fmt.Printf("Derived key (base64): %s\n", base64.StdEncoding.EncodeToString(key))
    fmt.Printf("Key length: %d bytes\n", len(key))
    
    // Create stored key format
    storedKey, err := deriver.CreateStoredKey(password)
    if err != nil {
        log.Fatalf("Failed to create stored key: %v", err)
    }
    fmt.Printf("\nStored key format: %s\n", storedKey)
    
    // Derive again with same salt - should produce same key
    key2 := deriver.DeriveKey(password, salt)
    fmt.Printf("\nKeys match: %t\n", string(key) == string(key2))
}
```

This example demonstrates PBKDF2 (Password-Based Key Derivation Function 2) for  
deriving encryption keys from passwords. PBKDF2 uses many iterations to slow down  
brute-force attacks. Always use a unique random salt for each password and store  
it with the derived key. PBKDF2 is widely supported and suitable for deriving  
keys for encryption, not for password hashing (use bcrypt/argon2 instead). The  
iteration count should be tuned to take at least 100ms on your system.  

## Secure key storage patterns

Proper key storage prevents unauthorized access to cryptographic keys.  

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/base64"
    "fmt"
    "io"
    "log"
    "os"
    
    "golang.org/x/crypto/scrypt"
)

type SecureKeyStore struct {
    masterPassword string
}

func NewSecureKeyStore(masterPassword string) *SecureKeyStore {
    return &SecureKeyStore{masterPassword: masterPassword}
}

// StoreKey encrypts and saves a key to file
func (sks *SecureKeyStore) StoreKey(keyData []byte, filename string) error {
    // Derive encryption key from master password
    salt := make([]byte, 32)
    if _, err := rand.Read(salt); err != nil {
        return fmt.Errorf("failed to generate salt: %w", err)
    }
    
    // Use scrypt for key derivation (more secure than PBKDF2)
    encryptionKey, err := scrypt.Key([]byte(sks.masterPassword), salt, 32768, 8, 1, 32)
    if err != nil {
        return fmt.Errorf("key derivation failed: %w", err)
    }
    
    // Encrypt the key data
    block, err := aes.NewCipher(encryptionKey)
    if err != nil {
        return fmt.Errorf("failed to create cipher: %w", err)
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return fmt.Errorf("failed to create GCM: %w", err)
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return fmt.Errorf("failed to generate nonce: %w", err)
    }
    
    ciphertext := gcm.Seal(nonce, nonce, keyData, nil)
    
    // Combine salt and ciphertext
    stored := append(salt, ciphertext...)
    
    // Encode and write to file
    encoded := base64.StdEncoding.EncodeToString(stored)
    if err := os.WriteFile(filename, []byte(encoded), 0600); err != nil {
        return fmt.Errorf("failed to write file: %w", err)
    }
    
    return nil
}

// LoadKey decrypts and loads a key from file
func (sks *SecureKeyStore) LoadKey(filename string) ([]byte, error) {
    // Read file
    encoded, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read file: %w", err)
    }
    
    // Decode
    stored, err := base64.StdEncoding.DecodeString(string(encoded))
    if err != nil {
        return nil, fmt.Errorf("failed to decode: %w", err)
    }
    
    if len(stored) < 32 {
        return nil, fmt.Errorf("invalid stored key format")
    }
    
    // Extract salt and ciphertext
    salt := stored[:32]
    ciphertext := stored[32:]
    
    // Derive encryption key
    encryptionKey, err := scrypt.Key([]byte(sks.masterPassword), salt, 32768, 8, 1, 32)
    if err != nil {
        return nil, fmt.Errorf("key derivation failed: %w", err)
    }
    
    // Decrypt
    block, err := aes.NewCipher(encryptionKey)
    if err != nil {
        return nil, fmt.Errorf("failed to create cipher: %w", err)
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, fmt.Errorf("failed to create GCM: %w", err)
    }
    
    nonceSize := gcm.NonceSize()
    if len(ciphertext) < nonceSize {
        return nil, fmt.Errorf("ciphertext too short")
    }
    
    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    
    plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, fmt.Errorf("decryption failed: %w", err)
    }
    
    return plaintext, nil
}

func main() {
    masterPassword := "StrongMasterPassword123!"
    keyStore := NewSecureKeyStore(masterPassword)
    
    // Generate a key to store
    apiKey := make([]byte, 32)
    if _, err := rand.Read(apiKey); err != nil {
        log.Fatalf("Failed to generate API key: %v", err)
    }
    
    fmt.Printf("Original API key: %x\n", apiKey)
    
    // Store the key
    filename := "encrypted_key.dat"
    if err := keyStore.StoreKey(apiKey, filename); err != nil {
        log.Fatalf("Failed to store key: %v", err)
    }
    fmt.Printf("Key stored securely in %s\n", filename)
    
    // Load the key
    loadedKey, err := keyStore.LoadKey(filename)
    if err != nil {
        log.Fatalf("Failed to load key: %v", err)
    }
    fmt.Printf("Loaded API key: %x\n", loadedKey)
    
    if string(apiKey) == string(loadedKey) {
        fmt.Println("\nKey stored and retrieved successfully!")
    }
    
    // Clean up
    os.Remove(filename)
}
```

This example demonstrates secure key storage using encryption. The master password  
is used to derive an encryption key via scrypt, which encrypts the actual key  
data. The file permissions are set to 0600 (owner read/write only). In production,  
use dedicated key management systems (KMS) like AWS KMS, Azure Key Vault, or  
HashiCorp Vault instead of file-based storage. Never hardcode keys in source code  
or store them in plain text.  


## Secret rotation implementation

Implementing secret rotation minimizes the impact of key compromise.  

```go
package main

import (
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "sync"
    "time"
)

type SecretRotator struct {
    currentSecret  string
    previousSecret string
    mu             sync.RWMutex
    rotationPeriod time.Duration
    stopChan       chan struct{}
}

func NewSecretRotator(initialSecret string, rotationPeriod time.Duration) *SecretRotator {
    return &SecretRotator{
        currentSecret:  initialSecret,
        previousSecret: "",
        rotationPeriod: rotationPeriod,
        stopChan:       make(chan struct{}),
    }
}

// generateSecret creates a new random secret
func (sr *SecretRotator) generateSecret() (string, error) {
    bytes := make([]byte, 32)
    if _, err := rand.Read(bytes); err != nil {
        return "", fmt.Errorf("failed to generate secret: %w", err)
    }
    return hex.EncodeToString(bytes), nil
}

// RotateSecret performs the rotation
func (sr *SecretRotator) RotateSecret() error {
    newSecret, err := sr.generateSecret()
    if err != nil {
        return err
    }
    
    sr.mu.Lock()
    sr.previousSecret = sr.currentSecret
    sr.currentSecret = newSecret
    sr.mu.Unlock()
    
    fmt.Printf("[%s] Secret rotated\n", time.Now().Format("15:04:05"))
    return nil
}

// GetCurrent returns the current secret
func (sr *SecretRotator) GetCurrent() string {
    sr.mu.RLock()
    defer sr.mu.RUnlock()
    return sr.currentSecret
}

// ValidateSecret checks if a secret is current or previous (grace period)
func (sr *SecretRotator) ValidateSecret(secret string) bool {
    sr.mu.RLock()
    defer sr.mu.RUnlock()
    return secret == sr.currentSecret || secret == sr.previousSecret
}

// StartAutoRotation begins automatic rotation
func (sr *SecretRotator) StartAutoRotation() {
    ticker := time.NewTicker(sr.rotationPeriod)
    go func() {
        for {
            select {
            case <-ticker.C:
                if err := sr.RotateSecret(); err != nil {
                    fmt.Printf("Rotation failed: %v\n", err)
                }
            case <-sr.stopChan:
                ticker.Stop()
                return
            }
        }
    }()
}

// Stop stops automatic rotation
func (sr *SecretRotator) Stop() {
    close(sr.stopChan)
}

func main() {
    // Create rotator with initial secret
    initialSecret, _ := hex.DecodeString("abcdef1234567890")
    rotator := NewSecretRotator(hex.EncodeToString(initialSecret), 5*time.Second)
    
    fmt.Printf("Initial secret: %s\n", rotator.GetCurrent())
    
    // Start automatic rotation
    rotator.StartAutoRotation()
    
    // Simulate secret usage
    for i := 0; i < 15; i++ {
        time.Sleep(2 * time.Second)
        current := rotator.GetCurrent()
        fmt.Printf("[%s] Using secret: %s...\n", 
            time.Now().Format("15:04:05"), current[:16])
    }
    
    // Stop rotation
    rotator.Stop()
    fmt.Println("Rotation stopped")
}
```

This example implements secret rotation with a grace period that accepts both  
current and previous secrets. This allows gradual rollout of new secrets without  
breaking active sessions. The rotation happens automatically on a schedule, and  
the system maintains both current and previous secrets during the transition  
period. In production, coordinate rotation across all services and use a  
centralized secret management system.  

## Basic TLS server setup

TLS servers provide encrypted communication channels for secure data transmission.  

```go
package main

import (
    "crypto/tls"
    "fmt"
    "log"
    "net/http"
)

type SecureServer struct {
    server *http.Server
}

func NewSecureServer(addr, certFile, keyFile string) *SecureServer {
    mux := http.NewServeMux()
    
    // Add handlers
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Secure connection established!\n")
        fmt.Fprintf(w, "TLS Version: %s\n", tlsVersionString(r.TLS.Version))
        fmt.Fprintf(w, "Cipher Suite: %s\n", tls.CipherSuiteName(r.TLS.CipherSuite))
    })
    
    mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, "OK\n")
    })
    
    // Configure TLS
    tlsConfig := &tls.Config{
        MinVersion: tls.VersionTLS12,
        CipherSuites: []uint16{
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
            tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
        },
        PreferServerCipherSuites: true,
    }
    
    server := &http.Server{
        Addr:      addr,
        Handler:   mux,
        TLSConfig: tlsConfig,
    }
    
    return &SecureServer{server: server}
}

func tlsVersionString(version uint16) string {
    switch version {
    case tls.VersionTLS10:
        return "TLS 1.0"
    case tls.VersionTLS11:
        return "TLS 1.1"
    case tls.VersionTLS12:
        return "TLS 1.2"
    case tls.VersionTLS13:
        return "TLS 1.3"
    default:
        return "Unknown"
    }
}

func (ss *SecureServer) Start(certFile, keyFile string) error {
    log.Printf("Starting secure server on %s", ss.server.Addr)
    return ss.server.ListenAndServeTLS(certFile, keyFile)
}

func main() {
    // In production, use real certificates from a CA
    // For testing, generate self-signed certificate:
    // openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365
    
    server := NewSecureServer(":8443", "cert.pem", "key.pem")
    
    log.Fatal(server.Start("cert.pem", "key.pem"))
}
```

This example demonstrates a basic TLS server with security best practices: TLS 1.2  
minimum version, strong cipher suites preferring ECDHE for forward secrecy, and  
proper TLS configuration. The server enforces secure ciphers and provides information  
about the established connection. In production, obtain certificates from Let's  
Encrypt or a commercial CA, configure proper timeouts, and implement graceful  
shutdown.  

## TLS client with certificate validation

TLS clients must validate server certificates to prevent man-in-the-middle attacks.  

```go
package main

import (
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "time"
)

type SecureTLSClient struct {
    httpClient *http.Client
}

func NewSecureTLSClient(caCertFile string) (*SecureTLSClient, error) {
    // Load CA certificate for server verification
    caCert, err := os.ReadFile(caCertFile)
    if err != nil {
        return nil, fmt.Errorf("failed to read CA cert: %w", err)
    }
    
    caCertPool := x509.NewCertPool()
    if !caCertPool.AppendCertsFromPEM(caCert) {
        return nil, fmt.Errorf("failed to parse CA cert")
    }
    
    // Configure TLS
    tlsConfig := &tls.Config{
        RootCAs:    caCertPool,
        MinVersion: tls.VersionTLS12,
        // Verify server certificate
        InsecureSkipVerify: false,
    }
    
    transport := &http.Transport{
        TLSClientConfig: tlsConfig,
        // Set reasonable timeouts
        TLSHandshakeTimeout:   10 * time.Second,
        ResponseHeaderTimeout: 10 * time.Second,
        IdleConnTimeout:       30 * time.Second,
    }
    
    httpClient := &http.Client{
        Transport: transport,
        Timeout:   30 * time.Second,
    }
    
    return &SecureTLSClient{httpClient: httpClient}, nil
}

// Get performs a secure GET request
func (stc *SecureTLSClient) Get(url string) ([]byte, error) {
    resp, err := stc.httpClient.Get(url)
    if err != nil {
        return nil, fmt.Errorf("request failed: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("unexpected status: %d", resp.StatusCode)
    }
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed to read response: %w", err)
    }
    
    return body, nil
}

// VerifyConnection checks TLS connection details
func (stc *SecureTLSClient) VerifyConnection(url string) error {
    resp, err := stc.httpClient.Get(url)
    if err != nil {
        return fmt.Errorf("connection failed: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.TLS == nil {
        return fmt.Errorf("connection is not TLS")
    }
    
    fmt.Printf("TLS Version: %s\n", tlsVersionString(resp.TLS.Version))
    fmt.Printf("Cipher Suite: %s\n", tls.CipherSuiteName(resp.TLS.CipherSuite))
    fmt.Printf("Server Name: %s\n", resp.TLS.ServerName)
    fmt.Printf("Peer Certificates: %d\n", len(resp.TLS.PeerCertificates))
    
    if len(resp.TLS.PeerCertificates) > 0 {
        cert := resp.TLS.PeerCertificates[0]
        fmt.Printf("Certificate Subject: %s\n", cert.Subject)
        fmt.Printf("Certificate Issuer: %s\n", cert.Issuer)
        fmt.Printf("Valid Until: %s\n", cert.NotAfter)
    }
    
    return nil
}

func tlsVersionString(version uint16) string {
    switch version {
    case tls.VersionTLS12:
        return "TLS 1.2"
    case tls.VersionTLS13:
        return "TLS 1.3"
    default:
        return fmt.Sprintf("Unknown (%d)", version)
    }
}

func main() {
    // Create client with CA certificate
    client, err := NewSecureTLSClient("ca-cert.pem")
    if err != nil {
        log.Fatalf("Failed to create client: %v", err)
    }
    
    // Make secure request
    url := "https://example.com"
    fmt.Printf("Connecting to %s\n", url)
    
    if err := client.VerifyConnection(url); err != nil {
        log.Printf("Connection verification failed: %v", err)
    }
    
    body, err := client.Get(url)
    if err != nil {
        log.Fatalf("Request failed: %v", err)
    }
    
    fmt.Printf("\nReceived %d bytes\n", len(body))
}
```

This example demonstrates a secure TLS client with proper certificate validation.  
The client loads a CA certificate bundle and verifies the server's certificate  
chain. Never use `InsecureSkipVerify: true` in production—it disables all  
certificate validation and makes connections vulnerable to MITM attacks. The  
example includes proper timeouts and connection verification to ensure secure  
communication.  


## Mutual TLS (mTLS) authentication

Mutual TLS requires both client and server to authenticate with certificates.  

```go
package main

import (
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
)

type MTLSServer struct {
    server *http.Server
}

func NewMTLSServer(addr, certFile, keyFile, clientCAFile string) (*MTLSServer, error) {
    // Load client CA certificate
    clientCACert, err := os.ReadFile(clientCAFile)
    if err != nil {
        return nil, fmt.Errorf("failed to read client CA: %w", err)
    }
    
    clientCAPool := x509.NewCertPool()
    if !clientCAPool.AppendCertsFromPEM(clientCACert) {
        return nil, fmt.Errorf("failed to parse client CA")
    }
    
    // Configure mTLS
    tlsConfig := &tls.Config{
        ClientAuth: tls.RequireAndVerifyClientCert,
        ClientCAs:  clientCAPool,
        MinVersion: tls.VersionTLS12,
    }
    
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // Extract client certificate info
        if r.TLS != nil && len(r.TLS.PeerCertificates) > 0 {
            cert := r.TLS.PeerCertificates[0]
            fmt.Fprintf(w, "Authenticated client: %s\n", cert.Subject.CommonName)
            fmt.Fprintf(w, "Certificate valid until: %s\n", cert.NotAfter)
        }
        fmt.Fprintf(w, "Mutual TLS connection established!\n")
    })
    
    server := &http.Server{
        Addr:      addr,
        Handler:   mux,
        TLSConfig: tlsConfig,
    }
    
    return &MTLSServer{server: server}, nil
}

func (ms *MTLSServer) Start(certFile, keyFile string) error {
    log.Printf("Starting mTLS server on %s", ms.server.Addr)
    return ms.server.ListenAndServeTLS(certFile, keyFile)
}

type MTLSClient struct {
    httpClient *http.Client
}

func NewMTLSClient(serverCAFile, clientCertFile, clientKeyFile string) (*MTLSClient, error) {
    // Load server CA
    serverCACert, err := os.ReadFile(serverCAFile)
    if err != nil {
        return nil, fmt.Errorf("failed to read server CA: %w", err)
    }
    
    serverCAPool := x509.NewCertPool()
    if !serverCAPool.AppendCertsFromPEM(serverCACert) {
        return nil, fmt.Errorf("failed to parse server CA")
    }
    
    // Load client certificate
    clientCert, err := tls.LoadX509KeyPair(clientCertFile, clientKeyFile)
    if err != nil {
        return nil, fmt.Errorf("failed to load client cert: %w", err)
    }
    
    tlsConfig := &tls.Config{
        RootCAs:      serverCAPool,
        Certificates: []tls.Certificate{clientCert},
        MinVersion:   tls.VersionTLS12,
    }
    
    transport := &http.Transport{
        TLSClientConfig: tlsConfig,
    }
    
    httpClient := &http.Client{
        Transport: transport,
    }
    
    return &MTLSClient{httpClient: httpClient}, nil
}

func (mc *MTLSClient) Get(url string) (string, error) {
    resp, err := mc.httpClient.Get(url)
    if err != nil {
        return "", fmt.Errorf("request failed: %w", err)
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return "", fmt.Errorf("failed to read response: %w", err)
    }
    
    return string(body), nil
}

func main() {
    fmt.Println("Mutual TLS (mTLS) authentication example")
    fmt.Println("This requires proper certificate setup:")
    fmt.Println("- Server cert/key")
    fmt.Println("- Client cert/key")
    fmt.Println("- CA certificates for both")
    
    // Start server in goroutine
    // server, _ := NewMTLSServer(":8443", "server-cert.pem", "server-key.pem", "client-ca.pem")
    // go server.Start("server-cert.pem", "server-key.pem")
    
    // Create client
    // client, _ := NewMTLSClient("server-ca.pem", "client-cert.pem", "client-key.pem")
    // response, _ := client.Get("https://localhost:8443/")
    // fmt.Println(response)
}
```

This example demonstrates mutual TLS where both parties authenticate each other  
with certificates. mTLS is essential for zero-trust architectures and service-to-service  
authentication. The server requires and verifies client certificates, while the  
client verifies the server certificate. mTLS provides strong authentication without  
passwords and is commonly used in microservices, API gateways, and Kubernetes.  

## Certificate pinning

Certificate pinning adds an extra layer of security by trusting specific  
certificates rather than any CA-signed certificate.  

```go
package main

import (
    "crypto/sha256"
    "crypto/tls"
    "crypto/x509"
    "encoding/hex"
    "fmt"
    "io"
    "net/http"
)

type CertificatePinner struct {
    pinnedFingerprints map[string]bool
    httpClient         *http.Client
}

func NewCertificatePinner(fingerprints []string) *CertificatePinner {
    pinnedMap := make(map[string]bool)
    for _, fp := range fingerprints {
        pinnedMap[fp] = true
    }
    
    cp := &CertificatePinner{
        pinnedFingerprints: pinnedMap,
    }
    
    // Custom TLS config with verification callback
    tlsConfig := &tls.Config{
        InsecureSkipVerify: false, // Still verify chain
        VerifyPeerCertificate: func(rawCerts [][]byte, verifiedChains [][]*x509.Certificate) error {
            return cp.verifyPinnedCertificate(rawCerts, verifiedChains)
        },
    }
    
    transport := &http.Transport{
        TLSClientConfig: tlsConfig,
    }
    
    cp.httpClient = &http.Client{
        Transport: transport,
    }
    
    return cp
}

func (cp *CertificatePinner) verifyPinnedCertificate(rawCerts [][]byte, verifiedChains [][]*x509.Certificate) error {
    // Check each certificate in the chain
    for _, rawCert := range rawCerts {
        fingerprint := sha256.Sum256(rawCert)
        fingerprintHex := hex.EncodeToString(fingerprint[:])
        
        if cp.pinnedFingerprints[fingerprintHex] {
            return nil // Found pinned certificate
        }
    }
    
    return fmt.Errorf("certificate pinning failed: no pinned certificate found")
}

// ComputeFingerprint calculates SHA-256 fingerprint of a certificate
func ComputeFingerprint(certPEM []byte) (string, error) {
    block, _ := pem.Decode(certPEM)
    if block == nil {
        return "", fmt.Errorf("failed to decode PEM")
    }
    
    fingerprint := sha256.Sum256(block.Bytes)
    return hex.EncodeToString(fingerprint[:]), nil
}

func (cp *CertificatePinner) Get(url string) (string, error) {
    resp, err := cp.httpClient.Get(url)
    if err != nil {
        return "", fmt.Errorf("request failed: %w", err)
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return "", fmt.Errorf("failed to read response: %w", err)
    }
    
    return string(body), nil
}

func main() {
    // Pin to specific certificate fingerprints
    // In production, obtain these fingerprints from your certificates
    pinnedFingerprints := []string{
        "abcdef1234567890...", // SHA-256 fingerprint of trusted cert
    }
    
    pinner := NewCertificatePinner(pinnedFingerprints)
    
    fmt.Println("Certificate pinning configured")
    fmt.Println("Only connections to pinned certificates will succeed")
    
    // Example: Make request with pinning
    // response, err := pinner.Get("https://example.com")
    // if err != nil {
    //     log.Printf("Pinning failed: %v", err)
    // }
}
```

This example demonstrates certificate pinning, which prevents MITM attacks even  
if a CA is compromised. By pinning specific certificate fingerprints, the client  
only accepts those certificates regardless of CA validation. Update pinned  
fingerprints before certificates expire. Certificate pinning is used in mobile  
apps and high-security environments but requires careful management to avoid  
breaking connections when certificates rotate.  


## TLS with SNI (Server Name Indication)

SNI allows multiple TLS certificates on a single IP address.  

```go
package main

import (
    "crypto/tls"
    "fmt"
    "log"
    "net/http"
)

type SNIServer struct {
    server *http.Server
}

func NewSNIServer(addr string) *SNIServer {
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Host: %s\n", r.Host)
        if r.TLS != nil {
            fmt.Fprintf(w, "Server Name: %s\n", r.TLS.ServerName)
        }
    })
    
    // Configure TLS with SNI support
    tlsConfig := &tls.Config{
        MinVersion: tls.VersionTLS12,
        GetCertificate: func(hello *tls.ClientHelloInfo) (*tls.Certificate, error) {
            // Select certificate based on SNI
            serverName := hello.ServerName
            
            var certFile, keyFile string
            switch serverName {
            case "example.com":
                certFile, keyFile = "example.com.crt", "example.com.key"
            case "api.example.com":
                certFile, keyFile = "api.example.com.crt", "api.example.com.key"
            default:
                certFile, keyFile = "default.crt", "default.key"
            }
            
            cert, err := tls.LoadX509KeyPair(certFile, keyFile)
            if err != nil {
                return nil, fmt.Errorf("failed to load certificate for %s: %w", serverName, err)
            }
            
            return &cert, nil
        },
    }
    
    server := &http.Server{
        Addr:      addr,
        Handler:   mux,
        TLSConfig: tlsConfig,
    }
    
    return &SNIServer{server: server}
}

func (ss *SNIServer) Start() error {
    log.Printf("Starting SNI-enabled server on %s", ss.server.Addr)
    // ListenAndServeTLS is called with empty strings because certs are loaded dynamically
    return ss.server.ListenAndServeTLS("", "")
}

func main() {
    server := NewSNIServer(":8443")
    
    fmt.Println("SNI server supports multiple domains:")
    fmt.Println("- example.com")
    fmt.Println("- api.example.com")
    fmt.Println("- (default for others)")
    
    // log.Fatal(server.Start())
}
```

This example demonstrates SNI (Server Name Indication) which allows hosting  
multiple domains with different TLS certificates on a single IP address. The  
`GetCertificate` callback dynamically selects the appropriate certificate based  
on the requested hostname. SNI is essential for modern hosting environments  
and enables efficient use of IP addresses. All modern browsers and clients  
support SNI.  

## Secure random number generation

Cryptographically secure random numbers are fundamental to security operations.  

```go
package main

import (
    "crypto/rand"
    "encoding/binary"
    "fmt"
    "log"
    "math/big"
)

type SecureRandom struct{}

func NewSecureRandom() *SecureRandom {
    return &SecureRandom{}
}

// Bytes generates n random bytes
func (sr *SecureRandom) Bytes(n int) ([]byte, error) {
    b := make([]byte, n)
    _, err := rand.Read(b)
    if err != nil {
        return nil, fmt.Errorf("failed to generate random bytes: %w", err)
    }
    return b, nil
}

// Int generates a random integer in range [0, max)
func (sr *SecureRandom) Int(max int64) (int64, error) {
    if max <= 0 {
        return 0, fmt.Errorf("max must be positive")
    }
    
    n, err := rand.Int(rand.Reader, big.NewInt(max))
    if err != nil {
        return 0, fmt.Errorf("failed to generate random int: %w", err)
    }
    
    return n.Int64(), nil
}

// IntRange generates random integer in range [min, max]
func (sr *SecureRandom) IntRange(min, max int64) (int64, error) {
    if min >= max {
        return 0, fmt.Errorf("min must be less than max")
    }
    
    rangeSize := max - min + 1
    n, err := sr.Int(rangeSize)
    if err != nil {
        return 0, err
    }
    
    return min + n, nil
}

// Uint64 generates a random uint64
func (sr *SecureRandom) Uint64() (uint64, error) {
    b := make([]byte, 8)
    _, err := rand.Read(b)
    if err != nil {
        return 0, fmt.Errorf("failed to generate random uint64: %w", err)
    }
    
    return binary.BigEndian.Uint64(b), nil
}

// Shuffle randomly shuffles a slice
func (sr *SecureRandom) Shuffle(slice []interface{}) error {
    n := len(slice)
    for i := n - 1; i > 0; i-- {
        j, err := sr.Int(int64(i + 1))
        if err != nil {
            return err
        }
        slice[i], slice[j] = slice[j], slice[i]
    }
    return nil
}

func main() {
    sr := NewSecureRandom()
    
    // Generate random bytes
    randomBytes, err := sr.Bytes(16)
    if err != nil {
        log.Fatalf("Failed to generate bytes: %v", err)
    }
    fmt.Printf("Random bytes: %x\n", randomBytes)
    
    // Generate random integers
    randomInt, err := sr.Int(100)
    if err != nil {
        log.Fatalf("Failed to generate int: %v", err)
    }
    fmt.Printf("Random int [0-100): %d\n", randomInt)
    
    // Generate in range
    randomRange, err := sr.IntRange(50, 150)
    if err != nil {
        log.Fatalf("Failed to generate range int: %v", err)
    }
    fmt.Printf("Random int [50-150]: %d\n", randomRange)
    
    // Generate uint64
    randomUint64, err := sr.Uint64()
    if err != nil {
        log.Fatalf("Failed to generate uint64: %v", err)
    }
    fmt.Printf("Random uint64: %d\n", randomUint64)
}
```

This example demonstrates generating cryptographically secure random numbers  
using `crypto/rand`. Never use `math/rand` for security purposes as it's  
deterministic and predictable. The crypto package provides a cryptographically  
secure random number generator (CSPRNG) that's suitable for generating keys,  
nonces, tokens, and other security-critical values. Always check for errors  
when generating random data.  

## JWT token generation with HS256

JWTs provide stateless authentication tokens for modern web applications.  

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "strings"
    "time"
)

type JWTClaims struct {
    Sub string `json:"sub"`
    Name string `json:"name"`
    Iat int64  `json:"iat"`
    Exp int64  `json:"exp"`
}

type JWTGenerator struct {
    secret []byte
}

func NewJWTGenerator(secret string) *JWTGenerator {
    return &JWTGenerator{secret: []byte(secret)}
}

func (jg *JWTGenerator) base64URLEncode(data []byte) string {
    return strings.TrimRight(base64.URLEncoding.EncodeToString(data), "=")
}

func (jg *JWTGenerator) base64URLDecode(s string) ([]byte, error) {
    // Add padding if needed
    if m := len(s) % 4; m != 0 {
        s += strings.Repeat("=", 4-m)
    }
    return base64.URLEncoding.DecodeString(s)
}

func (jg *JWTGenerator) GenerateToken(userID, userName string, expiration time.Duration) (string, error) {
    // Create header
    header := map[string]interface{}{
        "alg": "HS256",
        "typ": "JWT",
    }
    
    headerJSON, err := json.Marshal(header)
    if err != nil {
        return "", fmt.Errorf("failed to marshal header: %w", err)
    }
    
    // Create claims
    now := time.Now()
    claims := JWTClaims{
        Sub:  userID,
        Name: userName,
        Iat:  now.Unix(),
        Exp:  now.Add(expiration).Unix(),
    }
    
    claimsJSON, err := json.Marshal(claims)
    if err != nil {
        return "", fmt.Errorf("failed to marshal claims: %w", err)
    }
    
    // Encode header and claims
    encodedHeader := jg.base64URLEncode(headerJSON)
    encodedClaims := jg.base64URLEncode(claimsJSON)
    
    // Create signature
    message := fmt.Sprintf("%s.%s", encodedHeader, encodedClaims)
    signature := jg.sign([]byte(message))
    encodedSignature := jg.base64URLEncode(signature)
    
    // Combine all parts
    token := fmt.Sprintf("%s.%s.%s", encodedHeader, encodedClaims, encodedSignature)
    
    return token, nil
}

func (jg *JWTGenerator) sign(message []byte) []byte {
    mac := hmac.New(sha256.New, jg.secret)
    mac.Write(message)
    return mac.Sum(nil)
}

func (jg *JWTGenerator) VerifyToken(token string) (*JWTClaims, error) {
    parts := strings.Split(token, ".")
    if len(parts) != 3 {
        return nil, fmt.Errorf("invalid token format")
    }
    
    // Verify signature
    message := fmt.Sprintf("%s.%s", parts[0], parts[1])
    expectedSignature := jg.sign([]byte(message))
    encodedExpectedSignature := jg.base64URLEncode(expectedSignature)
    
    if parts[2] != encodedExpectedSignature {
        return nil, fmt.Errorf("invalid signature")
    }
    
    // Decode claims
    claimsJSON, err := jg.base64URLDecode(parts[1])
    if err != nil {
        return nil, fmt.Errorf("failed to decode claims: %w", err)
    }
    
    var claims JWTClaims
    if err := json.Unmarshal(claimsJSON, &claims); err != nil {
        return nil, fmt.Errorf("failed to unmarshal claims: %w", err)
    }
    
    // Check expiration
    if time.Now().Unix() > claims.Exp {
        return nil, fmt.Errorf("token expired")
    }
    
    return &claims, nil
}

func main() {
    generator := NewJWTGenerator("your-256-bit-secret-key-here!!")
    
    // Generate token
    token, err := generator.GenerateToken("user123", "John Doe", 1*time.Hour)
    if err != nil {
        fmt.Printf("Failed to generate token: %v\n", err)
        return
    }
    
    fmt.Println("Generated JWT Token:")
    fmt.Println(token)
    
    // Verify token
    claims, err := generator.VerifyToken(token)
    if err != nil {
        fmt.Printf("Token verification failed: %v\n", err)
        return
    }
    
    fmt.Println("\nToken verified successfully!")
    fmt.Printf("User ID: %s\n", claims.Sub)
    fmt.Printf("Name: %s\n", claims.Name)
    fmt.Printf("Issued At: %s\n", time.Unix(claims.Iat, 0))
    fmt.Printf("Expires At: %s\n", time.Unix(claims.Exp, 0))
}
```

This example demonstrates JWT generation and verification using HMAC-SHA256 (HS256).  
JWTs consist of three base64url-encoded parts: header, payload, and signature,  
separated by dots. The token is self-contained and can be verified without database  
lookups. Always use a strong secret key (at least 256 bits) and implement proper  
expiration checking. For production, consider using libraries like `golang-jwt/jwt`.  


## JWT with RSA signatures (RS256)

RS256 uses asymmetric keys for JWT signing, enabling public key verification.  

```go
package main

import (
    "crypto"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "strings"
    "time"
)

type RS256JWTManager struct {
    privateKey *rsa.PrivateKey
    publicKey  *rsa.PublicKey
}

func NewRS256JWTManager(bits int) (*RS256JWTManager, error) {
    privateKey, err := rsa.GenerateKey(rand.Reader, bits)
    if err != nil {
        return nil, fmt.Errorf("failed to generate key: %w", err)
    }
    
    return &RS256JWTManager{
        privateKey: privateKey,
        publicKey:  &privateKey.PublicKey,
    }, nil
}

func (jm *RS256JWTManager) base64URLEncode(data []byte) string {
    return strings.TrimRight(base64.URLEncoding.EncodeToString(data), "=")
}

func (jm *RS256JWTManager) base64URLDecode(s string) ([]byte, error) {
    if m := len(s) % 4; m != 0 {
        s += strings.Repeat("=", 4-m)
    }
    return base64.URLEncoding.DecodeString(s)
}

func (jm *RS256JWTManager) GenerateToken(subject, name string, duration time.Duration) (string, error) {
    header := map[string]string{
        "alg": "RS256",
        "typ": "JWT",
    }
    
    headerJSON, _ := json.Marshal(header)
    
    now := time.Now()
    claims := map[string]interface{}{
        "sub":  subject,
        "name": name,
        "iat":  now.Unix(),
        "exp":  now.Add(duration).Unix(),
    }
    
    claimsJSON, _ := json.Marshal(claims)
    
    encodedHeader := jm.base64URLEncode(headerJSON)
    encodedClaims := jm.base64URLEncode(claimsJSON)
    
    message := fmt.Sprintf("%s.%s", encodedHeader, encodedClaims)
    
    // Sign with RSA private key
    hashed := sha256.Sum256([]byte(message))
    signature, err := rsa.SignPKCS1v15(rand.Reader, jm.privateKey, crypto.SHA256, hashed[:])
    if err != nil {
        return "", fmt.Errorf("signing failed: %w", err)
    }
    
    encodedSignature := jm.base64URLEncode(signature)
    token := fmt.Sprintf("%s.%s.%s", encodedHeader, encodedClaims, encodedSignature)
    
    return token, nil
}

func (jm *RS256JWTManager) VerifyToken(token string) (map[string]interface{}, error) {
    parts := strings.Split(token, ".")
    if len(parts) != 3 {
        return nil, fmt.Errorf("invalid token format")
    }
    
    message := fmt.Sprintf("%s.%s", parts[0], parts[1])
    signature, err := jm.base64URLDecode(parts[2])
    if err != nil {
        return nil, fmt.Errorf("failed to decode signature: %w", err)
    }
    
    // Verify with RSA public key
    hashed := sha256.Sum256([]byte(message))
    err = rsa.VerifyPKCS1v15(jm.publicKey, crypto.SHA256, hashed[:], signature)
    if err != nil {
        return nil, fmt.Errorf("signature verification failed: %w", err)
    }
    
    // Decode and return claims
    claimsJSON, err := jm.base64URLDecode(parts[1])
    if err != nil {
        return nil, fmt.Errorf("failed to decode claims: %w", err)
    }
    
    var claims map[string]interface{}
    if err := json.Unmarshal(claimsJSON, &claims); err != nil {
        return nil, fmt.Errorf("failed to unmarshal claims: %w", err)
    }
    
    // Check expiration
    if exp, ok := claims["exp"].(float64); ok {
        if time.Now().Unix() > int64(exp) {
            return nil, fmt.Errorf("token expired")
        }
    }
    
    return claims, nil
}

func main() {
    manager, err := NewRS256JWTManager(2048)
    if err != nil {
        fmt.Printf("Failed to create manager: %v\n", err)
        return
    }
    
    token, err := manager.GenerateToken("user456", "Jane Smith", 2*time.Hour)
    if err != nil {
        fmt.Printf("Failed to generate token: %v\n", err)
        return
    }
    
    fmt.Println("Generated RS256 JWT Token:")
    fmt.Println(token)
    
    claims, err := manager.VerifyToken(token)
    if err != nil {
        fmt.Printf("Verification failed: %v\n", err)
        return
    }
    
    fmt.Println("\nToken verified!")
    fmt.Printf("Claims: %+v\n", claims)
}
```

This example demonstrates JWT with RS256 (RSA signature). Unlike HS256 which  
uses a shared secret, RS256 uses asymmetric keys: sign with private key, verify  
with public key. This allows public key distribution without compromising signing  
ability. RS256 is preferred for distributed systems where multiple services need  
to verify tokens but only an authorization server should sign them. The public  
key can be shared openly through JWKS endpoints.  

## JWT refresh tokens

Refresh tokens enable long-lived sessions without exposing access tokens.  

```go
package main

import (
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "sync"
    "time"
)

type TokenPair struct {
    AccessToken  string
    RefreshToken string
    ExpiresIn    int64
}

type RefreshTokenStore struct {
    tokens map[string]*RefreshTokenData
    mu     sync.RWMutex
}

type RefreshTokenData struct {
    UserID    string
    ExpiresAt time.Time
    Used      bool
}

func NewRefreshTokenStore() *RefreshTokenStore {
    return &RefreshTokenStore{
        tokens: make(map[string]*RefreshTokenData),
    }
}

func (rts *RefreshTokenStore) GenerateTokenPair(userID string) (*TokenPair, error) {
    // Generate short-lived access token (15 minutes)
    accessToken, err := generateRandomToken(32)
    if err != nil {
        return nil, fmt.Errorf("failed to generate access token: %w", err)
    }
    
    // Generate long-lived refresh token (7 days)
    refreshToken, err := generateRandomToken(32)
    if err != nil {
        return nil, fmt.Errorf("failed to generate refresh token: %w", err)
    }
    
    // Store refresh token
    rts.mu.Lock()
    rts.tokens[refreshToken] = &RefreshTokenData{
        UserID:    userID,
        ExpiresAt: time.Now().Add(7 * 24 * time.Hour),
        Used:      false,
    }
    rts.mu.Unlock()
    
    return &TokenPair{
        AccessToken:  accessToken,
        RefreshToken: refreshToken,
        ExpiresIn:    900, // 15 minutes in seconds
    }, nil
}

func (rts *RefreshTokenStore) RefreshAccess(refreshToken string) (*TokenPair, error) {
    rts.mu.Lock()
    defer rts.mu.Unlock()
    
    // Check if refresh token exists
    data, exists := rts.tokens[refreshToken]
    if !exists {
        return nil, fmt.Errorf("invalid refresh token")
    }
    
    // Check if already used (rotation)
    if data.Used {
        // Refresh token reuse detected - revoke all tokens for user
        return nil, fmt.Errorf("refresh token already used - possible attack")
    }
    
    // Check expiration
    if time.Now().After(data.ExpiresAt) {
        delete(rts.tokens, refreshToken)
        return nil, fmt.Errorf("refresh token expired")
    }
    
    // Mark as used (rotation)
    data.Used = true
    
    // Generate new token pair
    newAccessToken, _ := generateRandomToken(32)
    newRefreshToken, _ := generateRandomToken(32)
    
    // Store new refresh token
    rts.tokens[newRefreshToken] = &RefreshTokenData{
        UserID:    data.UserID,
        ExpiresAt: time.Now().Add(7 * 24 * time.Hour),
        Used:      false,
    }
    
    return &TokenPair{
        AccessToken:  newAccessToken,
        RefreshToken: newRefreshToken,
        ExpiresIn:    900,
    }, nil
}

func (rts *RefreshTokenStore) Revoke(refreshToken string) error {
    rts.mu.Lock()
    defer rts.mu.Unlock()
    
    delete(rts.tokens, refreshToken)
    return nil
}

func generateRandomToken(length int) (string, error) {
    bytes := make([]byte, length)
    if _, err := rand.Read(bytes); err != nil {
        return "", err
    }
    return hex.EncodeToString(bytes), nil
}

func main() {
    store := NewRefreshTokenStore()
    
    // Generate initial token pair
    tokens, err := store.GenerateTokenPair("user789")
    if err != nil {
        fmt.Printf("Failed to generate tokens: %v\n", err)
        return
    }
    
    fmt.Println("Initial Token Pair:")
    fmt.Printf("Access Token: %s...\n", tokens.AccessToken[:20])
    fmt.Printf("Refresh Token: %s...\n", tokens.RefreshToken[:20])
    fmt.Printf("Expires In: %d seconds\n", tokens.ExpiresIn)
    
    // Simulate token refresh
    time.Sleep(1 * time.Second)
    
    newTokens, err := store.RefreshAccess(tokens.RefreshToken)
    if err != nil {
        fmt.Printf("Refresh failed: %v\n", err)
        return
    }
    
    fmt.Println("\nRefreshed Token Pair:")
    fmt.Printf("New Access Token: %s...\n", newTokens.AccessToken[:20])
    fmt.Printf("New Refresh Token: %s...\n", newTokens.RefreshToken[:20])
    
    // Try to reuse old refresh token (should fail)
    _, err = store.RefreshAccess(tokens.RefreshToken)
    if err != nil {
        fmt.Printf("\nRefresh token reuse detected: %v\n", err)
    }
}
```

This example implements refresh token pattern with rotation. Access tokens are  
short-lived (15 min) for security, while refresh tokens are long-lived (7 days)  
but stored securely. Each refresh generates a new token pair and invalidates  
the old refresh token (rotation). Detecting refresh token reuse can indicate  
token theft. Store refresh tokens in httpOnly cookies and implement proper  
revocation mechanisms. Never store refresh tokens in localStorage.  


## JWT token blacklisting

Token blacklisting prevents revoked JWTs from being used until expiration.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type TokenBlacklist struct {
    blacklist map[string]time.Time
    mu        sync.RWMutex
}

func NewTokenBlacklist() *TokenBlacklist {
    bl := &TokenBlacklist{
        blacklist: make(map[string]time.Time),
    }
    
    // Start cleanup goroutine
    go bl.cleanupExpired()
    
    return bl
}

// Revoke adds a token to the blacklist
func (tb *TokenBlacklist) Revoke(tokenID string, expiresAt time.Time) {
    tb.mu.Lock()
    defer tb.mu.Unlock()
    
    tb.blacklist[tokenID] = expiresAt
}

// IsRevoked checks if a token is blacklisted
func (tb *TokenBlacklist) IsRevoked(tokenID string) bool {
    tb.mu.RLock()
    defer tb.mu.RUnlock()
    
    expiresAt, exists := tb.blacklist[tokenID]
    if !exists {
        return false
    }
    
    // If token expired naturally, remove from blacklist
    if time.Now().After(expiresAt) {
        tb.mu.RUnlock()
        tb.mu.Lock()
        delete(tb.blacklist, tokenID)
        tb.mu.Unlock()
        tb.mu.RLock()
        return false
    }
    
    return true
}

// cleanupExpired removes expired tokens from blacklist
func (tb *TokenBlacklist) cleanupExpired() {
    ticker := time.NewTicker(1 * time.Hour)
    defer ticker.Stop()
    
    for range ticker.C {
        tb.mu.Lock()
        now := time.Now()
        for tokenID, expiresAt := range tb.blacklist {
            if now.After(expiresAt) {
                delete(tb.blacklist, tokenID)
            }
        }
        tb.mu.Unlock()
    }
}

// RevokeAllForUser revokes all tokens for a user
func (tb *TokenBlacklist) RevokeAllForUser(userTokenIDs []string, expiresAt time.Time) {
    tb.mu.Lock()
    defer tb.mu.Unlock()
    
    for _, tokenID := range userTokenIDs {
        tb.blacklist[tokenID] = expiresAt
    }
}

// Size returns the number of blacklisted tokens
func (tb *TokenBlacklist) Size() int {
    tb.mu.RLock()
    defer tb.mu.RUnlock()
    return len(tb.blacklist)
}

func main() {
    blacklist := NewTokenBlacklist()
    
    // Revoke a token
    tokenID := "token-abc123"
    expiresAt := time.Now().Add(1 * time.Hour)
    blacklist.Revoke(tokenID, expiresAt)
    
    fmt.Printf("Token %s revoked\n", tokenID)
    fmt.Printf("Blacklist size: %d\n", blacklist.Size())
    
    // Check if token is revoked
    if blacklist.IsRevoked(tokenID) {
        fmt.Printf("Token %s is revoked\n", tokenID)
    }
    
    // Check a valid token
    validToken := "token-xyz789"
    if !blacklist.IsRevoked(validToken) {
        fmt.Printf("Token %s is valid\n", validToken)
    }
    
    // Revoke multiple tokens for a user
    userTokens := []string{"token-1", "token-2", "token-3"}
    blacklist.RevokeAllForUser(userTokens, expiresAt)
    fmt.Printf("\nRevoked %d tokens for user\n", len(userTokens))
    fmt.Printf("Blacklist size: %d\n", blacklist.Size())
}
```

This example implements JWT token blacklisting for revocation. When a user logs  
out or a token is compromised, add its JTI (JWT ID claim) to the blacklist.  
The blacklist automatically cleans up expired entries to prevent unbounded growth.  
For distributed systems, use Redis or similar for shared blacklist storage. Note  
that blacklisting requires checking storage on every request, reducing JWT's  
stateless advantage. Consider short token lifetimes to minimize blacklist size.  

## HTTP security headers

Security headers protect against common web vulnerabilities.  

```go
package main

import (
    "fmt"
    "net/http"
)

type SecurityHeaders struct {
    ContentSecurityPolicy string
    XFrameOptions         string
    XContentTypeOptions   string
    StrictTransportSecurity string
    ReferrerPolicy        string
    PermissionsPolicy     string
}

func DefaultSecurityHeaders() *SecurityHeaders {
    return &SecurityHeaders{
        ContentSecurityPolicy:   "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'none';",
        XFrameOptions:          "DENY",
        XContentTypeOptions:    "nosniff",
        StrictTransportSecurity: "max-age=31536000; includeSubDomains; preload",
        ReferrerPolicy:         "strict-origin-when-cross-origin",
        PermissionsPolicy:      "geolocation=(), microphone=(), camera=()",
    }
}

func SecurityHeadersMiddleware(headers *SecurityHeaders) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Set security headers
            if headers.ContentSecurityPolicy != "" {
                w.Header().Set("Content-Security-Policy", headers.ContentSecurityPolicy)
            }
            
            if headers.XFrameOptions != "" {
                w.Header().Set("X-Frame-Options", headers.XFrameOptions)
            }
            
            if headers.XContentTypeOptions != "" {
                w.Header().Set("X-Content-Type-Options", headers.XContentTypeOptions)
            }
            
            if headers.StrictTransportSecurity != "" {
                // Only set HSTS on HTTPS connections
                if r.TLS != nil {
                    w.Header().Set("Strict-Transport-Security", headers.StrictTransportSecurity)
                }
            }
            
            if headers.ReferrerPolicy != "" {
                w.Header().Set("Referrer-Policy", headers.ReferrerPolicy)
            }
            
            if headers.PermissionsPolicy != "" {
                w.Header().Set("Permissions-Policy", headers.PermissionsPolicy)
            }
            
            // Remove sensitive headers
            w.Header().Del("X-Powered-By")
            w.Header().Del("Server")
            
            next.ServeHTTP(w, r)
        })
    }
}

func main() {
    headers := DefaultSecurityHeaders()
    
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Secure page with security headers\n")
    })
    
    // Wrap with security headers middleware
    secureHandler := SecurityHeadersMiddleware(headers)(mux)
    
    fmt.Println("Server with security headers running on :8080")
    http.ListenAndServe(":8080", secureHandler)
}
```

This example implements essential HTTP security headers. CSP prevents XSS attacks,  
X-Frame-Options prevents clickjacking, HSTS enforces HTTPS, X-Content-Type-Options  
prevents MIME sniffing, and Referrer-Policy controls referrer information. These  
headers form a critical defense layer. Always test CSP policies carefully as they  
can break functionality. Use report-only mode initially to identify issues.  

## Input validation and sanitization

Input validation prevents injection attacks and data corruption.  

```go
package main

import (
    "fmt"
    "html"
    "net/mail"
    "regexp"
    "strings"
    "unicode"
)

type InputValidator struct {
    emailRegex    *regexp.Regexp
    usernameRegex *regexp.Regexp
    urlRegex      *regexp.Regexp
}

func NewInputValidator() *InputValidator {
    return &InputValidator{
        emailRegex:    regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
        usernameRegex: regexp.MustCompile(`^[a-zA-Z0-9_-]{3,20}$`),
        urlRegex:      regexp.MustCompile(`^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$`),
    }
}

// ValidateEmail validates email address
func (iv *InputValidator) ValidateEmail(email string) (bool, error) {
    email = strings.TrimSpace(email)
    
    if len(email) == 0 {
        return false, fmt.Errorf("email cannot be empty")
    }
    
    if len(email) > 254 {
        return false, fmt.Errorf("email too long")
    }
    
    // Use net/mail for standards-compliant validation
    _, err := mail.ParseAddress(email)
    if err != nil {
        return false, fmt.Errorf("invalid email format")
    }
    
    return true, nil
}

// ValidateUsername validates username
func (iv *InputValidator) ValidateUsername(username string) (bool, error) {
    username = strings.TrimSpace(username)
    
    if len(username) < 3 {
        return false, fmt.Errorf("username too short (min 3 characters)")
    }
    
    if len(username) > 20 {
        return false, fmt.Errorf("username too long (max 20 characters)")
    }
    
    if !iv.usernameRegex.MatchString(username) {
        return false, fmt.Errorf("username can only contain letters, numbers, underscore, and hyphen")
    }
    
    return true, nil
}

// ValidatePassword validates password strength
func (iv *InputValidator) ValidatePassword(password string) (bool, error) {
    if len(password) < 8 {
        return false, fmt.Errorf("password too short (min 8 characters)")
    }
    
    if len(password) > 128 {
        return false, fmt.Errorf("password too long (max 128 characters)")
    }
    
    var (
        hasUpper   = false
        hasLower   = false
        hasNumber  = false
        hasSpecial = false
    )
    
    for _, char := range password {
        switch {
        case unicode.IsUpper(char):
            hasUpper = true
        case unicode.IsLower(char):
            hasLower = true
        case unicode.IsNumber(char):
            hasNumber = true
        case unicode.IsPunct(char) || unicode.IsSymbol(char):
            hasSpecial = true
        }
    }
    
    if !hasUpper || !hasLower || !hasNumber || !hasSpecial {
        return false, fmt.Errorf("password must contain uppercase, lowercase, number, and special character")
    }
    
    return true, nil
}

// SanitizeHTML removes dangerous HTML tags
func (iv *InputValidator) SanitizeHTML(input string) string {
    // Escape HTML entities
    return html.EscapeString(input)
}

// ValidateURL validates URL format
func (iv *InputValidator) ValidateURL(url string) (bool, error) {
    url = strings.TrimSpace(url)
    
    if len(url) == 0 {
        return false, fmt.Errorf("URL cannot be empty")
    }
    
    if !iv.urlRegex.MatchString(url) {
        return false, fmt.Errorf("invalid URL format")
    }
    
    return true, nil
}

// SanitizeString removes control characters and trims
func (iv *InputValidator) SanitizeString(input string) string {
    // Remove control characters except newline and tab
    filtered := strings.Map(func(r rune) rune {
        if unicode.IsControl(r) && r != '\n' && r != '\t' {
            return -1
        }
        return r
    }, input)
    
    return strings.TrimSpace(filtered)
}

func main() {
    validator := NewInputValidator()
    
    // Test email validation
    emails := []string{"user@example.com", "invalid.email", "test@domain"}
    for _, email := range emails {
        valid, err := validator.ValidateEmail(email)
        fmt.Printf("Email '%s': valid=%t, error=%v\n", email, valid, err)
    }
    
    // Test username validation
    fmt.Println("\nUsername validation:")
    usernames := []string{"john_doe", "ab", "user@name", "valid_user123"}
    for _, username := range usernames {
        valid, err := validator.ValidateUsername(username)
        fmt.Printf("Username '%s': valid=%t, error=%v\n", username, valid, err)
    }
    
    // Test password validation
    fmt.Println("\nPassword validation:")
    passwords := []string{"weak", "StrongP@ss123", "NoSpecial123"}
    for _, password := range passwords {
        valid, err := validator.ValidatePassword(password)
        fmt.Printf("Password (hidden): valid=%t, error=%v\n", valid, err)
    }
    
    // Test HTML sanitization
    fmt.Println("\nHTML sanitization:")
    dangerous := "<script>alert('XSS')</script>Hello"
    sanitized := validator.SanitizeHTML(dangerous)
    fmt.Printf("Original: %s\n", dangerous)
    fmt.Printf("Sanitized: %s\n", sanitized)
}
```

This example demonstrates comprehensive input validation and sanitization.  
Always validate on the server side regardless of client validation. Use whitelist  
validation (allow known-good) rather than blacklist (block known-bad). Sanitize  
output based on context: HTML escaping for HTML, URL encoding for URLs, SQL  
parameterization for databases. Never trust user input and validate at system  
boundaries.  

