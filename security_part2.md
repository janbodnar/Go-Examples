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

