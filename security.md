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

