# Go Configuration Management

Configuration management is a fundamental aspect of application development  
that enables applications to adapt to different environments, customize  
behavior without code changes, and maintain separation between code and  
configuration data. Go provides excellent support for various configuration  
approaches, from simple environment variables to complex structured  
configuration files using JSON, YAML, and TOML formats.

Configuration management encompasses several key patterns and practices:  
loading configuration from multiple sources with precedence rules, validating  
configuration values for correctness and security, providing sensible defaults  
for optional settings, supporting hot-reloading for dynamic configuration  
updates, and handling sensitive information like passwords and API keys  
securely. Modern applications often combine multiple configuration sources,  
allowing configuration through files, environment variables, command-line  
flags, and remote configuration services.

Go's type system and reflection capabilities make configuration handling both  
type-safe and flexible. The use of struct tags enables declarative mapping  
between configuration sources and Go data structures, while interface  
implementations allow for pluggable configuration providers. The standard  
library's encoding packages provide robust support for JSON, while third-party  
libraries extend this to YAML and TOML formats with similar ease of use.

Effective configuration management strategies consider deployment environments  
(development, staging, production), configuration validation and error handling,  
secret management and security, configuration versioning and migration,  
performance implications of configuration loading, and observability through  
configuration logging and metrics. These considerations ensure applications  
remain maintainable, secure, and operational across different environments  
and deployment scenarios.

## Basic configuration structure

A well-designed configuration structure forms the foundation for robust  
configuration management in Go applications.  

```go
package main

import (
    "fmt"
    "time"
)

// DatabaseConfig holds database connection settings
type DatabaseConfig struct {
    Host     string `json:"host"`
    Port     int    `json:"port"`
    Name     string `json:"name"`
    User     string `json:"user"`
    Password string `json:"password"`
    SSLMode  string `json:"ssl_mode"`
}

// ServerConfig holds HTTP server settings
type ServerConfig struct {
    Host         string        `json:"host"`
    Port         int           `json:"port"`
    ReadTimeout  time.Duration `json:"read_timeout"`
    WriteTimeout time.Duration `json:"write_timeout"`
    IdleTimeout  time.Duration `json:"idle_timeout"`
}

// LoggingConfig holds logging configuration
type LoggingConfig struct {
    Level  string `json:"level"`
    Format string `json:"format"`
    Output string `json:"output"`
}

// AppConfig represents the complete application configuration
type AppConfig struct {
    Environment string          `json:"environment"`
    Debug       bool            `json:"debug"`
    Server      ServerConfig    `json:"server"`
    Database    DatabaseConfig  `json:"database"`
    Logging     LoggingConfig   `json:"logging"`
    Features    map[string]bool `json:"features"`
}

func main() {
    config := AppConfig{
        Environment: "development",
        Debug:       true,
        Server: ServerConfig{
            Host:         "localhost",
            Port:         8080,
            ReadTimeout:  30 * time.Second,
            WriteTimeout: 30 * time.Second,
            IdleTimeout:  60 * time.Second,
        },
        Database: DatabaseConfig{
            Host:     "localhost",
            Port:     5432,
            Name:     "myapp",
            User:     "postgres",
            Password: "secret",
            SSLMode:  "disable",
        },
        Logging: LoggingConfig{
            Level:  "debug",
            Format: "json",
            Output: "stdout",
        },
        Features: map[string]bool{
            "metrics":  true,
            "tracing":  false,
            "caching":  true,
        },
    }

    fmt.Printf("Environment: %s\n", config.Environment)
    fmt.Printf("Server: %s:%d\n", config.Server.Host, config.Server.Port)
    fmt.Printf("Database: %s@%s:%d/%s\n", 
        config.Database.User, config.Database.Host, 
        config.Database.Port, config.Database.Name)
    fmt.Printf("Logging: %s level, %s format\n", 
        config.Logging.Level, config.Logging.Format)
    
    for feature, enabled := range config.Features {
        fmt.Printf("Feature %s: %v\n", feature, enabled)
    }
}
```

This example establishes a well-structured configuration pattern using nested  
structs with JSON tags. Each configuration section is organized logically,  
and the use of Go's duration type provides type safety for time-based  
settings. The map for features demonstrates dynamic configuration options.  

## JSON configuration file loading

JSON configuration files provide a structured, readable format for  
application settings that integrates seamlessly with Go's type system.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "time"
)

type Config struct {
    AppName     string        `json:"app_name"`
    Version     string        `json:"version"`
    Environment string        `json:"environment"`
    Server      ServerConfig  `json:"server"`
    Database    DatabaseConfig `json:"database"`
    Redis       RedisConfig   `json:"redis"`
}

type ServerConfig struct {
    Host    string        `json:"host"`
    Port    int           `json:"port"`
    Timeout time.Duration `json:"timeout"`
    TLS     TLSConfig     `json:"tls"`
}

type TLSConfig struct {
    Enabled  bool   `json:"enabled"`
    CertFile string `json:"cert_file"`
    KeyFile  string `json:"key_file"`
}

type DatabaseConfig struct {
    Driver   string `json:"driver"`
    Host     string `json:"host"`
    Port     int    `json:"port"`
    Name     string `json:"name"`
    User     string `json:"user"`
    Password string `json:"password"`
    MaxConns int    `json:"max_connections"`
}

type RedisConfig struct {
    Host     string `json:"host"`
    Port     int    `json:"port"`
    DB       int    `json:"database"`
    Password string `json:"password"`
}

func LoadConfig(filename string) (*Config, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to open config file: %w", err)
    }
    defer file.Close()

    var config Config
    decoder := json.NewDecoder(file)
    if err := decoder.Decode(&config); err != nil {
        return nil, fmt.Errorf("failed to decode JSON config: %w", err)
    }

    return &config, nil
}

func (c *Config) SaveConfig(filename string) error {
    file, err := os.Create(filename)
    if err != nil {
        return fmt.Errorf("failed to create config file: %w", err)
    }
    defer file.Close()

    encoder := json.NewEncoder(file)
    encoder.SetIndent("", "  ")
    if err := encoder.Encode(c); err != nil {
        return fmt.Errorf("failed to encode JSON config: %w", err)
    }

    return nil
}

func main() {
    // Create sample configuration
    config := &Config{
        AppName:     "MyApp",
        Version:     "1.0.0",
        Environment: "production",
        Server: ServerConfig{
            Host:    "0.0.0.0",
            Port:    443,
            Timeout: 30 * time.Second,
            TLS: TLSConfig{
                Enabled:  true,
                CertFile: "/etc/ssl/certs/server.crt",
                KeyFile:  "/etc/ssl/private/server.key",
            },
        },
        Database: DatabaseConfig{
            Driver:   "postgres",
            Host:     "db.example.com",
            Port:     5432,
            Name:     "production_db",
            User:     "app_user",
            Password: "secure_password",
            MaxConns: 100,
        },
        Redis: RedisConfig{
            Host:     "redis.example.com",
            Port:     6379,
            DB:       0,
            Password: "redis_password",
        },
    }

    // Save configuration to file
    if err := config.SaveConfig("/tmp/config.json"); err != nil {
        fmt.Printf("Error saving config: %v\n", err)
        return
    }

    // Load configuration from file
    loadedConfig, err := LoadConfig("/tmp/config.json")
    if err != nil {
        fmt.Printf("Error loading config: %v\n", err)
        return
    }

    fmt.Printf("Loaded config for %s v%s\n", 
        loadedConfig.AppName, loadedConfig.Version)
    fmt.Printf("Server: %s:%d (TLS: %v)\n", 
        loadedConfig.Server.Host, loadedConfig.Server.Port, 
        loadedConfig.Server.TLS.Enabled)
    fmt.Printf("Database: %s://%s@%s:%d/%s\n", 
        loadedConfig.Database.Driver, loadedConfig.Database.User,
        loadedConfig.Database.Host, loadedConfig.Database.Port,
        loadedConfig.Database.Name)
    fmt.Printf("Redis: %s:%d (DB: %d)\n", 
        loadedConfig.Redis.Host, loadedConfig.Redis.Port, 
        loadedConfig.Redis.DB)
}
```

This example demonstrates JSON configuration file handling with proper error  
management and type safety. The use of json.Decoder and json.Encoder provides  
streaming JSON processing, while the SaveConfig method enables configuration  
persistence. Struct tags define the JSON field mapping explicitly.  

## YAML configuration with third-party library

YAML provides a more human-readable configuration format with support for  
comments and multi-line strings, commonly used in DevOps and cloud  
applications.  

```go
package main

import (
    "fmt"
    "os"
    "time"
    
    "gopkg.in/yaml.v3"
)

type AppConfig struct {
    Metadata struct {
        Name        string    `yaml:"name"`
        Version     string    `yaml:"version"`
        Description string    `yaml:"description"`
        CreatedAt   time.Time `yaml:"created_at"`
    } `yaml:"metadata"`
    
    Environment string `yaml:"environment"`
    Debug       bool   `yaml:"debug"`
    
    Server struct {
        Host         string        `yaml:"host"`
        Port         int           `yaml:"port"`
        ReadTimeout  time.Duration `yaml:"read_timeout"`
        WriteTimeout time.Duration `yaml:"write_timeout"`
        MaxConns     int           `yaml:"max_connections"`
        TLS          struct {
            Enabled  bool   `yaml:"enabled"`
            CertFile string `yaml:"cert_file"`
            KeyFile  string `yaml:"key_file"`
        } `yaml:"tls"`
    } `yaml:"server"`
    
    Database struct {
        Primary struct {
            Host     string `yaml:"host"`
            Port     int    `yaml:"port"`
            Name     string `yaml:"name"`
            User     string `yaml:"user"`
            Password string `yaml:"password"`
        } `yaml:"primary"`
        Replica struct {
            Host string `yaml:"host"`
            Port int    `yaml:"port"`
        } `yaml:"replica"`
        Pool struct {
            MaxOpen     int           `yaml:"max_open"`
            MaxIdle     int           `yaml:"max_idle"`
            MaxLifetime time.Duration `yaml:"max_lifetime"`
        } `yaml:"pool"`
    } `yaml:"database"`
    
    Logging struct {
        Level   string   `yaml:"level"`
        Format  string   `yaml:"format"`
        Outputs []string `yaml:"outputs"`
        Files   struct {
            Path       string `yaml:"path"`
            MaxSize    string `yaml:"max_size"`
            MaxBackups int    `yaml:"max_backups"`
            MaxAge     int    `yaml:"max_age"`
            Compress   bool   `yaml:"compress"`
        } `yaml:"files"`
    } `yaml:"logging"`
    
    Features map[string]interface{} `yaml:"features"`
}

func LoadYAMLConfig(filename string) (*AppConfig, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read YAML file: %w", err)
    }

    var config AppConfig
    if err := yaml.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("failed to parse YAML: %w", err)
    }

    return &config, nil
}

func (c *AppConfig) SaveYAMLConfig(filename string) error {
    data, err := yaml.Marshal(c)
    if err != nil {
        return fmt.Errorf("failed to marshal YAML: %w", err)
    }

    if err := os.WriteFile(filename, data, 0644); err != nil {
        return fmt.Errorf("failed to write YAML file: %w", err)
    }

    return nil
}

func main() {
    // Create sample YAML configuration
    yamlContent := `# Application Configuration
metadata:
  name: "MyApp"
  version: "1.2.0"
  description: "A sample application with YAML configuration"
  created_at: 2024-01-15T10:30:00Z

environment: "production"
debug: false

server:
  host: "0.0.0.0"
  port: 8443
  read_timeout: "30s"
  write_timeout: "30s" 
  max_connections: 1000
  tls:
    enabled: true
    cert_file: "/etc/ssl/certs/app.crt"
    key_file: "/etc/ssl/private/app.key"

database:
  primary:
    host: "primary.db.example.com"
    port: 5432
    name: "app_production"
    user: "app_user"
    password: "secure_db_password"
  replica:
    host: "replica.db.example.com"
    port: 5432
  pool:
    max_open: 25
    max_idle: 10
    max_lifetime: "5m"

logging:
  level: "info"
  format: "json"
  outputs: ["stdout", "file"]
  files:
    path: "/var/log/app.log"
    max_size: "100MB"
    max_backups: 5
    max_age: 30
    compress: true

features:
  authentication: true
  rate_limiting: true
  metrics_collection: true
  health_checks: true
  graceful_shutdown: true
  cors_enabled: false
  request_id_header: "X-Request-ID"
  max_request_size: "10MB"
`

    // Write sample YAML to file
    if err := os.WriteFile("/tmp/app.yaml", []byte(yamlContent), 0644); err != nil {
        fmt.Printf("Error writing YAML file: %v\n", err)
        return
    }

    // Load and process YAML configuration
    config, err := LoadYAMLConfig("/tmp/app.yaml")
    if err != nil {
        fmt.Printf("Error loading YAML config: %v\n", err)
        return
    }

    fmt.Printf("Application: %s v%s\n", 
        config.Metadata.Name, config.Metadata.Version)
    fmt.Printf("Environment: %s (Debug: %v)\n", 
        config.Environment, config.Debug)
    fmt.Printf("Server: %s:%d (Max Conns: %d)\n", 
        config.Server.Host, config.Server.Port, config.Server.MaxConns)
    fmt.Printf("Database Primary: %s@%s:%d/%s\n", 
        config.Database.Primary.User, config.Database.Primary.Host,
        config.Database.Primary.Port, config.Database.Primary.Name)
    fmt.Printf("Logging: %s level, outputs: %v\n", 
        config.Logging.Level, config.Logging.Outputs)
    
    fmt.Println("Features:")
    for feature, value := range config.Features {
        fmt.Printf("  %s: %v\n", feature, value)
    }
}
```

This example showcases YAML configuration with nested structures, comments,  
and various data types. YAML's readable format makes it excellent for complex  
configurations while maintaining all the benefits of structured data. The  
example includes realistic production configuration patterns.  

## TOML configuration parsing

TOML (Tom's Obvious, Minimal Language) provides a configuration format that  
balances human readability with machine parsing efficiency.  

```go
package main

import (
    "fmt"
    "os"
    "time"
    
    "github.com/BurntSushi/toml"
)

type Config struct {
    Title   string `toml:"title"`
    Version string `toml:"version"`
    
    Owner struct {
        Name         string    `toml:"name"`
        Organization string    `toml:"organization"`
        Bio          string    `toml:"bio"`
        DOB          time.Time `toml:"dob"`
    } `toml:"owner"`
    
    Database DatabaseConfig `toml:"database"`
    Servers  ServersConfig  `toml:"servers"`
    Clients  ClientsConfig  `toml:"clients"`
}

type DatabaseConfig struct {
    Server       string        `toml:"server"`
    Ports        []int         `toml:"ports"`
    ConnectionMax int          `toml:"connection_max"`
    Enabled      bool          `toml:"enabled"`
    Timeout      time.Duration `toml:"timeout"`
}

type ServersConfig struct {
    Alpha struct {
        IP   string `toml:"ip"`
        DC   string `toml:"dc"`
        Role string `toml:"role"`
    } `toml:"alpha"`
    
    Beta struct {
        IP   string `toml:"ip"`
        DC   string `toml:"dc"`
        Role string `toml:"role"`
    } `toml:"beta"`
}

type ClientsConfig struct {
    Data  [][]interface{} `toml:"data"`
    Hosts []string        `toml:"hosts"`
}

func LoadTOMLConfig(filename string) (*Config, error) {
    var config Config
    if _, err := toml.DecodeFile(filename, &config); err != nil {
        return nil, fmt.Errorf("failed to decode TOML file: %w", err)
    }
    return &config, nil
}

func (c *Config) SaveTOMLConfig(filename string) error {
    file, err := os.Create(filename)
    if err != nil {
        return fmt.Errorf("failed to create TOML file: %w", err)
    }
    defer file.Close()

    encoder := toml.NewEncoder(file)
    if err := encoder.Encode(c); err != nil {
        return fmt.Errorf("failed to encode TOML: %w", err)
    }

    return nil
}

func main() {
    // Create sample TOML configuration
    tomlContent := `# Application Configuration in TOML format

title = "TOML Configuration Example"
version = "1.0.0"

[owner]
name = "Tom Preston-Werner"
organization = "GitHub"
bio = "GitHub Cofounder & CEO\nLikes programming languages."
dob = 1979-05-27T07:32:00-08:00

[database]
server = "192.168.1.1"
ports = [8001, 8001, 8002]
connection_max = 5000
enabled = true
timeout = "5s"

[servers]

# Alpha server configuration
[servers.alpha]
ip = "10.0.0.1"
dc = "eqdc10"
role = "frontend"

# Beta server configuration  
[servers.beta]
ip = "10.0.0.2"
dc = "eqdc10"
role = "backend"

[clients]
data = [
  ["gamma", "delta"],
  [1, 2],
  ["alpha", "omega"]
]
hosts = [
  "alpha",
  "omega"
]
`

    // Write sample TOML to file
    if err := os.WriteFile("/tmp/config.toml", []byte(tomlContent), 0644); err != nil {
        fmt.Printf("Error writing TOML file: %v\n", err)
        return
    }

    // Load and parse TOML configuration
    config, err := LoadTOMLConfig("/tmp/config.toml")
    if err != nil {
        fmt.Printf("Error loading TOML config: %v\n", err)
        return
    }

    fmt.Printf("Title: %s\n", config.Title)
    fmt.Printf("Version: %s\n", config.Version)
    fmt.Printf("Owner: %s (%s)\n", config.Owner.Name, config.Owner.Organization)
    fmt.Printf("DOB: %s\n", config.Owner.DOB.Format("2006-01-02"))
    
    fmt.Printf("Database: %s (enabled: %v)\n", config.Database.Server, config.Database.Enabled)
    fmt.Printf("Database ports: %v\n", config.Database.Ports)
    fmt.Printf("Connection max: %d, Timeout: %v\n", 
        config.Database.ConnectionMax, config.Database.Timeout)
    
    fmt.Printf("Alpha server: %s (%s) in %s\n", 
        config.Servers.Alpha.IP, config.Servers.Alpha.Role, config.Servers.Alpha.DC)
    fmt.Printf("Beta server: %s (%s) in %s\n", 
        config.Servers.Beta.IP, config.Servers.Beta.Role, config.Servers.Beta.DC)
    
    fmt.Printf("Client hosts: %v\n", config.Clients.Hosts)
    fmt.Printf("Client data: %v\n", config.Clients.Data)

    // Demonstrate saving modified configuration
    config.Version = "1.1.0"
    config.Database.ConnectionMax = 10000
    
    if err := config.SaveTOMLConfig("/tmp/config_modified.toml"); err != nil {
        fmt.Printf("Error saving TOML config: %v\n", err)
        return
    }
    
    fmt.Println("Configuration saved to /tmp/config_modified.toml")
}
```

This example demonstrates TOML configuration parsing with complex nested  
structures, arrays, and various data types. TOML's syntax strikes a balance  
between readability and structure, making it popular for application  
configuration where humans need to edit files frequently.  

## Environment variable configuration

Environment variables provide a flexible way to configure applications,  
especially in containerized and cloud environments where configuration  
injection is common.  

```go
package main

import (
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"
)

type EnvironmentConfig struct {
    AppName        string        `env:"APP_NAME" default:"MyApp"`
    Environment    string        `env:"ENVIRONMENT" default:"development"`
    Debug          bool          `env:"DEBUG" default:"false"`
    Port           int           `env:"PORT" default:"8080"`
    Host           string        `env:"HOST" default:"localhost"`
    DatabaseURL    string        `env:"DATABASE_URL" required:"true"`
    RedisURL       string        `env:"REDIS_URL"`
    JWTSecret      string        `env:"JWT_SECRET" required:"true"`
    LogLevel       string        `env:"LOG_LEVEL" default:"info"`
    MaxWorkers     int           `env:"MAX_WORKERS" default:"10"`
    RequestTimeout time.Duration `env:"REQUEST_TIMEOUT" default:"30s"`
    AllowedOrigins []string      `env:"ALLOWED_ORIGINS" default:"*"`
    FeatureFlags   map[string]bool
}

func LoadFromEnvironment() (*EnvironmentConfig, error) {
    config := &EnvironmentConfig{}
    
    // Load basic string values
    config.AppName = getEnvWithDefault("APP_NAME", "MyApp")
    config.Environment = getEnvWithDefault("ENVIRONMENT", "development")
    config.Host = getEnvWithDefault("HOST", "localhost")
    config.LogLevel = getEnvWithDefault("LOG_LEVEL", "info")
    
    // Load and validate required values
    config.DatabaseURL = os.Getenv("DATABASE_URL")
    if config.DatabaseURL == "" {
        return nil, fmt.Errorf("DATABASE_URL environment variable is required")
    }
    
    config.JWTSecret = os.Getenv("JWT_SECRET")
    if config.JWTSecret == "" {
        return nil, fmt.Errorf("JWT_SECRET environment variable is required")
    }
    
    // Load optional values
    config.RedisURL = os.Getenv("REDIS_URL")
    
    // Load boolean values
    var err error
    config.Debug, err = parseBoolWithDefault("DEBUG", false)
    if err != nil {
        return nil, fmt.Errorf("invalid DEBUG value: %w", err)
    }
    
    // Load integer values
    config.Port, err = parseIntWithDefault("PORT", 8080)
    if err != nil {
        return nil, fmt.Errorf("invalid PORT value: %w", err)
    }
    
    config.MaxWorkers, err = parseIntWithDefault("MAX_WORKERS", 10)
    if err != nil {
        return nil, fmt.Errorf("invalid MAX_WORKERS value: %w", err)
    }
    
    // Load duration values
    config.RequestTimeout, err = parseDurationWithDefault("REQUEST_TIMEOUT", 30*time.Second)
    if err != nil {
        return nil, fmt.Errorf("invalid REQUEST_TIMEOUT value: %w", err)
    }
    
    // Load array values
    config.AllowedOrigins = parseStringSlice("ALLOWED_ORIGINS", []string{"*"})
    
    // Load feature flags from environment variables
    config.FeatureFlags = loadFeatureFlags()
    
    return config, nil
}

func getEnvWithDefault(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func parseBoolWithDefault(key string, defaultValue bool) (bool, error) {
    value := os.Getenv(key)
    if value == "" {
        return defaultValue, nil
    }
    return strconv.ParseBool(value)
}

func parseIntWithDefault(key string, defaultValue int) (int, error) {
    value := os.Getenv(key)
    if value == "" {
        return defaultValue, nil
    }
    return strconv.Atoi(value)
}

func parseDurationWithDefault(key string, defaultValue time.Duration) (time.Duration, error) {
    value := os.Getenv(key)
    if value == "" {
        return defaultValue, nil
    }
    return time.ParseDuration(value)
}

func parseStringSlice(key string, defaultValue []string) []string {
    value := os.Getenv(key)
    if value == "" {
        return defaultValue
    }
    return strings.Split(value, ",")
}

func loadFeatureFlags() map[string]bool {
    features := make(map[string]bool)
    
    // Look for environment variables starting with FEATURE_
    for _, env := range os.Environ() {
        if strings.HasPrefix(env, "FEATURE_") {
            parts := strings.SplitN(env, "=", 2)
            if len(parts) == 2 {
                featureName := strings.ToLower(strings.TrimPrefix(parts[0], "FEATURE_"))
                if value, err := strconv.ParseBool(parts[1]); err == nil {
                    features[featureName] = value
                }
            }
        }
    }
    
    return features
}

func (c *EnvironmentConfig) Validate() error {
    if c.Port < 1 || c.Port > 65535 {
        return fmt.Errorf("port must be between 1 and 65535, got %d", c.Port)
    }
    
    if c.MaxWorkers < 1 {
        return fmt.Errorf("max_workers must be positive, got %d", c.MaxWorkers)
    }
    
    validLogLevels := map[string]bool{
        "debug": true, "info": true, "warn": true, "error": true,
    }
    if !validLogLevels[c.LogLevel] {
        return fmt.Errorf("invalid log level: %s", c.LogLevel)
    }
    
    return nil
}

func (c *EnvironmentConfig) Print() {
    fmt.Printf("Application: %s\n", c.AppName)
    fmt.Printf("Environment: %s\n", c.Environment)
    fmt.Printf("Debug: %v\n", c.Debug)
    fmt.Printf("Server: %s:%d\n", c.Host, c.Port)
    fmt.Printf("Database URL: %s\n", maskSensitive(c.DatabaseURL))
    fmt.Printf("Redis URL: %s\n", maskSensitive(c.RedisURL))
    fmt.Printf("JWT Secret: %s\n", maskSensitive(c.JWTSecret))
    fmt.Printf("Log Level: %s\n", c.LogLevel)
    fmt.Printf("Max Workers: %d\n", c.MaxWorkers)
    fmt.Printf("Request Timeout: %v\n", c.RequestTimeout)
    fmt.Printf("Allowed Origins: %v\n", c.AllowedOrigins)
    
    if len(c.FeatureFlags) > 0 {
        fmt.Println("Feature Flags:")
        for feature, enabled := range c.FeatureFlags {
            fmt.Printf("  %s: %v\n", feature, enabled)
        }
    }
}

func maskSensitive(value string) string {
    if value == "" {
        return "(not set)"
    }
    if len(value) <= 8 {
        return "****"
    }
    return value[:4] + "****" + value[len(value)-4:]
}

func main() {
    // Set some example environment variables
    os.Setenv("APP_NAME", "ConfigDemo")
    os.Setenv("ENVIRONMENT", "production")
    os.Setenv("DEBUG", "false")
    os.Setenv("PORT", "9090")
    os.Setenv("DATABASE_URL", "postgres://user:pass@localhost:5432/mydb")
    os.Setenv("JWT_SECRET", "very-secret-jwt-key")
    os.Setenv("LOG_LEVEL", "info")
    os.Setenv("MAX_WORKERS", "20")
    os.Setenv("REQUEST_TIMEOUT", "45s")
    os.Setenv("ALLOWED_ORIGINS", "https://example.com,https://app.example.com")
    os.Setenv("FEATURE_METRICS", "true")
    os.Setenv("FEATURE_TRACING", "false")
    os.Setenv("FEATURE_CACHING", "true")

    config, err := LoadFromEnvironment()
    if err != nil {
        fmt.Printf("Error loading configuration: %v\n", err)
        return
    }

    if err := config.Validate(); err != nil {
        fmt.Printf("Configuration validation error: %v\n", err)
        return
    }

    fmt.Println("=== Environment Configuration ===")
    config.Print()
}
```

This example demonstrates comprehensive environment variable handling with  
type conversion, validation, and error handling. The pattern includes support  
for required and optional variables, default values, and dynamic feature flag  
loading, making it suitable for cloud-native applications.  

## Configuration with defaults and validation

Robust configuration management requires proper default values and validation  
to ensure application stability and security.  

```go
package main

import (
    "fmt"
    "net/url"
    "regexp"
    "strings"
    "time"
)

type AppConfig struct {
    Service  ServiceConfig  `json:"service"`
    Database DatabaseConfig `json:"database"`
    Cache    CacheConfig    `json:"cache"`
    Security SecurityConfig `json:"security"`
    Logging  LoggingConfig  `json:"logging"`
}

type ServiceConfig struct {
    Name            string        `json:"name" validate:"required"`
    Version         string        `json:"version" validate:"required,version"`
    Host            string        `json:"host" validate:"required,hostname"`
    Port            int           `json:"port" validate:"required,min=1,max=65535"`
    ReadTimeout     time.Duration `json:"read_timeout" validate:"required,min=1s"`
    WriteTimeout    time.Duration `json:"write_timeout" validate:"required,min=1s"`
    ShutdownTimeout time.Duration `json:"shutdown_timeout" validate:"required,min=1s"`
    MaxConnections  int           `json:"max_connections" validate:"min=1"`
}

type DatabaseConfig struct {
    Driver       string        `json:"driver" validate:"required,oneof=postgres mysql sqlite"`
    Host         string        `json:"host" validate:"required,hostname"`
    Port         int           `json:"port" validate:"required,min=1,max=65535"`
    Database     string        `json:"database" validate:"required"`
    Username     string        `json:"username" validate:"required"`
    Password     string        `json:"password" validate:"required,min=8"`
    MaxOpenConns int           `json:"max_open_conns" validate:"min=1"`
    MaxIdleConns int           `json:"max_idle_conns" validate:"min=1"`
    MaxLifetime  time.Duration `json:"max_lifetime" validate:"min=1m"`
    SSLMode      string        `json:"ssl_mode" validate:"oneof=disable require verify-ca verify-full"`
}

type CacheConfig struct {
    Provider string        `json:"provider" validate:"required,oneof=redis memcached memory"`
    Host     string        `json:"host" validate:"required_if=Provider redis,required_if=Provider memcached"`
    Port     int           `json:"port" validate:"required_if=Provider redis,required_if=Provider memcached"`
    Password string        `json:"password"`
    DB       int           `json:"db" validate:"min=0,max=15"`
    TTL      time.Duration `json:"ttl" validate:"min=1s"`
}

type SecurityConfig struct {
    JWTSecret      string        `json:"jwt_secret" validate:"required,min=32"`
    TokenLifetime  time.Duration `json:"token_lifetime" validate:"required,min=5m"`
    AllowedOrigins []string      `json:"allowed_origins" validate:"dive,uri"`
    RateLimit      struct {
        Requests int           `json:"requests" validate:"min=1"`
        Window   time.Duration `json:"window" validate:"min=1s"`
    } `json:"rate_limit"`
    PasswordMinLength int `json:"password_min_length" validate:"min=8,max=128"`
}

type LoggingConfig struct {
    Level  string `json:"level" validate:"required,oneof=debug info warn error"`
    Format string `json:"format" validate:"required,oneof=json text"`
    Output string `json:"output" validate:"required,oneof=stdout stderr file"`
    File   struct {
        Path     string `json:"path" validate:"required_if=Output file"`
        MaxSize  int    `json:"max_size" validate:"min=1"`  // MB
        MaxFiles int    `json:"max_files" validate:"min=1"`
        MaxAge   int    `json:"max_age" validate:"min=1"`   // days
    } `json:"file"`
}

func DefaultConfig() *AppConfig {
    return &AppConfig{
        Service: ServiceConfig{
            Name:            "myapp",
            Version:         "1.0.0",
            Host:            "localhost",
            Port:            8080,
            ReadTimeout:     30 * time.Second,
            WriteTimeout:    30 * time.Second,
            ShutdownTimeout: 10 * time.Second,
            MaxConnections:  1000,
        },
        Database: DatabaseConfig{
            Driver:       "postgres",
            Host:         "localhost",
            Port:         5432,
            Database:     "myapp",
            Username:     "postgres",
            Password:     "defaultpassword123",
            MaxOpenConns: 25,
            MaxIdleConns: 10,
            MaxLifetime:  5 * time.Minute,
            SSLMode:      "disable",
        },
        Cache: CacheConfig{
            Provider: "memory",
            Host:     "",
            Port:     0,
            Password: "",
            DB:       0,
            TTL:      1 * time.Hour,
        },
        Security: SecurityConfig{
            JWTSecret:     "this-is-a-very-long-jwt-secret-key-for-production-use-only",
            TokenLifetime: 24 * time.Hour,
            AllowedOrigins: []string{"http://localhost:3000"},
            RateLimit: struct {
                Requests int           `json:"requests" validate:"min=1"`
                Window   time.Duration `json:"window" validate:"min=1s"`
            }{
                Requests: 100,
                Window:   time.Minute,
            },
            PasswordMinLength: 8,
        },
        Logging: LoggingConfig{
            Level:  "info",
            Format: "json",
            Output: "stdout",
            File: struct {
                Path     string `json:"path" validate:"required_if=Output file"`
                MaxSize  int    `json:"max_size" validate:"min=1"`
                MaxFiles int    `json:"max_files" validate:"min=1"`
                MaxAge   int    `json:"max_age" validate:"min=1"`
            }{
                Path:     "",
                MaxSize:  100,
                MaxFiles: 5,
                MaxAge:   7,
            },
        },
    }
}

func (c *AppConfig) Validate() error {
    // Validate service configuration
    if err := c.validateService(); err != nil {
        return fmt.Errorf("service config: %w", err)
    }
    
    // Validate database configuration
    if err := c.validateDatabase(); err != nil {
        return fmt.Errorf("database config: %w", err)
    }
    
    // Validate cache configuration
    if err := c.validateCache(); err != nil {
        return fmt.Errorf("cache config: %w", err)
    }
    
    // Validate security configuration
    if err := c.validateSecurity(); err != nil {
        return fmt.Errorf("security config: %w", err)
    }
    
    // Validate logging configuration
    if err := c.validateLogging(); err != nil {
        return fmt.Errorf("logging config: %w", err)
    }
    
    return nil
}

func (c *AppConfig) validateService() error {
    if c.Service.Name == "" {
        return fmt.Errorf("service name is required")
    }
    
    if !isValidVersion(c.Service.Version) {
        return fmt.Errorf("invalid version format: %s", c.Service.Version)
    }
    
    if c.Service.Port < 1 || c.Service.Port > 65535 {
        return fmt.Errorf("port must be between 1 and 65535")
    }
    
    if c.Service.ReadTimeout < time.Second {
        return fmt.Errorf("read timeout must be at least 1 second")
    }
    
    return nil
}

func (c *AppConfig) validateDatabase() error {
    validDrivers := map[string]bool{
        "postgres": true, "mysql": true, "sqlite": true,
    }
    if !validDrivers[c.Database.Driver] {
        return fmt.Errorf("invalid database driver: %s", c.Database.Driver)
    }
    
    if len(c.Database.Password) < 8 {
        return fmt.Errorf("database password must be at least 8 characters")
    }
    
    if c.Database.MaxIdleConns > c.Database.MaxOpenConns {
        return fmt.Errorf("max idle connections cannot exceed max open connections")
    }
    
    return nil
}

func (c *AppConfig) validateCache() error {
    validProviders := map[string]bool{
        "redis": true, "memcached": true, "memory": true,
    }
    if !validProviders[c.Cache.Provider] {
        return fmt.Errorf("invalid cache provider: %s", c.Cache.Provider)
    }
    
    if c.Cache.Provider != "memory" && c.Cache.Host == "" {
        return fmt.Errorf("cache host is required for %s provider", c.Cache.Provider)
    }
    
    return nil
}

func (c *AppConfig) validateSecurity() error {
    if len(c.Security.JWTSecret) < 32 {
        return fmt.Errorf("JWT secret must be at least 32 characters")
    }
    
    for _, origin := range c.Security.AllowedOrigins {
        if origin != "*" {
            if _, err := url.Parse(origin); err != nil {
                return fmt.Errorf("invalid allowed origin: %s", origin)
            }
        }
    }
    
    if c.Security.RateLimit.Requests < 1 {
        return fmt.Errorf("rate limit requests must be positive")
    }
    
    return nil
}

func (c *AppConfig) validateLogging() error {
    validLevels := map[string]bool{
        "debug": true, "info": true, "warn": true, "error": true,
    }
    if !validLevels[c.Logging.Level] {
        return fmt.Errorf("invalid log level: %s", c.Logging.Level)
    }
    
    validFormats := map[string]bool{
        "json": true, "text": true,
    }
    if !validFormats[c.Logging.Format] {
        return fmt.Errorf("invalid log format: %s", c.Logging.Format)
    }
    
    if c.Logging.Output == "file" && c.Logging.File.Path == "" {
        return fmt.Errorf("log file path is required when output is file")
    }
    
    return nil
}

func isValidVersion(version string) bool {
    // Simple semantic version validation
    versionRegex := regexp.MustCompile(`^v?\d+\.\d+\.\d+(-[\w\.\-]+)?(\+[\w\.\-]+)?$`)
    return versionRegex.MatchString(version)
}

func (c *AppConfig) Print() {
    fmt.Printf("=== Application Configuration ===\n")
    fmt.Printf("Service: %s v%s on %s:%d\n", 
        c.Service.Name, c.Service.Version, c.Service.Host, c.Service.Port)
    fmt.Printf("Database: %s://%s@%s:%d/%s\n", 
        c.Database.Driver, c.Database.Username, c.Database.Host, 
        c.Database.Port, c.Database.Database)
    fmt.Printf("Cache: %s", c.Cache.Provider)
    if c.Cache.Provider != "memory" {
        fmt.Printf(" (%s:%d)", c.Cache.Host, c.Cache.Port)
    }
    fmt.Printf("\n")
    fmt.Printf("Security: JWT lifetime %v, %d allowed origins\n", 
        c.Security.TokenLifetime, len(c.Security.AllowedOrigins))
    fmt.Printf("Logging: %s level, %s format to %s\n", 
        c.Logging.Level, c.Logging.Format, c.Logging.Output)
}

func main() {
    // Start with default configuration
    config := DefaultConfig()
    
    // Modify some values to demonstrate validation
    config.Service.Name = "ProductionApp"
    config.Service.Version = "2.1.0"
    config.Database.Password = "super-secure-password-123"
    config.Cache.Provider = "redis"
    config.Cache.Host = "redis.example.com"
    config.Cache.Port = 6379
    config.Security.AllowedOrigins = []string{
        "https://app.example.com",
        "https://api.example.com",
    }
    
    // Validate configuration
    if err := config.Validate(); err != nil {
        fmt.Printf("Configuration validation failed: %v\n", err)
        return
    }
    
    fmt.Println("Configuration validation passed!")
    config.Print()
    
    // Test with invalid configuration
    fmt.Println("\n=== Testing Invalid Configuration ===")
    invalidConfig := DefaultConfig()
    invalidConfig.Service.Port = 70000  // Invalid port
    invalidConfig.Database.Password = "weak"  // Too short
    
    if err := invalidConfig.Validate(); err != nil {
        fmt.Printf("Expected validation error: %v\n", err)
    }
}
```

This example demonstrates comprehensive configuration validation with defaults,  
type checking, cross-field validation, and detailed error reporting. The  
validation ensures configuration consistency and prevents common deployment  
issues through early detection of configuration problems.  

## Configuration merging with precedence

Modern applications often need to merge configuration from multiple sources  
with a clear precedence order: defaults < file < environment < command-line.  

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "os"
    "strconv"
    "time"
)

type Config struct {
    Server struct {
        Host    string        `json:"host"`
        Port    int           `json:"port"`
        Timeout time.Duration `json:"timeout"`
    } `json:"server"`
    
    Database struct {
        Host     string `json:"host"`
        Port     int    `json:"port"`
        Name     string `json:"name"`
        User     string `json:"user"`
        Password string `json:"password"`
    } `json:"database"`
    
    Features struct {
        EnableMetrics bool `json:"enable_metrics"`
        EnableTracing bool `json:"enable_tracing"`
        EnableCaching bool `json:"enable_caching"`
    } `json:"features"`
    
    LogLevel string `json:"log_level"`
}

func NewConfig() *Config {
    return &Config{
        Server: struct {
            Host    string        `json:"host"`
            Port    int           `json:"port"`
            Timeout time.Duration `json:"timeout"`
        }{
            Host:    "localhost",
            Port:    8080,
            Timeout: 30 * time.Second,
        },
        Database: struct {
            Host     string `json:"host"`
            Port     int    `json:"port"`
            Name     string `json:"name"`
            User     string `json:"user"`
            Password string `json:"password"`
        }{
            Host:     "localhost",
            Port:     5432,
            Name:     "myapp",
            User:     "postgres",
            Password: "postgres",
        },
        Features: struct {
            EnableMetrics bool `json:"enable_metrics"`
            EnableTracing bool `json:"enable_tracing"`
            EnableCaching bool `json:"enable_caching"`
        }{
            EnableMetrics: false,
            EnableTracing: false,
            EnableCaching: true,
        },
        LogLevel: "info",
    }
}

func (c *Config) LoadFromFile(filename string) error {
    if filename == "" {
        return nil  // No file specified, skip
    }
    
    file, err := os.Open(filename)
    if err != nil {
        if os.IsNotExist(err) {
            fmt.Printf("Config file %s not found, skipping\n", filename)
            return nil
        }
        return fmt.Errorf("failed to open config file: %w", err)
    }
    defer file.Close()
    
    decoder := json.NewDecoder(file)
    fileConfig := &Config{}
    if err := decoder.Decode(fileConfig); err != nil {
        return fmt.Errorf("failed to decode config file: %w", err)
    }
    
    // Merge file config into current config
    c.mergeFrom(fileConfig)
    fmt.Printf("Loaded configuration from file: %s\n", filename)
    return nil
}

func (c *Config) LoadFromEnvironment() {
    fmt.Println("Loading configuration from environment variables...")
    
    // Server configuration
    if host := os.Getenv("SERVER_HOST"); host != "" {
        c.Server.Host = host
        fmt.Printf("  SERVER_HOST: %s\n", host)
    }
    if port := os.Getenv("SERVER_PORT"); port != "" {
        if p, err := strconv.Atoi(port); err == nil {
            c.Server.Port = p
            fmt.Printf("  SERVER_PORT: %d\n", p)
        }
    }
    if timeout := os.Getenv("SERVER_TIMEOUT"); timeout != "" {
        if t, err := time.ParseDuration(timeout); err == nil {
            c.Server.Timeout = t
            fmt.Printf("  SERVER_TIMEOUT: %v\n", t)
        }
    }
    
    // Database configuration
    if host := os.Getenv("DB_HOST"); host != "" {
        c.Database.Host = host
        fmt.Printf("  DB_HOST: %s\n", host)
    }
    if port := os.Getenv("DB_PORT"); port != "" {
        if p, err := strconv.Atoi(port); err == nil {
            c.Database.Port = p
            fmt.Printf("  DB_PORT: %d\n", p)
        }
    }
    if name := os.Getenv("DB_NAME"); name != "" {
        c.Database.Name = name
        fmt.Printf("  DB_NAME: %s\n", name)
    }
    if user := os.Getenv("DB_USER"); user != "" {
        c.Database.User = user
        fmt.Printf("  DB_USER: %s\n", user)
    }
    if password := os.Getenv("DB_PASSWORD"); password != "" {
        c.Database.Password = password
        fmt.Printf("  DB_PASSWORD: [REDACTED]\n")
    }
    
    // Features
    if metrics := os.Getenv("ENABLE_METRICS"); metrics != "" {
        if m, err := strconv.ParseBool(metrics); err == nil {
            c.Features.EnableMetrics = m
            fmt.Printf("  ENABLE_METRICS: %v\n", m)
        }
    }
    if tracing := os.Getenv("ENABLE_TRACING"); tracing != "" {
        if t, err := strconv.ParseBool(tracing); err == nil {
            c.Features.EnableTracing = t
            fmt.Printf("  ENABLE_TRACING: %v\n", t)
        }
    }
    if caching := os.Getenv("ENABLE_CACHING"); caching != "" {
        if c_val, err := strconv.ParseBool(caching); err == nil {
            c.Features.EnableCaching = c_val
            fmt.Printf("  ENABLE_CACHING: %v\n", c_val)
        }
    }
    
    // Log level
    if logLevel := os.Getenv("LOG_LEVEL"); logLevel != "" {
        c.LogLevel = logLevel
        fmt.Printf("  LOG_LEVEL: %s\n", logLevel)
    }
}

func (c *Config) LoadFromFlags() {
    fmt.Println("Loading configuration from command-line flags...")
    
    // Define command-line flags
    serverHost := flag.String("server-host", "", "Server host address")
    serverPort := flag.Int("server-port", 0, "Server port number")
    serverTimeout := flag.Duration("server-timeout", 0, "Server timeout duration")
    
    dbHost := flag.String("db-host", "", "Database host")
    dbPort := flag.Int("db-port", 0, "Database port")
    dbName := flag.String("db-name", "", "Database name")
    dbUser := flag.String("db-user", "", "Database user")
    dbPassword := flag.String("db-password", "", "Database password")
    
    enableMetrics := flag.Bool("enable-metrics", false, "Enable metrics collection")
    enableTracing := flag.Bool("enable-tracing", false, "Enable distributed tracing")
    enableCaching := flag.Bool("enable-caching", false, "Enable caching")
    
    logLevel := flag.String("log-level", "", "Log level (debug, info, warn, error)")
    
    // Parse command-line flags
    flag.Parse()
    
    // Apply flags if they were set
    if *serverHost != "" {
        c.Server.Host = *serverHost
        fmt.Printf("  --server-host: %s\n", *serverHost)
    }
    if *serverPort != 0 {
        c.Server.Port = *serverPort
        fmt.Printf("  --server-port: %d\n", *serverPort)
    }
    if *serverTimeout != 0 {
        c.Server.Timeout = *serverTimeout
        fmt.Printf("  --server-timeout: %v\n", *serverTimeout)
    }
    
    if *dbHost != "" {
        c.Database.Host = *dbHost
        fmt.Printf("  --db-host: %s\n", *dbHost)
    }
    if *dbPort != 0 {
        c.Database.Port = *dbPort
        fmt.Printf("  --db-port: %d\n", *dbPort)
    }
    if *dbName != "" {
        c.Database.Name = *dbName
        fmt.Printf("  --db-name: %s\n", *dbName)
    }
    if *dbUser != "" {
        c.Database.User = *dbUser
        fmt.Printf("  --db-user: %s\n", *dbUser)
    }
    if *dbPassword != "" {
        c.Database.Password = *dbPassword
        fmt.Printf("  --db-password: [REDACTED]\n")
    }
    
    // For boolean flags, we need to check if they were explicitly set
    flag.Visit(func(f *flag.Flag) {
        switch f.Name {
        case "enable-metrics":
            c.Features.EnableMetrics = *enableMetrics
            fmt.Printf("  --enable-metrics: %v\n", *enableMetrics)
        case "enable-tracing":
            c.Features.EnableTracing = *enableTracing
            fmt.Printf("  --enable-tracing: %v\n", *enableTracing)
        case "enable-caching":
            c.Features.EnableCaching = *enableCaching
            fmt.Printf("  --enable-caching: %v\n", *enableCaching)
        }
    })
    
    if *logLevel != "" {
        c.LogLevel = *logLevel
        fmt.Printf("  --log-level: %s\n", *logLevel)
    }
}

func (c *Config) mergeFrom(other *Config) {
    // Only merge non-zero values
    if other.Server.Host != "" {
        c.Server.Host = other.Server.Host
    }
    if other.Server.Port != 0 {
        c.Server.Port = other.Server.Port
    }
    if other.Server.Timeout != 0 {
        c.Server.Timeout = other.Server.Timeout
    }
    
    if other.Database.Host != "" {
        c.Database.Host = other.Database.Host
    }
    if other.Database.Port != 0 {
        c.Database.Port = other.Database.Port
    }
    if other.Database.Name != "" {
        c.Database.Name = other.Database.Name
    }
    if other.Database.User != "" {
        c.Database.User = other.Database.User
    }
    if other.Database.Password != "" {
        c.Database.Password = other.Database.Password
    }
    
    if other.LogLevel != "" {
        c.LogLevel = other.LogLevel
    }
}

func (c *Config) Print() {
    fmt.Println("\n=== Final Configuration ===")
    fmt.Printf("Server: %s:%d (timeout: %v)\n", 
        c.Server.Host, c.Server.Port, c.Server.Timeout)
    fmt.Printf("Database: %s@%s:%d/%s\n", 
        c.Database.User, c.Database.Host, c.Database.Port, c.Database.Name)
    fmt.Printf("Features: metrics=%v, tracing=%v, caching=%v\n", 
        c.Features.EnableMetrics, c.Features.EnableTracing, c.Features.EnableCaching)
    fmt.Printf("Log Level: %s\n", c.LogLevel)
}

func main() {
    // Create sample configuration file
    sampleConfig := `{
  "server": {
    "host": "0.0.0.0",
    "port": 9000,
    "timeout": "45s"
  },
  "database": {
    "host": "db.example.com",
    "name": "production"
  },
  "features": {
    "enable_metrics": true
  },
  "log_level": "warn"
}`
    
    if err := os.WriteFile("/tmp/app.json", []byte(sampleConfig), 0644); err != nil {
        fmt.Printf("Error creating sample config: %v\n", err)
        return
    }
    
    // Set some environment variables
    os.Setenv("SERVER_HOST", "api.example.com")
    os.Setenv("DB_PASSWORD", "super-secret-password")
    os.Setenv("ENABLE_TRACING", "true")
    os.Setenv("LOG_LEVEL", "debug")
    
    fmt.Println("=== Configuration Loading with Precedence ===")
    fmt.Println("Order: defaults < file < environment < command-line\n")
    
    // Start with defaults
    config := NewConfig()
    fmt.Println("1. Started with default configuration")
    
    // Load from file (overrides defaults)
    if err := config.LoadFromFile("/tmp/app.json"); err != nil {
        fmt.Printf("Error loading config file: %v\n", err)
    }
    
    // Load from environment (overrides file)
    config.LoadFromEnvironment()
    
    // Load from command-line flags (overrides everything)
    config.LoadFromFlags()
    
    // Print final configuration
    config.Print()
    
    fmt.Println("\nTry running with flags like:")
    fmt.Println("  go run main.go --server-port 3000 --enable-metrics")
}
```

This example demonstrates configuration merging with clear precedence rules.  
The pattern allows flexible configuration while maintaining predictable  
behavior. Each source only overrides explicitly set values, preserving  
defaults where no override is provided.  

## Configuration hot-reloading with file watching

Dynamic configuration updates allow applications to adapt to changing  
requirements without restarts, crucial for high-availability systems.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
    
    "github.com/fsnotify/fsnotify"
)

type AppConfig struct {
    Server struct {
        Host    string `json:"host"`
        Port    int    `json:"port"`
        Timeout int    `json:"timeout_seconds"`
    } `json:"server"`
    
    Database struct {
        MaxConnections int `json:"max_connections"`
        TimeoutSeconds int `json:"timeout_seconds"`
    } `json:"database"`
    
    Features struct {
        EnableMetrics bool `json:"enable_metrics"`
        EnableTracing bool `json:"enable_tracing"`
        RateLimit     int  `json:"rate_limit"`
    } `json:"features"`
    
    LogLevel string `json:"log_level"`
    Version  int    `json:"version"`
}

type ConfigManager struct {
    config     *AppConfig
    configPath string
    mutex      sync.RWMutex
    watcher    *fsnotify.Watcher
    callbacks  []func(*AppConfig)
}

func NewConfigManager(configPath string) (*ConfigManager, error) {
    watcher, err := fsnotify.NewWatcher()
    if err != nil {
        return nil, fmt.Errorf("failed to create file watcher: %w", err)
    }
    
    cm := &ConfigManager{
        configPath: configPath,
        watcher:    watcher,
        callbacks:  make([]func(*AppConfig), 0),
    }
    
    // Load initial configuration
    if err := cm.loadConfig(); err != nil {
        return nil, fmt.Errorf("failed to load initial config: %w", err)
    }
    
    // Start watching the config file
    if err := cm.startWatching(); err != nil {
        return nil, fmt.Errorf("failed to start watching config file: %w", err)
    }
    
    return cm, nil
}

func (cm *ConfigManager) loadConfig() error {
    data, err := os.ReadFile(cm.configPath)
    if err != nil {
        return fmt.Errorf("failed to read config file: %w", err)
    }
    
    var newConfig AppConfig
    if err := json.Unmarshal(data, &newConfig); err != nil {
        return fmt.Errorf("failed to parse config JSON: %w", err)
    }
    
    // Validate the new configuration
    if err := cm.validateConfig(&newConfig); err != nil {
        return fmt.Errorf("config validation failed: %w", err)
    }
    
    cm.mutex.Lock()
    oldConfig := cm.config
    cm.config = &newConfig
    cm.mutex.Unlock()
    
    fmt.Printf("Configuration loaded (version %d)\n", newConfig.Version)
    
    // Notify callbacks of config change
    if oldConfig != nil {
        cm.notifyCallbacks(&newConfig)
    }
    
    return nil
}

func (cm *ConfigManager) validateConfig(config *AppConfig) error {
    if config.Server.Port < 1 || config.Server.Port > 65535 {
        return fmt.Errorf("invalid server port: %d", config.Server.Port)
    }
    
    if config.Database.MaxConnections < 1 {
        return fmt.Errorf("invalid max connections: %d", config.Database.MaxConnections)
    }
    
    validLogLevels := map[string]bool{
        "debug": true, "info": true, "warn": true, "error": true,
    }
    if !validLogLevels[config.LogLevel] {
        return fmt.Errorf("invalid log level: %s", config.LogLevel)
    }
    
    if config.Features.RateLimit < 0 {
        return fmt.Errorf("invalid rate limit: %d", config.Features.RateLimit)
    }
    
    return nil
}

func (cm *ConfigManager) startWatching() error {
    if err := cm.watcher.Add(cm.configPath); err != nil {
        return fmt.Errorf("failed to watch config file: %w", err)
    }
    
    go cm.watchLoop()
    return nil
}

func (cm *ConfigManager) watchLoop() {
    debouncer := make(chan bool, 1)
    
    go func() {
        for {
            select {
            case <-debouncer:
                // Debounce rapid file changes
                time.Sleep(100 * time.Millisecond)
                fmt.Println("Configuration file changed, reloading...")
                
                if err := cm.loadConfig(); err != nil {
                    log.Printf("Failed to reload config: %v", err)
                } else {
                    fmt.Println("Configuration reloaded successfully")
                }
            }
        }
    }()
    
    for {
        select {
        case event, ok := <-cm.watcher.Events:
            if !ok {
                return
            }
            
            if event.Op&fsnotify.Write == fsnotify.Write {
                select {
                case debouncer <- true:
                default:
                    // Already pending reload
                }
            }
            
        case err, ok := <-cm.watcher.Errors:
            if !ok {
                return
            }
            log.Printf("Config watcher error: %v", err)
        }
    }
}

func (cm *ConfigManager) GetConfig() *AppConfig {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    
    // Return a copy to prevent race conditions
    configCopy := *cm.config
    return &configCopy
}

func (cm *ConfigManager) OnConfigChange(callback func(*AppConfig)) {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    cm.callbacks = append(cm.callbacks, callback)
}

func (cm *ConfigManager) notifyCallbacks(config *AppConfig) {
    for _, callback := range cm.callbacks {
        go callback(config)
    }
}

func (cm *ConfigManager) Close() error {
    return cm.watcher.Close()
}

// Example application using hot-reloadable configuration
type Application struct {
    configManager *ConfigManager
    currentConfig *AppConfig
}

func NewApplication(configPath string) (*Application, error) {
    cm, err := NewConfigManager(configPath)
    if err != nil {
        return nil, err
    }
    
    app := &Application{
        configManager: cm,
        currentConfig: cm.GetConfig(),
    }
    
    // Register for configuration changes
    cm.OnConfigChange(app.onConfigChange)
    
    return app, nil
}

func (app *Application) onConfigChange(newConfig *AppConfig) {
    oldConfig := app.currentConfig
    app.currentConfig = newConfig
    
    fmt.Printf("Configuration changed from version %d to %d\n", 
        oldConfig.Version, newConfig.Version)
    
    // React to specific configuration changes
    if oldConfig.LogLevel != newConfig.LogLevel {
        fmt.Printf("Log level changed: %s -> %s\n", 
            oldConfig.LogLevel, newConfig.LogLevel)
    }
    
    if oldConfig.Features.RateLimit != newConfig.Features.RateLimit {
        fmt.Printf("Rate limit changed: %d -> %d\n", 
            oldConfig.Features.RateLimit, newConfig.Features.RateLimit)
    }
    
    if oldConfig.Server.Port != newConfig.Server.Port {
        fmt.Printf("Server port changed: %d -> %d (restart required)\n", 
            oldConfig.Server.Port, newConfig.Server.Port)
    }
}

func (app *Application) Run() {
    fmt.Println("Application started, monitoring configuration changes...")
    
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            config := app.configManager.GetConfig()
            fmt.Printf("Current config: Port=%d, LogLevel=%s, RateLimit=%d\n", 
                config.Server.Port, config.LogLevel, config.Features.RateLimit)
        }
    }
}

func (app *Application) Shutdown() {
    app.configManager.Close()
}

func main() {
    // Create initial configuration file
    initialConfig := AppConfig{
        Server: struct {
            Host    string `json:"host"`
            Port    int    `json:"port"`
            Timeout int    `json:"timeout_seconds"`
        }{
            Host:    "localhost",
            Port:    8080,
            Timeout: 30,
        },
        Database: struct {
            MaxConnections int `json:"max_connections"`
            TimeoutSeconds int `json:"timeout_seconds"`
        }{
            MaxConnections: 10,
            TimeoutSeconds: 5,
        },
        Features: struct {
            EnableMetrics bool `json:"enable_metrics"`
            EnableTracing bool `json:"enable_tracing"`
            RateLimit     int  `json:"rate_limit"`
        }{
            EnableMetrics: false,
            EnableTracing: false,
            RateLimit:     100,
        },
        LogLevel: "info",
        Version:  1,
    }
    
    configData, _ := json.MarshalIndent(initialConfig, "", "  ")
    configPath := "/tmp/hot_reload_config.json"
    
    if err := os.WriteFile(configPath, configData, 0644); err != nil {
        log.Fatalf("Failed to create config file: %v", err)
    }
    
    // Create and start application
    app, err := NewApplication(configPath)
    if err != nil {
        log.Fatalf("Failed to create application: %v", err)
    }
    defer app.Shutdown()
    
    // Handle graceful shutdown
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    
    go app.Run()
    
    fmt.Println("Try modifying the config file:", configPath)
    fmt.Println("For example, change the log_level, rate_limit, or version number")
    fmt.Println("Press Ctrl+C to exit")
    
    <-sigChan
    fmt.Println("Shutting down...")
}
```

This example demonstrates sophisticated configuration hot-reloading with file  
watching, validation, and callback notifications. The pattern enables  
applications to adapt to configuration changes dynamically while maintaining  
consistency and preventing invalid configurations from being applied.  

## Secure configuration with encryption

Sensitive configuration data requires encryption at rest and secure handling  
to prevent credential exposure in logs and memory dumps.  

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "crypto/sha256"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "io"
    "os"
    "strings"
    
    "golang.org/x/crypto/pbkdf2"
)

type SecureConfig struct {
    AppName     string            `json:"app_name"`
    Environment string            `json:"environment"`
    Server      ServerConfig      `json:"server"`
    Secrets     map[string]string `json:"secrets"`
}

type ServerConfig struct {
    Host string `json:"host"`
    Port int    `json:"port"`
}

type ConfigEncryption struct {
    key []byte
}

func NewConfigEncryption(password string, salt []byte) *ConfigEncryption {
    // Derive key using PBKDF2
    key := pbkdf2.Key([]byte(password), salt, 100000, 32, sha256.New)
    return &ConfigEncryption{key: key}
}

func (ce *ConfigEncryption) Encrypt(plaintext string) (string, error) {
    block, err := aes.NewCipher(ce.key)
    if err != nil {
        return "", err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return "", err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return "", err
    }
    
    ciphertext := gcm.Seal(nonce, nonce, []byte(plaintext), nil)
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

func (ce *ConfigEncryption) Decrypt(ciphertext string) (string, error) {
    data, err := base64.StdEncoding.DecodeString(ciphertext)
    if err != nil {
        return "", err
    }
    
    block, err := aes.NewCipher(ce.key)
    if err != nil {
        return "", err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return "", err
    }
    
    nonceSize := gcm.NonceSize()
    if len(data) < nonceSize {
        return "", fmt.Errorf("ciphertext too short")
    }
    
    nonce, cipherData := data[:nonceSize], data[nonceSize:]
    plaintext, err := gcm.Open(nil, nonce, cipherData, nil)
    if err != nil {
        return "", err
    }
    
    return string(plaintext), nil
}

type SecureConfigManager struct {
    config     *SecureConfig
    encryption *ConfigEncryption
}

func NewSecureConfigManager(password string) *SecureConfigManager {
    // In production, use a proper salt storage mechanism
    salt := []byte("secure-config-salt-change-this-in-production")
    encryption := NewConfigEncryption(password, salt)
    
    return &SecureConfigManager{
        encryption: encryption,
    }
}

func (scm *SecureConfigManager) LoadFromFile(filename string) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return fmt.Errorf("failed to read config file: %w", err)
    }
    
    var rawConfig map[string]interface{}
    if err := json.Unmarshal(data, &rawConfig); err != nil {
        return fmt.Errorf("failed to parse config JSON: %w", err)
    }
    
    config := &SecureConfig{
        Secrets: make(map[string]string),
    }
    
    // Load non-secret fields directly
    if appName, ok := rawConfig["app_name"].(string); ok {
        config.AppName = appName
    }
    if env, ok := rawConfig["environment"].(string); ok {
        config.Environment = env
    }
    
    // Load server config
    if serverData, ok := rawConfig["server"].(map[string]interface{}); ok {
        if host, ok := serverData["host"].(string); ok {
            config.Server.Host = host
        }
        if port, ok := serverData["port"].(float64); ok {
            config.Server.Port = int(port)
        }
    }
    
    // Decrypt secret fields
    if secretsData, ok := rawConfig["secrets"].(map[string]interface{}); ok {
        for key, encryptedValue := range secretsData {
            if encStr, ok := encryptedValue.(string); ok {
                if strings.HasPrefix(encStr, "encrypted:") {
                    // Decrypt the value
                    encryptedData := strings.TrimPrefix(encStr, "encrypted:")
                    decrypted, err := scm.encryption.Decrypt(encryptedData)
                    if err != nil {
                        return fmt.Errorf("failed to decrypt secret %s: %w", key, err)
                    }
                    config.Secrets[key] = decrypted
                } else {
                    // Plain text value (for development)
                    config.Secrets[key] = encStr
                }
            }
        }
    }
    
    scm.config = config
    return nil
}

func (scm *SecureConfigManager) SaveToFile(filename string) error {
    if scm.config == nil {
        return fmt.Errorf("no configuration to save")
    }
    
    // Create a copy for serialization
    saveData := map[string]interface{}{
        "app_name":    scm.config.AppName,
        "environment": scm.config.Environment,
        "server": map[string]interface{}{
            "host": scm.config.Server.Host,
            "port": scm.config.Server.Port,
        },
        "secrets": make(map[string]string),
    }
    
    // Encrypt secret values
    secrets := saveData["secrets"].(map[string]string)
    for key, value := range scm.config.Secrets {
        encrypted, err := scm.encryption.Encrypt(value)
        if err != nil {
            return fmt.Errorf("failed to encrypt secret %s: %w", key, err)
        }
        secrets[key] = "encrypted:" + encrypted
    }
    
    data, err := json.MarshalIndent(saveData, "", "  ")
    if err != nil {
        return fmt.Errorf("failed to marshal config: %w", err)
    }
    
    return os.WriteFile(filename, data, 0600) // Restrictive permissions
}

func (scm *SecureConfigManager) SetSecret(key, value string) {
    if scm.config == nil {
        scm.config = &SecureConfig{Secrets: make(map[string]string)}
    }
    scm.config.Secrets[key] = value
}

func (scm *SecureConfigManager) GetSecret(key string) (string, bool) {
    if scm.config == nil {
        return "", false
    }
    value, exists := scm.config.Secrets[key]
    return value, exists
}

func (scm *SecureConfigManager) GetConfig() *SecureConfig {
    return scm.config
}

// SecureString provides a way to handle sensitive strings in memory
type SecureString struct {
    data []byte
}

func NewSecureString(value string) *SecureString {
    return &SecureString{data: []byte(value)}
}

func (ss *SecureString) Get() string {
    return string(ss.data)
}

func (ss *SecureString) Clear() {
    // Zero out the memory
    for i := range ss.data {
        ss.data[i] = 0
    }
    ss.data = nil
}

func (ss *SecureString) String() string {
    return "[REDACTED]"
}

// Example application using secure configuration
func main() {
    fmt.Println("=== Secure Configuration Management ===")
    
    // Get master password from environment (in production, use proper secret management)
    masterPassword := os.Getenv("CONFIG_MASTER_PASSWORD")
    if masterPassword == "" {
        masterPassword = "demo-password-change-in-production"
        fmt.Println("Using demo password (set CONFIG_MASTER_PASSWORD environment variable)")
    }
    
    // Create secure config manager
    scm := NewSecureConfigManager(masterPassword)
    
    // Create sample configuration with secrets
    scm.config = &SecureConfig{
        AppName:     "SecureApp",
        Environment: "production",
        Server: ServerConfig{
            Host: "api.example.com",
            Port: 443,
        },
        Secrets: map[string]string{
            "database_password": "super-secret-db-password-123",
            "jwt_secret":        "very-long-jwt-secret-key-for-token-signing",
            "api_key":          "sk-1234567890abcdef",
            "encryption_key":   "another-very-secret-key-for-data-encryption",
        },
    }
    
    configFile := "/tmp/secure_config.json"
    
    // Save encrypted configuration
    fmt.Println("Saving encrypted configuration...")
    if err := scm.SaveToFile(configFile); err != nil {
        fmt.Printf("Error saving config: %v\n", err)
        return
    }
    
    // Show the encrypted file content
    fmt.Println("\nEncrypted configuration file content:")
    if data, err := os.ReadFile(configFile); err == nil {
        fmt.Println(string(data))
    }
    
    // Load and decrypt configuration
    fmt.Println("\nLoading and decrypting configuration...")
    newScm := NewSecureConfigManager(masterPassword)
    if err := newScm.LoadFromFile(configFile); err != nil {
        fmt.Printf("Error loading config: %v\n", err)
        return
    }
    
    config := newScm.GetConfig()
    fmt.Printf("App: %s (%s)\n", config.AppName, config.Environment)
    fmt.Printf("Server: %s:%d\n", config.Server.Host, config.Server.Port)
    
    fmt.Println("Secrets (decrypted):")
    for key := range config.Secrets {
        // In production, avoid logging actual secret values
        fmt.Printf("  %s: [DECRYPTED - %d chars]\n", key, len(config.Secrets[key]))
    }
    
    // Demonstrate secure string handling
    fmt.Println("\n=== Secure String Handling ===")
    dbPassword, exists := newScm.GetSecret("database_password")
    if exists {
        securePass := NewSecureString(dbPassword)
        fmt.Printf("Database password loaded into secure string\n")
        
        // Use the password (simulated)
        fmt.Printf("Using password for database connection...\n")
        connectionString := fmt.Sprintf("postgres://user:%s@host/db", securePass.Get())
        _ = connectionString // Simulate usage
        
        // Clear sensitive data from memory
        securePass.Clear()
        fmt.Printf("Password cleared from memory: %s\n", securePass)
    }
    
    // Demonstrate wrong password handling
    fmt.Println("\n=== Wrong Password Test ===")
    wrongScm := NewSecureConfigManager("wrong-password")
    if err := wrongScm.LoadFromFile(configFile); err != nil {
        fmt.Printf("Expected error with wrong password: %v\n", err)
    }
    
    // Clean up
    os.Remove(configFile)
}
```

This example demonstrates comprehensive security patterns for configuration  
management including encryption at rest, secure string handling, and proper  
memory management for sensitive data. The approach protects against  
credential exposure while maintaining usability for development workflows.  

## Configuration with CLI flags integration

Combining configuration files with command-line flags provides flexible  
deployment options and operator-friendly interfaces.  

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"
)

type AppConfig struct {
    // Basic application settings
    Name        string `json:"name"`
    Version     string `json:"version"`
    Environment string `json:"environment"`
    Debug       bool   `json:"debug"`
    DryRun      bool   `json:"dry_run"`
    
    // Server configuration
    Server ServerConfig `json:"server"`
    
    // Database configuration
    Database DatabaseConfig `json:"database"`
    
    // Logging configuration
    Logging LoggingConfig `json:"logging"`
    
    // Feature flags
    Features map[string]bool `json:"features"`
    
    // Runtime settings
    Workers    int           `json:"workers"`
    Timeout    time.Duration `json:"timeout"`
    RetryCount int           `json:"retry_count"`
}

type ServerConfig struct {
    Host string `json:"host"`
    Port int    `json:"port"`
    TLS  bool   `json:"tls"`
}

type DatabaseConfig struct {
    Driver   string `json:"driver"`
    Host     string `json:"host"`
    Port     int    `json:"port"`
    Name     string `json:"name"`
    User     string `json:"user"`
    Password string `json:"password"`
}

type LoggingConfig struct {
    Level  string `json:"level"`
    Format string `json:"format"`
    File   string `json:"file"`
}

type CLIFlags struct {
    // Configuration file
    ConfigFile *string
    
    // Application flags
    Name        *string
    Environment *string
    Debug       *bool
    DryRun      *bool
    
    // Server flags
    Host *string
    Port *int
    TLS  *bool
    
    // Database flags
    DBDriver   *string
    DBHost     *string
    DBPort     *int
    DBName     *string
    DBUser     *string
    DBPassword *string
    
    // Logging flags
    LogLevel  *string
    LogFormat *string
    LogFile   *string
    
    // Runtime flags
    Workers    *int
    Timeout    *string
    RetryCount *int
    
    // Feature flags
    Features *string
    
    // Utility flags
    ShowConfig *bool
    Validate   *bool
    Help       *bool
}

func NewCLIFlags() *CLIFlags {
    return &CLIFlags{
        // Configuration file
        ConfigFile: flag.String("config", "", "Configuration file path"),
        
        // Application flags
        Name:        flag.String("name", "", "Application name"),
        Environment: flag.String("env", "", "Environment (dev, staging, prod)"),
        Debug:       flag.Bool("debug", false, "Enable debug mode"),
        DryRun:      flag.Bool("dry-run", false, "Enable dry-run mode"),
        
        // Server flags
        Host: flag.String("host", "", "Server host address"),
        Port: flag.Int("port", 0, "Server port number"),
        TLS:  flag.Bool("tls", false, "Enable TLS"),
        
        // Database flags
        DBDriver:   flag.String("db-driver", "", "Database driver (postgres, mysql, sqlite)"),
        DBHost:     flag.String("db-host", "", "Database host"),
        DBPort:     flag.Int("db-port", 0, "Database port"),
        DBName:     flag.String("db-name", "", "Database name"),
        DBUser:     flag.String("db-user", "", "Database user"),
        DBPassword: flag.String("db-password", "", "Database password"),
        
        // Logging flags
        LogLevel:  flag.String("log-level", "", "Log level (debug, info, warn, error)"),
        LogFormat: flag.String("log-format", "", "Log format (text, json)"),
        LogFile:   flag.String("log-file", "", "Log file path"),
        
        // Runtime flags
        Workers:    flag.Int("workers", 0, "Number of worker goroutines"),
        Timeout:    flag.String("timeout", "", "Request timeout (e.g., 30s, 5m)"),
        RetryCount: flag.Int("retry-count", 0, "Number of retries"),
        
        // Feature flags
        Features: flag.String("features", "", "Comma-separated feature flags (feature1=true,feature2=false)"),
        
        // Utility flags
        ShowConfig: flag.Bool("show-config", false, "Show final configuration and exit"),
        Validate:   flag.Bool("validate", false, "Validate configuration and exit"),
        Help:       flag.Bool("help", false, "Show help message"),
    }
}

func (cf *CLIFlags) ParseFlags() {
    flag.Parse()
}

func (cf *CLIFlags) ShowHelp() {
    fmt.Println("Configuration Management Example")
    fmt.Println("Usage: go run main.go [options]")
    fmt.Println()
    fmt.Println("Options:")
    flag.PrintDefaults()
    fmt.Println()
    fmt.Println("Examples:")
    fmt.Println("  go run main.go -config app.json")
    fmt.Println("  go run main.go -host 0.0.0.0 -port 8080 -debug")
    fmt.Println("  go run main.go -config app.json -db-password secret -features metrics=true,tracing=false")
}

func LoadConfig(flags *CLIFlags) (*AppConfig, error) {
    // Start with default configuration
    config := &AppConfig{
        Name:        "MyApp",
        Version:     "1.0.0",
        Environment: "development",
        Debug:       false,
        DryRun:      false,
        Server: ServerConfig{
            Host: "localhost",
            Port: 8080,
            TLS:  false,
        },
        Database: DatabaseConfig{
            Driver:   "postgres",
            Host:     "localhost",
            Port:     5432,
            Name:     "myapp",
            User:     "postgres",
            Password: "postgres",
        },
        Logging: LoggingConfig{
            Level:  "info",
            Format: "text",
            File:   "",
        },
        Features:   make(map[string]bool),
        Workers:    4,
        Timeout:    30 * time.Second,
        RetryCount: 3,
    }
    
    // Load from configuration file if specified
    if *flags.ConfigFile != "" {
        if err := loadConfigFromFile(*flags.ConfigFile, config); err != nil {
            return nil, fmt.Errorf("failed to load config file: %w", err)
        }
        fmt.Printf("Loaded configuration from: %s\n", *flags.ConfigFile)
    }
    
    // Override with command-line flags
    applyFlagsToConfig(flags, config)
    
    return config, nil
}

func loadConfigFromFile(filename string, config *AppConfig) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return err
    }
    
    return json.Unmarshal(data, config)
}

func applyFlagsToConfig(flags *CLIFlags, config *AppConfig) {
    // Application settings
    if *flags.Name != "" {
        config.Name = *flags.Name
    }
    if *flags.Environment != "" {
        config.Environment = *flags.Environment
    }
    
    // Boolean flags are special - they're set if the flag is present
    flag.Visit(func(f *flag.Flag) {
        switch f.Name {
        case "debug":
            config.Debug = *flags.Debug
        case "dry-run":
            config.DryRun = *flags.DryRun
        case "tls":
            config.Server.TLS = *flags.TLS
        }
    })
    
    // Server settings
    if *flags.Host != "" {
        config.Server.Host = *flags.Host
    }
    if *flags.Port != 0 {
        config.Server.Port = *flags.Port
    }
    
    // Database settings
    if *flags.DBDriver != "" {
        config.Database.Driver = *flags.DBDriver
    }
    if *flags.DBHost != "" {
        config.Database.Host = *flags.DBHost
    }
    if *flags.DBPort != 0 {
        config.Database.Port = *flags.DBPort
    }
    if *flags.DBName != "" {
        config.Database.Name = *flags.DBName
    }
    if *flags.DBUser != "" {
        config.Database.User = *flags.DBUser
    }
    if *flags.DBPassword != "" {
        config.Database.Password = *flags.DBPassword
    }
    
    // Logging settings
    if *flags.LogLevel != "" {
        config.Logging.Level = *flags.LogLevel
    }
    if *flags.LogFormat != "" {
        config.Logging.Format = *flags.LogFormat
    }
    if *flags.LogFile != "" {
        config.Logging.File = *flags.LogFile
    }
    
    // Runtime settings
    if *flags.Workers != 0 {
        config.Workers = *flags.Workers
    }
    if *flags.Timeout != "" {
        if timeout, err := time.ParseDuration(*flags.Timeout); err == nil {
            config.Timeout = timeout
        }
    }
    if *flags.RetryCount != 0 {
        config.RetryCount = *flags.RetryCount
    }
    
    // Parse feature flags
    if *flags.Features != "" {
        parseFeatureFlags(*flags.Features, config)
    }
}

func parseFeatureFlags(features string, config *AppConfig) {
    if config.Features == nil {
        config.Features = make(map[string]bool)
    }
    
    pairs := strings.Split(features, ",")
    for _, pair := range pairs {
        parts := strings.Split(strings.TrimSpace(pair), "=")
        if len(parts) == 2 {
            feature := strings.TrimSpace(parts[0])
            if value, err := strconv.ParseBool(strings.TrimSpace(parts[1])); err == nil {
                config.Features[feature] = value
            }
        }
    }
}

func (config *AppConfig) Validate() error {
    // Validate environment
    validEnvs := map[string]bool{
        "development": true, "staging": true, "production": true, "test": true,
    }
    if !validEnvs[config.Environment] {
        return fmt.Errorf("invalid environment: %s", config.Environment)
    }
    
    // Validate server settings
    if config.Server.Port < 1 || config.Server.Port > 65535 {
        return fmt.Errorf("invalid port: %d", config.Server.Port)
    }
    
    // Validate database settings
    validDrivers := map[string]bool{
        "postgres": true, "mysql": true, "sqlite": true,
    }
    if !validDrivers[config.Database.Driver] {
        return fmt.Errorf("invalid database driver: %s", config.Database.Driver)
    }
    
    // Validate logging settings
    validLevels := map[string]bool{
        "debug": true, "info": true, "warn": true, "error": true,
    }
    if !validLevels[config.Logging.Level] {
        return fmt.Errorf("invalid log level: %s", config.Logging.Level)
    }
    
    validFormats := map[string]bool{
        "text": true, "json": true,
    }
    if !validFormats[config.Logging.Format] {
        return fmt.Errorf("invalid log format: %s", config.Logging.Format)
    }
    
    // Validate runtime settings
    if config.Workers < 1 {
        return fmt.Errorf("workers must be positive: %d", config.Workers)
    }
    
    if config.Timeout < time.Second {
        return fmt.Errorf("timeout must be at least 1 second: %v", config.Timeout)
    }
    
    return nil
}

func (config *AppConfig) Print() {
    fmt.Println("\n=== Final Configuration ===")
    fmt.Printf("Application: %s v%s (%s)\n", config.Name, config.Version, config.Environment)
    fmt.Printf("Debug: %v, Dry Run: %v\n", config.Debug, config.DryRun)
    fmt.Printf("Server: %s:%d (TLS: %v)\n", config.Server.Host, config.Server.Port, config.Server.TLS)
    fmt.Printf("Database: %s://%s@%s:%d/%s\n", 
        config.Database.Driver, config.Database.User, config.Database.Host, 
        config.Database.Port, config.Database.Name)
    fmt.Printf("Logging: %s level, %s format", config.Logging.Level, config.Logging.Format)
    if config.Logging.File != "" {
        fmt.Printf(" -> %s", config.Logging.File)
    }
    fmt.Printf("\n")
    fmt.Printf("Runtime: %d workers, %v timeout, %d retries\n", 
        config.Workers, config.Timeout, config.RetryCount)
    
    if len(config.Features) > 0 {
        fmt.Printf("Features: ")
        first := true
        for feature, enabled := range config.Features {
            if !first {
                fmt.Printf(", ")
            }
            fmt.Printf("%s=%v", feature, enabled)
            first = false
        }
        fmt.Printf("\n")
    }
}

func main() {
    // Create and parse CLI flags
    flags := NewCLIFlags()
    flags.ParseFlags()
    
    // Show help if requested
    if *flags.Help {
        flags.ShowHelp()
        return
    }
    
    // Create sample configuration file for demonstration
    if *flags.ConfigFile == "" {
        sampleConfig := &AppConfig{
            Name:        "SampleApp",
            Version:     "2.0.0",
            Environment: "staging",
            Debug:       false,
            Server: ServerConfig{
                Host: "0.0.0.0",
                Port: 9000,
                TLS:  true,
            },
            Database: DatabaseConfig{
                Driver: "postgres",
                Host:   "db.staging.example.com",
                Port:   5432,
                Name:   "staging_db",
                User:   "app_user",
            },
            Logging: LoggingConfig{
                Level:  "warn",
                Format: "json",
                File:   "/var/log/app.log",
            },
            Features: map[string]bool{
                "metrics":       true,
                "tracing":       false,
                "rate_limiting": true,
            },
            Workers:    8,
            Timeout:    45 * time.Second,
            RetryCount: 5,
        }
        
        data, _ := json.MarshalIndent(sampleConfig, "", "  ")
        sampleFile := "/tmp/sample_config.json"
        os.WriteFile(sampleFile, data, 0644)
        
        fmt.Printf("Created sample config file: %s\n", sampleFile)
        fmt.Printf("Try: go run main.go -config %s\n", sampleFile)
        fmt.Printf("Or:  go run main.go -config %s -port 3000 -debug -features metrics=false,caching=true\n", sampleFile)
    }
    
    // Load configuration
    config, err := LoadConfig(flags)
    if err != nil {
        fmt.Printf("Error loading configuration: %v\n", err)
        os.Exit(1)
    }
    
    // Validate configuration
    if *flags.Validate || *flags.ShowConfig {
        if err := config.Validate(); err != nil {
            fmt.Printf("Configuration validation failed: %v\n", err)
            os.Exit(1)
        }
        fmt.Println("Configuration validation passed!")
    }
    
    // Show configuration if requested
    if *flags.ShowConfig {
        config.Print()
        return
    }
    
    // Run application with configuration
    fmt.Printf("Starting %s with configuration from ", config.Name)
    if *flags.ConfigFile != "" {
        fmt.Printf("file and ")
    }
    fmt.Printf("command-line overrides\n")
    
    config.Print()
}
```

This example demonstrates comprehensive CLI flag integration with configuration  
files, providing operators with flexible deployment options while maintaining  
configuration file benefits. The pattern supports configuration validation,  
help generation, and clear precedence rules for configuration sources.  

## Configuration providers with interfaces

Interface-based configuration providers enable pluggable configuration  
sources and support for remote configuration services.  

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
    "strings"
    "sync"
    "time"
)

// ConfigProvider defines the interface for configuration sources
type ConfigProvider interface {
    Name() string
    Load(ctx context.Context) (map[string]interface{}, error)
    Watch(ctx context.Context, callback func(map[string]interface{})) error
}

// FileConfigProvider loads configuration from local files
type FileConfigProvider struct {
    path string
}

func NewFileConfigProvider(path string) *FileConfigProvider {
    return &FileConfigProvider{path: path}
}

func (f *FileConfigProvider) Name() string {
    return fmt.Sprintf("file:%s", f.path)
}

func (f *FileConfigProvider) Load(ctx context.Context) (map[string]interface{}, error) {
    data, err := os.ReadFile(f.path)
    if err != nil {
        return nil, fmt.Errorf("failed to read file: %w", err)
    }
    
    var config map[string]interface{}
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("failed to parse JSON: %w", err)
    }
    
    return config, nil
}

func (f *FileConfigProvider) Watch(ctx context.Context, callback func(map[string]interface{})) error {
    // Simple polling implementation for demonstration
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()
    
    var lastModTime time.Time
    if stat, err := os.Stat(f.path); err == nil {
        lastModTime = stat.ModTime()
    }
    
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
            stat, err := os.Stat(f.path)
            if err != nil {
                continue
            }
            
            if stat.ModTime().After(lastModTime) {
                lastModTime = stat.ModTime()
                if config, err := f.Load(ctx); err == nil {
                    callback(config)
                }
            }
        }
    }
}

// HTTPConfigProvider loads configuration from HTTP endpoints
type HTTPConfigProvider struct {
    url    string
    client *http.Client
    headers map[string]string
}

func NewHTTPConfigProvider(url string) *HTTPConfigProvider {
    return &HTTPConfigProvider{
        url: url,
        client: &http.Client{
            Timeout: 10 * time.Second,
        },
        headers: make(map[string]string),
    }
}

func (h *HTTPConfigProvider) SetHeader(key, value string) {
    h.headers[key] = value
}

func (h *HTTPConfigProvider) Name() string {
    return fmt.Sprintf("http:%s", h.url)
}

func (h *HTTPConfigProvider) Load(ctx context.Context) (map[string]interface{}, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", h.url, nil)
    if err != nil {
        return nil, fmt.Errorf("failed to create request: %w", err)
    }
    
    // Add custom headers
    for key, value := range h.headers {
        req.Header.Set(key, value)
    }
    
    resp, err := h.client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch config: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("HTTP error: %d", resp.StatusCode)
    }
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed to read response: %w", err)
    }
    
    var config map[string]interface{}
    if err := json.Unmarshal(body, &config); err != nil {
        return nil, fmt.Errorf("failed to parse JSON: %w", err)
    }
    
    return config, nil
}

func (h *HTTPConfigProvider) Watch(ctx context.Context, callback func(map[string]interface{})) error {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    var lastConfig map[string]interface{}
    
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
            config, err := h.Load(ctx)
            if err != nil {
                fmt.Printf("Error loading config from %s: %v\n", h.url, err)
                continue
            }
            
            // Simple change detection using JSON string comparison
            if !configEqual(lastConfig, config) {
                lastConfig = config
                callback(config)
            }
        }
    }
}

// EnvironmentConfigProvider loads configuration from environment variables
type EnvironmentConfigProvider struct {
    prefix string
}

func NewEnvironmentConfigProvider(prefix string) *EnvironmentConfigProvider {
    return &EnvironmentConfigProvider{prefix: prefix}
}

func (e *EnvironmentConfigProvider) Name() string {
    return fmt.Sprintf("env:%s", e.prefix)
}

func (e *EnvironmentConfigProvider) Load(ctx context.Context) (map[string]interface{}, error) {
    config := make(map[string]interface{})
    
    for _, env := range os.Environ() {
        if strings.HasPrefix(env, e.prefix) {
            parts := strings.SplitN(env, "=", 2)
            if len(parts) == 2 {
                key := strings.ToLower(strings.TrimPrefix(parts[0], e.prefix))
                key = strings.ReplaceAll(key, "_", ".")
                config[key] = parts[1]
            }
        }
    }
    
    return config, nil
}

func (e *EnvironmentConfigProvider) Watch(ctx context.Context, callback func(map[string]interface{})) error {
    // Environment variables don't typically change during runtime
    // This is a no-op implementation
    <-ctx.Done()
    return ctx.Err()
}

// ConfigManager orchestrates multiple configuration providers
type ConfigManager struct {
    providers []ConfigProvider
    config    map[string]interface{}
    mutex     sync.RWMutex
    callbacks []func(map[string]interface{})
}

func NewConfigManager() *ConfigManager {
    return &ConfigManager{
        providers: make([]ConfigProvider, 0),
        config:    make(map[string]interface{}),
        callbacks: make([]func(map[string]interface{}), 0),
    }
}

func (cm *ConfigManager) AddProvider(provider ConfigProvider) {
    cm.providers = append(cm.providers, provider)
}

func (cm *ConfigManager) OnConfigChange(callback func(map[string]interface{})) {
    cm.callbacks = append(cm.callbacks, callback)
}

func (cm *ConfigManager) LoadConfig(ctx context.Context) error {
    allConfig := make(map[string]interface{})
    
    // Load from all providers in order (later providers override earlier ones)
    for _, provider := range cm.providers {
        fmt.Printf("Loading config from %s...\n", provider.Name())
        
        config, err := provider.Load(ctx)
        if err != nil {
            fmt.Printf("Warning: failed to load from %s: %v\n", provider.Name(), err)
            continue
        }
        
        // Merge configuration
        mergeConfig(allConfig, config)
        fmt.Printf("Loaded %d keys from %s\n", len(config), provider.Name())
    }
    
    cm.mutex.Lock()
    cm.config = allConfig
    cm.mutex.Unlock()
    
    return nil
}

func (cm *ConfigManager) StartWatching(ctx context.Context) {
    for _, provider := range cm.providers {
        go func(p ConfigProvider) {
            err := p.Watch(ctx, func(config map[string]interface{}) {
                fmt.Printf("Configuration change detected from %s\n", p.Name())
                cm.handleConfigChange(config)
            })
            if err != nil && err != context.Canceled {
                fmt.Printf("Error watching provider %s: %v\n", p.Name(), err)
            }
        }(provider)
    }
}

func (cm *ConfigManager) handleConfigChange(newConfig map[string]interface{}) {
    cm.mutex.Lock()
    mergeConfig(cm.config, newConfig)
    configCopy := copyConfig(cm.config)
    cm.mutex.Unlock()
    
    // Notify callbacks
    for _, callback := range cm.callbacks {
        go callback(configCopy)
    }
}

func (cm *ConfigManager) GetConfig() map[string]interface{} {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    return copyConfig(cm.config)
}

func (cm *ConfigManager) GetString(key string, defaultValue string) string {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    
    if value, exists := cm.config[key]; exists {
        if str, ok := value.(string); ok {
            return str
        }
    }
    return defaultValue
}

func (cm *ConfigManager) GetInt(key string, defaultValue int) int {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    
    if value, exists := cm.config[key]; exists {
        switch v := value.(type) {
        case int:
            return v
        case float64:
            return int(v)
        case string:
            // Try to parse string as int
            if i, err := fmt.Sscanf(v, "%d", new(int)); err == nil && i == 1 {
                var result int
                fmt.Sscanf(v, "%d", &result)
                return result
            }
        }
    }
    return defaultValue
}

func (cm *ConfigManager) GetBool(key string, defaultValue bool) bool {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    
    if value, exists := cm.config[key]; exists {
        switch v := value.(type) {
        case bool:
            return v
        case string:
            return strings.ToLower(v) == "true"
        }
    }
    return defaultValue
}

// Helper functions
func mergeConfig(dst, src map[string]interface{}) {
    for key, value := range src {
        dst[key] = value
    }
}

func copyConfig(src map[string]interface{}) map[string]interface{} {
    dst := make(map[string]interface{})
    for key, value := range src {
        dst[key] = value
    }
    return dst
}

func configEqual(a, b map[string]interface{}) bool {
    if len(a) != len(b) {
        return false
    }
    
    for key, valueA := range a {
        valueB, exists := b[key]
        if !exists || valueA != valueB {
            return false
        }
    }
    
    return true
}

func main() {
    fmt.Println("=== Configuration Providers Example ===")
    
    // Create sample configuration files and HTTP endpoint
    fileConfig := map[string]interface{}{
        "app.name":     "ProviderApp",
        "app.version":  "1.0.0",
        "server.host":  "localhost",
        "server.port":  8080,
        "database.url": "postgres://localhost:5432/app",
        "features.metrics": true,
    }
    
    fileData, _ := json.MarshalIndent(fileConfig, "", "  ")
    configFile := "/tmp/provider_config.json"
    os.WriteFile(configFile, fileData, 0644)
    
    // Set environment variables
    os.Setenv("APP_SERVER_HOST", "0.0.0.0")
    os.Setenv("APP_SERVER_PORT", "9090")
    os.Setenv("APP_DATABASE_PASSWORD", "secret")
    os.Setenv("APP_FEATURES_TRACING", "true")
    
    // Create configuration manager
    cm := NewConfigManager()
    
    // Add providers in order of precedence (lowest to highest)
    cm.AddProvider(NewFileConfigProvider(configFile))
    cm.AddProvider(NewEnvironmentConfigProvider("APP_"))
    
    // Optional: Add HTTP provider (commented out for demo)
    // httpProvider := NewHTTPConfigProvider("http://config-service.example.com/config")
    // httpProvider.SetHeader("Authorization", "Bearer token")
    // cm.AddProvider(httpProvider)
    
    // Set up configuration change callback
    cm.OnConfigChange(func(config map[string]interface{}) {
        fmt.Println("\n=== Configuration Changed ===")
        printConfig(config)
    })
    
    // Load initial configuration
    ctx := context.Background()
    if err := cm.LoadConfig(ctx); err != nil {
        fmt.Printf("Error loading config: %v\n", err)
        return
    }
    
    // Start watching for changes
    watchCtx, cancel := context.WithCancel(ctx)
    defer cancel()
    cm.StartWatching(watchCtx)
    
    // Print initial configuration
    fmt.Println("\n=== Initial Configuration ===")
    printConfig(cm.GetConfig())
    
    // Demonstrate typed getters
    fmt.Println("\n=== Typed Configuration Access ===")
    fmt.Printf("App Name: %s\n", cm.GetString("app.name", "Unknown"))
    fmt.Printf("Server Host: %s\n", cm.GetString("server.host", "localhost"))
    fmt.Printf("Server Port: %d\n", cm.GetInt("server.port", 8080))
    fmt.Printf("Metrics Enabled: %v\n", cm.GetBool("features.metrics", false))
    fmt.Printf("Tracing Enabled: %v\n", cm.GetBool("features.tracing", false))
    
    // Simulate configuration changes
    fmt.Println("\n=== Simulating Configuration Changes ===")
    fmt.Println("Modifying configuration file...")
    
    // Update file configuration
    fileConfig["app.version"] = "1.1.0"
    fileConfig["server.port"] = 3000
    fileConfig["features.caching"] = true
    
    updatedData, _ := json.MarshalIndent(fileConfig, "", "  ")
    os.WriteFile(configFile, updatedData, 0644)
    
    // Wait for file change detection
    time.Sleep(6 * time.Second)
    
    fmt.Println("\n=== Updated Configuration ===")
    printConfig(cm.GetConfig())
    
    // Clean up
    os.Remove(configFile)
}

func printConfig(config map[string]interface{}) {
    for key, value := range config {
        fmt.Printf("  %s: %v\n", key, value)
    }
}
```

This example demonstrates a flexible configuration provider system that  
supports multiple sources with clear precedence rules. The interface-based  
design enables easy addition of new providers like etcd, Consul, or cloud  
configuration services while maintaining consistent behavior.  

## Configuration testing patterns

Proper testing of configuration handling ensures robust deployment and  
prevents configuration-related failures in production environments.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "path/filepath"
    "reflect"
    "strings"
    "testing"
    "time"
)

// ApplicationConfig represents our application configuration
type ApplicationConfig struct {
    Service  ServiceConfig  `json:"service" validate:"required"`
    Database DatabaseConfig `json:"database" validate:"required"`
    Cache    CacheConfig    `json:"cache"`
    Security SecurityConfig `json:"security" validate:"required"`
}

type ServiceConfig struct {
    Name    string        `json:"name" validate:"required"`
    Port    int           `json:"port" validate:"min=1,max=65535"`
    Timeout time.Duration `json:"timeout" validate:"min=1s"`
}

type DatabaseConfig struct {
    Driver   string `json:"driver" validate:"required,oneof=postgres mysql"`
    Host     string `json:"host" validate:"required"`
    Port     int    `json:"port" validate:"min=1,max=65535"`
    Database string `json:"database" validate:"required"`
    Username string `json:"username" validate:"required"`
    Password string `json:"password" validate:"required,min=8"`
}

type CacheConfig struct {
    Type string `json:"type" validate:"oneof=redis memory"`
    TTL  int    `json:"ttl" validate:"min=1"`
}

type SecurityConfig struct {
    JWTSecret    string   `json:"jwt_secret" validate:"required,min=32"`
    AllowedHosts []string `json:"allowed_hosts" validate:"required"`
}

// ConfigLoader handles configuration loading and validation
type ConfigLoader struct {
    validator ConfigValidator
}

type ConfigValidator interface {
    Validate(config *ApplicationConfig) []string
}

type DefaultValidator struct{}

func (v *DefaultValidator) Validate(config *ApplicationConfig) []string {
    var errors []string
    
    // Service validation
    if config.Service.Name == "" {
        errors = append(errors, "service.name is required")
    }
    if config.Service.Port < 1 || config.Service.Port > 65535 {
        errors = append(errors, "service.port must be between 1 and 65535")
    }
    if config.Service.Timeout < time.Second {
        errors = append(errors, "service.timeout must be at least 1 second")
    }
    
    // Database validation
    if config.Database.Driver == "" {
        errors = append(errors, "database.driver is required")
    } else if config.Database.Driver != "postgres" && config.Database.Driver != "mysql" {
        errors = append(errors, "database.driver must be postgres or mysql")
    }
    if config.Database.Host == "" {
        errors = append(errors, "database.host is required")
    }
    if config.Database.Port < 1 || config.Database.Port > 65535 {
        errors = append(errors, "database.port must be between 1 and 65535")
    }
    if config.Database.Database == "" {
        errors = append(errors, "database.database is required")
    }
    if config.Database.Username == "" {
        errors = append(errors, "database.username is required")
    }
    if len(config.Database.Password) < 8 {
        errors = append(errors, "database.password must be at least 8 characters")
    }
    
    // Cache validation (optional)
    if config.Cache.Type != "" {
        if config.Cache.Type != "redis" && config.Cache.Type != "memory" {
            errors = append(errors, "cache.type must be redis or memory")
        }
        if config.Cache.TTL < 1 {
            errors = append(errors, "cache.ttl must be positive")
        }
    }
    
    // Security validation
    if len(config.Security.JWTSecret) < 32 {
        errors = append(errors, "security.jwt_secret must be at least 32 characters")
    }
    if len(config.Security.AllowedHosts) == 0 {
        errors = append(errors, "security.allowed_hosts is required")
    }
    
    return errors
}

func NewConfigLoader(validator ConfigValidator) *ConfigLoader {
    return &ConfigLoader{validator: validator}
}

func (cl *ConfigLoader) LoadFromFile(filename string) (*ApplicationConfig, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read config file: %w", err)
    }
    
    var config ApplicationConfig
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("failed to parse JSON: %w", err)
    }
    
    // Validate configuration
    if errors := cl.validator.Validate(&config); len(errors) > 0 {
        return nil, fmt.Errorf("validation errors: %s", strings.Join(errors, ", "))
    }
    
    return &config, nil
}

// Test utilities for configuration testing
type ConfigTestSuite struct {
    tempDir      string
    configLoader *ConfigLoader
}

func NewConfigTestSuite() *ConfigTestSuite {
    tempDir, _ := os.MkdirTemp("", "config_test_")
    return &ConfigTestSuite{
        tempDir:      tempDir,
        configLoader: NewConfigLoader(&DefaultValidator{}),
    }
}

func (cts *ConfigTestSuite) Cleanup() {
    os.RemoveAll(cts.tempDir)
}

func (cts *ConfigTestSuite) CreateConfigFile(name string, config interface{}) string {
    data, _ := json.MarshalIndent(config, "", "  ")
    filename := filepath.Join(cts.tempDir, name)
    os.WriteFile(filename, data, 0644)
    return filename
}

func (cts *ConfigTestSuite) CreateInvalidConfigFile(name string, content string) string {
    filename := filepath.Join(cts.tempDir, name)
    os.WriteFile(filename, []byte(content), 0644)
    return filename
}

// Example test functions (normally these would be in *_test.go files)

func TestValidConfiguration(t *testing.T) {
    suite := NewConfigTestSuite()
    defer suite.Cleanup()
    
    validConfig := ApplicationConfig{
        Service: ServiceConfig{
            Name:    "TestApp",
            Port:    8080,
            Timeout: 30 * time.Second,
        },
        Database: DatabaseConfig{
            Driver:   "postgres",
            Host:     "localhost",
            Port:     5432,
            Database: "testdb",
            Username: "testuser",
            Password: "testpassword123",
        },
        Cache: CacheConfig{
            Type: "redis",
            TTL:  3600,
        },
        Security: SecurityConfig{
            JWTSecret:    "this-is-a-very-long-jwt-secret-key-for-testing-purposes-only",
            AllowedHosts: []string{"localhost", "test.example.com"},
        },
    }
    
    configFile := suite.CreateConfigFile("valid.json", validConfig)
    
    loadedConfig, err := suite.configLoader.LoadFromFile(configFile)
    if err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
    
    if !reflect.DeepEqual(loadedConfig, &validConfig) {
        t.Errorf("Loaded config doesn't match expected config")
    }
    
    fmt.Println("✓ Valid configuration test passed")
}

func TestInvalidConfiguration(t *testing.T) {
    suite := NewConfigTestSuite()
    defer suite.Cleanup()
    
    testCases := []struct {
        name   string
        config ApplicationConfig
        expectedErrors []string
    }{
        {
            name: "missing service name",
            config: ApplicationConfig{
                Service: ServiceConfig{
                    Port:    8080,
                    Timeout: 30 * time.Second,
                },
                Database: DatabaseConfig{
                    Driver:   "postgres",
                    Host:     "localhost",
                    Port:     5432,
                    Database: "testdb",
                    Username: "testuser",
                    Password: "testpassword123",
                },
                Security: SecurityConfig{
                    JWTSecret:    "this-is-a-very-long-jwt-secret-key-for-testing-purposes-only",
                    AllowedHosts: []string{"localhost"},
                },
            },
            expectedErrors: []string{"service.name is required"},
        },
        {
            name: "invalid port",
            config: ApplicationConfig{
                Service: ServiceConfig{
                    Name:    "TestApp",
                    Port:    70000,
                    Timeout: 30 * time.Second,
                },
                Database: DatabaseConfig{
                    Driver:   "postgres",
                    Host:     "localhost",
                    Port:     5432,
                    Database: "testdb",
                    Username: "testuser",
                    Password: "testpassword123",
                },
                Security: SecurityConfig{
                    JWTSecret:    "this-is-a-very-long-jwt-secret-key-for-testing-purposes-only",
                    AllowedHosts: []string{"localhost"},
                },
            },
            expectedErrors: []string{"service.port must be between 1 and 65535"},
        },
        {
            name: "weak password",
            config: ApplicationConfig{
                Service: ServiceConfig{
                    Name:    "TestApp",
                    Port:    8080,
                    Timeout: 30 * time.Second,
                },
                Database: DatabaseConfig{
                    Driver:   "postgres",
                    Host:     "localhost",
                    Port:     5432,
                    Database: "testdb",
                    Username: "testuser",
                    Password: "weak",
                },
                Security: SecurityConfig{
                    JWTSecret:    "this-is-a-very-long-jwt-secret-key-for-testing-purposes-only",
                    AllowedHosts: []string{"localhost"},
                },
            },
            expectedErrors: []string{"database.password must be at least 8 characters"},
        },
    }
    
    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            configFile := suite.CreateConfigFile(tc.name+".json", tc.config)
            
            _, err := suite.configLoader.LoadFromFile(configFile)
            if err == nil {
                t.Fatalf("Expected validation error, got none")
            }
            
            errorStr := err.Error()
            for _, expectedError := range tc.expectedErrors {
                if !strings.Contains(errorStr, expectedError) {
                    t.Errorf("Expected error '%s' not found in: %s", expectedError, errorStr)
                }
            }
        })
    }
    
    fmt.Println("✓ Invalid configuration tests passed")
}

func TestMalformedJSON(t *testing.T) {
    suite := NewConfigTestSuite()
    defer suite.Cleanup()
    
    malformedJSON := `{
        "service": {
            "name": "TestApp",
            "port": 8080,
            "timeout": "30s"
        },
        "database": {
            "driver": "postgres",
            "host": "localhost",
            "port": 5432,
            "database": "testdb",
            "username": "testuser",
            "password": "testpassword123"
        },
        // This comment makes it invalid JSON
        "security": {
            "jwt_secret": "this-is-a-very-long-jwt-secret-key-for-testing-purposes-only",
            "allowed_hosts": ["localhost"]
        }
    }`
    
    configFile := suite.CreateInvalidConfigFile("malformed.json", malformedJSON)
    
    _, err := suite.configLoader.LoadFromFile(configFile)
    if err == nil {
        t.Fatalf("Expected JSON parsing error, got none")
    }
    
    if !strings.Contains(err.Error(), "failed to parse JSON") {
        t.Errorf("Expected JSON parsing error, got: %v", err)
    }
    
    fmt.Println("✓ Malformed JSON test passed")
}

func TestMissingConfigFile(t *testing.T) {
    suite := NewConfigTestSuite()
    defer suite.Cleanup()
    
    nonExistentFile := filepath.Join(suite.tempDir, "nonexistent.json")
    
    _, err := suite.configLoader.LoadFromFile(nonExistentFile)
    if err == nil {
        t.Fatalf("Expected file not found error, got none")
    }
    
    if !strings.Contains(err.Error(), "failed to read config file") {
        t.Errorf("Expected file read error, got: %v", err)
    }
    
    fmt.Println("✓ Missing config file test passed")
}

// Benchmark configuration loading performance
func BenchmarkConfigLoading(b *testing.B) {
    suite := NewConfigTestSuite()
    defer suite.Cleanup()
    
    config := ApplicationConfig{
        Service: ServiceConfig{
            Name:    "BenchmarkApp",
            Port:    8080,
            Timeout: 30 * time.Second,
        },
        Database: DatabaseConfig{
            Driver:   "postgres",
            Host:     "localhost",
            Port:     5432,
            Database: "benchdb",
            Username: "benchuser",
            Password: "benchpassword123",
        },
        Security: SecurityConfig{
            JWTSecret:    "this-is-a-very-long-jwt-secret-key-for-benchmarking-purposes-only",
            AllowedHosts: []string{"localhost", "bench.example.com"},
        },
    }
    
    configFile := suite.CreateConfigFile("benchmark.json", config)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, err := suite.configLoader.LoadFromFile(configFile)
        if err != nil {
            b.Fatalf("Benchmark failed: %v", err)
        }
    }
}

// Mock validator for testing
type MockValidator struct {
    ShouldFail bool
    Errors     []string
}

func (mv *MockValidator) Validate(config *ApplicationConfig) []string {
    if mv.ShouldFail {
        return mv.Errors
    }
    return nil
}

func TestWithMockValidator(t *testing.T) {
    suite := NewConfigTestSuite()
    defer suite.Cleanup()
    
    // Test with failing validator
    mockValidator := &MockValidator{
        ShouldFail: true,
        Errors:     []string{"mock validation error"},
    }
    
    suite.configLoader = NewConfigLoader(mockValidator)
    
    config := ApplicationConfig{
        Service: ServiceConfig{Name: "Test", Port: 8080, Timeout: 30 * time.Second},
        Database: DatabaseConfig{
            Driver: "postgres", Host: "localhost", Port: 5432,
            Database: "test", Username: "test", Password: "testpassword123",
        },
        Security: SecurityConfig{
            JWTSecret: "this-is-a-very-long-jwt-secret-key", 
            AllowedHosts: []string{"localhost"},
        },
    }
    
    configFile := suite.CreateConfigFile("mock.json", config)
    
    _, err := suite.configLoader.LoadFromFile(configFile)
    if err == nil {
        t.Fatalf("Expected validation error from mock, got none")
    }
    
    if !strings.Contains(err.Error(), "mock validation error") {
        t.Errorf("Expected mock validation error, got: %v", err)
    }
    
    fmt.Println("✓ Mock validator test passed")
}

func main() {
    fmt.Println("=== Configuration Testing Patterns ===")
    
    // Create a mock testing.T for demonstration
    t := &testing.T{}
    
    // Run tests
    TestValidConfiguration(t)
    TestInvalidConfiguration(t)
    TestMalformedJSON(t)
    TestMissingConfigFile(t)
    TestWithMockValidator(t)
    
    // Run benchmark
    fmt.Println("\n=== Performance Benchmark ===")
    result := testing.Benchmark(BenchmarkConfigLoading)
    fmt.Printf("Benchmark results: %s\n", result.String())
    
    fmt.Println("\n=== Test Suite Complete ===")
    fmt.Println("All configuration tests passed successfully!")
}
```

This example demonstrates comprehensive testing patterns for configuration  
management including unit tests, integration tests, error cases, performance  
benchmarks, and mock validators. The patterns ensure configuration handling  
is robust and reliable across different deployment scenarios.  

## Configuration schema and documentation

Schema-driven configuration provides self-documenting configuration with  
automatic validation and IDE support for configuration files.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "reflect"
    "strconv"
    "strings"
    "time"
)

// ConfigSchema defines the structure and validation rules for configuration
type ConfigSchema struct {
    Fields map[string]FieldSchema `json:"fields"`
}

type FieldSchema struct {
    Type        string      `json:"type"`
    Description string      `json:"description"`
    Required    bool        `json:"required"`
    Default     interface{} `json:"default,omitempty"`
    Validation  Validation  `json:"validation,omitempty"`
    Example     interface{} `json:"example,omitempty"`
    Deprecated  bool        `json:"deprecated,omitempty"`
    Since       string      `json:"since,omitempty"`
}

type Validation struct {
    Min         *float64 `json:"min,omitempty"`
    Max         *float64 `json:"max,omitempty"`
    MinLength   *int     `json:"min_length,omitempty"`
    MaxLength   *int     `json:"max_length,omitempty"`
    Pattern     string   `json:"pattern,omitempty"`
    OneOf       []string `json:"one_of,omitempty"`
    MinDuration string   `json:"min_duration,omitempty"`
    MaxDuration string   `json:"max_duration,omitempty"`
}

// ConfigDocumentationGenerator generates documentation from schema
type ConfigDocumentationGenerator struct {
    schema ConfigSchema
}

func NewConfigDocumentationGenerator(schema ConfigSchema) *ConfigDocumentationGenerator {
    return &ConfigDocumentationGenerator{schema: schema}
}

func (cdg *ConfigDocumentationGenerator) GenerateMarkdown() string {
    var md strings.Builder
    
    md.WriteString("# Configuration Reference\n\n")
    md.WriteString("This document describes all available configuration options.\n\n")
    
    // Group fields by category
    categories := make(map[string][]string)
    for fieldName := range cdg.schema.Fields {
        category := extractCategory(fieldName)
        categories[category] = append(categories[category], fieldName)
    }
    
    for category, fields := range categories {
        md.WriteString(fmt.Sprintf("## %s\n\n", strings.Title(category)))
        
        for _, fieldName := range fields {
            field := cdg.schema.Fields[fieldName]
            md.WriteString(cdg.generateFieldDoc(fieldName, field))
        }
        
        md.WriteString("\n")
    }
    
    return md.String()
}

func (cdg *ConfigDocumentationGenerator) generateFieldDoc(name string, field FieldSchema) string {
    var doc strings.Builder
    
    doc.WriteString(fmt.Sprintf("### %s\n\n", name))
    doc.WriteString(fmt.Sprintf("**Type:** %s  \n", field.Type))
    doc.WriteString(fmt.Sprintf("**Required:** %v  \n", field.Required))
    
    if field.Default != nil {
        doc.WriteString(fmt.Sprintf("**Default:** `%v`  \n", field.Default))
    }
    
    if field.Since != "" {
        doc.WriteString(fmt.Sprintf("**Since:** %s  \n", field.Since))
    }
    
    if field.Deprecated {
        doc.WriteString("**Deprecated:** This field is deprecated and will be removed in a future version.  \n")
    }
    
    doc.WriteString(fmt.Sprintf("\n%s\n\n", field.Description))
    
    // Validation rules
    if field.Validation.Min != nil || field.Validation.Max != nil {
        doc.WriteString("**Constraints:**\n")
        if field.Validation.Min != nil {
            doc.WriteString(fmt.Sprintf("- Minimum value: %v\n", *field.Validation.Min))
        }
        if field.Validation.Max != nil {
            doc.WriteString(fmt.Sprintf("- Maximum value: %v\n", *field.Validation.Max))
        }
    }
    
    if field.Validation.MinLength != nil || field.Validation.MaxLength != nil {
        doc.WriteString("**Length constraints:**\n")
        if field.Validation.MinLength != nil {
            doc.WriteString(fmt.Sprintf("- Minimum length: %d\n", *field.Validation.MinLength))
        }
        if field.Validation.MaxLength != nil {
            doc.WriteString(fmt.Sprintf("- Maximum length: %d\n", *field.Validation.MaxLength))
        }
    }
    
    if len(field.Validation.OneOf) > 0 {
        doc.WriteString(fmt.Sprintf("**Allowed values:** %s\n", strings.Join(field.Validation.OneOf, ", ")))
    }
    
    if field.Example != nil {
        doc.WriteString(fmt.Sprintf("\n**Example:**\n```json\n\"%s\": %v\n```\n", name, formatJSON(field.Example)))
    }
    
    doc.WriteString("\n---\n\n")
    
    return doc.String()
}

func extractCategory(fieldName string) string {
    parts := strings.Split(fieldName, ".")
    if len(parts) > 1 {
        return parts[0]
    }
    return "general"
}

func formatJSON(value interface{}) string {
    if str, ok := value.(string); ok {
        return fmt.Sprintf("\"%s\"", str)
    }
    data, _ := json.Marshal(value)
    return string(data)
}

// SchemaValidator validates configuration against schema
type SchemaValidator struct {
    schema ConfigSchema
}

func NewSchemaValidator(schema ConfigSchema) *SchemaValidator {
    return &SchemaValidator{schema: schema}
}

func (sv *SchemaValidator) Validate(config map[string]interface{}) []ValidationError {
    var errors []ValidationError
    
    // Check required fields
    for fieldName, fieldSchema := range sv.schema.Fields {
        if fieldSchema.Required {
            if _, exists := config[fieldName]; !exists {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Message: "required field is missing",
                })
            }
        }
    }
    
    // Validate existing fields
    for fieldName, value := range config {
        if fieldSchema, exists := sv.schema.Fields[fieldName]; exists {
            fieldErrors := sv.validateField(fieldName, value, fieldSchema)
            errors = append(errors, fieldErrors...)
        } else {
            errors = append(errors, ValidationError{
                Field:   fieldName,
                Message: "unknown field",
            })
        }
    }
    
    return errors
}

func (sv *SchemaValidator) validateField(fieldName string, value interface{}, schema FieldSchema) []ValidationError {
    var errors []ValidationError
    
    // Type validation
    if !sv.validateType(value, schema.Type) {
        errors = append(errors, ValidationError{
            Field:   fieldName,
            Message: fmt.Sprintf("expected type %s, got %T", schema.Type, value),
        })
        return errors // Don't continue if type is wrong
    }
    
    // Validate constraints
    switch schema.Type {
    case "string":
        if str, ok := value.(string); ok {
            if schema.Validation.MinLength != nil && len(str) < *schema.Validation.MinLength {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Message: fmt.Sprintf("string too short, minimum length is %d", *schema.Validation.MinLength),
                })
            }
            if schema.Validation.MaxLength != nil && len(str) > *schema.Validation.MaxLength {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Message: fmt.Sprintf("string too long, maximum length is %d", *schema.Validation.MaxLength),
                })
            }
            if len(schema.Validation.OneOf) > 0 {
                valid := false
                for _, allowed := range schema.Validation.OneOf {
                    if str == allowed {
                        valid = true
                        break
                    }
                }
                if !valid {
                    errors = append(errors, ValidationError{
                        Field:   fieldName,
                        Message: fmt.Sprintf("value must be one of: %s", strings.Join(schema.Validation.OneOf, ", ")),
                    })
                }
            }
        }
        
    case "number", "integer":
        if num := sv.getNumberValue(value); num != nil {
            if schema.Validation.Min != nil && *num < *schema.Validation.Min {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Message: fmt.Sprintf("value too small, minimum is %v", *schema.Validation.Min),
                })
            }
            if schema.Validation.Max != nil && *num > *schema.Validation.Max {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Message: fmt.Sprintf("value too large, maximum is %v", *schema.Validation.Max),
                })
            }
        }
        
    case "duration":
        if str, ok := value.(string); ok {
            if duration, err := time.ParseDuration(str); err == nil {
                if schema.Validation.MinDuration != "" {
                    if minDur, err := time.ParseDuration(schema.Validation.MinDuration); err == nil && duration < minDur {
                        errors = append(errors, ValidationError{
                            Field:   fieldName,
                            Message: fmt.Sprintf("duration too short, minimum is %s", schema.Validation.MinDuration),
                        })
                    }
                }
                if schema.Validation.MaxDuration != "" {
                    if maxDur, err := time.ParseDuration(schema.Validation.MaxDuration); err == nil && duration > maxDur {
                        errors = append(errors, ValidationError{
                            Field:   fieldName,
                            Message: fmt.Sprintf("duration too long, maximum is %s", schema.Validation.MaxDuration),
                        })
                    }
                }
            } else {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Message: "invalid duration format",
                })
            }
        }
    }
    
    return errors
}

func (sv *SchemaValidator) validateType(value interface{}, expectedType string) bool {
    switch expectedType {
    case "string":
        _, ok := value.(string)
        return ok
    case "number":
        switch value.(type) {
        case int, int8, int16, int32, int64, uint, uint8, uint16, uint32, uint64, float32, float64:
            return true
        }
        return false
    case "integer":
        switch value.(type) {
        case int, int8, int16, int32, int64, uint, uint8, uint16, uint32, uint64:
            return true
        case float64:
            // JSON numbers come as float64, check if it's actually an integer
            if f, ok := value.(float64); ok {
                return f == float64(int64(f))
            }
        }
        return false
    case "boolean":
        _, ok := value.(bool)
        return ok
    case "duration":
        if str, ok := value.(string); ok {
            _, err := time.ParseDuration(str)
            return err == nil
        }
        return false
    }
    return false
}

func (sv *SchemaValidator) getNumberValue(value interface{}) *float64 {
    switch v := value.(type) {
    case int:
        f := float64(v)
        return &f
    case int8:
        f := float64(v)
        return &f
    case int16:
        f := float64(v)
        return &f
    case int32:
        f := float64(v)
        return &f
    case int64:
        f := float64(v)
        return &f
    case uint:
        f := float64(v)
        return &f
    case uint8:
        f := float64(v)
        return &f
    case uint16:
        f := float64(v)
        return &f
    case uint32:
        f := float64(v)
        return &f
    case uint64:
        f := float64(v)
        return &f
    case float32:
        f := float64(v)
        return &f
    case float64:
        return &v
    }
    return nil
}

type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
}

func (ve ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", ve.Field, ve.Message)
}

func main() {
    fmt.Println("=== Configuration Schema and Documentation ===")
    
    // Define comprehensive configuration schema
    schema := ConfigSchema{
        Fields: map[string]FieldSchema{
            "app.name": {
                Type:        "string",
                Description: "The name of the application. Used for logging and monitoring.",
                Required:    true,
                Example:     "my-awesome-app",
                Since:       "1.0.0",
                Validation: Validation{
                    MinLength: intPtr(1),
                    MaxLength: intPtr(50),
                },
            },
            "app.version": {
                Type:        "string",
                Description: "The version of the application in semantic versioning format.",
                Required:    false,
                Default:     "1.0.0",
                Example:     "2.1.3",
                Since:       "1.0.0",
            },
            "server.host": {
                Type:        "string",
                Description: "The host address the server will bind to. Use 0.0.0.0 to bind to all interfaces.",
                Required:    false,
                Default:     "localhost",
                Example:     "0.0.0.0",
                Since:       "1.0.0",
            },
            "server.port": {
                Type:        "integer",
                Description: "The port number the server will listen on.",
                Required:    false,
                Default:     8080,
                Example:     3000,
                Since:       "1.0.0",
                Validation: Validation{
                    Min: float64Ptr(1),
                    Max: float64Ptr(65535),
                },
            },
            "server.timeout": {
                Type:        "duration",
                Description: "The maximum duration for server requests before timing out.",
                Required:    false,
                Default:     "30s",
                Example:     "1m30s",
                Since:       "1.1.0",
                Validation: Validation{
                    MinDuration: "1s",
                    MaxDuration: "10m",
                },
            },
            "database.driver": {
                Type:        "string",
                Description: "The database driver to use for connections.",
                Required:    true,
                Example:     "postgres",
                Since:       "1.0.0",
                Validation: Validation{
                    OneOf: []string{"postgres", "mysql", "sqlite"},
                },
            },
            "database.url": {
                Type:        "string",
                Description: "The database connection URL including credentials.",
                Required:    true,
                Example:     "postgres://user:pass@localhost:5432/mydb",
                Since:       "1.0.0",
                Validation: Validation{
                    MinLength: intPtr(10),
                },
            },
            "logging.level": {
                Type:        "string",
                Description: "The minimum log level to output.",
                Required:    false,
                Default:     "info",
                Example:     "debug",
                Since:       "1.0.0",
                Validation: Validation{
                    OneOf: []string{"debug", "info", "warn", "error"},
                },
            },
            "logging.format": {
                Type:        "string",
                Description: "The format for log output.",
                Required:    false,
                Default:     "text",
                Example:     "json",
                Since:       "1.2.0",
                Validation: Validation{
                    OneOf: []string{"text", "json"},
                },
            },
            "features.metrics": {
                Type:        "boolean",
                Description: "Enable metrics collection and exposition.",
                Required:    false,
                Default:     false,
                Example:     true,
                Since:       "1.1.0",
            },
            "cache.type": {
                Type:        "string",
                Description: "DEPRECATED: Use cache.provider instead. The type of cache to use.",
                Required:    false,
                Default:     "memory",
                Example:     "redis",
                Since:       "1.0.0",
                Deprecated:  true,
                Validation: Validation{
                    OneOf: []string{"memory", "redis"},
                },
            },
            "cache.provider": {
                Type:        "string",
                Description: "The cache provider to use for caching operations.",
                Required:    false,
                Default:     "memory",
                Example:     "redis",
                Since:       "1.3.0",
                Validation: Validation{
                    OneOf: []string{"memory", "redis", "memcached"},
                },
            },
        },
    }
    
    // Generate documentation
    docGen := NewConfigDocumentationGenerator(schema)
    markdown := docGen.GenerateMarkdown()
    
    // Save documentation to file
    docFile := "/tmp/config_reference.md"
    if err := os.WriteFile(docFile, []byte(markdown), 0644); err != nil {
        fmt.Printf("Error writing documentation: %v\n", err)
        return
    }
    
    fmt.Printf("Generated configuration documentation: %s\n", docFile)
    fmt.Println("\nFirst few lines of documentation:")
    lines := strings.Split(markdown, "\n")
    for i, line := range lines[:min(20, len(lines))] {
        fmt.Printf("%2d: %s\n", i+1, line)
    }
    
    // Test configuration validation
    fmt.Println("\n=== Configuration Validation ===")
    
    validator := NewSchemaValidator(schema)
    
    // Test valid configuration
    validConfig := map[string]interface{}{
        "app.name":         "TestApp",
        "app.version":      "2.0.0",
        "server.host":      "0.0.0.0",
        "server.port":      3000,
        "server.timeout":   "45s",
        "database.driver":  "postgres",
        "database.url":     "postgres://user:pass@localhost:5432/testdb",
        "logging.level":    "debug",
        "logging.format":   "json",
        "features.metrics": true,
        "cache.provider":   "redis",
    }
    
    errors := validator.Validate(validConfig)
    if len(errors) == 0 {
        fmt.Println("✓ Valid configuration passed validation")
    } else {
        fmt.Printf("✗ Unexpected validation errors: %v\n", errors)
    }
    
    // Test invalid configuration
    invalidConfig := map[string]interface{}{
        // Missing required app.name
        "server.port":      70000, // Invalid port
        "server.timeout":   "30ms", // Too short
        "database.driver":  "oracle", // Invalid driver
        "database.url":     "short", // Too short
        "logging.level":    "verbose", // Invalid level
        "features.metrics": "yes", // Wrong type
        "unknown.field":    "value", // Unknown field
    }
    
    errors = validator.Validate(invalidConfig)
    fmt.Printf("\nValidation errors for invalid config:\n")
    for _, err := range errors {
        fmt.Printf("  - %s\n", err.Error())
    }
    
    // Test deprecated field warning
    configWithDeprecated := map[string]interface{}{
        "app.name":       "TestApp",
        "database.driver": "postgres",
        "database.url":   "postgres://user:pass@localhost:5432/testdb",
        "cache.type":     "redis", // Deprecated field
    }
    
    errors = validator.Validate(configWithDeprecated)
    if len(errors) == 0 {
        fmt.Println("\n✓ Configuration with deprecated field is valid (consider migrating)")
        if fieldSchema, exists := schema.Fields["cache.type"]; exists && fieldSchema.Deprecated {
            fmt.Println("  Warning: cache.type is deprecated, use cache.provider instead")
        }
    }
    
    fmt.Println("\n=== Schema Validation Complete ===")
}

func intPtr(i int) *int {
    return &i
}

func float64Ptr(f float64) *float64 {
    return &f
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

This example demonstrates comprehensive configuration schema definition with  
automatic documentation generation and validation. The pattern provides  
self-documenting configuration with IDE support and prevents configuration  
errors through schema-based validation and clear constraint definitions.  

## Configuration migration patterns

Configuration migration enables smooth transitions between configuration  
versions while maintaining backward compatibility and data integrity.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "strconv"
    "time"
)

// ConfigMigrator handles configuration version migrations
type ConfigMigrator struct {
    migrations []Migration
}

type Migration interface {
    Version() int
    Description() string
    Up(config map[string]interface{}) error
    Down(config map[string]interface{}) error
}

// V1ToV2Migration migrates from version 1 to version 2
type V1ToV2Migration struct{}

func (m *V1ToV2Migration) Version() int {
    return 2
}

func (m *V1ToV2Migration) Description() string {
    return "Restructure server configuration and add new fields"
}

func (m *V1ToV2Migration) Up(config map[string]interface{}) error {
    fmt.Println("Migrating to version 2: restructuring server configuration")
    
    // Migrate flat server config to nested structure
    if host, exists := config["host"]; exists {
        delete(config, "host")
        if _, hasServer := config["server"]; !hasServer {
            config["server"] = make(map[string]interface{})
        }
        config["server"].(map[string]interface{})["host"] = host
    }
    
    if port, exists := config["port"]; exists {
        delete(config, "port")
        if _, hasServer := config["server"]; !hasServer {
            config["server"] = make(map[string]interface{})
        }
        config["server"].(map[string]interface{})["port"] = port
    }
    
    // Add default timeout if server section exists
    if server, hasServer := config["server"]; hasServer {
        serverMap := server.(map[string]interface{})
        if _, hasTimeout := serverMap["timeout"]; !hasTimeout {
            serverMap["timeout"] = "30s"
        }
    }
    
    // Add new logging section with defaults
    if _, hasLogging := config["logging"]; !hasLogging {
        config["logging"] = map[string]interface{}{
            "level":  "info",
            "format": "text",
        }
    }
    
    return nil
}

func (m *V1ToV2Migration) Down(config map[string]interface{}) error {
    fmt.Println("Rolling back to version 1: flattening server configuration")
    
    // Move server config back to top level
    if server, exists := config["server"]; exists {
        serverMap := server.(map[string]interface{})
        if host, hasHost := serverMap["host"]; hasHost {
            config["host"] = host
        }
        if port, hasPort := serverMap["port"]; hasPort {
            config["port"] = port
        }
        delete(config, "server")
    }
    
    // Remove logging section (didn't exist in v1)
    delete(config, "logging")
    
    return nil
}

// V2ToV3Migration migrates from version 2 to version 3
type V2ToV3Migration struct{}

func (m *V2ToV3Migration) Version() int {
    return 3
}

func (m *V2ToV3Migration) Description() string {
    return "Add database configuration and feature flags"
}

func (m *V2ToV3Migration) Up(config map[string]interface{}) error {
    fmt.Println("Migrating to version 3: adding database and features")
    
    // Add database configuration with defaults
    if _, hasDatabase := config["database"]; !hasDatabase {
        config["database"] = map[string]interface{}{
            "driver": "postgres",
            "host":   "localhost",
            "port":   5432,
            "name":   "app",
        }
    }
    
    // Add feature flags section
    if _, hasFeatures := config["features"]; !hasFeatures {
        config["features"] = map[string]interface{}{
            "metrics": false,
            "tracing": false,
        }
    }
    
    // Migrate old debug flag to features.debug if it exists
    if debug, hasDebug := config["debug"]; hasDebug {
        features := config["features"].(map[string]interface{})
        features["debug"] = debug
        delete(config, "debug")
    }
    
    return nil
}

func (m *V2ToV3Migration) Down(config map[string]interface{}) error {
    fmt.Println("Rolling back to version 2: removing database and features")
    
    // Restore debug flag if it exists in features
    if features, hasFeatures := config["features"]; hasFeatures {
        featuresMap := features.(map[string]interface{})
        if debug, hasDebug := featuresMap["debug"]; hasDebug {
            config["debug"] = debug
        }
    }
    
    // Remove new sections
    delete(config, "database")
    delete(config, "features")
    
    return nil
}

func NewConfigMigrator() *ConfigMigrator {
    return &ConfigMigrator{
        migrations: []Migration{
            &V1ToV2Migration{},
            &V2ToV3Migration{},
        },
    }
}

func (cm *ConfigMigrator) GetCurrentVersion(config map[string]interface{}) int {
    if version, exists := config["_version"]; exists {
        switch v := version.(type) {
        case int:
            return v
        case float64:
            return int(v)
        case string:
            if parsed, err := strconv.Atoi(v); err == nil {
                return parsed
            }
        }
    }
    return 1 // Default to version 1 if no version field
}

func (cm *ConfigMigrator) GetLatestVersion() int {
    maxVersion := 1
    for _, migration := range cm.migrations {
        if migration.Version() > maxVersion {
            maxVersion = migration.Version()
        }
    }
    return maxVersion
}

func (cm *ConfigMigrator) MigrateToLatest(config map[string]interface{}) error {
    currentVersion := cm.GetCurrentVersion(config)
    latestVersion := cm.GetLatestVersion()
    
    fmt.Printf("Current configuration version: %d\n", currentVersion)
    fmt.Printf("Latest configuration version: %d\n", latestVersion)
    
    if currentVersion == latestVersion {
        fmt.Println("Configuration is already at the latest version")
        return nil
    }
    
    if currentVersion > latestVersion {
        return fmt.Errorf("configuration version %d is newer than supported version %d", 
            currentVersion, latestVersion)
    }
    
    // Apply migrations in order
    for targetVersion := currentVersion + 1; targetVersion <= latestVersion; targetVersion++ {
        migration := cm.findMigration(targetVersion)
        if migration == nil {
            return fmt.Errorf("no migration found for version %d", targetVersion)
        }
        
        fmt.Printf("Applying migration to version %d: %s\n", 
            targetVersion, migration.Description())
        
        if err := migration.Up(config); err != nil {
            return fmt.Errorf("migration to version %d failed: %w", targetVersion, err)
        }
        
        // Update version
        config["_version"] = targetVersion
        fmt.Printf("Successfully migrated to version %d\n", targetVersion)
    }
    
    return nil
}

func (cm *ConfigMigrator) MigrateToVersion(config map[string]interface{}, targetVersion int) error {
    currentVersion := cm.GetCurrentVersion(config)
    
    if currentVersion == targetVersion {
        fmt.Printf("Configuration is already at version %d\n", targetVersion)
        return nil
    }
    
    if currentVersion < targetVersion {
        // Migrate up
        for version := currentVersion + 1; version <= targetVersion; version++ {
            migration := cm.findMigration(version)
            if migration == nil {
                return fmt.Errorf("no migration found for version %d", version)
            }
            
            if err := migration.Up(config); err != nil {
                return fmt.Errorf("migration to version %d failed: %w", version, err)
            }
            config["_version"] = version
        }
    } else {
        // Migrate down
        for version := currentVersion; version > targetVersion; version-- {
            migration := cm.findMigration(version)
            if migration == nil {
                return fmt.Errorf("no migration found for version %d", version)
            }
            
            if err := migration.Down(config); err != nil {
                return fmt.Errorf("rollback from version %d failed: %w", version, err)
            }
            config["_version"] = version - 1
        }
    }
    
    return nil
}

func (cm *ConfigMigrator) findMigration(version int) Migration {
    for _, migration := range cm.migrations {
        if migration.Version() == version {
            return migration
        }
    }
    return nil
}

func (cm *ConfigMigrator) CreateBackup(config map[string]interface{}, filename string) error {
    // Add backup metadata
    backup := map[string]interface{}{
        "backup_timestamp": time.Now().Unix(),
        "backup_version":   cm.GetCurrentVersion(config),
        "config":          config,
    }
    
    data, err := json.MarshalIndent(backup, "", "  ")
    if err != nil {
        return fmt.Errorf("failed to marshal backup: %w", err)
    }
    
    if err := os.WriteFile(filename, data, 0644); err != nil {
        return fmt.Errorf("failed to write backup file: %w", err)
    }
    
    fmt.Printf("Configuration backup created: %s\n", filename)
    return nil
}

func (cm *ConfigMigrator) RestoreFromBackup(filename string) (map[string]interface{}, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read backup file: %w", err)
    }
    
    var backup map[string]interface{}
    if err := json.Unmarshal(data, &backup); err != nil {
        return nil, fmt.Errorf("failed to parse backup file: %w", err)
    }
    
    config, ok := backup["config"].(map[string]interface{})
    if !ok {
        return nil, fmt.Errorf("invalid backup file format")
    }
    
    if timestamp, hasTimestamp := backup["backup_timestamp"]; hasTimestamp {
        if ts, ok := timestamp.(float64); ok {
            backupTime := time.Unix(int64(ts), 0)
            fmt.Printf("Restoring backup from: %s\n", backupTime.Format(time.RFC3339))
        }
    }
    
    return config, nil
}

func printConfig(config map[string]interface{}, title string) {
    fmt.Printf("\n=== %s ===\n", title)
    data, _ := json.MarshalIndent(config, "", "  ")
    fmt.Println(string(data))
}

func main() {
    fmt.Println("=== Configuration Migration Example ===")
    
    // Create version 1 configuration
    v1Config := map[string]interface{}{
        "_version": 1,
        "app_name": "MigrationDemo",
        "host":     "localhost",
        "port":     8080,
        "debug":    true,
    }
    
    printConfig(v1Config, "Version 1 Configuration")
    
    migrator := NewConfigMigrator()
    
    // Create backup before migration
    backupFile := "/tmp/config_backup.json"
    if err := migrator.CreateBackup(v1Config, backupFile); err != nil {
        fmt.Printf("Error creating backup: %v\n", err)
        return
    }
    
    // Migrate to latest version
    fmt.Println("\n=== Starting Migration Process ===")
    if err := migrator.MigrateToLatest(v1Config); err != nil {
        fmt.Printf("Migration failed: %v\n", err)
        return
    }
    
    printConfig(v1Config, "Latest Version Configuration")
    
    // Demonstrate rollback to version 2
    fmt.Println("\n=== Demonstrating Rollback ===")
    if err := migrator.MigrateToVersion(v1Config, 2); err != nil {
        fmt.Printf("Rollback failed: %v\n", err)
        return
    }
    
    printConfig(v1Config, "Rolled Back to Version 2")
    
    // Migrate back to latest
    fmt.Println("\n=== Migrating Back to Latest ===")
    if err := migrator.MigrateToLatest(v1Config); err != nil {
        fmt.Printf("Migration failed: %v\n", err)
        return
    }
    
    printConfig(v1Config, "Back to Latest Version")
    
    // Demonstrate backup restoration
    fmt.Println("\n=== Demonstrating Backup Restoration ===")
    restoredConfig, err := migrator.RestoreFromBackup(backupFile)
    if err != nil {
        fmt.Printf("Backup restoration failed: %v\n", err)
        return
    }
    
    printConfig(restoredConfig, "Restored from Backup")
    
    // Clean up
    os.Remove(backupFile)
    
    fmt.Println("\n=== Migration Demo Complete ===")
}
```

This example demonstrates comprehensive configuration migration patterns  
including versioning, backup/restore, rollback capabilities, and data  
transformation. The pattern ensures safe configuration evolution while  
maintaining backward compatibility and providing recovery options.  

## Configuration templating with variables

Configuration templating enables dynamic configuration generation with  
variable substitution and conditional logic for different environments.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "regexp"
    "strconv"
    "strings"
    "text/template"
    "time"
)

// ConfigTemplateEngine handles configuration template processing
type ConfigTemplateEngine struct {
    variables map[string]interface{}
    functions template.FuncMap
}

func NewConfigTemplateEngine() *ConfigTemplateEngine {
    return &ConfigTemplateEngine{
        variables: make(map[string]interface{}),
        functions: template.FuncMap{
            "env":           getEnvWithDefault,
            "envRequired":   getEnvRequired,
            "default":       defaultValue,
            "upper":         strings.ToUpper,
            "lower":         strings.ToLower,
            "replace":       strings.ReplaceAll,
            "contains":      strings.Contains,
            "hasPrefix":     strings.HasPrefix,
            "hasSuffix":     strings.HasSuffix,
            "split":         strings.Split,
            "join":          strings.Join,
            "now":           time.Now,
            "formatTime":    formatTime,
            "add":           add,
            "sub":           sub,
            "mul":           mul,
            "div":           div,
            "mod":           mod,
            "toString":      toString,
            "toInt":         toInt,
            "toBool":        toBool,
            "toFloat":       toFloat,
            "json":          toJSON,
            "indent":        indent,
        },
    }
}

func (cte *ConfigTemplateEngine) SetVariable(key string, value interface{}) {
    cte.variables[key] = value
}

func (cte *ConfigTemplateEngine) SetVariables(vars map[string]interface{}) {
    for key, value := range vars {
        cte.variables[key] = value
    }
}

func (cte *ConfigTemplateEngine) ProcessTemplate(templateContent string) (string, error) {
    tmpl, err := template.New("config").Funcs(cte.functions).Parse(templateContent)
    if err != nil {
        return "", fmt.Errorf("failed to parse template: %w", err)
    }
    
    var result strings.Builder
    if err := tmpl.Execute(&result, cte.variables); err != nil {
        return "", fmt.Errorf("failed to execute template: %w", err)
    }
    
    return result.String(), nil
}

func (cte *ConfigTemplateEngine) ProcessTemplateFile(templatePath, outputPath string) error {
    templateContent, err := os.ReadFile(templatePath)
    if err != nil {
        return fmt.Errorf("failed to read template file: %w", err)
    }
    
    result, err := cte.ProcessTemplate(string(templateContent))
    if err != nil {
        return err
    }
    
    if err := os.WriteFile(outputPath, []byte(result), 0644); err != nil {
        return fmt.Errorf("failed to write output file: %w", err)
    }
    
    return nil
}

// SimpleVariableSubstitutor handles simple ${VAR} style substitution
type SimpleVariableSubstitutor struct {
    variables map[string]string
}

func NewSimpleVariableSubstitutor() *SimpleVariableSubstitutor {
    return &SimpleVariableSubstitutor{
        variables: make(map[string]string),
    }
}

func (svs *SimpleVariableSubstitutor) SetVariable(key, value string) {
    svs.variables[key] = value
}

func (svs *SimpleVariableSubstitutor) LoadFromEnvironment(prefix string) {
    for _, env := range os.Environ() {
        if strings.HasPrefix(env, prefix) {
            parts := strings.SplitN(env, "=", 2)
            if len(parts) == 2 {
                key := strings.TrimPrefix(parts[0], prefix)
                svs.variables[key] = parts[1]
            }
        }
    }
}

func (svs *SimpleVariableSubstitutor) Substitute(content string) string {
    // Pattern for ${VAR} or ${VAR:default}
    pattern := regexp.MustCompile(`\$\{([^}:]+)(?::([^}]*))?\}`)
    
    return pattern.ReplaceAllStringFunc(content, func(match string) string {
        // Extract variable name and default value
        submatches := pattern.FindStringSubmatch(match)
        if len(submatches) < 2 {
            return match
        }
        
        varName := submatches[1]
        defaultValue := ""
        if len(submatches) > 2 {
            defaultValue = submatches[2]
        }
        
        // Look for variable value
        if value, exists := svs.variables[varName]; exists {
            return value
        }
        
        // Check environment
        if value := os.Getenv(varName); value != "" {
            return value
        }
        
        // Return default value
        return defaultValue
    })
}

// Template function implementations
func getEnvWithDefault(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func getEnvRequired(key string) (string, error) {
    value := os.Getenv(key)
    if value == "" {
        return "", fmt.Errorf("required environment variable %s is not set", key)
    }
    return value, nil
}

func defaultValue(value, defaultVal interface{}) interface{} {
    if value == nil || value == "" {
        return defaultVal
    }
    return value
}

func formatTime(format string, t time.Time) string {
    return t.Format(format)
}

func add(a, b int) int { return a + b }
func sub(a, b int) int { return a - b }
func mul(a, b int) int { return a * b }
func div(a, b int) int { return a / b }
func mod(a, b int) int { return a % b }

func toString(v interface{}) string {
    return fmt.Sprintf("%v", v)
}

func toInt(v interface{}) (int, error) {
    switch val := v.(type) {
    case int:
        return val, nil
    case float64:
        return int(val), nil
    case string:
        return strconv.Atoi(val)
    default:
        return 0, fmt.Errorf("cannot convert %T to int", v)
    }
}

func toBool(v interface{}) (bool, error) {
    switch val := v.(type) {
    case bool:
        return val, nil
    case string:
        return strconv.ParseBool(val)
    default:
        return false, fmt.Errorf("cannot convert %T to bool", v)
    }
}

func toFloat(v interface{}) (float64, error) {
    switch val := v.(type) {
    case float64:
        return val, nil
    case int:
        return float64(val), nil
    case string:
        return strconv.ParseFloat(val, 64)
    default:
        return 0, fmt.Errorf("cannot convert %T to float64", v)
    }
}

func toJSON(v interface{}) (string, error) {
    data, err := json.Marshal(v)
    if err != nil {
        return "", err
    }
    return string(data), nil
}

func indent(spaces int, text string) string {
    indentation := strings.Repeat(" ", spaces)
    lines := strings.Split(text, "\n")
    for i, line := range lines {
        if line != "" {
            lines[i] = indentation + line
        }
    }
    return strings.Join(lines, "\n")
}

func main() {
    fmt.Println("=== Configuration Templating Example ===")
    
    // Set environment variables for demonstration
    os.Setenv("ENVIRONMENT", "production")
    os.Setenv("DB_PASSWORD", "secret-db-password")
    os.Setenv("API_KEY", "sk-1234567890abcdef")
    os.Setenv("REPLICA_COUNT", "3")
    os.Setenv("ENABLE_METRICS", "true")
    
    // Example 1: Go template engine
    fmt.Println("\n=== Go Template Engine ===")
    
    templateContent := `{
  "app": {
    "name": "{{.AppName}}",
    "version": "{{.Version}}",
    "environment": "{{env "ENVIRONMENT" "development"}}",
    "debug": {{if eq (env "ENVIRONMENT") "development"}}true{{else}}false{{end}},
    "instance_id": "{{.AppName}}-{{env "HOSTNAME" "unknown"}}-{{now.Unix}}"
  },
  "server": {
    "host": "{{default .ServerHost "0.0.0.0"}}",
    "port": {{.ServerPort}},
    "workers": {{mul (toInt (env "REPLICA_COUNT" "1")) 2}},
    "timeout": "{{.Timeout}}"
  },
  "database": {
    "driver": "postgres",
    "host": "{{.DatabaseHost}}",
    "port": 5432,
    "name": "{{.AppName | lower}}_{{env "ENVIRONMENT"}}",
    "user": "{{.AppName | lower}}_user",
    "password": "{{envRequired "DB_PASSWORD"}}"
  },
  "external": {
    "api_key": "{{env "API_KEY"}}",
    "endpoints": [
      {{range $i, $endpoint := .Endpoints}}{{if $i}},{{end}}
      "{{$endpoint}}"
      {{end}}
    ]
  },
  "features": {
    "metrics": {{env "ENABLE_METRICS" "false" | toBool}},
    "tracing": {{if eq (env "ENVIRONMENT") "production"}}true{{else}}false{{end}},
    "caching": true
  },
  "generated": {
    "timestamp": "{{now.Format "2006-01-02T15:04:05Z07:00"}}",
    "config_version": "{{.ConfigVersion}}"
  }
}`
    
    engine := NewConfigTemplateEngine()
    engine.SetVariables(map[string]interface{}{
        "AppName":      "TemplateApp",
        "Version":      "2.1.0",
        "ServerHost":   "api.example.com",
        "ServerPort":   8080,
        "Timeout":      "30s",
        "DatabaseHost": "db.example.com",
        "Endpoints": []string{
            "https://api1.example.com",
            "https://api2.example.com",
            "https://api3.example.com",
        },
        "ConfigVersion": "1.0",
    })
    
    result, err := engine.ProcessTemplate(templateContent)
    if err != nil {
        fmt.Printf("Template processing failed: %v\n", err)
        return
    }
    
    fmt.Println("Generated configuration:")
    fmt.Println(result)
    
    // Example 2: Simple variable substitution
    fmt.Println("\n=== Simple Variable Substitution ===")
    
    simpleTemplate := `{
  "service": {
    "name": "${SERVICE_NAME:myapp}",
    "port": ${PORT:8080},
    "environment": "${ENVIRONMENT:development}"
  },
  "database": {
    "url": "postgres://${DB_USER:postgres}:${DB_PASSWORD}@${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:myapp}",
    "max_connections": ${DB_MAX_CONNECTIONS:10}
  },
  "cache": {
    "type": "${CACHE_TYPE:memory}",
    "url": "${CACHE_URL:}"
  },
  "security": {
    "jwt_secret": "${JWT_SECRET}",
    "allowed_origins": "${ALLOWED_ORIGINS:*}"
  }
}`
    
    substitutor := NewSimpleVariableSubstitutor()
    substitutor.SetVariable("SERVICE_NAME", "SimpleApp")
    substitutor.SetVariable("PORT", "9000")
    substitutor.SetVariable("DB_USER", "app_user")
    substitutor.SetVariable("DB_HOST", "prod-db.example.com")
    substitutor.SetVariable("DB_NAME", "production")
    substitutor.SetVariable("JWT_SECRET", "super-secret-jwt-key")
    substitutor.LoadFromEnvironment("CONFIG_")
    
    simpleResult := substitutor.Substitute(simpleTemplate)
    
    fmt.Println("Simple substitution result:")
    fmt.Println(simpleResult)
    
    // Example 3: Environment-specific templates
    fmt.Println("\n=== Environment-Specific Templates ===")
    
    environments := []string{"development", "staging", "production"}
    
    for _, env := range environments {
        os.Setenv("ENVIRONMENT", env)
        
        envTemplate := `{
  "environment": "{{env "ENVIRONMENT"}}",
  "log_level": "{{if eq (env "ENVIRONMENT") "development"}}debug{{else if eq (env "ENVIRONMENT") "staging"}}info{{else}}warn{{end}}",
  "database": {
    "pool_size": {{if eq (env "ENVIRONMENT") "production"}}20{{else if eq (env "ENVIRONMENT") "staging"}}10{{else}}5{{end}},
    "ssl_mode": "{{if eq (env "ENVIRONMENT") "production"}}require{{else}}disable{{end}}"
  },
  "features": {
    "debug_mode": {{if eq (env "ENVIRONMENT") "development"}}true{{else}}false{{end}},
    "profiling": {{if ne (env "ENVIRONMENT") "production"}}true{{else}}false{{end}}
  }
}`
        
        envResult, err := engine.ProcessTemplate(envTemplate)
        if err != nil {
            fmt.Printf("Error processing %s template: %v\n", env, err)
            continue
        }
        
        fmt.Printf("\n%s configuration:\n", strings.Title(env))
        fmt.Println(envResult)
    }
    
    // Example 4: Save template to file
    fmt.Println("\n=== File Template Processing ===")
    
    templateFile := "/tmp/config_template.json"
    outputFile := "/tmp/generated_config.json"
    
    if err := os.WriteFile(templateFile, []byte(templateContent), 0644); err != nil {
        fmt.Printf("Error writing template file: %v\n", err)
        return
    }
    
    if err := engine.ProcessTemplateFile(templateFile, outputFile); err != nil {
        fmt.Printf("Error processing template file: %v\n", err)
        return
    }
    
    fmt.Printf("Template processed successfully!\n")
    fmt.Printf("Template file: %s\n", templateFile)
    fmt.Printf("Output file: %s\n", outputFile)
    
    // Verify generated file
    if generatedContent, err := os.ReadFile(outputFile); err == nil {
        var config map[string]interface{}
        if err := json.Unmarshal(generatedContent, &config); err == nil {
            fmt.Println("Generated configuration is valid JSON")
        } else {
            fmt.Printf("Generated configuration is invalid JSON: %v\n", err)
        }
    }
    
    // Clean up
    os.Remove(templateFile)
    os.Remove(outputFile)
    
    fmt.Println("\n=== Templating Demo Complete ===")
}
```

This example demonstrates comprehensive configuration templating including  
Go templates with custom functions, simple variable substitution, environment-  
specific configuration generation, and file-based template processing. The  
pattern enables dynamic configuration generation while maintaining type safety  
and validation capabilities.  

## Configuration observability and monitoring

Configuration observability provides visibility into configuration usage,  
changes, and health to support operational debugging and compliance.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "sync"
    "time"
)

// ConfigObserver defines the interface for configuration observation
type ConfigObserver interface {
    OnConfigLoad(source string, config map[string]interface{})
    OnConfigChange(field string, oldValue, newValue interface{})
    OnConfigError(source string, err error)
    OnConfigAccess(field string, value interface{})
}

// ConfigMetrics tracks configuration usage and health metrics
type ConfigMetrics struct {
    LoadCount       int64                    `json:"load_count"`
    ChangeCount     int64                    `json:"change_count"`
    ErrorCount      int64                    `json:"error_count"`
    AccessCount     map[string]int64         `json:"access_count"`
    LastLoaded      time.Time                `json:"last_loaded"`
    LastChanged     time.Time                `json:"last_changed"`
    LastError       time.Time                `json:"last_error"`
    FieldUsage      map[string]time.Time     `json:"field_usage"`
    ConfigSources   map[string]int64         `json:"config_sources"`
    HealthStatus    string                   `json:"health_status"`
    ValidationErrors []string                `json:"validation_errors"`
    mutex           sync.RWMutex
}

func NewConfigMetrics() *ConfigMetrics {
    return &ConfigMetrics{
        AccessCount:   make(map[string]int64),
        FieldUsage:    make(map[string]time.Time),
        ConfigSources: make(map[string]int64),
        HealthStatus:  "healthy",
        ValidationErrors: make([]string, 0),
    }
}

func (cm *ConfigMetrics) RecordLoad(source string) {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    
    cm.LoadCount++
    cm.LastLoaded = time.Now()
    cm.ConfigSources[source]++
    
    if cm.ErrorCount == 0 {
        cm.HealthStatus = "healthy"
    }
}

func (cm *ConfigMetrics) RecordChange(field string) {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    
    cm.ChangeCount++
    cm.LastChanged = time.Now()
    cm.FieldUsage[field] = time.Now()
}

func (cm *ConfigMetrics) RecordError(source string, err error) {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    
    cm.ErrorCount++
    cm.LastError = time.Now()
    cm.HealthStatus = "degraded"
    cm.ValidationErrors = append(cm.ValidationErrors, fmt.Sprintf("%s: %v", source, err))
    
    // Keep only last 10 errors
    if len(cm.ValidationErrors) > 10 {
        cm.ValidationErrors = cm.ValidationErrors[len(cm.ValidationErrors)-10:]
    }
}

func (cm *ConfigMetrics) RecordAccess(field string) {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    
    cm.AccessCount[field]++
    cm.FieldUsage[field] = time.Now()
}

func (cm *ConfigMetrics) GetSnapshot() ConfigMetrics {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    
    // Create a deep copy
    snapshot := ConfigMetrics{
        LoadCount:        cm.LoadCount,
        ChangeCount:      cm.ChangeCount,
        ErrorCount:       cm.ErrorCount,
        LastLoaded:       cm.LastLoaded,
        LastChanged:      cm.LastChanged,
        LastError:        cm.LastError,
        HealthStatus:     cm.HealthStatus,
        AccessCount:      make(map[string]int64),
        FieldUsage:       make(map[string]time.Time),
        ConfigSources:    make(map[string]int64),
        ValidationErrors: make([]string, len(cm.ValidationErrors)),
    }
    
    for k, v := range cm.AccessCount {
        snapshot.AccessCount[k] = v
    }
    for k, v := range cm.FieldUsage {
        snapshot.FieldUsage[k] = v
    }
    for k, v := range cm.ConfigSources {
        snapshot.ConfigSources[k] = v
    }
    copy(snapshot.ValidationErrors, cm.ValidationErrors)
    
    return snapshot
}

// AuditLogger logs configuration changes for compliance and debugging
type AuditLogger struct {
    logFile   *os.File
    formatter AuditFormatter
}

type AuditFormatter interface {
    Format(event AuditEvent) string
}

type AuditEvent struct {
    Timestamp time.Time   `json:"timestamp"`
    Type      string      `json:"type"`
    Source    string      `json:"source"`
    Field     string      `json:"field,omitempty"`
    OldValue  interface{} `json:"old_value,omitempty"`
    NewValue  interface{} `json:"new_value,omitempty"`
    Error     string      `json:"error,omitempty"`
    User      string      `json:"user,omitempty"`
    Context   map[string]interface{} `json:"context,omitempty"`
}

type JSONAuditFormatter struct{}

func (f *JSONAuditFormatter) Format(event AuditEvent) string {
    data, _ := json.Marshal(event)
    return string(data)
}

func NewAuditLogger(filename string) (*AuditLogger, error) {
    file, err := os.OpenFile(filename, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        return nil, fmt.Errorf("failed to open audit log file: %w", err)
    }
    
    return &AuditLogger{
        logFile:   file,
        formatter: &JSONAuditFormatter{},
    }, nil
}

func (al *AuditLogger) LogLoad(source string, config map[string]interface{}) {
    event := AuditEvent{
        Timestamp: time.Now(),
        Type:      "config_load",
        Source:    source,
        Context: map[string]interface{}{
            "field_count": len(config),
        },
    }
    al.writeEvent(event)
}

func (al *AuditLogger) LogChange(field string, oldValue, newValue interface{}) {
    event := AuditEvent{
        Timestamp: time.Now(),
        Type:      "config_change",
        Field:     field,
        OldValue:  al.sanitizeValue(oldValue),
        NewValue:  al.sanitizeValue(newValue),
    }
    al.writeEvent(event)
}

func (al *AuditLogger) LogError(source string, err error) {
    event := AuditEvent{
        Timestamp: time.Now(),
        Type:      "config_error",
        Source:    source,
        Error:     err.Error(),
    }
    al.writeEvent(event)
}

func (al *AuditLogger) LogAccess(field string, value interface{}) {
    event := AuditEvent{
        Timestamp: time.Now(),
        Type:      "config_access",
        Field:     field,
        NewValue:  al.sanitizeValue(value),
    }
    al.writeEvent(event)
}

func (al *AuditLogger) sanitizeValue(value interface{}) interface{} {
    if str, ok := value.(string); ok {
        // Mask potential secrets
        if len(str) > 8 && containsSecretKeywords(str) {
            return "[REDACTED]"
        }
    }
    return value
}

func containsSecretKeywords(str string) bool {
    keywords := []string{"password", "secret", "key", "token", "credential"}
    lowerStr := fmt.Sprintf("%v", str)
    for _, keyword := range keywords {
        if len(lowerStr) > 10 && len(lowerStr) < 200 {
            return true // Likely a secret based on length
        }
    }
    return false
}

func (al *AuditLogger) writeEvent(event AuditEvent) {
    logLine := al.formatter.Format(event) + "\n"
    al.logFile.WriteString(logLine)
    al.logFile.Sync()
}

func (al *AuditLogger) Close() error {
    return al.logFile.Close()
}

// ObservableConfig wraps configuration with observability features
type ObservableConfig struct {
    config    map[string]interface{}
    metrics   *ConfigMetrics
    auditor   *AuditLogger
    observers []ConfigObserver
    mutex     sync.RWMutex
}

func NewObservableConfig(auditLogPath string) (*ObservableConfig, error) {
    auditor, err := NewAuditLogger(auditLogPath)
    if err != nil {
        return nil, err
    }
    
    return &ObservableConfig{
        config:    make(map[string]interface{}),
        metrics:   NewConfigMetrics(),
        auditor:   auditor,
        observers: make([]ConfigObserver, 0),
    }, nil
}

func (oc *ObservableConfig) AddObserver(observer ConfigObserver) {
    oc.observers = append(oc.observers, observer)
}

func (oc *ObservableConfig) LoadConfig(source string, config map[string]interface{}) error {
    oc.mutex.Lock()
    defer oc.mutex.Unlock()
    
    // Record metrics
    oc.metrics.RecordLoad(source)
    
    // Log audit event
    oc.auditor.LogLoad(source, config)
    
    // Notify observers
    for _, observer := range oc.observers {
        observer.OnConfigLoad(source, config)
    }
    
    // Store config
    for key, value := range config {
        if oldValue, exists := oc.config[key]; exists && oldValue != value {
            oc.metrics.RecordChange(key)
            oc.auditor.LogChange(key, oldValue, value)
            
            for _, observer := range oc.observers {
                observer.OnConfigChange(key, oldValue, value)
            }
        }
        oc.config[key] = value
    }
    
    return nil
}

func (oc *ObservableConfig) Get(key string) (interface{}, bool) {
    oc.mutex.RLock()
    defer oc.mutex.RUnlock()
    
    value, exists := oc.config[key]
    if exists {
        oc.metrics.RecordAccess(key)
        oc.auditor.LogAccess(key, value)
        
        for _, observer := range oc.observers {
            observer.OnConfigAccess(key, value)
        }
    }
    
    return value, exists
}

func (oc *ObservableConfig) GetString(key, defaultValue string) string {
    if value, exists := oc.Get(key); exists {
        if str, ok := value.(string); ok {
            return str
        }
    }
    return defaultValue
}

func (oc *ObservableConfig) GetInt(key string, defaultValue int) int {
    if value, exists := oc.Get(key); exists {
        switch v := value.(type) {
        case int:
            return v
        case float64:
            return int(v)
        }
    }
    return defaultValue
}

func (oc *ObservableConfig) GetBool(key string, defaultValue bool) bool {
    if value, exists := oc.Get(key); exists {
        if b, ok := value.(bool); ok {
            return b
        }
    }
    return defaultValue
}

func (oc *ObservableConfig) GetMetrics() ConfigMetrics {
    return oc.metrics.GetSnapshot()
}

func (oc *ObservableConfig) GetHealthStatus() string {
    metrics := oc.GetMetrics()
    return metrics.HealthStatus
}

func (oc *ObservableConfig) Close() error {
    return oc.auditor.Close()
}

// ConsoleObserver prints configuration events to the console
type ConsoleObserver struct {
    verbose bool
}

func NewConsoleObserver(verbose bool) *ConsoleObserver {
    return &ConsoleObserver{verbose: verbose}
}

func (co *ConsoleObserver) OnConfigLoad(source string, config map[string]interface{}) {
    fmt.Printf("Config loaded from %s (%d fields)\n", source, len(config))
}

func (co *ConsoleObserver) OnConfigChange(field string, oldValue, newValue interface{}) {
    if co.verbose {
        fmt.Printf("Config changed: %s = %v -> %v\n", field, oldValue, newValue)
    } else {
        fmt.Printf("Config changed: %s\n", field)
    }
}

func (co *ConsoleObserver) OnConfigError(source string, err error) {
    fmt.Printf("Config error from %s: %v\n", source, err)
}

func (co *ConsoleObserver) OnConfigAccess(field string, value interface{}) {
    if co.verbose {
        fmt.Printf("Config accessed: %s = %v\n", field, value)
    }
}

func main() {
    fmt.Println("=== Configuration Observability Example ===")
    
    // Create observable configuration
    auditLogPath := "/tmp/config_audit.log"
    observableConfig, err := NewObservableConfig(auditLogPath)
    if err != nil {
        fmt.Printf("Error creating observable config: %v\n", err)
        return
    }
    defer observableConfig.Close()
    
    // Add console observer
    observer := NewConsoleObserver(false) // Set to true for verbose output
    observableConfig.AddObserver(observer)
    
    // Load initial configuration
    fmt.Println("\n=== Loading Initial Configuration ===")
    initialConfig := map[string]interface{}{
        "app.name":         "ObservableApp",
        "app.version":      "1.0.0",
        "server.host":      "localhost",
        "server.port":      8080,
        "database.url":     "postgres://localhost:5432/app",
        "database.password": "secret123",
        "logging.level":    "info",
        "features.metrics": true,
    }
    
    observableConfig.LoadConfig("file:/etc/app/config.json", initialConfig)
    
    // Simulate configuration access
    fmt.Println("\n=== Accessing Configuration Values ===")
    appName := observableConfig.GetString("app.name", "unknown")
    serverPort := observableConfig.GetInt("server.port", 8080)
    metricsEnabled := observableConfig.GetBool("features.metrics", false)
    
    fmt.Printf("App: %s on port %d (metrics: %v)\n", appName, serverPort, metricsEnabled)
    
    // Simulate configuration changes
    fmt.Println("\n=== Simulating Configuration Changes ===")
    updatedConfig := map[string]interface{}{
        "server.port":      9090,
        "logging.level":    "debug",
        "features.tracing": true,
        "cache.type":       "redis",
    }
    
    observableConfig.LoadConfig("env:APP_", updatedConfig)
    
    // Access updated values
    newPort := observableConfig.GetInt("server.port", 8080)
    logLevel := observableConfig.GetString("logging.level", "info")
    
    fmt.Printf("Updated port: %d, log level: %s\n", newPort, logLevel)
    
    // Get metrics snapshot
    fmt.Println("\n=== Configuration Metrics ===")
    metrics := observableConfig.GetMetrics()
    fmt.Printf("Load count: %d\n", metrics.LoadCount)
    fmt.Printf("Change count: %d\n", metrics.ChangeCount)
    fmt.Printf("Error count: %d\n", metrics.ErrorCount)
    fmt.Printf("Health status: %s\n", metrics.HealthStatus)
    fmt.Printf("Last loaded: %s\n", metrics.LastLoaded.Format(time.RFC3339))
    
    if len(metrics.AccessCount) > 0 {
        fmt.Println("Field access counts:")
        for field, count := range metrics.AccessCount {
            fmt.Printf("  %s: %d\n", field, count)
        }
    }
    
    if len(metrics.ConfigSources) > 0 {
        fmt.Println("Configuration sources:")
        for source, count := range metrics.ConfigSources {
            fmt.Printf("  %s: %d loads\n", source, count)
        }
    }
    
    // Simulate error
    fmt.Println("\n=== Simulating Configuration Error ===")
    for _, observer := range observableConfig.observers {
        observer.OnConfigError("file:/invalid/path.json", fmt.Errorf("file not found"))
    }
    
    // Check health after error
    fmt.Printf("Health status after error: %s\n", observableConfig.GetHealthStatus())
    
    // Show audit log contents
    fmt.Println("\n=== Audit Log Contents ===")
    if auditData, err := os.ReadFile(auditLogPath); err == nil {
        fmt.Printf("Audit log (%s):\n", auditLogPath)
        fmt.Println(string(auditData))
    } else {
        fmt.Printf("Error reading audit log: %v\n", err)
    }
    
    // Performance metrics
    fmt.Println("\n=== Performance Test ===")
    start := time.Now()
    
    for i := 0; i < 1000; i++ {
        observableConfig.GetString("app.name", "default")
        observableConfig.GetInt("server.port", 8080)
    }
    
    duration := time.Since(start)
    fmt.Printf("1000 config accesses took: %v\n", duration)
    fmt.Printf("Average per access: %v\n", duration/1000)
    
    // Final metrics
    finalMetrics := observableConfig.GetMetrics()
    fmt.Printf("Total accesses: %d\n", finalMetrics.AccessCount["app.name"]+finalMetrics.AccessCount["server.port"])
    
    // Clean up
    os.Remove(auditLogPath)
    
    fmt.Println("\n=== Observability Demo Complete ===")
}
```

This example demonstrates comprehensive configuration observability including  
metrics collection, audit logging, observer patterns, and health monitoring.  
The pattern provides visibility into configuration usage and changes while  
maintaining performance and security through proper data sanitization.  

## Configuration caching and performance optimization

Configuration caching improves application performance by reducing I/O  
operations and providing fast access to frequently used configuration values.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "sync"
    "time"
)

// CacheEntry represents a cached configuration value
type CacheEntry struct {
    Value     interface{}
    Timestamp time.Time
    TTL       time.Duration
    AccessCount int64
    LastAccess  time.Time
}

func (ce *CacheEntry) IsExpired() bool {
    if ce.TTL == 0 {
        return false // No expiration
    }
    return time.Since(ce.Timestamp) > ce.TTL
}

func (ce *CacheEntry) Touch() {
    ce.AccessCount++
    ce.LastAccess = time.Now()
}

// ConfigCache provides caching capabilities for configuration values
type ConfigCache struct {
    entries   map[string]*CacheEntry
    mutex     sync.RWMutex
    defaultTTL time.Duration
    maxSize   int
    hitCount  int64
    missCount int64
    evictions int64
}

func NewConfigCache(defaultTTL time.Duration, maxSize int) *ConfigCache {
    cache := &ConfigCache{
        entries:    make(map[string]*CacheEntry),
        defaultTTL: defaultTTL,
        maxSize:    maxSize,
    }
    
    // Start cleanup goroutine
    go cache.cleanupExpired()
    
    return cache
}

func (cc *ConfigCache) Set(key string, value interface{}, ttl time.Duration) {
    cc.mutex.Lock()
    defer cc.mutex.Unlock()
    
    if ttl == 0 {
        ttl = cc.defaultTTL
    }
    
    // Check if we need to evict entries
    if len(cc.entries) >= cc.maxSize {
        cc.evictLRU()
    }
    
    cc.entries[key] = &CacheEntry{
        Value:      value,
        Timestamp:  time.Now(),
        TTL:        ttl,
        AccessCount: 0,
        LastAccess:  time.Now(),
    }
}

func (cc *ConfigCache) Get(key string) (interface{}, bool) {
    cc.mutex.Lock()
    defer cc.mutex.Unlock()
    
    entry, exists := cc.entries[key]
    if !exists {
        cc.missCount++
        return nil, false
    }
    
    if entry.IsExpired() {
        delete(cc.entries, key)
        cc.missCount++
        return nil, false
    }
    
    entry.Touch()
    cc.hitCount++
    return entry.Value, true
}

func (cc *ConfigCache) Delete(key string) {
    cc.mutex.Lock()
    defer cc.mutex.Unlock()
    delete(cc.entries, key)
}

func (cc *ConfigCache) Clear() {
    cc.mutex.Lock()
    defer cc.mutex.Unlock()
    cc.entries = make(map[string]*CacheEntry)
}

func (cc *ConfigCache) evictLRU() {
    if len(cc.entries) == 0 {
        return
    }
    
    var oldestKey string
    var oldestTime time.Time
    
    for key, entry := range cc.entries {
        if oldestKey == "" || entry.LastAccess.Before(oldestTime) {
            oldestKey = key
            oldestTime = entry.LastAccess
        }
    }
    
    if oldestKey != "" {
        delete(cc.entries, oldestKey)
        cc.evictions++
    }
}

func (cc *ConfigCache) cleanupExpired() {
    ticker := time.NewTicker(1 * time.Minute)
    defer ticker.Stop()
    
    for range ticker.C {
        cc.mutex.Lock()
        for key, entry := range cc.entries {
            if entry.IsExpired() {
                delete(cc.entries, key)
            }
        }
        cc.mutex.Unlock()
    }
}

func (cc *ConfigCache) GetStats() CacheStats {
    cc.mutex.RLock()
    defer cc.mutex.RUnlock()
    
    total := cc.hitCount + cc.missCount
    hitRatio := float64(0)
    if total > 0 {
        hitRatio = float64(cc.hitCount) / float64(total)
    }
    
    return CacheStats{
        Size:      len(cc.entries),
        MaxSize:   cc.maxSize,
        HitCount:  cc.hitCount,
        MissCount: cc.missCount,
        HitRatio:  hitRatio,
        Evictions: cc.evictions,
    }
}

type CacheStats struct {
    Size      int     `json:"size"`
    MaxSize   int     `json:"max_size"`
    HitCount  int64   `json:"hit_count"`
    MissCount int64   `json:"miss_count"`
    HitRatio  float64 `json:"hit_ratio"`
    Evictions int64   `json:"evictions"`
}

// PerformantConfigManager combines configuration management with caching
type PerformantConfigManager struct {
    cache       *ConfigCache
    configFiles map[string]string
    lastLoaded  map[string]time.Time
    watchers    map[string]*time.Ticker
    mutex       sync.RWMutex
    loadCount   int64
}

func NewPerformantConfigManager(cacheSize int, defaultTTL time.Duration) *PerformantConfigManager {
    return &PerformantConfigManager{
        cache:       NewConfigCache(defaultTTL, cacheSize),
        configFiles: make(map[string]string),
        lastLoaded:  make(map[string]time.Time),
        watchers:    make(map[string]*time.Ticker),
    }
}

func (pcm *PerformantConfigManager) LoadConfigFile(key, filename string, refreshInterval time.Duration) error {
    // Load initial configuration
    if err := pcm.loadFromFile(key, filename); err != nil {
        return err
    }
    
    pcm.mutex.Lock()
    pcm.configFiles[key] = filename
    pcm.mutex.Unlock()
    
    // Start auto-refresh if specified
    if refreshInterval > 0 {
        pcm.startAutoRefresh(key, filename, refreshInterval)
    }
    
    return nil
}

func (pcm *PerformantConfigManager) loadFromFile(key, filename string) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return fmt.Errorf("failed to read config file %s: %w", filename, err)
    }
    
    var config map[string]interface{}
    if err := json.Unmarshal(data, &config); err != nil {
        return fmt.Errorf("failed to parse config file %s: %w", filename, err)
    }
    
    // Cache configuration values with different TTLs based on type
    for configKey, value := range config {
        fullKey := fmt.Sprintf("%s.%s", key, configKey)
        ttl := pcm.getTTLForKey(configKey)
        pcm.cache.Set(fullKey, value, ttl)
    }
    
    pcm.mutex.Lock()
    pcm.lastLoaded[key] = time.Now()
    pcm.loadCount++
    pcm.mutex.Unlock()
    
    fmt.Printf("Loaded configuration %s from %s (%d keys)\n", key, filename, len(config))
    return nil
}

func (pcm *PerformantConfigManager) getTTLForKey(key string) time.Duration {
    // Different TTL strategies based on key patterns
    switch {
    case key == "version" || key == "app_name":
        return 1 * time.Hour // Static values, long TTL
    case key == "debug" || key == "log_level":
        return 10 * time.Minute // Development settings, medium TTL
    case key == "feature_flags" || key == "rate_limits":
        return 30 * time.Second // Dynamic settings, short TTL
    default:
        return 5 * time.Minute // Default TTL
    }
}

func (pcm *PerformantConfigManager) startAutoRefresh(key, filename string, interval time.Duration) {
    pcm.mutex.Lock()
    defer pcm.mutex.Unlock()
    
    // Stop existing watcher if any
    if ticker, exists := pcm.watchers[key]; exists {
        ticker.Stop()
    }
    
    ticker := time.NewTicker(interval)
    pcm.watchers[key] = ticker
    
    go func() {
        for range ticker.C {
            pcm.mutex.RLock()
            currentFile, exists := pcm.configFiles[key]
            pcm.mutex.RUnlock()
            
            if !exists || currentFile != filename {
                return // Configuration removed or changed
            }
            
            if err := pcm.loadFromFile(key, filename); err != nil {
                fmt.Printf("Auto-refresh failed for %s: %v\n", key, err)
            }
        }
    }()
}

func (pcm *PerformantConfigManager) GetString(key, defaultValue string) string {
    if value, exists := pcm.cache.Get(key); exists {
        if str, ok := value.(string); ok {
            return str
        }
    }
    return defaultValue
}

func (pcm *PerformantConfigManager) GetInt(key string, defaultValue int) int {
    if value, exists := pcm.cache.Get(key); exists {
        switch v := value.(type) {
        case int:
            return v
        case float64:
            return int(v)
        }
    }
    return defaultValue
}

func (pcm *PerformantConfigManager) GetBool(key string, defaultValue bool) bool {
    if value, exists := pcm.cache.Get(key); exists {
        if b, ok := value.(bool); ok {
            return b
        }
    }
    return defaultValue
}

func (pcm *PerformantConfigManager) GetFloat(key string, defaultValue float64) float64 {
    if value, exists := pcm.cache.Get(key); exists {
        switch v := value.(type) {
        case float64:
            return v
        case int:
            return float64(v)
        }
    }
    return defaultValue
}

func (pcm *PerformantConfigManager) InvalidateKey(key string) {
    pcm.cache.Delete(key)
}

func (pcm *PerformantConfigManager) InvalidateAll() {
    pcm.cache.Clear()
}

func (pcm *PerformantConfigManager) GetCacheStats() CacheStats {
    return pcm.cache.GetStats()
}

func (pcm *PerformantConfigManager) GetLoadCount() int64 {
    pcm.mutex.RLock()
    defer pcm.mutex.RUnlock()
    return pcm.loadCount
}

func (pcm *PerformantConfigManager) Stop() {
    pcm.mutex.Lock()
    defer pcm.mutex.Unlock()
    
    for _, ticker := range pcm.watchers {
        ticker.Stop()
    }
    pcm.watchers = make(map[string]*time.Ticker)
}

func main() {
    fmt.Println("=== Configuration Caching and Performance Example ===")
    
    // Create performant config manager
    manager := NewPerformantConfigManager(100, 5*time.Minute)
    defer manager.Stop()
    
    // Create sample configuration files
    appConfig := map[string]interface{}{
        "app_name":    "CachedApp",
        "version":     "1.0.0",
        "debug":       false,
        "log_level":   "info",
        "server_port": 8080,
        "timeout":     30.5,
    }
    
    featureConfig := map[string]interface{}{
        "feature_flags": map[string]bool{
            "metrics": true,
            "tracing": false,
            "caching": true,
        },
        "rate_limits": map[string]int{
            "api_requests": 1000,
            "file_uploads": 10,
        },
    }
    
    // Write config files
    appData, _ := json.MarshalIndent(appConfig, "", "  ")
    featureData, _ := json.MarshalIndent(featureConfig, "", "  ")
    
    appFile := "/tmp/app_config.json"
    featureFile := "/tmp/feature_config.json"
    
    os.WriteFile(appFile, appData, 0644)
    os.WriteFile(featureFile, featureData, 0644)
    
    // Load configurations with auto-refresh
    fmt.Println("\n=== Loading Configurations ===")
    if err := manager.LoadConfigFile("app", appFile, 30*time.Second); err != nil {
        fmt.Printf("Error loading app config: %v\n", err)
        return
    }
    
    if err := manager.LoadConfigFile("features", featureFile, 10*time.Second); err != nil {
        fmt.Printf("Error loading feature config: %v\n", err)
        return
    }
    
    // Test configuration access
    fmt.Println("\n=== Testing Configuration Access ===")
    
    appName := manager.GetString("app.app_name", "unknown")
    serverPort := manager.GetInt("app.server_port", 8080)
    timeout := manager.GetFloat("app.timeout", 30.0)
    debugMode := manager.GetBool("app.debug", true)
    
    fmt.Printf("App: %s\n", appName)
    fmt.Printf("Server port: %d\n", serverPort)
    fmt.Printf("Timeout: %.1f seconds\n", timeout)
    fmt.Printf("Debug mode: %v\n", debugMode)
    
    // Performance test
    fmt.Println("\n=== Performance Test ===")
    
    iterations := 10000
    start := time.Now()
    
    for i := 0; i < iterations; i++ {
        manager.GetString("app.app_name", "default")
        manager.GetInt("app.server_port", 8080)
        manager.GetBool("app.debug", false)
        manager.GetFloat("app.timeout", 30.0)
    }
    
    duration := time.Since(start)
    fmt.Printf("Performed %d cached config accesses in %v\n", iterations*4, duration)
    fmt.Printf("Average per access: %v\n", duration/time.Duration(iterations*4))
    
    // Show cache statistics
    fmt.Println("\n=== Cache Statistics ===")
    stats := manager.GetCacheStats()
    fmt.Printf("Cache size: %d/%d\n", stats.Size, stats.MaxSize)
    fmt.Printf("Hit count: %d\n", stats.HitCount)
    fmt.Printf("Miss count: %d\n", stats.MissCount)
    fmt.Printf("Hit ratio: %.2f%%\n", stats.HitRatio*100)
    fmt.Printf("Evictions: %d\n", stats.Evictions)
    fmt.Printf("Load count: %d\n", manager.GetLoadCount())
    
    // Test cache invalidation
    fmt.Println("\n=== Testing Cache Invalidation ===")
    fmt.Printf("Before invalidation - App name: %s\n", manager.GetString("app.app_name", "unknown"))
    
    manager.InvalidateKey("app.app_name")
    fmt.Printf("After invalidation - App name: %s\n", manager.GetString("app.app_name", "unknown"))
    
    // Test with different TTL for sensitive data
    fmt.Println("\n=== Testing TTL-based Expiration ===")
    
    // Simulate rapid access to show caching benefits
    rapidStart := time.Now()
    for i := 0; i < 1000; i++ {
        manager.GetString("app.version", "1.0.0")
    }
    rapidDuration := time.Since(rapidStart)
    
    fmt.Printf("1000 rapid accesses took: %v\n", rapidDuration)
    
    // Final cache stats
    finalStats := manager.GetCacheStats()
    fmt.Printf("\nFinal cache hit ratio: %.2f%%\n", finalStats.HitRatio*100)
    
    // Clean up
    os.Remove(appFile)
    os.Remove(featureFile)
    
    fmt.Println("\n=== Caching Demo Complete ===")
}
```

This example demonstrates comprehensive configuration caching and performance  
optimization including TTL-based expiration, LRU eviction, auto-refresh  
capabilities, and detailed performance metrics. The pattern significantly  
improves configuration access performance while maintaining data freshness  
through intelligent caching strategies.  

## Remote configuration services

Remote configuration services enable centralized configuration management  
with real-time updates and distributed coordination capabilities.  

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "sync"
    "time"
)

// RemoteConfigClient provides access to remote configuration services
type RemoteConfigClient struct {
    baseURL    string
    apiKey     string
    client     *http.Client
    cache      map[string]ConfigValue
    callbacks  []func(key string, value interface{})
    mutex      sync.RWMutex
    polling    bool
    pollCancel context.CancelFunc
}

type ConfigValue struct {
    Value     interface{} `json:"value"`
    Version   int64       `json:"version"`
    UpdatedAt time.Time   `json:"updated_at"`
    Source    string      `json:"source"`
}

func NewRemoteConfigClient(baseURL, apiKey string) *RemoteConfigClient {
    return &RemoteConfigClient{
        baseURL: baseURL,
        apiKey:  apiKey,
        client: &http.Client{
            Timeout: 10 * time.Second,
        },
        cache:     make(map[string]ConfigValue),
        callbacks: make([]func(string, interface{}), 0),
    }
}

func (rcc *RemoteConfigClient) GetValue(key string) (interface{}, error) {
    // Check cache first
    rcc.mutex.RLock()
    if cached, exists := rcc.cache[key]; exists {
        rcc.mutex.RUnlock()
        return cached.Value, nil
    }
    rcc.mutex.RUnlock()
    
    // Fetch from remote service
    url := fmt.Sprintf("%s/config/%s", rcc.baseURL, key)
    req, err := http.NewRequest("GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Authorization", "Bearer "+rcc.apiKey)
    req.Header.Set("Accept", "application/json")
    
    resp, err := rcc.client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch config: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("config service returned status: %d", resp.StatusCode)
    }
    
    var configValue ConfigValue
    if err := json.NewDecoder(resp.Body).Decode(&configValue); err != nil {
        return nil, fmt.Errorf("failed to decode response: %w", err)
    }
    
    // Update cache
    rcc.mutex.Lock()
    rcc.cache[key] = configValue
    rcc.mutex.Unlock()
    
    return configValue.Value, nil
}

func (rcc *RemoteConfigClient) StartPolling(ctx context.Context, interval time.Duration) {
    if rcc.polling {
        return
    }
    
    pollCtx, cancel := context.WithCancel(ctx)
    rcc.pollCancel = cancel
    rcc.polling = true
    
    go func() {
        ticker := time.NewTicker(interval)
        defer ticker.Stop()
        
        for {
            select {
            case <-pollCtx.Done():
                rcc.polling = false
                return
            case <-ticker.C:
                rcc.pollUpdates()
            }
        }
    }()
}

func (rcc *RemoteConfigClient) StopPolling() {
    if rcc.pollCancel != nil {
        rcc.pollCancel()
    }
}

func (rcc *RemoteConfigClient) pollUpdates() {
    rcc.mutex.RLock()
    keys := make([]string, 0, len(rcc.cache))
    for key := range rcc.cache {
        keys = append(keys, key)
    }
    rcc.mutex.RUnlock()
    
    for _, key := range keys {
        if newValue, err := rcc.GetValue(key); err == nil {
            rcc.mutex.RLock()
            cached, exists := rcc.cache[key]
            rcc.mutex.RUnlock()
            
            if exists && cached.Value != newValue {
                for _, callback := range rcc.callbacks {
                    callback(key, newValue)
                }
            }
        }
    }
}

func (rcc *RemoteConfigClient) OnChange(callback func(key string, value interface{})) {
    rcc.callbacks = append(rcc.callbacks, callback)
}

// Simulate a remote config service
func startMockConfigService() *http.Server {
    mux := http.NewServeMux()
    
    // Mock configuration data
    configs := map[string]ConfigValue{
        "app.name": {
            Value:     "RemoteApp",
            Version:   1,
            UpdatedAt: time.Now(),
            Source:    "remote",
        },
        "feature.metrics": {
            Value:     true,
            Version:   1,
            UpdatedAt: time.Now(),
            Source:    "remote",
        },
        "rate.limit": {
            Value:     100,
            Version:   1,
            UpdatedAt: time.Now(),
            Source:    "remote",
        },
    }
    
    mux.HandleFunc("/config/", func(w http.ResponseWriter, r *http.Request) {
        key := r.URL.Path[8:] // Remove "/config/" prefix
        
        config, exists := configs[key]
        if !exists {
            http.NotFound(w, r)
            return
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(config)
    })
    
    server := &http.Server{
        Addr:    ":8090",
        Handler: mux,
    }
    
    go server.ListenAndServe()
    return server
}

func main() {
    fmt.Println("=== Remote Configuration Service Example ===")
    
    // Start mock config service
    server := startMockConfigService()
    defer server.Shutdown(context.Background())
    
    // Give server time to start
    time.Sleep(100 * time.Millisecond)
    
    // Create remote config client
    client := NewRemoteConfigClient("http://localhost:8090", "demo-api-key")
    
    // Set up change callback
    client.OnChange(func(key string, value interface{}) {
        fmt.Printf("Configuration changed: %s = %v\n", key, value)
    })
    
    // Fetch some configuration values
    fmt.Println("\n=== Fetching Remote Configuration ===")
    
    appName, err := client.GetValue("app.name")
    if err != nil {
        fmt.Printf("Error fetching app.name: %v\n", err)
    } else {
        fmt.Printf("App name: %v\n", appName)
    }
    
    metricsEnabled, err := client.GetValue("feature.metrics")
    if err != nil {
        fmt.Printf("Error fetching feature.metrics: %v\n", err)
    } else {
        fmt.Printf("Metrics enabled: %v\n", metricsEnabled)
    }
    
    rateLimit, err := client.GetValue("rate.limit")
    if err != nil {
        fmt.Printf("Error fetching rate.limit: %v\n", err)
    } else {
        fmt.Printf("Rate limit: %v\n", rateLimit)
    }
    
    // Test caching (second access should be faster)
    fmt.Println("\n=== Testing Cache Performance ===")
    
    start := time.Now()
    for i := 0; i < 100; i++ {
        client.GetValue("app.name")
    }
    duration := time.Since(start)
    fmt.Printf("100 cached accesses took: %v\n", duration)
    
    fmt.Println("\n=== Remote Configuration Complete ===")
}
```

This example demonstrates integration with remote configuration services  
including caching, polling for updates, and change notifications. This  
pattern enables centralized configuration management across distributed  
systems with real-time update capabilities.  

## Configuration feature flags and A/B testing

Feature flags provide runtime configuration control for application behavior  
enabling safe deployments and A/B testing capabilities.  

```go
package main

import (
    "crypto/md5"
    "encoding/json"
    "fmt"
    "math/rand"
    "os"
    "strconv"
    "sync"
    "time"
)

// FeatureFlag represents a feature flag configuration
type FeatureFlag struct {
    Key          string                 `json:"key"`
    Name         string                 `json:"name"`
    Description  string                 `json:"description"`
    Enabled      bool                   `json:"enabled"`
    Rules        []FeatureRule          `json:"rules"`
    Variations   map[string]interface{} `json:"variations"`
    DefaultValue interface{}            `json:"default_value"`
    CreatedAt    time.Time              `json:"created_at"`
    UpdatedAt    time.Time              `json:"updated_at"`
}

type FeatureRule struct {
    ID          string                 `json:"id"`
    Condition   RuleCondition          `json:"condition"`
    Variation   string                 `json:"variation"`
    Percentage  int                    `json:"percentage"`
    UserFilters map[string]interface{} `json:"user_filters"`
}

type RuleCondition struct {
    Operator string      `json:"operator"`
    Field    string      `json:"field"`
    Value    interface{} `json:"value"`
}

type User struct {
    ID         string            `json:"id"`
    Email      string            `json:"email"`
    Properties map[string]string `json:"properties"`
    Groups     []string          `json:"groups"`
}

// FeatureFlagManager manages feature flags and evaluations
type FeatureFlagManager struct {
    flags   map[string]*FeatureFlag
    mutex   sync.RWMutex
    metrics map[string]*FlagMetrics
}

type FlagMetrics struct {
    EvaluationCount map[string]int64 `json:"evaluation_count"`
    LastEvaluated   time.Time        `json:"last_evaluated"`
    TotalEvaluations int64           `json:"total_evaluations"`
}

func NewFeatureFlagManager() *FeatureFlagManager {
    return &FeatureFlagManager{
        flags:   make(map[string]*FeatureFlag),
        metrics: make(map[string]*FlagMetrics),
    }
}

func (ffm *FeatureFlagManager) LoadFlags(filename string) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return fmt.Errorf("failed to read flags file: %w", err)
    }
    
    var flags []*FeatureFlag
    if err := json.Unmarshal(data, &flags); err != nil {
        return fmt.Errorf("failed to parse flags: %w", err)
    }
    
    ffm.mutex.Lock()
    defer ffm.mutex.Unlock()
    
    for _, flag := range flags {
        ffm.flags[flag.Key] = flag
        if _, exists := ffm.metrics[flag.Key]; !exists {
            ffm.metrics[flag.Key] = &FlagMetrics{
                EvaluationCount: make(map[string]int64),
            }
        }
    }
    
    fmt.Printf("Loaded %d feature flags\n", len(flags))
    return nil
}

func (ffm *FeatureFlagManager) EvaluateFlag(key string, user *User, defaultValue interface{}) interface{} {
    ffm.mutex.RLock()
    flag, exists := ffm.flags[key]
    if !exists {
        ffm.mutex.RUnlock()
        return defaultValue
    }
    
    metrics := ffm.metrics[key]
    ffm.mutex.RUnlock()
    
    // Update metrics
    ffm.mutex.Lock()
    metrics.TotalEvaluations++
    metrics.LastEvaluated = time.Now()
    ffm.mutex.Unlock()
    
    // If flag is disabled, return default
    if !flag.Enabled {
        ffm.recordEvaluation(key, "default")
        return flag.DefaultValue
    }
    
    // Evaluate rules in order
    for _, rule := range flag.Rules {
        if ffm.evaluateRule(rule, user) {
            variation := ffm.selectVariation(flag, rule, user)
            ffm.recordEvaluation(key, variation)
            return flag.Variations[variation]
        }
    }
    
    // No rules matched, return default
    ffm.recordEvaluation(key, "default")
    return flag.DefaultValue
}

func (ffm *FeatureFlagManager) evaluateRule(rule FeatureRule, user *User) bool {
    // Check user filters
    for filterKey, filterValue := range rule.UserFilters {
        switch filterKey {
        case "user_id":
            if user.ID != filterValue {
                return false
            }
        case "email_domain":
            if !ffm.matchesEmailDomain(user.Email, filterValue.(string)) {
                return false
            }
        case "group":
            if !ffm.userInGroup(user, filterValue.(string)) {
                return false
            }
        case "property":
            propFilter := filterValue.(map[string]interface{})
            propKey := propFilter["key"].(string)
            propValue := propFilter["value"].(string)
            if user.Properties[propKey] != propValue {
                return false
            }
        }
    }
    
    // Check percentage rollout
    if rule.Percentage < 100 {
        hash := ffm.getUserHash(user.ID, rule.ID)
        if hash%100 >= rule.Percentage {
            return false
        }
    }
    
    return true
}

func (ffm *FeatureFlagManager) selectVariation(flag *FeatureFlag, rule FeatureRule, user *User) string {
    if rule.Variation != "" {
        return rule.Variation
    }
    
    // A/B test logic - use user hash for consistent assignment
    hash := ffm.getUserHash(user.ID, flag.Key)
    variations := make([]string, 0, len(flag.Variations))
    for variation := range flag.Variations {
        variations = append(variations, variation)
    }
    
    if len(variations) > 0 {
        return variations[hash%len(variations)]
    }
    
    return "default"
}

func (ffm *FeatureFlagManager) getUserHash(userID, salt string) int {
    hasher := md5.New()
    hasher.Write([]byte(userID + salt))
    hashBytes := hasher.Sum(nil)
    
    // Convert first 4 bytes to int
    hash := int(hashBytes[0])<<24 | int(hashBytes[1])<<16 | int(hashBytes[2])<<8 | int(hashBytes[3])
    if hash < 0 {
        hash = -hash
    }
    return hash
}

func (ffm *FeatureFlagManager) matchesEmailDomain(email, domain string) bool {
    if len(email) == 0 {
        return false
    }
    
    for i := len(email) - 1; i >= 0; i-- {
        if email[i] == '@' {
            emailDomain := email[i+1:]
            return emailDomain == domain
        }
    }
    return false
}

func (ffm *FeatureFlagManager) userInGroup(user *User, group string) bool {
    for _, userGroup := range user.Groups {
        if userGroup == group {
            return true
        }
    }
    return false
}

func (ffm *FeatureFlagManager) recordEvaluation(flagKey, variation string) {
    ffm.mutex.Lock()
    defer ffm.mutex.Unlock()
    
    if metrics, exists := ffm.metrics[flagKey]; exists {
        metrics.EvaluationCount[variation]++
    }
}

func (ffm *FeatureFlagManager) GetMetrics(flagKey string) *FlagMetrics {
    ffm.mutex.RLock()
    defer ffm.mutex.RUnlock()
    
    if metrics, exists := ffm.metrics[flagKey]; exists {
        // Return a copy
        metricsCopy := &FlagMetrics{
            EvaluationCount:  make(map[string]int64),
            LastEvaluated:    metrics.LastEvaluated,
            TotalEvaluations: metrics.TotalEvaluations,
        }
        for k, v := range metrics.EvaluationCount {
            metricsCopy.EvaluationCount[k] = v
        }
        return metricsCopy
    }
    
    return nil
}

func (ffm *FeatureFlagManager) IsEnabled(key string, user *User) bool {
    result := ffm.EvaluateFlag(key, user, false)
    if b, ok := result.(bool); ok {
        return b
    }
    return false
}

func (ffm *FeatureFlagManager) GetString(key string, user *User, defaultValue string) string {
    result := ffm.EvaluateFlag(key, user, defaultValue)
    if s, ok := result.(string); ok {
        return s
    }
    return defaultValue
}

func (ffm *FeatureFlagManager) GetInt(key string, user *User, defaultValue int) int {
    result := ffm.EvaluateFlag(key, user, defaultValue)
    switch v := result.(type) {
    case int:
        return v
    case float64:
        return int(v)
    case string:
        if i, err := strconv.Atoi(v); err == nil {
            return i
        }
    }
    return defaultValue
}

func main() {
    fmt.Println("=== Feature Flags and A/B Testing Example ===")
    
    // Create feature flag configuration
    flags := []*FeatureFlag{
        {
            Key:         "new_ui",
            Name:        "New UI Design",
            Description: "Enable the new user interface design",
            Enabled:     true,
            DefaultValue: false,
            Variations: map[string]interface{}{
                "control":   false,
                "treatment": true,
            },
            Rules: []FeatureRule{
                {
                    ID:         "beta_users",
                    Percentage: 100,
                    Variation:  "treatment",
                    UserFilters: map[string]interface{}{
                        "group": "beta",
                    },
                },
                {
                    ID:         "gradual_rollout",
                    Percentage: 25, // 25% rollout
                    Variation:  "treatment",
                },
            },
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
        {
            Key:         "recommendation_algorithm",
            Name:        "Recommendation Algorithm",
            Description: "A/B test for recommendation algorithms",
            Enabled:     true,
            DefaultValue: "collaborative",
            Variations: map[string]interface{}{
                "collaborative": "collaborative",
                "content_based": "content_based",
                "hybrid":        "hybrid",
            },
            Rules: []FeatureRule{
                {
                    ID:         "algorithm_test",
                    Percentage: 100,
                    UserFilters: map[string]interface{}{
                        "email_domain": "example.com",
                    },
                },
            },
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
        {
            Key:         "premium_features",
            Name:        "Premium Features",
            Description: "Enable premium features for eligible users",
            Enabled:     true,
            DefaultValue: false,
            Variations: map[string]interface{}{
                "disabled": false,
                "enabled":  true,
            },
            Rules: []FeatureRule{
                {
                    ID:         "premium_users",
                    Percentage: 100,
                    Variation:  "enabled",
                    UserFilters: map[string]interface{}{
                        "property": map[string]interface{}{
                            "key":   "subscription",
                            "value": "premium",
                        },
                    },
                },
            },
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
    }
    
    // Save flags to file
    flagsData, _ := json.MarshalIndent(flags, "", "  ")
    flagsFile := "/tmp/feature_flags.json"
    os.WriteFile(flagsFile, flagsData, 0644)
    
    // Create feature flag manager
    manager := NewFeatureFlagManager()
    if err := manager.LoadFlags(flagsFile); err != nil {
        fmt.Printf("Error loading flags: %v\n", err)
        return
    }
    
    // Test with different users
    fmt.Println("\n=== Testing Feature Flag Evaluations ===")
    
    users := []*User{
        {
            ID:    "user1",
            Email: "alice@example.com",
            Properties: map[string]string{
                "subscription": "premium",
            },
            Groups: []string{"beta"},
        },
        {
            ID:    "user2",
            Email: "bob@company.com",
            Properties: map[string]string{
                "subscription": "free",
            },
            Groups: []string{"regular"},
        },
        {
            ID:    "user3",
            Email: "charlie@example.com",
            Properties: map[string]string{
                "subscription": "free",
            },
            Groups: []string{"regular"},
        },
    }
    
    for _, user := range users {
        fmt.Printf("\nUser: %s (%s)\n", user.ID, user.Email)
        
        newUI := manager.IsEnabled("new_ui", user)
        fmt.Printf("  New UI: %v\n", newUI)
        
        algorithm := manager.GetString("recommendation_algorithm", user, "default")
        fmt.Printf("  Recommendation Algorithm: %s\n", algorithm)
        
        premiumFeatures := manager.IsEnabled("premium_features", user)
        fmt.Printf("  Premium Features: %v\n", premiumFeatures)
    }
    
    // Simulate many evaluations for metrics
    fmt.Println("\n=== Simulating Traffic for A/B Test ===")
    
    rand.Seed(time.Now().UnixNano())
    
    for i := 0; i < 1000; i++ {
        userID := fmt.Sprintf("user_%d", rand.Intn(10000))
        user := &User{
            ID:    userID,
            Email: fmt.Sprintf("%s@test.com", userID),
            Groups: []string{"regular"},
        }
        
        manager.EvaluateFlag("new_ui", user, false)
        manager.EvaluateFlag("recommendation_algorithm", user, "collaborative")
    }
    
    // Show metrics
    fmt.Println("\n=== Feature Flag Metrics ===")
    
    flagKeys := []string{"new_ui", "recommendation_algorithm", "premium_features"}
    for _, key := range flagKeys {
        metrics := manager.GetMetrics(key)
        if metrics != nil {
            fmt.Printf("\nFlag: %s\n", key)
            fmt.Printf("  Total evaluations: %d\n", metrics.TotalEvaluations)
            fmt.Printf("  Last evaluated: %s\n", metrics.LastEvaluated.Format(time.RFC3339))
            fmt.Printf("  Variation breakdown:\n")
            
            for variation, count := range metrics.EvaluationCount {
                percentage := float64(count) / float64(metrics.TotalEvaluations) * 100
                fmt.Printf("    %s: %d (%.1f%%)\n", variation, count, percentage)
            }
        }
    }
    
    // Clean up
    os.Remove(flagsFile)
    
    fmt.Println("\n=== Feature Flags Demo Complete ===")
}
```

This example demonstrates comprehensive feature flag management including  
rule-based evaluation, A/B testing capabilities, user targeting, percentage  
rollouts, and detailed metrics collection. The pattern enables safe feature  
deployments and data-driven decision making through controlled rollouts.  

## Configuration deployment patterns

Configuration deployment patterns ensure safe and reliable configuration  
updates across different environments and deployment scenarios.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "path/filepath"
    "time"
)

// ConfigDeployment represents a configuration deployment
type ConfigDeployment struct {
    ID          string                 `json:"id"`
    Version     string                 `json:"version"`
    Environment string                 `json:"environment"`
    Config      map[string]interface{} `json:"config"`
    Metadata    DeploymentMetadata     `json:"metadata"`
    Status      string                 `json:"status"`
    CreatedAt   time.Time              `json:"created_at"`
    DeployedAt  *time.Time             `json:"deployed_at,omitempty"`
    RolledBackAt *time.Time            `json:"rolled_back_at,omitempty"`
}

type DeploymentMetadata struct {
    Author      string            `json:"author"`
    Description string            `json:"description"`
    Changes     []string          `json:"changes"`
    Tags        map[string]string `json:"tags"`
    Checksum    string            `json:"checksum"`
}

// ConfigDeploymentManager manages configuration deployments
type ConfigDeploymentManager struct {
    deploymentDir string
    environments  map[string]*EnvironmentConfig
}

type EnvironmentConfig struct {
    Name           string   `json:"name"`
    RequiresApproval bool   `json:"requires_approval"`
    Validators     []string `json:"validators"`
    PreDeployHooks []string `json:"pre_deploy_hooks"`
    PostDeployHooks []string `json:"post_deploy_hooks"`
}

func NewConfigDeploymentManager(deploymentDir string) *ConfigDeploymentManager {
    return &ConfigDeploymentManager{
        deploymentDir: deploymentDir,
        environments: map[string]*EnvironmentConfig{
            "development": {
                Name:           "development",
                RequiresApproval: false,
                Validators:     []string{"syntax", "basic"},
            },
            "staging": {
                Name:           "staging",
                RequiresApproval: true,
                Validators:     []string{"syntax", "basic", "integration"},
                PreDeployHooks: []string{"backup", "notify"},
            },
            "production": {
                Name:           "production",
                RequiresApproval: true,
                Validators:     []string{"syntax", "basic", "integration", "security"},
                PreDeployHooks: []string{"backup", "notify", "maintenance_mode"},
                PostDeployHooks: []string{"health_check", "notify", "disable_maintenance"},
            },
        },
    }
}

func (cdm *ConfigDeploymentManager) CreateDeployment(
    version, environment string,
    config map[string]interface{},
    metadata DeploymentMetadata,
) (*ConfigDeployment, error) {
    
    // Validate environment
    envConfig, exists := cdm.environments[environment]
    if !exists {
        return nil, fmt.Errorf("unknown environment: %s", environment)
    }
    
    // Create deployment
    deployment := &ConfigDeployment{
        ID:          fmt.Sprintf("%s-%s-%d", environment, version, time.Now().Unix()),
        Version:     version,
        Environment: environment,
        Config:      config,
        Metadata:    metadata,
        Status:      "created",
        CreatedAt:   time.Now(),
    }
    
    // Calculate checksum
    deployment.Metadata.Checksum = cdm.calculateChecksum(config)
    
    // Validate configuration
    if err := cdm.validateDeployment(deployment, envConfig); err != nil {
        deployment.Status = "validation_failed"
        return deployment, fmt.Errorf("validation failed: %w", err)
    }
    
    deployment.Status = "validated"
    
    // Save deployment
    if err := cdm.saveDeployment(deployment); err != nil {
        return deployment, fmt.Errorf("failed to save deployment: %w", err)
    }
    
    return deployment, nil
}

func (cdm *ConfigDeploymentManager) DeployConfiguration(deploymentID string) error {
    deployment, err := cdm.loadDeployment(deploymentID)
    if err != nil {
        return fmt.Errorf("failed to load deployment: %w", err)
    }
    
    envConfig := cdm.environments[deployment.Environment]
    
    fmt.Printf("Deploying configuration %s to %s\n", deploymentID, deployment.Environment)
    
    // Check approval requirements
    if envConfig.RequiresApproval && deployment.Status != "approved" {
        return fmt.Errorf("deployment requires approval")
    }
    
    deployment.Status = "deploying"
    cdm.saveDeployment(deployment)
    
    // Execute pre-deploy hooks
    for _, hook := range envConfig.PreDeployHooks {
        fmt.Printf("Executing pre-deploy hook: %s\n", hook)
        if err := cdm.executeHook(hook, deployment); err != nil {
            deployment.Status = "deploy_failed"
            cdm.saveDeployment(deployment)
            return fmt.Errorf("pre-deploy hook %s failed: %w", hook, err)
        }
    }
    
    // Deploy configuration
    if err := cdm.deployToEnvironment(deployment); err != nil {
        deployment.Status = "deploy_failed"
        cdm.saveDeployment(deployment)
        return fmt.Errorf("deployment failed: %w", err)
    }
    
    // Execute post-deploy hooks
    for _, hook := range envConfig.PostDeployHooks {
        fmt.Printf("Executing post-deploy hook: %s\n", hook)
        if err := cdm.executeHook(hook, deployment); err != nil {
            fmt.Printf("Warning: post-deploy hook %s failed: %v\n", hook, err)
        }
    }
    
    now := time.Now()
    deployment.DeployedAt = &now
    deployment.Status = "deployed"
    cdm.saveDeployment(deployment)
    
    fmt.Printf("Configuration %s deployed successfully\n", deploymentID)
    return nil
}

func (cdm *ConfigDeploymentManager) RollbackDeployment(deploymentID string) error {
    deployment, err := cdm.loadDeployment(deploymentID)
    if err != nil {
        return fmt.Errorf("failed to load deployment: %w", err)
    }
    
    if deployment.Status != "deployed" {
        return fmt.Errorf("can only rollback deployed configurations")
    }
    
    fmt.Printf("Rolling back deployment %s\n", deploymentID)
    
    // Find previous deployment
    previousDeployment, err := cdm.findPreviousDeployment(deployment.Environment, deployment.ID)
    if err != nil {
        return fmt.Errorf("failed to find previous deployment: %w", err)
    }
    
    if previousDeployment == nil {
        return fmt.Errorf("no previous deployment found for rollback")
    }
    
    // Deploy previous configuration
    if err := cdm.deployToEnvironment(previousDeployment); err != nil {
        return fmt.Errorf("rollback failed: %w", err)
    }
    
    now := time.Now()
    deployment.RolledBackAt = &now
    deployment.Status = "rolled_back"
    cdm.saveDeployment(deployment)
    
    fmt.Printf("Deployment %s rolled back successfully\n", deploymentID)
    return nil
}

func (cdm *ConfigDeploymentManager) validateDeployment(deployment *ConfigDeployment, envConfig *EnvironmentConfig) error {
    for _, validator := range envConfig.Validators {
        switch validator {
        case "syntax":
            if err := cdm.validateSyntax(deployment.Config); err != nil {
                return fmt.Errorf("syntax validation failed: %w", err)
            }
        case "basic":
            if err := cdm.validateBasic(deployment.Config); err != nil {
                return fmt.Errorf("basic validation failed: %w", err)
            }
        case "integration":
            if err := cdm.validateIntegration(deployment.Config, deployment.Environment); err != nil {
                return fmt.Errorf("integration validation failed: %w", err)
            }
        case "security":
            if err := cdm.validateSecurity(deployment.Config); err != nil {
                return fmt.Errorf("security validation failed: %w", err)
            }
        }
    }
    return nil
}

func (cdm *ConfigDeploymentManager) validateSyntax(config map[string]interface{}) error {
    // Validate JSON syntax by marshaling/unmarshaling
    _, err := json.Marshal(config)
    return err
}

func (cdm *ConfigDeploymentManager) validateBasic(config map[string]interface{}) error {
    // Check required fields
    requiredFields := []string{"app_name", "version"}
    for _, field := range requiredFields {
        if _, exists := config[field]; !exists {
            return fmt.Errorf("required field missing: %s", field)
        }
    }
    return nil
}

func (cdm *ConfigDeploymentManager) validateIntegration(config map[string]interface{}, environment string) error {
    // Environment-specific validation
    if environment == "production" {
        if debug, exists := config["debug"]; exists && debug.(bool) {
            return fmt.Errorf("debug mode not allowed in production")
        }
    }
    return nil
}

func (cdm *ConfigDeploymentManager) validateSecurity(config map[string]interface{}) error {
    // Check for potential secrets in configuration
    for key, value := range config {
        if str, ok := value.(string); ok {
            if len(str) > 10 && len(str) < 100 {
                // Potential secret - should be externalized
                fmt.Printf("Warning: potential secret in field %s\n", key)
            }
        }
    }
    return nil
}

func (cdm *ConfigDeploymentManager) executeHook(hook string, deployment *ConfigDeployment) error {
    switch hook {
    case "backup":
        return cdm.createBackup(deployment.Environment)
    case "notify":
        return cdm.sendNotification(deployment)
    case "maintenance_mode":
        return cdm.enableMaintenanceMode(deployment.Environment)
    case "health_check":
        return cdm.performHealthCheck(deployment.Environment)
    case "disable_maintenance":
        return cdm.disableMaintenanceMode(deployment.Environment)
    default:
        return fmt.Errorf("unknown hook: %s", hook)
    }
}

func (cdm *ConfigDeploymentManager) createBackup(environment string) error {
    fmt.Printf("Creating backup for environment: %s\n", environment)
    // Implementation would backup current configuration
    return nil
}

func (cdm *ConfigDeploymentManager) sendNotification(deployment *ConfigDeployment) error {
    fmt.Printf("Sending notification for deployment: %s\n", deployment.ID)
    // Implementation would send notifications via email, Slack, etc.
    return nil
}

func (cdm *ConfigDeploymentManager) enableMaintenanceMode(environment string) error {
    fmt.Printf("Enabling maintenance mode for: %s\n", environment)
    // Implementation would enable maintenance mode
    return nil
}

func (cdm *ConfigDeploymentManager) performHealthCheck(environment string) error {
    fmt.Printf("Performing health check for: %s\n", environment)
    // Implementation would check application health
    return nil
}

func (cdm *ConfigDeploymentManager) disableMaintenanceMode(environment string) error {
    fmt.Printf("Disabling maintenance mode for: %s\n", environment)
    // Implementation would disable maintenance mode
    return nil
}

func (cdm *ConfigDeploymentManager) deployToEnvironment(deployment *ConfigDeployment) error {
    fmt.Printf("Deploying to environment: %s\n", deployment.Environment)
    
    // Write configuration to environment-specific location
    envDir := filepath.Join(cdm.deploymentDir, "environments", deployment.Environment)
    os.MkdirAll(envDir, 0755)
    
    configFile := filepath.Join(envDir, "config.json")
    data, _ := json.MarshalIndent(deployment.Config, "", "  ")
    
    return os.WriteFile(configFile, data, 0644)
}

func (cdm *ConfigDeploymentManager) calculateChecksum(config map[string]interface{}) string {
    data, _ := json.Marshal(config)
    return fmt.Sprintf("%x", len(data)) // Simplified checksum
}

func (cdm *ConfigDeploymentManager) saveDeployment(deployment *ConfigDeployment) error {
    deploymentFile := filepath.Join(cdm.deploymentDir, "deployments", deployment.ID+".json")
    os.MkdirAll(filepath.Dir(deploymentFile), 0755)
    
    data, _ := json.MarshalIndent(deployment, "", "  ")
    return os.WriteFile(deploymentFile, data, 0644)
}

func (cdm *ConfigDeploymentManager) loadDeployment(deploymentID string) (*ConfigDeployment, error) {
    deploymentFile := filepath.Join(cdm.deploymentDir, "deployments", deploymentID+".json")
    
    data, err := os.ReadFile(deploymentFile)
    if err != nil {
        return nil, err
    }
    
    var deployment ConfigDeployment
    if err := json.Unmarshal(data, &deployment); err != nil {
        return nil, err
    }
    
    return &deployment, nil
}

func (cdm *ConfigDeploymentManager) findPreviousDeployment(environment, currentID string) (*ConfigDeployment, error) {
    // Implementation would find the most recent successful deployment
    // This is simplified for the example
    return nil, fmt.Errorf("no previous deployment found")
}

func main() {
    fmt.Println("=== Configuration Deployment Patterns Example ===")
    
    // Create deployment manager
    deploymentDir := "/tmp/config_deployments"
    manager := NewConfigDeploymentManager(deploymentDir)
    
    // Sample configuration
    config := map[string]interface{}{
        "app_name":    "DeploymentApp",
        "version":     "2.1.0",
        "environment": "production",
        "debug":       false,
        "server": map[string]interface{}{
            "host": "0.0.0.0",
            "port": 8080,
        },
        "database": map[string]interface{}{
            "host": "prod-db.example.com",
            "port": 5432,
        },
        "features": map[string]interface{}{
            "metrics": true,
            "tracing": true,
        },
    }
    
    metadata := DeploymentMetadata{
        Author:      "deployment-system",
        Description: "Production deployment with new features",
        Changes: []string{
            "Enable metrics collection",
            "Enable distributed tracing",
            "Update database host",
        },
        Tags: map[string]string{
            "release": "v2.1.0",
            "hotfix":  "false",
        },
    }
    
    // Test development deployment (no approval required)
    fmt.Println("\n=== Development Deployment ===")
    devDeployment, err := manager.CreateDeployment("2.1.0", "development", config, metadata)
    if err != nil {
        fmt.Printf("Error creating development deployment: %v\n", err)
    } else {
        fmt.Printf("Created deployment: %s\n", devDeployment.ID)
        
        if err := manager.DeployConfiguration(devDeployment.ID); err != nil {
            fmt.Printf("Development deployment failed: %v\n", err)
        }
    }
    
    // Test staging deployment (requires approval)
    fmt.Println("\n=== Staging Deployment ===")
    stagingDeployment, err := manager.CreateDeployment("2.1.0", "staging", config, metadata)
    if err != nil {
        fmt.Printf("Error creating staging deployment: %v\n", err)
    } else {
        fmt.Printf("Created deployment: %s\n", stagingDeployment.ID)
        
        // Simulate approval
        stagingDeployment.Status = "approved"
        manager.saveDeployment(stagingDeployment)
        
        if err := manager.DeployConfiguration(stagingDeployment.ID); err != nil {
            fmt.Printf("Staging deployment failed: %v\n", err)
        }
    }
    
    // Test production deployment
    fmt.Println("\n=== Production Deployment ===")
    prodDeployment, err := manager.CreateDeployment("2.1.0", "production", config, metadata)
    if err != nil {
        fmt.Printf("Error creating production deployment: %v\n", err)
    } else {
        fmt.Printf("Created deployment: %s\n", prodDeployment.ID)
        
        // Simulate approval
        prodDeployment.Status = "approved"
        manager.saveDeployment(prodDeployment)
        
        if err := manager.DeployConfiguration(prodDeployment.ID); err != nil {
            fmt.Printf("Production deployment failed: %v\n", err)
        }
    }
    
    // Test validation failure
    fmt.Println("\n=== Testing Validation Failure ===")
    invalidConfig := map[string]interface{}{
        "debug": true, // Not allowed in production
    }
    
    _, err = manager.CreateDeployment("2.1.1", "production", invalidConfig, metadata)
    if err != nil {
        fmt.Printf("Expected validation failure: %v\n", err)
    }
    
    // Clean up
    os.RemoveAll(deploymentDir)
    
    fmt.Println("\n=== Deployment Patterns Demo Complete ===")
}
```

This example demonstrates comprehensive configuration deployment patterns  
including environment-specific validation, approval workflows, deployment  
hooks, rollback capabilities, and audit trails. The pattern ensures safe  
and controlled configuration updates across different environments.  

## Configuration monitoring and health checks

Configuration monitoring ensures application configuration remains healthy  
and detects configuration-related issues before they impact users.  

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "sync"
    "time"
)

// ConfigHealthChecker monitors configuration health
type ConfigHealthChecker struct {
    checks      map[string]HealthCheck
    results     map[string]HealthResult
    mutex       sync.RWMutex
    interval    time.Duration
    running     bool
    stopChan    chan struct{}
    httpServer  *http.Server
}

type HealthCheck interface {
    Name() string
    Check(ctx context.Context) HealthResult
}

type HealthResult struct {
    Status    string    `json:"status"`
    Message   string    `json:"message"`
    Details   map[string]interface{} `json:"details"`
    CheckedAt time.Time `json:"checked_at"`
    Duration  time.Duration `json:"duration"`
}

// ConfigFileHealthCheck checks if config files are accessible
type ConfigFileHealthCheck struct {
    name string
    path string
}

func (c *ConfigFileHealthCheck) Name() string {
    return c.name
}

func (c *ConfigFileHealthCheck) Check(ctx context.Context) HealthResult {
    start := time.Now()
    
    if _, err := os.Stat(c.path); err != nil {
        return HealthResult{
            Status:    "unhealthy",
            Message:   fmt.Sprintf("Config file not accessible: %v", err),
            CheckedAt: time.Now(),
            Duration:  time.Since(start),
        }
    }
    
    return HealthResult{
        Status:    "healthy",
        Message:   "Config file accessible",
        CheckedAt: time.Now(),
        Duration:  time.Since(start),
    }
}

// ConfigValidationHealthCheck validates current configuration
type ConfigValidationHealthCheck struct {
    name      string
    validator func() error
}

func (c *ConfigValidationHealthCheck) Name() string {
    return c.name
}

func (c *ConfigValidationHealthCheck) Check(ctx context.Context) HealthResult {
    start := time.Now()
    
    if err := c.validator(); err != nil {
        return HealthResult{
            Status:    "unhealthy", 
            Message:   fmt.Sprintf("Configuration validation failed: %v", err),
            CheckedAt: time.Now(),
            Duration:  time.Since(start),
        }
    }
    
    return HealthResult{
        Status:    "healthy",
        Message:   "Configuration is valid",
        CheckedAt: time.Now(),
        Duration:  time.Since(start),
    }
}

func NewConfigHealthChecker(interval time.Duration) *ConfigHealthChecker {
    return &ConfigHealthChecker{
        checks:   make(map[string]HealthCheck),
        results:  make(map[string]HealthResult),
        interval: interval,
        stopChan: make(chan struct{}),
    }
}

func (chc *ConfigHealthChecker) AddCheck(check HealthCheck) {
    chc.mutex.Lock()
    defer chc.mutex.Unlock()
    chc.checks[check.Name()] = check
}

func (chc *ConfigHealthChecker) Start() {
    if chc.running {
        return
    }
    
    chc.running = true
    go chc.runHealthChecks()
    chc.startHTTPServer()
}

func (chc *ConfigHealthChecker) Stop() {
    if !chc.running {
        return
    }
    
    close(chc.stopChan)
    chc.running = false
    
    if chc.httpServer != nil {
        chc.httpServer.Shutdown(context.Background())
    }
}

func (chc *ConfigHealthChecker) runHealthChecks() {
    ticker := time.NewTicker(chc.interval)
    defer ticker.Stop()
    
    // Run initial check
    chc.performHealthChecks()
    
    for {
        select {
        case <-ticker.C:
            chc.performHealthChecks()
        case <-chc.stopChan:
            return
        }
    }
}

func (chc *ConfigHealthChecker) performHealthChecks() {
    chc.mutex.RLock()
    checks := make(map[string]HealthCheck)
    for name, check := range chc.checks {
        checks[name] = check
    }
    chc.mutex.RUnlock()
    
    var wg sync.WaitGroup
    resultsChan := make(chan struct{name string; result HealthResult}, len(checks))
    
    for name, check := range checks {
        wg.Add(1)
        go func(name string, check HealthCheck) {
            defer wg.Done()
            
            ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
            defer cancel()
            
            result := check.Check(ctx)
            resultsChan <- struct{name string; result HealthResult}{name: name, result: result}
        }(name, check)
    }
    
    go func() {
        wg.Wait()
        close(resultsChan)
    }()
    
    chc.mutex.Lock()
    for result := range resultsChan {
        chc.results[result.name] = result.result
    }
    chc.mutex.Unlock()
}

func (chc *ConfigHealthChecker) GetHealthStatus() map[string]HealthResult {
    chc.mutex.RLock()
    defer chc.mutex.RUnlock()
    
    status := make(map[string]HealthResult)
    for name, result := range chc.results {
        status[name] = result
    }
    return status
}

func (chc *ConfigHealthChecker) IsHealthy() bool {
    status := chc.GetHealthStatus()
    for _, result := range status {
        if result.Status != "healthy" {
            return false
        }
    }
    return true
}

func (chc *ConfigHealthChecker) startHTTPServer() {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/health", chc.healthHandler)
    mux.HandleFunc("/health/config", chc.configHealthHandler)
    
    chc.httpServer = &http.Server{
        Addr:    ":8091",
        Handler: mux,
    }
    
    go chc.httpServer.ListenAndServe()
}

func (chc *ConfigHealthChecker) healthHandler(w http.ResponseWriter, r *http.Request) {
    if chc.IsHealthy() {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    } else {
        w.WriteHeader(http.StatusServiceUnavailable)
        w.Write([]byte("UNHEALTHY"))
    }
}

func (chc *ConfigHealthChecker) configHealthHandler(w http.ResponseWriter, r *http.Request) {
    status := chc.GetHealthStatus()
    
    w.Header().Set("Content-Type", "application/json")
    if chc.IsHealthy() {
        w.WriteHeader(http.StatusOK)
    } else {
        w.WriteHeader(http.StatusServiceUnavailable)
    }
    
    json.NewEncoder(w).Encode(status)
}

func main() {
    fmt.Println("=== Configuration Health Monitoring Example ===")
    
    // Create config files for testing
    configFile := "/tmp/app_config.json"
    config := map[string]interface{}{
        "app_name": "HealthApp",
        "version":  "1.0.0",
        "server": map[string]interface{}{
            "port": 8080,
        },
    }
    data, _ := json.MarshalIndent(config, "", "  ")
    os.WriteFile(configFile, data, 0644)
    
    // Create health checker
    checker := NewConfigHealthChecker(10 * time.Second)
    
    // Add health checks
    checker.AddCheck(&ConfigFileHealthCheck{
        name: "config_file",
        path: configFile,
    })
    
    checker.AddCheck(&ConfigValidationHealthCheck{
        name: "config_validation",
        validator: func() error {
            // Simulate validation logic
            if config["app_name"] == nil {
                return fmt.Errorf("app_name is required")
            }
            return nil
        },
    })
    
    // Start monitoring
    checker.Start()
    defer checker.Stop()
    
    fmt.Println("Health monitoring started on :8091")
    fmt.Println("Endpoints:")
    fmt.Println("  GET /health - Overall health status")
    fmt.Println("  GET /health/config - Detailed config health")
    
    // Wait and show health status
    time.Sleep(2 * time.Second)
    
    fmt.Println("\n=== Initial Health Status ===")
    status := checker.GetHealthStatus()
    for name, result := range status {
        fmt.Printf("%s: %s - %s\n", name, result.Status, result.Message)
    }
    
    // Simulate config file issue
    fmt.Println("\n=== Simulating Config File Issue ===")
    os.Remove(configFile)
    
    time.Sleep(11 * time.Second) // Wait for next health check
    
    fmt.Println("Health status after removing config file:")
    status = checker.GetHealthStatus()
    for name, result := range status {
        fmt.Printf("%s: %s - %s\n", name, result.Status, result.Message)
    }
    
    // Restore config file
    fmt.Println("\n=== Restoring Config File ===")
    os.WriteFile(configFile, data, 0644)
    
    time.Sleep(11 * time.Second) // Wait for next health check
    
    fmt.Println("Health status after restoring config file:")
    status = checker.GetHealthStatus()
    for name, result := range status {
        fmt.Printf("%s: %s - %s\n", name, result.Status, result.Message)
    }
    
    // Clean up
    os.Remove(configFile)
    
    fmt.Println("\n=== Health Monitoring Demo Complete ===")
}
```

This example demonstrates configuration health monitoring with periodic  
checks, HTTP health endpoints, and real-time status reporting to detect  
configuration issues proactively.  

## Configuration backup and restore

Configuration backup and restore capabilities ensure configuration data  
can be recovered in case of failures or corruption.  

```go
package main

import (
    "compress/gzip"
    "encoding/json"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "sort"
    "strings"
    "time"
)

// ConfigBackupManager handles configuration backup and restore operations
type ConfigBackupManager struct {
    backupDir    string
    maxBackups   int
    compression  bool
    encryption   bool
    encryptionKey []byte
}

type BackupMetadata struct {
    ID          string            `json:"id"`
    Timestamp   time.Time         `json:"timestamp"`
    Description string            `json:"description"`
    Environment string            `json:"environment"`
    Version     string            `json:"version"`
    Files       map[string]string `json:"files"`
    Checksum    string            `json:"checksum"`
    Size        int64             `json:"size"`
    Compressed  bool              `json:"compressed"`
    Encrypted   bool              `json:"encrypted"`
}

func NewConfigBackupManager(backupDir string, maxBackups int) *ConfigBackupManager {
    return &ConfigBackupManager{
        backupDir:   backupDir,
        maxBackups:  maxBackups,
        compression: true,
        encryption:  false,
    }
}

func (cbm *ConfigBackupManager) CreateBackup(environment, version, description string, configFiles map[string]string) (*BackupMetadata, error) {
    backupID := fmt.Sprintf("%s_%s_%d", environment, version, time.Now().Unix())
    backupPath := filepath.Join(cbm.backupDir, backupID)
    
    if err := os.MkdirAll(backupPath, 0755); err != nil {
        return nil, fmt.Errorf("failed to create backup directory: %w", err)
    }
    
    metadata := &BackupMetadata{
        ID:          backupID,
        Timestamp:   time.Now(),
        Description: description,
        Environment: environment,
        Version:     version,
        Files:       make(map[string]string),
        Compressed:  cbm.compression,
        Encrypted:   cbm.encryption,
    }
    
    var totalSize int64
    
    // Backup each configuration file
    for configName, filePath := range configFiles {
        fmt.Printf("Backing up %s from %s\n", configName, filePath)
        
        backupFileName := configName + ".json"
        if cbm.compression {
            backupFileName += ".gz"
        }
        
        backupFilePath := filepath.Join(backupPath, backupFileName)
        size, err := cbm.backupFile(filePath, backupFilePath)
        if err != nil {
            return nil, fmt.Errorf("failed to backup file %s: %w", configName, err)
        }
        
        metadata.Files[configName] = backupFileName
        totalSize += size
    }
    
    metadata.Size = totalSize
    metadata.Checksum = cbm.calculateChecksum(metadata)
    
    // Save metadata
    metadataPath := filepath.Join(backupPath, "metadata.json")
    if err := cbm.saveMetadata(metadata, metadataPath); err != nil {
        return nil, fmt.Errorf("failed to save metadata: %w", err)
    }
    
    // Clean up old backups
    if err := cbm.cleanupOldBackups(); err != nil {
        fmt.Printf("Warning: failed to cleanup old backups: %v\n", err)
    }
    
    fmt.Printf("Backup created: %s (size: %d bytes)\n", backupID, totalSize)
    return metadata, nil
}

func (cbm *ConfigBackupManager) backupFile(sourcePath, backupPath string) (int64, error) {
    sourceFile, err := os.Open(sourcePath)
    if err != nil {
        return 0, err
    }
    defer sourceFile.Close()
    
    backupFile, err := os.Create(backupPath)
    if err != nil {
        return 0, err
    }
    defer backupFile.Close()
    
    var writer io.Writer = backupFile
    var closer io.Closer
    
    if cbm.compression {
        gzipWriter := gzip.NewWriter(backupFile)
        writer = gzipWriter
        closer = gzipWriter
    }
    
    size, err := io.Copy(writer, sourceFile)
    if err != nil {
        return 0, err
    }
    
    if closer != nil {
        closer.Close()
    }
    
    return size, nil
}

func (cbm *ConfigBackupManager) RestoreBackup(backupID string, targetDir string) error {
    backupPath := filepath.Join(cbm.backupDir, backupID)
    metadataPath := filepath.Join(backupPath, "metadata.json")
    
    metadata, err := cbm.loadMetadata(metadataPath)
    if err != nil {
        return fmt.Errorf("failed to load backup metadata: %w", err)
    }
    
    fmt.Printf("Restoring backup: %s (%s)\n", backupID, metadata.Description)
    
    if err := os.MkdirAll(targetDir, 0755); err != nil {
        return fmt.Errorf("failed to create target directory: %w", err)
    }
    
    // Restore each file
    for configName, backupFileName := range metadata.Files {
        sourcePath := filepath.Join(backupPath, backupFileName)
        targetPath := filepath.Join(targetDir, configName+".json")
        
        fmt.Printf("Restoring %s to %s\n", configName, targetPath)
        
        if err := cbm.restoreFile(sourcePath, targetPath, metadata.Compressed); err != nil {
            return fmt.Errorf("failed to restore file %s: %w", configName, err)
        }
    }
    
    fmt.Printf("Backup restored successfully to %s\n", targetDir)
    return nil
}

func (cbm *ConfigBackupManager) restoreFile(sourcePath, targetPath string, compressed bool) error {
    sourceFile, err := os.Open(sourcePath)
    if err != nil {
        return err
    }
    defer sourceFile.Close()
    
    targetFile, err := os.Create(targetPath)
    if err != nil {
        return err
    }
    defer targetFile.Close()
    
    var reader io.Reader = sourceFile
    var closer io.Closer
    
    if compressed {
        gzipReader, err := gzip.NewReader(sourceFile)
        if err != nil {
            return err
        }
        reader = gzipReader
        closer = gzipReader
    }
    
    _, err = io.Copy(targetFile, reader)
    if closer != nil {
        closer.Close()
    }
    
    return err
}

func (cbm *ConfigBackupManager) ListBackups() ([]*BackupMetadata, error) {
    entries, err := os.ReadDir(cbm.backupDir)
    if err != nil {
        return nil, err
    }
    
    var backups []*BackupMetadata
    
    for _, entry := range entries {
        if entry.IsDir() {
            metadataPath := filepath.Join(cbm.backupDir, entry.Name(), "metadata.json")
            if metadata, err := cbm.loadMetadata(metadataPath); err == nil {
                backups = append(backups, metadata)
            }
        }
    }
    
    // Sort by timestamp (newest first)
    sort.Slice(backups, func(i, j int) bool {
        return backups[i].Timestamp.After(backups[j].Timestamp)
    })
    
    return backups, nil
}

func (cbm *ConfigBackupManager) DeleteBackup(backupID string) error {
    backupPath := filepath.Join(cbm.backupDir, backupID)
    
    if err := os.RemoveAll(backupPath); err != nil {
        return fmt.Errorf("failed to delete backup: %w", err)
    }
    
    fmt.Printf("Backup deleted: %s\n", backupID)
    return nil
}

func (cbm *ConfigBackupManager) cleanupOldBackups() error {
    backups, err := cbm.ListBackups()
    if err != nil {
        return err
    }
    
    if len(backups) > cbm.maxBackups {
        // Delete oldest backups
        toDelete := backups[cbm.maxBackups:]
        for _, backup := range toDelete {
            if err := cbm.DeleteBackup(backup.ID); err != nil {
                fmt.Printf("Warning: failed to delete old backup %s: %v\n", backup.ID, err)
            }
        }
    }
    
    return nil
}

func (cbm *ConfigBackupManager) calculateChecksum(metadata *BackupMetadata) string {
    // Simplified checksum calculation
    data := fmt.Sprintf("%s%s%d", metadata.Environment, metadata.Version, metadata.Timestamp.Unix())
    return fmt.Sprintf("%x", len(data))
}

func (cbm *ConfigBackupManager) saveMetadata(metadata *BackupMetadata, path string) error {
    data, err := json.MarshalIndent(metadata, "", "  ")
    if err != nil {
        return err
    }
    return os.WriteFile(path, data, 0644)
}

func (cbm *ConfigBackupManager) loadMetadata(path string) (*BackupMetadata, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, err
    }
    
    var metadata BackupMetadata
    if err := json.Unmarshal(data, &metadata); err != nil {
        return nil, err
    }
    
    return &metadata, nil
}

func main() {
    fmt.Println("=== Configuration Backup and Restore Example ===")
    
    // Create sample configuration files
    configFiles := map[string]string{
        "app":      "/tmp/app_config.json",
        "database": "/tmp/db_config.json",
        "features": "/tmp/features_config.json",
    }
    
    // Create sample configs
    configs := map[string]interface{}{
        "app": map[string]interface{}{
            "name":    "BackupApp",
            "version": "1.0.0",
            "environment": "production",
        },
        "database": map[string]interface{}{
            "host": "db.example.com",
            "port": 5432,
            "name": "production_db",
        },
        "features": map[string]interface{}{
            "metrics": true,
            "tracing": false,
            "caching": true,
        },
    }
    
    for name, config := range configs {
        data, _ := json.MarshalIndent(config, "", "  ")
        os.WriteFile(configFiles[name], data, 0644)
    }
    
    // Create backup manager
    backupDir := "/tmp/config_backups"
    manager := NewConfigBackupManager(backupDir, 5)
    
    // Create backup
    fmt.Println("\n=== Creating Backup ===")
    backup, err := manager.CreateBackup(
        "production",
        "1.0.0",
        "Pre-deployment backup",
        configFiles,
    )
    if err != nil {
        fmt.Printf("Error creating backup: %v\n", err)
        return
    }
    
    // List backups
    fmt.Println("\n=== Listing Backups ===")
    backups, err := manager.ListBackups()
    if err != nil {
        fmt.Printf("Error listing backups: %v\n", err)
        return
    }
    
    for _, backup := range backups {
        fmt.Printf("Backup: %s\n", backup.ID)
        fmt.Printf("  Timestamp: %s\n", backup.Timestamp.Format(time.RFC3339))
        fmt.Printf("  Environment: %s\n", backup.Environment)
        fmt.Printf("  Version: %s\n", backup.Version)
        fmt.Printf("  Description: %s\n", backup.Description)
        fmt.Printf("  Size: %d bytes\n", backup.Size)
        fmt.Printf("  Files: %s\n", strings.Join(getKeys(backup.Files), ", "))
        fmt.Println()
    }
    
    // Simulate configuration corruption
    fmt.Println("=== Simulating Configuration Corruption ===")
    for _, filePath := range configFiles {
        os.WriteFile(filePath, []byte("corrupted"), 0644)
    }
    
    // Restore from backup
    fmt.Println("\n=== Restoring from Backup ===")
    restoreDir := "/tmp/config_restore"
    if err := manager.RestoreBackup(backup.ID, restoreDir); err != nil {
        fmt.Printf("Error restoring backup: %v\n", err)
        return
    }
    
    // Verify restored files
    fmt.Println("\n=== Verifying Restored Files ===")
    for configName := range configFiles {
        restoredPath := filepath.Join(restoreDir, configName+".json")
        if data, err := os.ReadFile(restoredPath); err == nil {
            var config map[string]interface{}
            if err := json.Unmarshal(data, &config); err == nil {
                fmt.Printf("Restored %s: %v\n", configName, config)
            }
        }
    }
    
    // Create another backup to test cleanup
    fmt.Println("\n=== Testing Backup Cleanup ===")
    for i := 0; i < 3; i++ {
        time.Sleep(100 * time.Millisecond) // Ensure different timestamps
        manager.CreateBackup(
            "staging",
            fmt.Sprintf("1.0.%d", i+1),
            fmt.Sprintf("Test backup %d", i+1),
            configFiles,
        )
    }
    
    finalBackups, _ := manager.ListBackups()
    fmt.Printf("Total backups after cleanup: %d\n", len(finalBackups))
    
    // Clean up
    for _, filePath := range configFiles {
        os.Remove(filePath)
    }
    os.RemoveAll(backupDir)
    os.RemoveAll(restoreDir)
    
    fmt.Println("\n=== Backup and Restore Demo Complete ===")
}

func getKeys(m map[string]string) []string {
    keys := make([]string, 0, len(m))
    for k := range m {
        keys = append(keys, k)
    }
    return keys
}
```

This example demonstrates comprehensive configuration backup and restore  
capabilities including compression, metadata tracking, cleanup policies,  
and recovery verification to ensure configuration data protection.  

## Configuration versioning and change tracking

Configuration versioning tracks changes over time and enables rollback  
to previous configurations when issues are detected.  

```go
package main

import (
    "crypto/md5"
    "encoding/json"
    "fmt"
    "os"
    "sort"
    "time"
)

// ConfigVersion represents a version of configuration
type ConfigVersion struct {
    ID          string                 `json:"id"`
    Version     int                    `json:"version"`
    Config      map[string]interface{} `json:"config"`
    Changes     []ConfigChange         `json:"changes"`
    Metadata    VersionMetadata        `json:"metadata"`
    Timestamp   time.Time              `json:"timestamp"`
    ParentID    string                 `json:"parent_id,omitempty"`
    Hash        string                 `json:"hash"`
}

type ConfigChange struct {
    Field     string      `json:"field"`
    Operation string      `json:"operation"` // "add", "update", "delete"
    OldValue  interface{} `json:"old_value,omitempty"`
    NewValue  interface{} `json:"new_value,omitempty"`
}

type VersionMetadata struct {
    Author      string            `json:"author"`
    Description string            `json:"description"`
    Tags        map[string]string `json:"tags"`
    Environment string            `json:"environment"`
}

// ConfigVersionManager manages configuration versions
type ConfigVersionManager struct {
    versions    map[string]*ConfigVersion
    currentID   string
    nextVersion int
    storageDir  string
}

func NewConfigVersionManager(storageDir string) *ConfigVersionManager {
    return &ConfigVersionManager{
        versions:    make(map[string]*ConfigVersion),
        nextVersion: 1,
        storageDir:  storageDir,
    }
}

func (cvm *ConfigVersionManager) CreateVersion(
    config map[string]interface{},
    metadata VersionMetadata,
) (*ConfigVersion, error) {
    
    var changes []ConfigChange
    var parentID string
    
    // Calculate changes from current version
    if cvm.currentID != "" {
        currentVersion := cvm.versions[cvm.currentID]
        changes = cvm.calculateChanges(currentVersion.Config, config)
        parentID = cvm.currentID
    }
    
    version := &ConfigVersion{
        ID:        fmt.Sprintf("v%d_%d", cvm.nextVersion, time.Now().Unix()),
        Version:   cvm.nextVersion,
        Config:    cvm.deepCopyConfig(config),
        Changes:   changes,
        Metadata:  metadata,
        Timestamp: time.Now(),
        ParentID:  parentID,
        Hash:      cvm.calculateHash(config),
    }
    
    // Store version
    cvm.versions[version.ID] = version
    cvm.currentID = version.ID
    cvm.nextVersion++
    
    // Persist to storage
    if err := cvm.saveVersion(version); err != nil {
        return nil, fmt.Errorf("failed to save version: %w", err)
    }
    
    fmt.Printf("Created configuration version %s (v%d)\n", version.ID, version.Version)
    return version, nil
}

func (cvm *ConfigVersionManager) calculateChanges(oldConfig, newConfig map[string]interface{}) []ConfigChange {
    var changes []ConfigChange
    
    // Find added and updated fields
    for key, newValue := range newConfig {
        if oldValue, exists := oldConfig[key]; exists {
            if !cvm.isEqual(oldValue, newValue) {
                changes = append(changes, ConfigChange{
                    Field:     key,
                    Operation: "update",
                    OldValue:  oldValue,
                    NewValue:  newValue,
                })
            }
        } else {
            changes = append(changes, ConfigChange{
                Field:     key,
                Operation: "add",
                NewValue:  newValue,
            })
        }
    }
    
    // Find deleted fields
    for key, oldValue := range oldConfig {
        if _, exists := newConfig[key]; !exists {
            changes = append(changes, ConfigChange{
                Field:     key,
                Operation: "delete",
                OldValue:  oldValue,
            })
        }
    }
    
    return changes
}

func (cvm *ConfigVersionManager) isEqual(a, b interface{}) bool {
    aJSON, _ := json.Marshal(a)
    bJSON, _ := json.Marshal(b)
    return string(aJSON) == string(bJSON)
}

func (cvm *ConfigVersionManager) deepCopyConfig(config map[string]interface{}) map[string]interface{} {
    data, _ := json.Marshal(config)
    var copy map[string]interface{}
    json.Unmarshal(data, &copy)
    return copy
}

func (cvm *ConfigVersionManager) calculateHash(config map[string]interface{}) string {
    data, _ := json.Marshal(config)
    hash := md5.Sum(data)
    return fmt.Sprintf("%x", hash)
}

func (cvm *ConfigVersionManager) GetVersion(versionID string) (*ConfigVersion, error) {
    version, exists := cvm.versions[versionID]
    if !exists {
        return nil, fmt.Errorf("version not found: %s", versionID)
    }
    return version, nil
}

func (cvm *ConfigVersionManager) GetCurrentVersion() *ConfigVersion {
    if cvm.currentID == "" {
        return nil
    }
    return cvm.versions[cvm.currentID]
}

func (cvm *ConfigVersionManager) ListVersions() []*ConfigVersion {
    versions := make([]*ConfigVersion, 0, len(cvm.versions))
    for _, version := range cvm.versions {
        versions = append(versions, version)
    }
    
    // Sort by version number
    sort.Slice(versions, func(i, j int) bool {
        return versions[i].Version > versions[j].Version
    })
    
    return versions
}

func (cvm *ConfigVersionManager) RollbackToVersion(versionID string) error {
    version, exists := cvm.versions[versionID]
    if !exists {
        return fmt.Errorf("version not found: %s", versionID)
    }
    
    // Create a new version based on the target version
    rollbackMetadata := VersionMetadata{
        Author:      "system",
        Description: fmt.Sprintf("Rollback to version %d", version.Version),
        Tags: map[string]string{
            "rollback": "true",
            "target":   versionID,
        },
        Environment: version.Metadata.Environment,
    }
    
    _, err := cvm.CreateVersion(version.Config, rollbackMetadata)
    if err != nil {
        return fmt.Errorf("failed to create rollback version: %w", err)
    }
    
    fmt.Printf("Rolled back to version %s\n", versionID)
    return nil
}

func (cvm *ConfigVersionManager) GetChangeHistory(field string) []ConfigChange {
    var history []ConfigChange
    
    versions := cvm.ListVersions()
    for _, version := range versions {
        for _, change := range version.Changes {
            if change.Field == field {
                history = append(history, change)
            }
        }
    }
    
    return history
}

func (cvm *ConfigVersionManager) CompareVersions(versionID1, versionID2 string) ([]ConfigChange, error) {
    version1, exists := cvm.versions[versionID1]
    if !exists {
        return nil, fmt.Errorf("version not found: %s", versionID1)
    }
    
    version2, exists := cvm.versions[versionID2]
    if !exists {
        return nil, fmt.Errorf("version not found: %s", versionID2)
    }
    
    return cvm.calculateChanges(version1.Config, version2.Config), nil
}

func (cvm *ConfigVersionManager) saveVersion(version *ConfigVersion) error {
    if cvm.storageDir == "" {
        return nil // In-memory only
    }
    
    os.MkdirAll(cvm.storageDir, 0755)
    filename := fmt.Sprintf("%s.json", version.ID)
    filepath := fmt.Sprintf("%s/%s", cvm.storageDir, filename)
    
    data, err := json.MarshalIndent(version, "", "  ")
    if err != nil {
        return err
    }
    
    return os.WriteFile(filepath, data, 0644)
}

func (cvm *ConfigVersionManager) LoadVersions() error {
    if cvm.storageDir == "" {
        return nil
    }
    
    entries, err := os.ReadDir(cvm.storageDir)
    if err != nil {
        return err
    }
    
    for _, entry := range entries {
        if !entry.IsDir() && strings.HasSuffix(entry.Name(), ".json") {
            filepath := fmt.Sprintf("%s/%s", cvm.storageDir, entry.Name())
            data, err := os.ReadFile(filepath)
            if err != nil {
                continue
            }
            
            var version ConfigVersion
            if err := json.Unmarshal(data, &version); err != nil {
                continue
            }
            
            cvm.versions[version.ID] = &version
            if version.Version >= cvm.nextVersion {
                cvm.nextVersion = version.Version + 1
                cvm.currentID = version.ID
            }
        }
    }
    
    return nil
}

func main() {
    fmt.Println("=== Configuration Versioning and Change Tracking Example ===")
    
    // Create version manager
    storageDir := "/tmp/config_versions"
    manager := NewConfigVersionManager(storageDir)
    
    // Create initial configuration
    fmt.Println("\n=== Creating Initial Configuration ===")
    initialConfig := map[string]interface{}{
        "app_name":    "VersionApp",
        "version":     "1.0.0",
        "server_port": 8080,
        "debug":       false,
        "features": map[string]interface{}{
            "metrics": false,
            "tracing": false,
        },
    }
    
    v1, _ := manager.CreateVersion(initialConfig, VersionMetadata{
        Author:      "developer",
        Description: "Initial configuration",
        Environment: "development",
        Tags: map[string]string{
            "milestone": "initial",
        },
    })
    
    // Update configuration
    fmt.Println("\n=== Updating Configuration ===")
    updatedConfig := map[string]interface{}{
        "app_name":    "VersionApp",
        "version":     "1.1.0",
        "server_port": 9090,
        "debug":       true,
        "features": map[string]interface{}{
            "metrics": true,
            "tracing": false,
            "caching": true,
        },
        "database_url": "postgres://localhost:5432/app",
    }
    
    v2, _ := manager.CreateVersion(updatedConfig, VersionMetadata{
        Author:      "developer",
        Description: "Enable metrics and update server port",
        Environment: "development",
        Tags: map[string]string{
            "feature": "metrics",
        },
    })
    
    // Another update
    fmt.Println("\n=== Another Update ===")
    finalConfig := map[string]interface{}{
        "app_name":    "VersionApp",
        "version":     "1.2.0",
        "server_port": 9090,
        "debug":       false,
        "features": map[string]interface{}{
            "metrics": true,
            "tracing": true,
            "caching": true,
        },
        "database_url": "postgres://prod-db:5432/app",
    }
    
    v3, _ := manager.CreateVersion(finalConfig, VersionMetadata{
        Author:      "ops-team",
        Description: "Production deployment with tracing",
        Environment: "production",
        Tags: map[string]string{
            "deployment": "production",
            "feature":    "tracing",
        },
    })
    
    // List all versions
    fmt.Println("\n=== Version History ===")
    versions := manager.ListVersions()
    for _, version := range versions {
        fmt.Printf("Version %d (%s):\n", version.Version, version.ID)
        fmt.Printf("  Author: %s\n", version.Metadata.Author)
        fmt.Printf("  Description: %s\n", version.Metadata.Description)
        fmt.Printf("  Timestamp: %s\n", version.Timestamp.Format(time.RFC3339))
        fmt.Printf("  Hash: %s\n", version.Hash)
        fmt.Printf("  Changes: %d\n", len(version.Changes))
        
        for _, change := range version.Changes {
            fmt.Printf("    %s %s", change.Operation, change.Field)
            if change.Operation == "update" {
                fmt.Printf(": %v -> %v", change.OldValue, change.NewValue)
            } else if change.Operation == "add" {
                fmt.Printf(": %v", change.NewValue)
            } else if change.Operation == "delete" {
                fmt.Printf(": %v", change.OldValue)
            }
            fmt.Println()
        }
        fmt.Println()
    }
    
    // Show change history for specific field
    fmt.Println("=== Change History for 'server_port' ===")
    portHistory := manager.GetChangeHistory("server_port")
    for _, change := range portHistory {
        fmt.Printf("%s: %v -> %v\n", change.Operation, change.OldValue, change.NewValue)
    }
    
    // Compare versions
    fmt.Println("\n=== Comparing Versions ===")
    changes, _ := manager.CompareVersions(v1.ID, v3.ID)
    fmt.Printf("Changes from v1 to v3:\n")
    for _, change := range changes {
        fmt.Printf("  %s %s", change.Operation, change.Field)
        if change.Operation == "update" {
            fmt.Printf(": %v -> %v", change.OldValue, change.NewValue)
        } else if change.Operation == "add" {
            fmt.Printf(": %v", change.NewValue)
        }
        fmt.Println()
    }
    
    // Rollback to previous version
    fmt.Println("\n=== Rolling Back ===")
    manager.RollbackToVersion(v2.ID)
    
    current := manager.GetCurrentVersion()
    fmt.Printf("Current version after rollback: %d\n", current.Version)
    fmt.Printf("Debug mode: %v\n", current.Config["debug"])
    
    // Clean up
    os.RemoveAll(storageDir)
    
    fmt.Println("\n=== Versioning Demo Complete ===")
}
```

This example demonstrates comprehensive configuration versioning with change  
tracking, diff calculation, rollback capabilities, and change history  
analysis to provide full visibility into configuration evolution.  

## Multi-tenant configuration management

Multi-tenant applications require isolated configuration management per  
tenant while maintaining shared defaults and efficient resource usage.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "sync"
    "time"
)

// TenantConfig represents configuration for a specific tenant
type TenantConfig struct {
    TenantID    string                 `json:"tenant_id"`
    Config      map[string]interface{} `json:"config"`
    Overrides   map[string]interface{} `json:"overrides"`
    LastUpdated time.Time              `json:"last_updated"`
    Version     int                    `json:"version"`
}

// MultiTenantConfigManager manages configuration for multiple tenants
type MultiTenantConfigManager struct {
    globalConfig  map[string]interface{}
    tenantConfigs map[string]*TenantConfig
    mutex         sync.RWMutex
    hierarchy     []string // Priority order for configuration resolution
}

func NewMultiTenantConfigManager() *MultiTenantConfigManager {
    return &MultiTenantConfigManager{
        globalConfig:  make(map[string]interface{}),
        tenantConfigs: make(map[string]*TenantConfig),
        hierarchy:     []string{"tenant", "global", "default"},
    }
}

func (mtcm *MultiTenantConfigManager) SetGlobalConfig(config map[string]interface{}) {
    mtcm.mutex.Lock()
    defer mtcm.mutex.Unlock()
    mtcm.globalConfig = config
}

func (mtcm *MultiTenantConfigManager) SetTenantConfig(tenantID string, config map[string]interface{}) {
    mtcm.mutex.Lock()
    defer mtcm.mutex.Unlock()
    
    tenantConfig, exists := mtcm.tenantConfigs[tenantID]
    if !exists {
        tenantConfig = &TenantConfig{
            TenantID:  tenantID,
            Config:    make(map[string]interface{}),
            Overrides: make(map[string]interface{}),
            Version:   1,
        }
        mtcm.tenantConfigs[tenantID] = tenantConfig
    } else {
        tenantConfig.Version++
    }
    
    tenantConfig.Config = config
    tenantConfig.LastUpdated = time.Now()
}

func (mtcm *MultiTenantConfigManager) SetTenantOverride(tenantID, key string, value interface{}) {
    mtcm.mutex.Lock()
    defer mtcm.mutex.Unlock()
    
    tenantConfig, exists := mtcm.tenantConfigs[tenantID]
    if !exists {
        tenantConfig = &TenantConfig{
            TenantID:  tenantID,
            Config:    make(map[string]interface{}),
            Overrides: make(map[string]interface{}),
            Version:   1,
        }
        mtcm.tenantConfigs[tenantID] = tenantConfig
    }
    
    tenantConfig.Overrides[key] = value
    tenantConfig.LastUpdated = time.Now()
}

func (mtcm *MultiTenantConfigManager) GetValue(tenantID, key string, defaultValue interface{}) interface{} {
    mtcm.mutex.RLock()
    defer mtcm.mutex.RUnlock()
    
    // Check tenant overrides first
    if tenantConfig, exists := mtcm.tenantConfigs[tenantID]; exists {
        if value, hasOverride := tenantConfig.Overrides[key]; hasOverride {
            return value
        }
        
        // Check tenant-specific config
        if value, hasConfig := tenantConfig.Config[key]; hasConfig {
            return value
        }
    }
    
    // Check global config
    if value, hasGlobal := mtcm.globalConfig[key]; hasGlobal {
        return value
    }
    
    return defaultValue
}

func (mtcm *MultiTenantConfigManager) GetAllConfig(tenantID string) map[string]interface{} {
    mtcm.mutex.RLock()
    defer mtcm.mutex.RUnlock()
    
    result := make(map[string]interface{})
    
    // Start with global config
    for key, value := range mtcm.globalConfig {
        result[key] = value
    }
    
    // Override with tenant-specific config
    if tenantConfig, exists := mtcm.tenantConfigs[tenantID]; exists {
        for key, value := range tenantConfig.Config {
            result[key] = value
        }
        
        // Override with tenant overrides
        for key, value := range tenantConfig.Overrides {
            result[key] = value
        }
    }
    
    return result
}

func (mtcm *MultiTenantConfigManager) ListTenants() []string {
    mtcm.mutex.RLock()
    defer mtcm.mutex.RUnlock()
    
    tenants := make([]string, 0, len(mtcm.tenantConfigs))
    for tenantID := range mtcm.tenantConfigs {
        tenants = append(tenants, tenantID)
    }
    return tenants
}

func (mtcm *MultiTenantConfigManager) GetTenantInfo(tenantID string) *TenantConfig {
    mtcm.mutex.RLock()
    defer mtcm.mutex.RUnlock()
    
    if config, exists := mtcm.tenantConfigs[tenantID]; exists {
        // Return a copy
        return &TenantConfig{
            TenantID:    config.TenantID,
            Config:      copyMap(config.Config),
            Overrides:   copyMap(config.Overrides),
            LastUpdated: config.LastUpdated,
            Version:     config.Version,
        }
    }
    return nil
}

func copyMap(original map[string]interface{}) map[string]interface{} {
    copy := make(map[string]interface{})
    for key, value := range original {
        copy[key] = value
    }
    return copy
}

func main() {
    fmt.Println("=== Multi-Tenant Configuration Management Example ===")
    
    manager := NewMultiTenantConfigManager()
    
    // Set global configuration
    fmt.Println("\n=== Setting Global Configuration ===")
    globalConfig := map[string]interface{}{
        "app_name":        "MultiTenantApp",
        "max_connections": 100,
        "timeout":         30,
        "features": map[string]interface{}{
            "logging": true,
            "metrics": true,
            "tracing": false,
        },
        "limits": map[string]interface{}{
            "api_calls_per_hour": 1000,
            "storage_gb":         10,
        },
    }
    
    manager.SetGlobalConfig(globalConfig)
    fmt.Println("Global configuration set")
    
    // Configure tenant-specific settings
    fmt.Println("\n=== Configuring Tenant-Specific Settings ===")
    
    // Enterprise tenant with higher limits
    enterpriseConfig := map[string]interface{}{
        "max_connections": 500,
        "features": map[string]interface{}{
            "logging":       true,
            "metrics":       true,
            "tracing":       true,
            "premium_apis":  true,
        },
        "limits": map[string]interface{}{
            "api_calls_per_hour": 10000,
            "storage_gb":         100,
        },
        "support_level": "enterprise",
    }
    manager.SetTenantConfig("enterprise-corp", enterpriseConfig)
    
    // Startup tenant with basic limits
    startupConfig := map[string]interface{}{
        "max_connections": 50,
        "limits": map[string]interface{}{
            "api_calls_per_hour": 500,
            "storage_gb":         5,
        },
        "support_level": "basic",
    }
    manager.SetTenantConfig("startup-inc", startupConfig)
    
    // Free tier tenant
    freeConfig := map[string]interface{}{
        "max_connections": 10,
        "features": map[string]interface{}{
            "logging": true,
            "metrics": false,
            "tracing": false,
        },
        "limits": map[string]interface{}{
            "api_calls_per_hour": 100,
            "storage_gb":         1,
        },
        "support_level": "community",
    }
    manager.SetTenantConfig("free-user", freeConfig)
    
    // Set some tenant-specific overrides
    fmt.Println("\n=== Setting Tenant Overrides ===")
    manager.SetTenantOverride("enterprise-corp", "timeout", 60)
    manager.SetTenantOverride("startup-inc", "timeout", 45)
    manager.SetTenantOverride("free-user", "max_connections", 5)
    
    // Test configuration resolution for each tenant
    fmt.Println("\n=== Configuration Resolution Testing ===")
    
    tenants := []string{"enterprise-corp", "startup-inc", "free-user", "new-tenant"}
    
    for _, tenantID := range tenants {
        fmt.Printf("\nTenant: %s\n", tenantID)
        fmt.Printf("  Max Connections: %v\n", manager.GetValue(tenantID, "max_connections", "default"))
        fmt.Printf("  Timeout: %v\n", manager.GetValue(tenantID, "timeout", 30))
        fmt.Printf("  API Calls/Hour: %v\n", manager.GetValue(tenantID, "limits.api_calls_per_hour", "unlimited"))
        fmt.Printf("  Storage GB: %v\n", manager.GetValue(tenantID, "limits.storage_gb", "unlimited"))
        fmt.Printf("  Support Level: %v\n", manager.GetValue(tenantID, "support_level", "none"))
        fmt.Printf("  Tracing Enabled: %v\n", manager.GetValue(tenantID, "features.tracing", false))
        
        if tenantInfo := manager.GetTenantInfo(tenantID); tenantInfo != nil {
            fmt.Printf("  Config Version: %d\n", tenantInfo.Version)
            fmt.Printf("  Last Updated: %s\n", tenantInfo.LastUpdated.Format(time.RFC3339))
            fmt.Printf("  Override Count: %d\n", len(tenantInfo.Overrides))
        }
    }
    
    // Show complete configuration for a tenant
    fmt.Println("\n=== Complete Configuration for Enterprise Tenant ===")
    enterpriseFullConfig := manager.GetAllConfig("enterprise-corp")
    data, _ := json.MarshalIndent(enterpriseFullConfig, "", "  ")
    fmt.Println(string(data))
    
    // List all tenants
    fmt.Println("\n=== All Configured Tenants ===")
    allTenants := manager.ListTenants()
    for _, tenantID := range allTenants {
        tenantInfo := manager.GetTenantInfo(tenantID)
        fmt.Printf("- %s (v%d, %d config items, %d overrides)\n", 
            tenantID, tenantInfo.Version, len(tenantInfo.Config), len(tenantInfo.Overrides))
    }
    
    // Demonstrate configuration hierarchy
    fmt.Println("\n=== Configuration Hierarchy Demo ===")
    testKey := "new_feature_enabled"
    
    fmt.Printf("Testing resolution for key: %s\n", testKey)
    for _, tenantID := range allTenants {
        value := manager.GetValue(tenantID, testKey, "default_value")
        fmt.Printf("  %s: %v\n", tenantID, value)
    }
    
    // Add global setting and test again
    fmt.Println("\nAdding global setting...")
    globalConfig[testKey] = true
    manager.SetGlobalConfig(globalConfig)
    
    for _, tenantID := range allTenants {
        value := manager.GetValue(tenantID, testKey, "default_value")
        fmt.Printf("  %s: %v\n", tenantID, value)
    }
    
    // Add tenant override and test
    fmt.Println("\nAdding tenant override for startup-inc...")
    manager.SetTenantOverride("startup-inc", testKey, false)
    
    for _, tenantID := range allTenants {
        value := manager.GetValue(tenantID, testKey, "default_value")
        fmt.Printf("  %s: %v\n", tenantID, value)
    }
    
    fmt.Println("\n=== Multi-Tenant Configuration Demo Complete ===")
}
```

This example demonstrates multi-tenant configuration management with  
tenant isolation, configuration hierarchy, override capabilities, and  
efficient resource sharing while maintaining tenant-specific customization.  

## Configuration performance profiling

Configuration performance profiling identifies bottlenecks and optimizes  
configuration access patterns for high-performance applications.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "math/rand"
    "os"
    "runtime"
    "sync"
    "time"
)

// ConfigProfiler profiles configuration access patterns and performance
type ConfigProfiler struct {
    accessCounts    map[string]int64
    accessTimes     map[string][]time.Duration
    totalAccesses   int64
    hotKeys         []string
    mutex           sync.RWMutex
    startTime       time.Time
    memStats        []MemorySnapshot
    concurrentTests map[string]*ConcurrentTest
}

type MemorySnapshot struct {
    Timestamp   time.Time `json:"timestamp"`
    HeapAlloc   uint64    `json:"heap_alloc"`
    HeapSys     uint64    `json:"heap_sys"`
    NumGC       uint32    `json:"num_gc"`
    GCCPUFraction float64 `json:"gc_cpu_fraction"`
}

type ConcurrentTest struct {
    Name        string        `json:"name"`
    Goroutines  int           `json:"goroutines"`
    Operations  int           `json:"operations"`
    Duration    time.Duration `json:"duration"`
    Throughput  float64       `json:"throughput"`
    Errors      int64         `json:"errors"`
}

type ProfileResult struct {
    TotalAccesses    int64                        `json:"total_accesses"`
    TestDuration     time.Duration                `json:"test_duration"`
    AverageAccessTime time.Duration               `json:"average_access_time"`
    TopKeys          []KeyStats                   `json:"top_keys"`
    MemoryStats      []MemorySnapshot             `json:"memory_stats"`
    ConcurrentTests  map[string]*ConcurrentTest   `json:"concurrent_tests"`
    Performance      PerformanceMetrics           `json:"performance"`
}

type KeyStats struct {
    Key          string        `json:"key"`
    AccessCount  int64         `json:"access_count"`
    AverageTime  time.Duration `json:"average_time"`
    TotalTime    time.Duration `json:"total_time"`
}

type PerformanceMetrics struct {
    AccessesPerSecond   float64 `json:"accesses_per_second"`
    MemoryEfficiency    float64 `json:"memory_efficiency"`
    CacheHitRatio      float64 `json:"cache_hit_ratio"`
    ConcurrencyScaling float64 `json:"concurrency_scaling"`
}

func NewConfigProfiler() *ConfigProfiler {
    return &ConfigProfiler{
        accessCounts:    make(map[string]int64),
        accessTimes:     make(map[string][]time.Duration),
        startTime:       time.Now(),
        memStats:        make([]MemorySnapshot, 0),
        concurrentTests: make(map[string]*ConcurrentTest),
    }
}

func (cp *ConfigProfiler) RecordAccess(key string, duration time.Duration) {
    cp.mutex.Lock()
    defer cp.mutex.Unlock()
    
    cp.accessCounts[key]++
    cp.totalAccesses++
    
    if _, exists := cp.accessTimes[key]; !exists {
        cp.accessTimes[key] = make([]time.Duration, 0)
    }
    cp.accessTimes[key] = append(cp.accessTimes[key], duration)
    
    // Keep only last 1000 measurements per key to manage memory
    if len(cp.accessTimes[key]) > 1000 {
        cp.accessTimes[key] = cp.accessTimes[key][len(cp.accessTimes[key])-1000:]
    }
}

func (cp *ConfigProfiler) TakeMemorySnapshot() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    cp.mutex.Lock()
    defer cp.mutex.Unlock()
    
    snapshot := MemorySnapshot{
        Timestamp:     time.Now(),
        HeapAlloc:     m.HeapAlloc,
        HeapSys:       m.HeapSys,
        NumGC:         m.NumGC,
        GCCPUFraction: m.GCCPUFraction,
    }
    
    cp.memStats = append(cp.memStats, snapshot)
}

func (cp *ConfigProfiler) RunConcurrentTest(name string, goroutines, operationsPerGoroutine int, testFunc func()) {
    test := &ConcurrentTest{
        Name:       name,
        Goroutines: goroutines,
        Operations: goroutines * operationsPerGoroutine,
    }
    
    start := time.Now()
    var wg sync.WaitGroup
    var errors int64
    
    for i := 0; i < goroutines; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            
            for j := 0; j < operationsPerGoroutine; j++ {
                func() {
                    defer func() {
                        if r := recover(); r != nil {
                            errors++
                        }
                    }()
                    testFunc()
                }()
            }
        }()
    }
    
    wg.Wait()
    test.Duration = time.Since(start)
    test.Throughput = float64(test.Operations) / test.Duration.Seconds()
    test.Errors = errors
    
    cp.mutex.Lock()
    cp.concurrentTests[name] = test
    cp.mutex.Unlock()
}

func (cp *ConfigProfiler) GenerateReport() *ProfileResult {
    cp.mutex.RLock()
    defer cp.mutex.RUnlock()
    
    result := &ProfileResult{
        TotalAccesses:   cp.totalAccesses,
        TestDuration:    time.Since(cp.startTime),
        TopKeys:         make([]KeyStats, 0),
        MemoryStats:     make([]MemorySnapshot, len(cp.memStats)),
        ConcurrentTests: make(map[string]*ConcurrentTest),
    }
    
    // Copy memory stats
    copy(result.MemoryStats, cp.memStats)
    
    // Copy concurrent tests
    for name, test := range cp.concurrentTests {
        result.ConcurrentTests[name] = test
    }
    
    // Calculate key statistics
    var totalTime time.Duration
    for key, count := range cp.accessCounts {
        keyTimes := cp.accessTimes[key]
        var keyTotalTime time.Duration
        for _, t := range keyTimes {
            keyTotalTime += t
        }
        
        avgTime := keyTotalTime / time.Duration(len(keyTimes))
        totalTime += keyTotalTime
        
        result.TopKeys = append(result.TopKeys, KeyStats{
            Key:         key,
            AccessCount: count,
            AverageTime: avgTime,
            TotalTime:   keyTotalTime,
        })
    }
    
    // Sort top keys by access count
    for i := 0; i < len(result.TopKeys)-1; i++ {
        for j := i + 1; j < len(result.TopKeys); j++ {
            if result.TopKeys[j].AccessCount > result.TopKeys[i].AccessCount {
                result.TopKeys[i], result.TopKeys[j] = result.TopKeys[j], result.TopKeys[i]
            }
        }
    }
    
    // Keep only top 10
    if len(result.TopKeys) > 10 {
        result.TopKeys = result.TopKeys[:10]
    }
    
    // Calculate average access time
    if cp.totalAccesses > 0 {
        result.AverageAccessTime = totalTime / time.Duration(cp.totalAccesses)
    }
    
    // Calculate performance metrics
    result.Performance = PerformanceMetrics{
        AccessesPerSecond:  float64(cp.totalAccesses) / result.TestDuration.Seconds(),
        MemoryEfficiency:   cp.calculateMemoryEfficiency(),
        CacheHitRatio:     0.95, // Simulated
        ConcurrencyScaling: cp.calculateConcurrencyScaling(),
    }
    
    return result
}

func (cp *ConfigProfiler) calculateMemoryEfficiency() float64 {
    if len(cp.memStats) < 2 {
        return 1.0
    }
    
    first := cp.memStats[0]
    last := cp.memStats[len(cp.memStats)-1]
    
    memoryGrowth := float64(last.HeapAlloc - first.HeapAlloc)
    accessGrowth := float64(cp.totalAccesses)
    
    if accessGrowth == 0 {
        return 1.0
    }
    
    // Lower values are better (less memory per access)
    efficiency := 1.0 - (memoryGrowth / accessGrowth / 1000.0) // Normalize
    if efficiency < 0 {
        efficiency = 0
    }
    return efficiency
}

func (cp *ConfigProfiler) calculateConcurrencyScaling() float64 {
    if len(cp.concurrentTests) < 2 {
        return 1.0
    }
    
    // Find single-threaded and multi-threaded tests
    var singleThreaded, multiThreaded *ConcurrentTest
    
    for _, test := range cp.concurrentTests {
        if test.Goroutines == 1 {
            singleThreaded = test
        } else if multiThreaded == nil || test.Goroutines > multiThreaded.Goroutines {
            multiThreaded = test
        }
    }
    
    if singleThreaded == nil || multiThreaded == nil {
        return 1.0
    }
    
    // Calculate scaling factor
    expectedScaling := float64(multiThreaded.Goroutines)
    actualScaling := multiThreaded.Throughput / singleThreaded.Throughput
    
    return actualScaling / expectedScaling
}

// Mock configuration manager for testing
type MockConfigManager struct {
    config map[string]interface{}
    mutex  sync.RWMutex
}

func NewMockConfigManager() *MockConfigManager {
    return &MockConfigManager{
        config: map[string]interface{}{
            "app_name":      "ProfileApp",
            "server_port":   8080,
            "database_url":  "postgres://localhost:5432/app",
            "cache_enabled": true,
            "log_level":     "info",
            "max_workers":   10,
            "timeout":       30,
            "features": map[string]interface{}{
                "metrics": true,
                "tracing": false,
            },
        },
    }
}

func (mcm *MockConfigManager) Get(key string) interface{} {
    mcm.mutex.RLock()
    defer mcm.mutex.RUnlock()
    
    // Simulate some processing time
    time.Sleep(time.Microsecond * time.Duration(rand.Intn(100)))
    
    return mcm.config[key]
}

func main() {
    fmt.Println("=== Configuration Performance Profiling Example ===")
    
    profiler := NewConfigProfiler()
    configManager := NewMockConfigManager()
    
    // Test keys with different access patterns
    hotKeys := []string{"app_name", "server_port", "cache_enabled"}
    warmKeys := []string{"database_url", "log_level", "max_workers"}
    coldKeys := []string{"timeout", "features"}
    
    fmt.Println("\n=== Running Performance Tests ===")
    
    // Memory snapshot before tests
    profiler.TakeMemorySnapshot()
    
    // Test 1: Single-threaded access pattern
    fmt.Println("Running single-threaded test...")
    profiler.RunConcurrentTest("single_threaded", 1, 10000, func() {
        key := hotKeys[rand.Intn(len(hotKeys))]
        start := time.Now()
        configManager.Get(key)
        profiler.RecordAccess(key, time.Since(start))
    })
    
    // Test 2: Multi-threaded access pattern
    fmt.Println("Running multi-threaded test...")
    profiler.RunConcurrentTest("multi_threaded", 10, 1000, func() {
        var key string
        switch rand.Intn(10) {
        case 0, 1, 2, 3, 4, 5: // 60% hot keys
            key = hotKeys[rand.Intn(len(hotKeys))]
        case 6, 7, 8: // 30% warm keys
            key = warmKeys[rand.Intn(len(warmKeys))]
        case 9: // 10% cold keys
            key = coldKeys[rand.Intn(len(coldKeys))]
        }
        
        start := time.Now()
        configManager.Get(key)
        profiler.RecordAccess(key, time.Since(start))
    })
    
    // Test 3: High concurrency test
    fmt.Println("Running high concurrency test...")
    profiler.RunConcurrentTest("high_concurrency", 100, 100, func() {
        key := hotKeys[rand.Intn(len(hotKeys))]
        start := time.Now()
        configManager.Get(key)
        profiler.RecordAccess(key, time.Since(start))
    })
    
    // Take periodic memory snapshots
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        profiler.TakeMemorySnapshot()
    }
    
    // Test 4: Mixed workload
    fmt.Println("Running mixed workload test...")
    profiler.RunConcurrentTest("mixed_workload", 20, 500, func() {
        allKeys := append(append(hotKeys, warmKeys...), coldKeys...)
        key := allKeys[rand.Intn(len(allKeys))]
        start := time.Now()
        configManager.Get(key)
        profiler.RecordAccess(key, time.Since(start))
    })
    
    // Final memory snapshot
    profiler.TakeMemorySnapshot()
    
    // Generate and display report
    fmt.Println("\n=== Performance Analysis Report ===")
    report := profiler.GenerateReport()
    
    fmt.Printf("Total Accesses: %d\n", report.TotalAccesses)
    fmt.Printf("Test Duration: %v\n", report.TestDuration)
    fmt.Printf("Average Access Time: %v\n", report.AverageAccessTime)
    fmt.Printf("Accesses per Second: %.2f\n", report.Performance.AccessesPerSecond)
    fmt.Printf("Memory Efficiency: %.2f\n", report.Performance.MemoryEfficiency)
    fmt.Printf("Cache Hit Ratio: %.2f\n", report.Performance.CacheHitRatio)
    fmt.Printf("Concurrency Scaling: %.2f\n", report.Performance.ConcurrencyScaling)
    
    fmt.Println("\n=== Top Accessed Keys ===")
    for i, keyStats := range report.TopKeys {
        fmt.Printf("%d. %s: %d accesses, avg: %v, total: %v\n",
            i+1, keyStats.Key, keyStats.AccessCount, 
            keyStats.AverageTime, keyStats.TotalTime)
    }
    
    fmt.Println("\n=== Concurrent Test Results ===")
    for name, test := range report.ConcurrentTests {
        fmt.Printf("%s:\n", name)
        fmt.Printf("  Goroutines: %d\n", test.Goroutines)
        fmt.Printf("  Operations: %d\n", test.Operations)
        fmt.Printf("  Duration: %v\n", test.Duration)
        fmt.Printf("  Throughput: %.2f ops/sec\n", test.Throughput)
        fmt.Printf("  Errors: %d\n", test.Errors)
        fmt.Println()
    }
    
    fmt.Println("=== Memory Usage Over Time ===")
    for i, snapshot := range report.MemoryStats {
        if i%2 == 0 { // Show every other snapshot
            fmt.Printf("T+%.1fs: Heap: %.2f MB, GC: %d\n",
                snapshot.Timestamp.Sub(report.MemoryStats[0].Timestamp).Seconds(),
                float64(snapshot.HeapAlloc)/1024/1024,
                snapshot.NumGC)
        }
    }
    
    // Save detailed report to file
    reportFile := "/tmp/config_performance_report.json"
    if data, err := json.MarshalIndent(report, "", "  "); err == nil {
        os.WriteFile(reportFile, data, 0644)
        fmt.Printf("\nDetailed report saved to: %s\n", reportFile)
    }
    
    // Performance recommendations
    fmt.Println("\n=== Performance Recommendations ===")
    if report.Performance.AccessesPerSecond < 10000 {
        fmt.Println("- Consider implementing configuration caching")
    }
    if report.Performance.ConcurrencyScaling < 0.7 {
        fmt.Println("- Configuration access may have concurrency bottlenecks")
    }
    if report.Performance.MemoryEfficiency < 0.8 {
        fmt.Println("- Memory usage could be optimized")
    }
    
    hotKeyThreshold := report.TotalAccesses / 5 // Top 20% threshold
    fmt.Printf("- Hot keys (>%d accesses) should be prioritized for caching\n", hotKeyThreshold)
    
    // Clean up
    os.Remove(reportFile)
    
    fmt.Println("\n=== Performance Profiling Demo Complete ===")
}
```

This example demonstrates comprehensive configuration performance profiling  
including access pattern analysis, concurrency testing, memory profiling,  
and performance optimization recommendations for high-performance  
configuration management.  

## Real-time configuration synchronization

Real-time configuration synchronization ensures configuration consistency  
across distributed systems with minimal latency and conflict resolution.  

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "sync"
    "time"
)

// ConfigSyncNode represents a node in the distributed configuration system
type ConfigSyncNode struct {
    nodeID      string
    config      map[string]interface{}
    version     int64
    peers       map[string]*ConfigSyncNode
    changeLog   []ConfigChange
    subscribers []chan ConfigUpdate
    mutex       sync.RWMutex
    syncChan    chan ConfigSync
    running     bool
}

type ConfigChange struct {
    NodeID    string      `json:"node_id"`
    Key       string      `json:"key"`
    Value     interface{} `json:"value"`
    Version   int64       `json:"version"`
    Timestamp time.Time   `json:"timestamp"`
    Operation string      `json:"operation"` // "set", "delete"
}

type ConfigUpdate struct {
    Key       string      `json:"key"`
    Value     interface{} `json:"value"`
    OldValue  interface{} `json:"old_value"`
    Version   int64       `json:"version"`
    Source    string      `json:"source"`
    Operation string      `json:"operation"`
}

type ConfigSync struct {
    FromNode string        `json:"from_node"`
    ToNode   string        `json:"to_node"`
    Changes  []ConfigChange `json:"changes"`
    Version  int64         `json:"version"`
}

func NewConfigSyncNode(nodeID string) *ConfigSyncNode {
    return &ConfigSyncNode{
        nodeID:      nodeID,
        config:      make(map[string]interface{}),
        version:     0,
        peers:       make(map[string]*ConfigSyncNode),
        changeLog:   make([]ConfigChange, 0),
        subscribers: make([]chan ConfigUpdate, 0),
        syncChan:    make(chan ConfigSync, 100),
    }
}

func (csn *ConfigSyncNode) Start() {
    csn.mutex.Lock()
    if csn.running {
        csn.mutex.Unlock()
        return
    }
    csn.running = true
    csn.mutex.Unlock()
    
    go csn.syncLoop()
}

func (csn *ConfigSyncNode) Stop() {
    csn.mutex.Lock()
    csn.running = false
    csn.mutex.Unlock()
    close(csn.syncChan)
}

func (csn *ConfigSyncNode) AddPeer(peer *ConfigSyncNode) {
    csn.mutex.Lock()
    defer csn.mutex.Unlock()
    csn.peers[peer.nodeID] = peer
}

func (csn *ConfigSyncNode) Subscribe() chan ConfigUpdate {
    csn.mutex.Lock()
    defer csn.mutex.Unlock()
    
    updateChan := make(chan ConfigUpdate, 10)
    csn.subscribers = append(csn.subscribers, updateChan)
    return updateChan
}

func (csn *ConfigSyncNode) Set(key string, value interface{}) {
    csn.mutex.Lock()
    defer csn.mutex.Unlock()
    
    oldValue := csn.config[key]
    csn.config[key] = value
    csn.version++
    
    change := ConfigChange{
        NodeID:    csn.nodeID,
        Key:       key,
        Value:     value,
        Version:   csn.version,
        Timestamp: time.Now(),
        Operation: "set",
    }
    
    csn.changeLog = append(csn.changeLog, change)
    
    // Notify subscribers
    update := ConfigUpdate{
        Key:       key,
        Value:     value,
        OldValue:  oldValue,
        Version:   csn.version,
        Source:    csn.nodeID,
        Operation: "set",
    }
    
    csn.notifySubscribers(update)
    
    // Propagate to peers
    go csn.propagateChange(change)
}

func (csn *ConfigSyncNode) Delete(key string) {
    csn.mutex.Lock()
    defer csn.mutex.Unlock()
    
    oldValue, exists := csn.config[key]
    if !exists {
        return
    }
    
    delete(csn.config, key)
    csn.version++
    
    change := ConfigChange{
        NodeID:    csn.nodeID,
        Key:       key,
        Value:     nil,
        Version:   csn.version,
        Timestamp: time.Now(),
        Operation: "delete",
    }
    
    csn.changeLog = append(csn.changeLog, change)
    
    // Notify subscribers
    update := ConfigUpdate{
        Key:       key,
        Value:     nil,
        OldValue:  oldValue,
        Version:   csn.version,
        Source:    csn.nodeID,
        Operation: "delete",
    }
    
    csn.notifySubscribers(update)
    
    // Propagate to peers
    go csn.propagateChange(change)
}

func (csn *ConfigSyncNode) Get(key string) (interface{}, bool) {
    csn.mutex.RLock()
    defer csn.mutex.RUnlock()
    
    value, exists := csn.config[key]
    return value, exists
}

func (csn *ConfigSyncNode) GetAll() map[string]interface{} {
    csn.mutex.RLock()
    defer csn.mutex.RUnlock()
    
    result := make(map[string]interface{})
    for key, value := range csn.config {
        result[key] = value
    }
    return result
}

func (csn *ConfigSyncNode) GetVersion() int64 {
    csn.mutex.RLock()
    defer csn.mutex.RUnlock()
    return csn.version
}

func (csn *ConfigSyncNode) propagateChange(change ConfigChange) {
    csn.mutex.RLock()
    peers := make([]*ConfigSyncNode, 0, len(csn.peers))
    for _, peer := range csn.peers {
        peers = append(peers, peer)
    }
    csn.mutex.RUnlock()
    
    sync := ConfigSync{
        FromNode: csn.nodeID,
        Changes:  []ConfigChange{change},
        Version:  change.Version,
    }
    
    for _, peer := range peers {
        sync.ToNode = peer.nodeID
        select {
        case peer.syncChan <- sync:
        default:
            fmt.Printf("Warning: sync channel full for peer %s\n", peer.nodeID)
        }
    }
}

func (csn *ConfigSyncNode) syncLoop() {
    for sync := range csn.syncChan {
        csn.processSyncMessage(sync)
    }
}

func (csn *ConfigSyncNode) processSyncMessage(sync ConfigSync) {
    csn.mutex.Lock()
    defer csn.mutex.Unlock()
    
    for _, change := range sync.Changes {
        // Avoid processing our own changes
        if change.NodeID == csn.nodeID {
            continue
        }
        
        // Check if we already have this change (conflict resolution)
        if csn.hasChange(change) {
            continue
        }
        
        // Apply change with conflict resolution
        csn.applyChange(change)
    }
}

func (csn *ConfigSyncNode) hasChange(change ConfigChange) bool {
    for _, existingChange := range csn.changeLog {
        if existingChange.NodeID == change.NodeID &&
           existingChange.Key == change.Key &&
           existingChange.Version == change.Version {
            return true
        }
    }
    return false
}

func (csn *ConfigSyncNode) applyChange(change ConfigChange) {
    oldValue := csn.config[change.Key]
    
    // Conflict resolution: last-write-wins with timestamp tiebreaker
    shouldApply := true
    
    for _, existingChange := range csn.changeLog {
        if existingChange.Key == change.Key {
            if existingChange.Timestamp.After(change.Timestamp) {
                shouldApply = false
                break
            } else if existingChange.Timestamp.Equal(change.Timestamp) {
                // Tiebreaker: use lexicographically later node ID
                if existingChange.NodeID > change.NodeID {
                    shouldApply = false
                    break
                }
            }
        }
    }
    
    if !shouldApply {
        return
    }
    
    // Apply the change
    switch change.Operation {
    case "set":
        csn.config[change.Key] = change.Value
    case "delete":
        delete(csn.config, change.Key)
    }
    
    // Update our version if the change is newer
    if change.Version > csn.version {
        csn.version = change.Version
    }
    
    // Add to change log
    csn.changeLog = append(csn.changeLog, change)
    
    // Notify subscribers
    update := ConfigUpdate{
        Key:       change.Key,
        Value:     change.Value,
        OldValue:  oldValue,
        Version:   change.Version,
        Source:    change.NodeID,
        Operation: change.Operation,
    }
    
    csn.notifySubscribers(update)
}

func (csn *ConfigSyncNode) notifySubscribers(update ConfigUpdate) {
    for _, subscriber := range csn.subscribers {
        select {
        case subscriber <- update:
        default:
            // Subscriber channel full, skip
        }
    }
}

func (csn *ConfigSyncNode) GetChangeLog() []ConfigChange {
    csn.mutex.RLock()
    defer csn.mutex.RUnlock()
    
    log := make([]ConfigChange, len(csn.changeLog))
    copy(log, csn.changeLog)
    return log
}

func main() {
    fmt.Println("=== Real-time Configuration Synchronization Example ===")
    
    // Create a cluster of configuration nodes
    node1 := NewConfigSyncNode("node-1")
    node2 := NewConfigSyncNode("node-2")
    node3 := NewConfigSyncNode("node-3")
    
    // Set up peer relationships
    node1.AddPeer(node2)
    node1.AddPeer(node3)
    node2.AddPeer(node1)
    node2.AddPeer(node3)
    node3.AddPeer(node1)
    node3.AddPeer(node2)
    
    // Start all nodes
    node1.Start()
    node2.Start()
    node3.Start()
    
    defer func() {
        node1.Stop()
        node2.Stop()
        node3.Stop()
    }()
    
    // Subscribe to changes on node2
    updates := node2.Subscribe()
    go func() {
        fmt.Println("\n=== Change Notifications (Node 2) ===")
        for update := range updates {
            fmt.Printf("Update: %s = %v (from %s, v%d)\n", 
                update.Key, update.Value, update.Source, update.Version)
        }
    }()
    
    // Test configuration synchronization
    fmt.Println("\n=== Testing Configuration Synchronization ===")
    
    // Node 1 sets some configuration
    fmt.Println("Node 1 setting initial configuration...")
    node1.Set("app_name", "SyncApp")
    node1.Set("version", "1.0.0")
    node1.Set("debug", true)
    
    time.Sleep(100 * time.Millisecond) // Allow propagation
    
    // Check if changes propagated
    fmt.Println("\nConfiguration state after Node 1 updates:")
    printNodeState("Node 1", node1)
    printNodeState("Node 2", node2)
    printNodeState("Node 3", node3)
    
    // Node 2 updates configuration
    fmt.Println("\nNode 2 updating configuration...")
    node2.Set("version", "1.1.0")
    node2.Set("max_connections", 100)
    
    time.Sleep(100 * time.Millisecond)
    
    fmt.Println("\nConfiguration state after Node 2 updates:")
    printNodeState("Node 1", node1)
    printNodeState("Node 2", node2)
    printNodeState("Node 3", node3)
    
    // Node 3 deletes a key
    fmt.Println("\nNode 3 deleting debug setting...")
    node3.Delete("debug")
    
    time.Sleep(100 * time.Millisecond)
    
    fmt.Println("\nConfiguration state after Node 3 deletion:")
    printNodeState("Node 1", node1)
    printNodeState("Node 2", node2)
    printNodeState("Node 3", node3)
    
    // Test concurrent updates (conflict resolution)
    fmt.Println("\n=== Testing Conflict Resolution ===")
    
    // Simulate near-simultaneous updates to the same key
    go func() {
        node1.Set("concurrent_test", "from_node1")
    }()
    go func() {
        node2.Set("concurrent_test", "from_node2")
    }()
    go func() {
        node3.Set("concurrent_test", "from_node3")
    }()
    
    time.Sleep(200 * time.Millisecond)
    
    fmt.Println("Final state after concurrent updates:")
    printNodeState("Node 1", node1)
    printNodeState("Node 2", node2)
    printNodeState("Node 3", node3)
    
    // Show change logs
    fmt.Println("\n=== Change Logs ===")
    
    nodes := map[string]*ConfigSyncNode{
        "Node 1": node1,
        "Node 2": node2,
        "Node 3": node3,
    }
    
    for name, node := range nodes {
        fmt.Printf("\n%s Change Log:\n", name)
        changeLog := node.GetChangeLog()
        for i, change := range changeLog {
            fmt.Printf("  %d. %s %s=%v by %s (v%d) at %s\n",
                i+1, change.Operation, change.Key, change.Value,
                change.NodeID, change.Version,
                change.Timestamp.Format("15:04:05.000"))
        }
    }
    
    // Test network partition simulation
    fmt.Println("\n=== Simulating Network Partition ===")
    
    // Remove peer connections to simulate partition
    node1.mutex.Lock()
    delete(node1.peers, "node-2")
    delete(node1.peers, "node-3")
    node1.mutex.Unlock()
    
    // Node 1 makes changes while partitioned
    node1.Set("partition_test", "isolated_update")
    
    // Nodes 2 and 3 continue to sync with each other
    node2.Set("partition_test", "connected_update")
    
    time.Sleep(100 * time.Millisecond)
    
    fmt.Println("During partition:")
    printNodeState("Node 1 (isolated)", node1)
    printNodeState("Node 2 (connected)", node2)
    printNodeState("Node 3 (connected)", node3)
    
    // Restore connections (heal partition)
    fmt.Println("\nHealing network partition...")
    node1.AddPeer(node2)
    node1.AddPeer(node3)
    
    // Trigger sync by making a new change
    node1.Set("heal_trigger", "sync_now")
    
    time.Sleep(200 * time.Millisecond)
    
    fmt.Println("After healing partition:")
    printNodeState("Node 1", node1)
    printNodeState("Node 2", node2)
    printNodeState("Node 3", node3)
    
    fmt.Println("\n=== Synchronization Demo Complete ===")
}

func printNodeState(name string, node *ConfigSyncNode) {
    config := node.GetAll()
    version := node.GetVersion()
    
    fmt.Printf("%s (v%d): ", name, version)
    
    if len(config) == 0 {
        fmt.Println("(empty)")
        return
    }
    
    configJSON, _ := json.Marshal(config)
    fmt.Println(string(configJSON))
}
```

This example demonstrates real-time configuration synchronization across  
distributed nodes with conflict resolution, network partition handling,  
and eventual consistency guarantees for robust distributed configuration  
management.  

## Configuration compliance and governance

Configuration compliance ensures applications meet organizational policies  
and regulatory requirements through automated validation and reporting.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "regexp"
    "strings"
    "time"
)

// ComplianceRule defines a configuration compliance rule
type ComplianceRule struct {
    ID          string      `json:"id"`
    Name        string      `json:"name"`
    Description string      `json:"description"`
    Category    string      `json:"category"`
    Severity    string      `json:"severity"`
    Condition   Condition   `json:"condition"`
    Remediation string      `json:"remediation"`
    Required    bool        `json:"required"`
    Tags        []string    `json:"tags"`
}

type Condition struct {
    Field    string      `json:"field"`
    Operator string      `json:"operator"`
    Value    interface{} `json:"value"`
    Pattern  string      `json:"pattern,omitempty"`
}

type ComplianceViolation struct {
    RuleID      string      `json:"rule_id"`
    RuleName    string      `json:"rule_name"`
    Severity    string      `json:"severity"`
    Field       string      `json:"field"`
    ActualValue interface{} `json:"actual_value"`
    ExpectedValue interface{} `json:"expected_value"`
    Message     string      `json:"message"`
    Remediation string      `json:"remediation"`
    Timestamp   time.Time   `json:"timestamp"`
}

type ComplianceReport struct {
    Timestamp        time.Time             `json:"timestamp"`
    TotalRules       int                   `json:"total_rules"`
    PassedRules      int                   `json:"passed_rules"`
    FailedRules      int                   `json:"failed_rules"`
    ComplianceScore  float64               `json:"compliance_score"`
    Violations       []ComplianceViolation `json:"violations"`
    Summary          ComplianceSummary     `json:"summary"`
}

type ComplianceSummary struct {
    CriticalViolations int               `json:"critical_violations"`
    HighViolations     int               `json:"high_violations"`
    MediumViolations   int               `json:"medium_violations"`
    LowViolations      int               `json:"low_violations"`
    CategoriesFailed   map[string]int    `json:"categories_failed"`
}

// ComplianceEngine evaluates configuration against compliance rules
type ComplianceEngine struct {
    rules map[string]ComplianceRule
}

func NewComplianceEngine() *ComplianceEngine {
    return &ComplianceEngine{
        rules: make(map[string]ComplianceRule),
    }
}

func (ce *ComplianceEngine) AddRule(rule ComplianceRule) {
    ce.rules[rule.ID] = rule
}

func (ce *ComplianceEngine) LoadStandardRules() {
    rules := []ComplianceRule{
        {
            ID:          "SEC-001",
            Name:        "TLS Required for Production",
            Description: "Production environments must use TLS encryption",
            Category:    "Security",
            Severity:    "Critical",
            Condition: Condition{
                Field:    "server.tls.enabled",
                Operator: "equals",
                Value:    true,
            },
            Remediation: "Enable TLS by setting server.tls.enabled to true",
            Required:    true,
            Tags:        []string{"security", "encryption", "production"},
        },
        {
            ID:          "SEC-002",
            Name:        "Debug Mode Disabled in Production",
            Description: "Debug mode must be disabled in production environments",
            Category:    "Security",
            Severity:    "High",
            Condition: Condition{
                Field:    "debug",
                Operator: "equals",
                Value:    false,
            },
            Remediation: "Set debug to false for production deployments",
            Required:    true,
            Tags:        []string{"security", "production"},
        },
        {
            ID:          "SEC-003",
            Name:        "Strong Database Passwords",
            Description: "Database passwords must meet complexity requirements",
            Category:    "Security",
            Severity:    "High",
            Condition: Condition{
                Field:    "database.password",
                Operator: "matches",
                Pattern:  `^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{12,}$`,
            },
            Remediation: "Use passwords with 12+ chars, including uppercase, lowercase, numbers, and special characters",
            Required:    true,
            Tags:        []string{"security", "passwords"},
        },
        {
            ID:          "PERF-001",
            Name:        "Connection Pool Limits",
            Description: "Database connection pools should have reasonable limits",
            Category:    "Performance",
            Severity:    "Medium",
            Condition: Condition{
                Field:    "database.max_connections",
                Operator: "less_than_or_equal",
                Value:    1000,
            },
            Remediation: "Set database.max_connections to a reasonable value (≤1000)",
            Required:    false,
            Tags:        []string{"performance", "database"},
        },
        {
            ID:          "OPS-001",
            Name:        "Logging Level Compliance",
            Description: "Production systems should use appropriate log levels",
            Category:    "Operations",
            Severity:    "Medium",
            Condition: Condition{
                Field:    "logging.level",
                Operator: "in",
                Value:    []string{"info", "warn", "error"},
            },
            Remediation: "Set logging.level to info, warn, or error for production",
            Required:    false,
            Tags:        []string{"operations", "logging"},
        },
        {
            ID:          "COMP-001",
            Name:        "Required Application Metadata",
            Description: "Applications must have name and version specified",
            Category:    "Compliance",
            Severity:    "Medium",
            Condition: Condition{
                Field:    "app.name",
                Operator: "exists",
            },
            Remediation: "Specify app.name and app.version in configuration",
            Required:    true,
            Tags:        []string{"compliance", "metadata"},
        },
    }
    
    for _, rule := range rules {
        ce.AddRule(rule)
    }
}

func (ce *ComplianceEngine) EvaluateCompliance(config map[string]interface{}, environment string) *ComplianceReport {
    report := &ComplianceReport{
        Timestamp:   time.Now(),
        TotalRules:  len(ce.rules),
        Violations:  make([]ComplianceViolation, 0),
        Summary: ComplianceSummary{
            CategoriesFailed: make(map[string]int),
        },
    }
    
    for _, rule := range ce.rules {
        // Skip non-required rules for non-production environments
        if !rule.Required && environment != "production" {
            continue
        }
        
        violation := ce.evaluateRule(rule, config, environment)
        if violation != nil {
            report.Violations = append(report.Violations, *violation)
            report.FailedRules++
            
            // Update summary counters
            switch violation.Severity {
            case "Critical":
                report.Summary.CriticalViolations++
            case "High":
                report.Summary.HighViolations++
            case "Medium":
                report.Summary.MediumViolations++
            case "Low":
                report.Summary.LowViolations++
            }
            
            report.Summary.CategoriesFailed[rule.Category]++
        } else {
            report.PassedRules++
        }
    }
    
    // Calculate compliance score
    if report.TotalRules > 0 {
        report.ComplianceScore = float64(report.PassedRules) / float64(report.TotalRules) * 100
    }
    
    return report
}

func (ce *ComplianceEngine) evaluateRule(rule ComplianceRule, config map[string]interface{}, environment string) *ComplianceViolation {
    value := ce.getNestedValue(config, rule.Condition.Field)
    
    passed := ce.evaluateCondition(rule.Condition, value)
    
    if !passed {
        return &ComplianceViolation{
            RuleID:        rule.ID,
            RuleName:      rule.Name,
            Severity:      rule.Severity,
            Field:         rule.Condition.Field,
            ActualValue:   value,
            ExpectedValue: rule.Condition.Value,
            Message:       fmt.Sprintf("Rule '%s' failed: %s", rule.Name, rule.Description),
            Remediation:   rule.Remediation,
            Timestamp:     time.Now(),
        }
    }
    
    return nil
}

func (ce *ComplianceEngine) evaluateCondition(condition Condition, value interface{}) bool {
    switch condition.Operator {
    case "exists":
        return value != nil
    case "equals":
        return ce.valuesEqual(value, condition.Value)
    case "not_equals":
        return !ce.valuesEqual(value, condition.Value)
    case "greater_than":
        return ce.compareValues(value, condition.Value) > 0
    case "less_than":
        return ce.compareValues(value, condition.Value) < 0
    case "greater_than_or_equal":
        return ce.compareValues(value, condition.Value) >= 0
    case "less_than_or_equal":
        return ce.compareValues(value, condition.Value) <= 0
    case "in":
        if slice, ok := condition.Value.([]interface{}); ok {
            for _, item := range slice {
                if ce.valuesEqual(value, item) {
                    return true
                }
            }
        } else if slice, ok := condition.Value.([]string); ok {
            if str, ok := value.(string); ok {
                for _, item := range slice {
                    if str == item {
                        return true
                    }
                }
            }
        }
        return false
    case "matches":
        if str, ok := value.(string); ok && condition.Pattern != "" {
            matched, err := regexp.MatchString(condition.Pattern, str)
            return err == nil && matched
        }
        return false
    default:
        return false
    }
}

func (ce *ComplianceEngine) getNestedValue(config map[string]interface{}, path string) interface{} {
    parts := strings.Split(path, ".")
    current := config
    
    for i, part := range parts {
        if i == len(parts)-1 {
            // Last part - return the value
            return current[part]
        }
        
        // Navigate deeper
        if next, ok := current[part].(map[string]interface{}); ok {
            current = next
        } else {
            return nil
        }
    }
    
    return nil
}

func (ce *ComplianceEngine) valuesEqual(a, b interface{}) bool {
    if a == nil && b == nil {
        return true
    }
    if a == nil || b == nil {
        return false
    }
    
    // Convert to JSON and compare strings for complex comparison
    aJSON, _ := json.Marshal(a)
    bJSON, _ := json.Marshal(b)
    return string(aJSON) == string(bJSON)
}

func (ce *ComplianceEngine) compareValues(a, b interface{}) int {
    switch va := a.(type) {
    case int:
        if vb, ok := b.(int); ok {
            if va > vb {
                return 1
            } else if va < vb {
                return -1
            }
            return 0
        }
    case float64:
        if vb, ok := b.(float64); ok {
            if va > vb {
                return 1
            } else if va < vb {
                return -1
            }
            return 0
        }
        if vb, ok := b.(int); ok {
            if va > float64(vb) {
                return 1
            } else if va < float64(vb) {
                return -1
            }
            return 0
        }
    case string:
        if vb, ok := b.(string); ok {
            return strings.Compare(va, vb)
        }
    }
    return 0
}

func main() {
    fmt.Println("=== Configuration Compliance and Governance Example ===")
    
    // Create compliance engine and load standard rules
    engine := NewComplianceEngine()
    engine.LoadStandardRules()
    
    fmt.Printf("Loaded %d compliance rules\n", len(engine.rules))
    
    // Test configurations for different environments
    configurations := map[string]map[string]interface{}{
        "development": {
            "app": map[string]interface{}{
                "name":    "TestApp",
                "version": "1.0.0",
            },
            "debug": true,
            "server": map[string]interface{}{
                "port": 8080,
                "tls": map[string]interface{}{
                    "enabled": false,
                },
            },
            "database": map[string]interface{}{
                "host":            "localhost",
                "port":            5432,
                "password":        "simple123",
                "max_connections": 10,
            },
            "logging": map[string]interface{}{
                "level": "debug",
            },
        },
        "production": {
            "app": map[string]interface{}{
                "name":    "TestApp",
                "version": "1.0.0",
            },
            "debug": false,
            "server": map[string]interface{}{
                "port": 443,
                "tls": map[string]interface{}{
                    "enabled": true,
                },
            },
            "database": map[string]interface{}{
                "host":            "prod-db.example.com",
                "port":            5432,
                "password":        "SecureP@ssw0rd123!",
                "max_connections": 100,
            },
            "logging": map[string]interface{}{
                "level": "info",
            },
        },
        "non_compliant": {
            "debug": true,  // Violation: debug enabled in production
            "server": map[string]interface{}{
                "port": 8080,
                "tls": map[string]interface{}{
                    "enabled": false,  // Violation: TLS disabled
                },
            },
            "database": map[string]interface{}{
                "password":        "weak",  // Violation: weak password
                "max_connections": 5000,     // Violation: too many connections
            },
            "logging": map[string]interface{}{
                "level": "debug",  // Violation: debug logging in production
            },
            // Missing app.name - violation
        },
    }
    
    // Evaluate compliance for each configuration
    for envName, config := range configurations {
        fmt.Printf("\n=== Compliance Report for %s Environment ===\n", strings.Title(envName))
        
        environment := envName
        if envName == "non_compliant" {
            environment = "production"  // Test as production for strictest rules
        }
        
        report := engine.EvaluateCompliance(config, environment)
        
        fmt.Printf("Compliance Score: %.1f%%\n", report.ComplianceScore)
        fmt.Printf("Rules Passed: %d/%d\n", report.PassedRules, report.TotalRules)
        
        if len(report.Violations) > 0 {
            fmt.Printf("\nViolations Found:\n")
            for i, violation := range report.Violations {
                fmt.Printf("%d. [%s] %s\n", i+1, violation.Severity, violation.RuleName)
                fmt.Printf("   Field: %s\n", violation.Field)
                fmt.Printf("   Current: %v\n", violation.ActualValue)
                if violation.ExpectedValue != nil {
                    fmt.Printf("   Expected: %v\n", violation.ExpectedValue)
                }
                fmt.Printf("   Remediation: %s\n", violation.Remediation)
                fmt.Println()
            }
            
            fmt.Printf("Summary:\n")
            fmt.Printf("  Critical: %d\n", report.Summary.CriticalViolations)
            fmt.Printf("  High: %d\n", report.Summary.HighViolations)
            fmt.Printf("  Medium: %d\n", report.Summary.MediumViolations)
            fmt.Printf("  Low: %d\n", report.Summary.LowViolations)
            
            if len(report.Summary.CategoriesFailed) > 0 {
                fmt.Printf("  Categories with failures:\n")
                for category, count := range report.Summary.CategoriesFailed {
                    fmt.Printf("    %s: %d violations\n", category, count)
                }
            }
        } else {
            fmt.Printf("✓ All compliance rules passed!\n")
        }
    }
    
    // Generate detailed compliance report
    fmt.Println("\n=== Detailed Compliance Analysis ===")
    
    prodReport := engine.EvaluateCompliance(configurations["production"], "production")
    nonCompliantReport := engine.EvaluateCompliance(configurations["non_compliant"], "production")
    
    fmt.Printf("Production Environment Compliance: %.1f%%\n", prodReport.ComplianceScore)
    fmt.Printf("Non-Compliant Configuration: %.1f%%\n", nonCompliantReport.ComplianceScore)
    
    improvement := prodReport.ComplianceScore - nonCompliantReport.ComplianceScore
    fmt.Printf("Compliance Improvement: +%.1f%%\n", improvement)
    
    // Save detailed report to file
    reportData, _ := json.MarshalIndent(nonCompliantReport, "", "  ")
    reportFile := "/tmp/compliance_report.json"
    os.WriteFile(reportFile, reportData, 0644)
    fmt.Printf("\nDetailed compliance report saved to: %s\n", reportFile)
    
    // Compliance recommendations
    fmt.Println("\n=== Compliance Recommendations ===")
    if nonCompliantReport.Summary.CriticalViolations > 0 {
        fmt.Println("⚠️  CRITICAL: Address critical violations immediately")
    }
    if nonCompliantReport.Summary.HighViolations > 0 {
        fmt.Println("⚠️  HIGH: Schedule high-priority violations for next release")
    }
    if nonCompliantReport.Summary.MediumViolations > 0 {
        fmt.Println("ℹ️  MEDIUM: Consider addressing medium violations in upcoming sprints")
    }
    
    fmt.Println("\nTop Priority Actions:")
    criticalAndHigh := 0
    for _, violation := range nonCompliantReport.Violations {
        if violation.Severity == "Critical" || violation.Severity == "High" {
            criticalAndHigh++
            if criticalAndHigh <= 3 {  // Show top 3
                fmt.Printf("%d. %s: %s\n", criticalAndHigh, violation.RuleName, violation.Remediation)
            }
        }
    }
    
    // Clean up
    os.Remove(reportFile)
    
    fmt.Println("\n=== Compliance and Governance Demo Complete ===")
}
```

This example demonstrates comprehensive configuration compliance and governance  
with automated rule evaluation, violation reporting, and remediation guidance  
to ensure configurations meet organizational and regulatory requirements.  

## Configuration A/B testing framework

A/B testing framework for configuration enables data-driven configuration  
decisions through controlled experiments and statistical analysis.  

```go
package main

import (
    "fmt"
    "math"
    "math/rand"
    "sort"
    "time"
)

// ABTest represents a configuration A/B test
type ABTest struct {
    ID          string                 `json:"id"`
    Name        string                 `json:"name"`
    Description string                 `json:"description"`
    ConfigKey   string                 `json:"config_key"`
    Variants    map[string]ABVariant   `json:"variants"`
    Traffic     TrafficAllocation      `json:"traffic"`
    Metrics     []string               `json:"metrics"`
    StartTime   time.Time              `json:"start_time"`
    EndTime     *time.Time             `json:"end_time,omitempty"`
    Status      string                 `json:"status"`
    Results     *ABTestResults         `json:"results,omitempty"`
}

type ABVariant struct {
    Name        string      `json:"name"`
    Value       interface{} `json:"value"`
    Description string      `json:"description"`
    Weight      float64     `json:"weight"`
}

type TrafficAllocation struct {
    TotalPercentage float64            `json:"total_percentage"`
    VariantSplit    map[string]float64 `json:"variant_split"`
}

type ABTestResults struct {
    TotalUsers      int64                      `json:"total_users"`
    VariantResults  map[string]*VariantResults `json:"variant_results"`
    WinningVariant  string                     `json:"winning_variant"`
    ConfidenceLevel float64                    `json:"confidence_level"`
    StatisticalPower float64                   `json:"statistical_power"`
    Recommendations []string                   `json:"recommendations"`
}

type VariantResults struct {
    UserCount     int64                  `json:"user_count"`
    Metrics       map[string]MetricStats `json:"metrics"`
    ConversionRate float64               `json:"conversion_rate"`
}

type MetricStats struct {
    Count    int64   `json:"count"`
    Sum      float64 `json:"sum"`
    Mean     float64 `json:"mean"`
    StdDev   float64 `json:"std_dev"`
    Min      float64 `json:"min"`
    Max      float64 `json:"max"`
    Samples  []float64 `json:"-"` // Not exported, for calculation
}

// ABTestFramework manages A/B tests for configuration
type ABTestFramework struct {
    tests   map[string]*ABTest
    metrics map[string]*MetricStats
    rand    *rand.Rand
}

func NewABTestFramework() *ABTestFramework {
    return &ABTestFramework{
        tests:   make(map[string]*ABTest),
        metrics: make(map[string]*MetricStats),
        rand:    rand.New(rand.NewSource(time.Now().UnixNano())),
    }
}

func (atf *ABTestFramework) CreateTest(test *ABTest) error {
    // Validate test configuration
    if err := atf.validateTest(test); err != nil {
        return fmt.Errorf("test validation failed: %w", err)
    }
    
    test.Status = "created"
    test.StartTime = time.Now()
    atf.tests[test.ID] = test
    
    fmt.Printf("Created A/B test: %s\n", test.Name)
    return nil
}

func (atf *ABTestFramework) validateTest(test *ABTest) error {
    if test.ID == "" || test.Name == "" || test.ConfigKey == "" {
        return fmt.Errorf("test ID, name, and config key are required")
    }
    
    if len(test.Variants) < 2 {
        return fmt.Errorf("at least 2 variants are required")
    }
    
    totalWeight := 0.0
    for _, variant := range test.Variants {
        totalWeight += variant.Weight
    }
    
    if math.Abs(totalWeight-1.0) > 0.001 {
        return fmt.Errorf("variant weights must sum to 1.0, got %.3f", totalWeight)
    }
    
    return nil
}

func (atf *ABTestFramework) StartTest(testID string) error {
    test, exists := atf.tests[testID]
    if !exists {
        return fmt.Errorf("test not found: %s", testID)
    }
    
    if test.Status != "created" {
        return fmt.Errorf("test cannot be started, current status: %s", test.Status)
    }
    
    test.Status = "running"
    test.StartTime = time.Now()
    
    fmt.Printf("Started A/B test: %s\n", test.Name)
    return nil
}

func (atf *ABTestFramework) GetVariantForUser(testID, userID string) (string, interface{}, error) {
    test, exists := atf.tests[testID]
    if !exists {
        return "", nil, fmt.Errorf("test not found: %s", testID)
    }
    
    if test.Status != "running" {
        return "", nil, fmt.Errorf("test is not running")
    }
    
    // Determine if user is in the test
    userHash := atf.hashUser(userID, testID)
    if userHash > test.Traffic.TotalPercentage {
        return "control", nil, nil // User not in test
    }
    
    // Assign variant based on weights
    variantHash := atf.hashUser(userID, testID+"_variant")
    
    var cumulativeWeight float64
    for variantName, variant := range test.Variants {
        cumulativeWeight += variant.Weight
        if variantHash <= cumulativeWeight {
            return variantName, variant.Value, nil
        }
    }
    
    // Fallback to first variant
    for variantName, variant := range test.Variants {
        return variantName, variant.Value, nil
    }
    
    return "", nil, fmt.Errorf("no variant assigned")
}

func (atf *ABTestFramework) hashUser(userID, salt string) float64 {
    // Simple hash function for demo - in production use crypto/hash
    hash := 0
    for _, char := range userID + salt {
        hash = (hash*31 + int(char)) % 1000000
    }
    return float64(hash%1000) / 1000.0
}

func (atf *ABTestFramework) RecordMetric(testID, userID, variantName, metricName string, value float64) {
    test, exists := atf.tests[testID]
    if !exists || test.Status != "running" {
        return
    }
    
    // Initialize results if needed
    if test.Results == nil {
        test.Results = &ABTestResults{
            VariantResults: make(map[string]*VariantResults),
        }
    }
    
    // Initialize variant results if needed
    if test.Results.VariantResults[variantName] == nil {
        test.Results.VariantResults[variantName] = &VariantResults{
            Metrics: make(map[string]MetricStats),
        }
    }
    
    variant := test.Results.VariantResults[variantName]
    
    // Update metric statistics
    metric := variant.Metrics[metricName]
    metric.Count++
    metric.Sum += value
    metric.Mean = metric.Sum / float64(metric.Count)
    
    if metric.Samples == nil {
        metric.Samples = make([]float64, 0)
    }
    metric.Samples = append(metric.Samples, value)
    
    if metric.Count == 1 {
        metric.Min = value
        metric.Max = value
    } else {
        if value < metric.Min {
            metric.Min = value
        }
        if value > metric.Max {
            metric.Max = value
        }
    }
    
    // Calculate standard deviation
    if metric.Count > 1 {
        variance := 0.0
        for _, sample := range metric.Samples {
            variance += math.Pow(sample-metric.Mean, 2)
        }
        variance /= float64(metric.Count - 1)
        metric.StdDev = math.Sqrt(variance)
    }
    
    variant.Metrics[metricName] = metric
    test.Results.TotalUsers++
}

func (atf *ABTestFramework) AnalyzeTest(testID string) (*ABTestResults, error) {
    test, exists := atf.tests[testID]
    if !exists {
        return nil, fmt.Errorf("test not found: %s", testID)
    }
    
    if test.Results == nil {
        return nil, fmt.Errorf("no data available for analysis")
    }
    
    results := test.Results
    
    // Find winning variant based on primary metric (assume first metric is primary)
    if len(test.Metrics) > 0 {
        primaryMetric := test.Metrics[0]
        
        var bestVariant string
        var bestValue float64
        var hasData bool
        
        for variantName, variantResults := range results.VariantResults {
            if metric, exists := variantResults.Metrics[primaryMetric]; exists && metric.Count > 0 {
                if !hasData || metric.Mean > bestValue {
                    bestVariant = variantName
                    bestValue = metric.Mean
                    hasData = true
                }
            }
        }
        
        if hasData {
            results.WinningVariant = bestVariant
            results.ConfidenceLevel = atf.calculateConfidence(test, primaryMetric)
            results.StatisticalPower = atf.calculatePower(test, primaryMetric)
        }
    }
    
    // Generate recommendations
    results.Recommendations = atf.generateRecommendations(test)
    
    return results, nil
}

func (atf *ABTestFramework) calculateConfidence(test *ABTest, metricName string) float64 {
    // Simplified confidence calculation - in production use proper statistical tests
    variants := make([]*VariantResults, 0)
    for _, variant := range test.Results.VariantResults {
        if metric, exists := variant.Metrics[metricName]; exists && metric.Count > 30 {
            variants = append(variants, variant)
        }
    }
    
    if len(variants) < 2 {
        return 0.0
    }
    
    // Simple t-test approximation
    baselineMetric := variants[0].Metrics[metricName]
    variantMetric := variants[1].Metrics[metricName]
    
    if baselineMetric.StdDev == 0 || variantMetric.StdDev == 0 {
        return 0.0
    }
    
    meanDiff := math.Abs(variantMetric.Mean - baselineMetric.Mean)
    pooledStdErr := math.Sqrt(
        (math.Pow(baselineMetric.StdDev, 2)/float64(baselineMetric.Count)) +
        (math.Pow(variantMetric.StdDev, 2)/float64(variantMetric.Count)),
    )
    
    if pooledStdErr == 0 {
        return 0.0
    }
    
    tStat := meanDiff / pooledStdErr
    
    // Convert t-statistic to confidence (simplified)
    if tStat > 2.0 {
        return 0.95
    } else if tStat > 1.65 {
        return 0.90
    } else if tStat > 1.28 {
        return 0.80
    }
    
    return 0.50
}

func (atf *ABTestFramework) calculatePower(test *ABTest, metricName string) float64 {
    // Simplified power calculation
    if test.Results.TotalUsers < 1000 {
        return 0.50
    } else if test.Results.TotalUsers < 5000 {
        return 0.70
    } else if test.Results.TotalUsers < 10000 {
        return 0.80
    }
    return 0.90
}

func (atf *ABTestFramework) generateRecommendations(test *ABTest) []string {
    recommendations := make([]string, 0)
    
    if test.Results.ConfidenceLevel < 0.90 {
        recommendations = append(recommendations, 
            "Increase sample size to achieve statistical significance")
    }
    
    if test.Results.StatisticalPower < 0.80 {
        recommendations = append(recommendations, 
            "Test may be underpowered - consider longer runtime")
    }
    
    if test.Results.TotalUsers < 1000 {
        recommendations = append(recommendations, 
            "Sample size is too small for reliable conclusions")
    }
    
    if test.Results.WinningVariant != "" {
        recommendations = append(recommendations, 
            fmt.Sprintf("Consider implementing variant '%s' based on test results", 
                test.Results.WinningVariant))
    }
    
    return recommendations
}

func (atf *ABTestFramework) StopTest(testID string) error {
    test, exists := atf.tests[testID]
    if !exists {
        return fmt.Errorf("test not found: %s", testID)
    }
    
    if test.Status != "running" {
        return fmt.Errorf("test is not running")
    }
    
    test.Status = "completed"
    now := time.Now()
    test.EndTime = &now
    
    // Perform final analysis
    _, err := atf.AnalyzeTest(testID)
    if err != nil {
        fmt.Printf("Warning: final analysis failed: %v\n", err)
    }
    
    fmt.Printf("Stopped A/B test: %s\n", test.Name)
    return nil
}

func main() {
    fmt.Println("=== Configuration A/B Testing Framework Example ===")
    
    framework := NewABTestFramework()
    
    // Create an A/B test for cache timeout configuration
    test := &ABTest{
        ID:          "cache_timeout_test",
        Name:        "Cache Timeout Optimization",
        Description: "Test different cache timeout values to optimize performance",
        ConfigKey:   "cache.timeout",
        Variants: map[string]ABVariant{
            "control": {
                Name:        "Control (30s)",
                Value:       30,
                Description: "Current 30-second timeout",
                Weight:      0.40,
            },
            "variant_a": {
                Name:        "Variant A (60s)",
                Value:       60,
                Description: "Increased 60-second timeout",
                Weight:      0.30,
            },
            "variant_b": {
                Name:        "Variant B (15s)",
                Value:       15,
                Description: "Decreased 15-second timeout",
                Weight:      0.30,
            },
        },
        Traffic: TrafficAllocation{
            TotalPercentage: 0.20, // 20% of users in test
            VariantSplit: map[string]float64{
                "control":   0.40,
                "variant_a": 0.30,
                "variant_b": 0.30,
            },
        },
        Metrics: []string{"response_time", "cache_hit_ratio", "error_rate"},
    }
    
    // Create and start the test
    if err := framework.CreateTest(test); err != nil {
        fmt.Printf("Error creating test: %v\n", err)
        return
    }
    
    if err := framework.StartTest(test.ID); err != nil {
        fmt.Printf("Error starting test: %v\n", err)
        return
    }
    
    fmt.Println("\n=== Simulating User Traffic ===")
    
    // Simulate user traffic and metrics collection
    for i := 0; i < 10000; i++ {
        userID := fmt.Sprintf("user_%d", i)
        
        variant, configValue, err := framework.GetVariantForUser(test.ID, userID)
        if err != nil {
            continue
        }
        
        if variant == "control" {
            continue // User not in test
        }
        
        // Simulate metrics based on variant
        var responseTime, cacheHitRatio, errorRate float64
        
        switch variant {
        case "control":
            responseTime = 100 + rand.Float64()*50     // 100-150ms
            cacheHitRatio = 0.70 + rand.Float64()*0.20 // 70-90%
            errorRate = 0.01 + rand.Float64()*0.02     // 1-3%
        case "variant_a":
            responseTime = 90 + rand.Float64()*40      // 90-130ms (better)
            cacheHitRatio = 0.75 + rand.Float64()*0.20 // 75-95% (better)
            errorRate = 0.008 + rand.Float64()*0.015   // 0.8-2.3% (better)
        case "variant_b":
            responseTime = 110 + rand.Float64()*60     // 110-170ms (worse)
            cacheHitRatio = 0.60 + rand.Float64()*0.25 // 60-85% (worse)
            errorRate = 0.015 + rand.Float64()*0.025   // 1.5-4% (worse)
        }
        
        // Record metrics
        framework.RecordMetric(test.ID, userID, variant, "response_time", responseTime)
        framework.RecordMetric(test.ID, userID, variant, "cache_hit_ratio", cacheHitRatio)
        framework.RecordMetric(test.ID, userID, variant, "error_rate", errorRate)
        
        // Show progress periodically
        if i%2000 == 0 {
            fmt.Printf("Processed %d users, config value for %s: %v\n", i, variant, configValue)
        }
    }
    
    // Analyze test results
    fmt.Println("\n=== Analyzing Test Results ===")
    
    results, err := framework.AnalyzeTest(test.ID)
    if err != nil {
        fmt.Printf("Error analyzing test: %v\n", err)
        return
    }
    
    fmt.Printf("Total Users: %d\n", results.TotalUsers)
    fmt.Printf("Winning Variant: %s\n", results.WinningVariant)
    fmt.Printf("Confidence Level: %.2f\n", results.ConfidenceLevel)
    fmt.Printf("Statistical Power: %.2f\n", results.StatisticalPower)
    
    fmt.Println("\n=== Variant Performance ===")
    
    variants := make([]string, 0)
    for variant := range results.VariantResults {
        variants = append(variants, variant)
    }
    sort.Strings(variants)
    
    for _, variant := range variants {
        variantResults := results.VariantResults[variant]
        fmt.Printf("\n%s:\n", variant)
        fmt.Printf("  Users: %d\n", variantResults.UserCount)
        
        for metricName, metric := range variantResults.Metrics {
            fmt.Printf("  %s: %.2f ± %.2f (min: %.2f, max: %.2f, n=%d)\n",
                metricName, metric.Mean, metric.StdDev, 
                metric.Min, metric.Max, metric.Count)
        }
    }
    
    fmt.Println("\n=== Recommendations ===")
    for i, recommendation := range results.Recommendations {
        fmt.Printf("%d. %s\n", i+1, recommendation)
    }
    
    // Stop the test
    framework.StopTest(test.ID)
    
    // Final analysis summary
    fmt.Println("\n=== Test Summary ===")
    fmt.Printf("Test Duration: %v\n", test.EndTime.Sub(test.StartTime))
    fmt.Printf("Status: %s\n", test.Status)
    
    if results.ConfidenceLevel >= 0.90 {
        fmt.Printf("✓ Test achieved statistical significance\n")
        fmt.Printf("Recommended action: Implement %s variant\n", results.WinningVariant)
    } else {
        fmt.Printf("⚠ Test did not achieve statistical significance\n")
        fmt.Printf("Recommended action: Continue testing or increase sample size\n")
    }
    
    fmt.Println("\n=== A/B Testing Framework Demo Complete ===")
}
```

This example demonstrates a comprehensive A/B testing framework for  
configuration experiments with statistical analysis, confidence calculations,  
and data-driven recommendations for configuration optimization.  

## Configuration internationalization (i18n)

Configuration internationalization enables applications to adapt settings  
and behavior based on locale and regional requirements.  

```go
package main

import (
    "fmt"
    "os"
    "strings"
    "time"
)

// LocalizedConfig represents locale-specific configuration
type LocalizedConfig struct {
    Locale   string            `json:"locale"`
    Language string            `json:"language"`
    Country  string            `json:"country"`
    Currency string            `json:"currency"`
    Timezone string            `json:"timezone"`
    Messages map[string]string `json:"messages"`
    Formats  FormatConfig      `json:"formats"`
    Features LocaleFeatures    `json:"features"`
}

type FormatConfig struct {
    DateFormat     string `json:"date_format"`
    TimeFormat     string `json:"time_format"`
    NumberFormat   string `json:"number_format"`
    CurrencyFormat string `json:"currency_format"`
    DecimalSep     string `json:"decimal_separator"`
    ThousandsSep   string `json:"thousands_separator"`
}

type LocaleFeatures struct {
    RTLSupport     bool     `json:"rtl_support"`
    PluralRules    []string `json:"plural_rules"`
    SortOrder      string   `json:"sort_order"`
    FirstDayOfWeek int      `json:"first_day_of_week"`
    WeekendDays    []int    `json:"weekend_days"`
}

// I18nConfigManager manages internationalized configurations
type I18nConfigManager struct {
    defaultLocale string
    configs       map[string]*LocalizedConfig
    fallbackChain map[string][]string
}

func NewI18nConfigManager(defaultLocale string) *I18nConfigManager {
    return &I18nConfigManager{
        defaultLocale: defaultLocale,
        configs:       make(map[string]*LocalizedConfig),
        fallbackChain: make(map[string][]string),
    }
}

func (icm *I18nConfigManager) LoadLocaleConfig(locale string, config *LocalizedConfig) {
    icm.configs[locale] = config
}

func (icm *I18nConfigManager) SetFallbackChain(locale string, fallbacks []string) {
    icm.fallbackChain[locale] = fallbacks
}

func (icm *I18nConfigManager) GetConfig(locale string) *LocalizedConfig {
    // Try exact match first
    if config, exists := icm.configs[locale]; exists {
        return config
    }
    
    // Try fallback chain
    if fallbacks, exists := icm.fallbackChain[locale]; exists {
        for _, fallback := range fallbacks {
            if config, exists := icm.configs[fallback]; exists {
                return config
            }
        }
    }
    
    // Try language part of locale (e.g., "en" from "en-US")
    if parts := strings.Split(locale, "-"); len(parts) > 1 {
        if config, exists := icm.configs[parts[0]]; exists {
            return config
        }
    }
    
    // Return default locale configuration
    if config, exists := icm.configs[icm.defaultLocale]; exists {
        return config
    }
    
    return nil
}

func (icm *I18nConfigManager) GetMessage(locale, key string) string {
    config := icm.GetConfig(locale)
    if config == nil {
        return key // Return key if no config found
    }
    
    if message, exists := config.Messages[key]; exists {
        return message
    }
    
    return key // Return key if message not found
}

func (icm *I18nConfigManager) FormatDate(locale string, t time.Time) string {
    config := icm.GetConfig(locale)
    if config == nil {
        return t.Format("2006-01-02")
    }
    
    // Convert Go time format
    format := strings.ReplaceAll(config.Formats.DateFormat, "DD", "02")
    format = strings.ReplaceAll(format, "MM", "01")
    format = strings.ReplaceAll(format, "YYYY", "2006")
    
    return t.Format(format)
}

func (icm *I18nConfigManager) FormatNumber(locale string, number float64) string {
    config := icm.GetConfig(locale)
    if config == nil {
        return fmt.Sprintf("%.2f", number)
    }
    
    // Simple number formatting based on locale
    numberStr := fmt.Sprintf("%.2f", number)
    
    if config.Formats.DecimalSep != "." {
        numberStr = strings.ReplaceAll(numberStr, ".", config.Formats.DecimalSep)
    }
    
    return numberStr
}

func DetectLocaleFromEnvironment() string {
    // Check various environment variables
    for _, env := range []string{"LC_ALL", "LC_MESSAGES", "LANG"} {
        if locale := os.Getenv(env); locale != "" {
            // Extract locale part (before any encoding suffix)
            if idx := strings.Index(locale, "."); idx > 0 {
                locale = locale[:idx]
            }
            return locale
        }
    }
    return "en-US" // Default fallback
}

func main() {
    fmt.Println("=== Configuration Internationalization Example ===")
    
    manager := NewI18nConfigManager("en-US")
    
    // Load English (US) configuration
    enUSConfig := &LocalizedConfig{
        Locale:   "en-US",
        Language: "English",
        Country:  "United States",
        Currency: "USD",
        Timezone: "America/New_York",
        Messages: map[string]string{
            "welcome":     "Welcome",
            "goodbye":     "Goodbye",
            "hello_user":  "Hello, %s!",
            "save_success": "Data saved successfully",
            "error_occurred": "An error occurred",
        },
        Formats: FormatConfig{
            DateFormat:     "MM/DD/YYYY",
            TimeFormat:     "HH:mm:ss",
            NumberFormat:   "#,##0.00",
            CurrencyFormat: "$#,##0.00",
            DecimalSep:     ".",
            ThousandsSep:   ",",
        },
        Features: LocaleFeatures{
            RTLSupport:     false,
            PluralRules:    []string{"one", "other"},
            SortOrder:      "alphabetical",
            FirstDayOfWeek: 0, // Sunday
            WeekendDays:    []int{0, 6}, // Sunday, Saturday
        },
    }
    
    // Load German configuration
    deConfig := &LocalizedConfig{
        Locale:   "de-DE",
        Language: "Deutsch",
        Country:  "Deutschland",
        Currency: "EUR",
        Timezone: "Europe/Berlin",
        Messages: map[string]string{
            "welcome":     "Willkommen",
            "goodbye":     "Auf Wiedersehen",
            "hello_user":  "Hallo, %s!",
            "save_success": "Daten erfolgreich gespeichert",
            "error_occurred": "Ein Fehler ist aufgetreten",
        },
        Formats: FormatConfig{
            DateFormat:     "DD.MM.YYYY",
            TimeFormat:     "HH:mm:ss",
            NumberFormat:   "#.##0,00",
            CurrencyFormat: "#.##0,00 €",
            DecimalSep:     ",",
            ThousandsSep:   ".",
        },
        Features: LocaleFeatures{
            RTLSupport:     false,
            PluralRules:    []string{"one", "other"},
            SortOrder:      "alphabetical",
            FirstDayOfWeek: 1, // Monday
            WeekendDays:    []int{0, 6}, // Sunday, Saturday
        },
    }
    
    // Load Japanese configuration
    jaConfig := &LocalizedConfig{
        Locale:   "ja-JP",
        Language: "日本語",
        Country:  "日本",
        Currency: "JPY",
        Timezone: "Asia/Tokyo",
        Messages: map[string]string{
            "welcome":     "いらっしゃいませ",
            "goodbye":     "さようなら",
            "hello_user":  "こんにちは、%sさん！",
            "save_success": "データが正常に保存されました",
            "error_occurred": "エラーが発生しました",
        },
        Formats: FormatConfig{
            DateFormat:     "YYYY/MM/DD",
            TimeFormat:     "HH:mm:ss",
            NumberFormat:   "#,##0",
            CurrencyFormat: "¥#,##0",
            DecimalSep:     ".",
            ThousandsSep:   ",",
        },
        Features: LocaleFeatures{
            RTLSupport:     false,
            PluralRules:    []string{"other"},
            SortOrder:      "kana",
            FirstDayOfWeek: 0, // Sunday
            WeekendDays:    []int{0, 6}, // Sunday, Saturday
        },
    }
    
    // Register configurations
    manager.LoadLocaleConfig("en-US", enUSConfig)
    manager.LoadLocaleConfig("de-DE", deConfig)
    manager.LoadLocaleConfig("ja-JP", jaConfig)
    
    // Set up fallback chains
    manager.SetFallbackChain("en-GB", []string{"en-US"})
    manager.SetFallbackChain("de-AT", []string{"de-DE"})
    manager.SetFallbackChain("ja", []string{"ja-JP"})
    
    // Detect locale from environment
    detectedLocale := DetectLocaleFromEnvironment()
    fmt.Printf("Detected locale: %s\n", detectedLocale)
    
    // Test different locales
    locales := []string{"en-US", "de-DE", "ja-JP", "en-GB", "de-AT", "fr-FR"}
    
    for _, locale := range locales {
        fmt.Printf("\n=== Locale: %s ===\n", locale)
        
        config := manager.GetConfig(locale)
        if config == nil {
            fmt.Printf("No configuration available for %s\n", locale)
            continue
        }
        
        fmt.Printf("Language: %s\n", config.Language)
        fmt.Printf("Country: %s\n", config.Country)
        fmt.Printf("Currency: %s\n", config.Currency)
        
        // Test message localization
        fmt.Printf("Welcome message: %s\n", manager.GetMessage(locale, "welcome"))
        fmt.Printf("Goodbye message: %s\n", manager.GetMessage(locale, "goodbye"))
        fmt.Printf("Success message: %s\n", manager.GetMessage(locale, "save_success"))
        
        // Test date formatting
        now := time.Now()
        fmt.Printf("Formatted date: %s\n", manager.FormatDate(locale, now))
        
        // Test number formatting
        number := 1234.56
        fmt.Printf("Formatted number: %s\n", manager.FormatNumber(locale, number))
        
        // Display locale features
        fmt.Printf("RTL Support: %v\n", config.Features.RTLSupport)
        fmt.Printf("First day of week: %d\n", config.Features.FirstDayOfWeek)
        fmt.Printf("Weekend days: %v\n", config.Features.WeekendDays)
    }
    
    // Test fallback behavior
    fmt.Println("\n=== Fallback Testing ===")
    
    // Try unsupported locale
    unsupportedLocale := "ar-SA"
    config := manager.GetConfig(unsupportedLocale)
    if config != nil {
        fmt.Printf("Fallback for %s: %s\n", unsupportedLocale, config.Locale)
    } else {
        fmt.Printf("No fallback available for %s\n", unsupportedLocale)
    }
    
    // Test partial locale match
    partialLocale := "de"
    config = manager.GetConfig(partialLocale)
    if config != nil {
        fmt.Printf("Partial match for %s: %s\n", partialLocale, config.Locale)
    }
    
    fmt.Println("\n=== Internationalization Demo Complete ===")
}
```

This example demonstrates comprehensive internationalization configuration with  
locale-specific settings, message localization, format customization, fallback  
chains, and environment-based locale detection for global applications.  

## Configuration microservices orchestration

Configuration orchestration in microservices enables centralized management  
and coordination of configurations across distributed service architectures.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "sync"
    "time"
)

// ServiceConfig represents configuration for a microservice
type ServiceConfig struct {
    ServiceName    string                 `json:"service_name"`
    Version        string                 `json:"version"`
    Environment    string                 `json:"environment"`
    Dependencies   []ServiceDependency    `json:"dependencies"`
    Resources      ResourceConfig         `json:"resources"`
    Networking     NetworkConfig          `json:"networking"`
    Monitoring     MonitoringConfig       `json:"monitoring"`
    Security       SecurityConfig         `json:"security"`
    CustomConfig   map[string]interface{} `json:"custom_config"`
    LastUpdated    time.Time              `json:"last_updated"`
    ConfigVersion  int64                  `json:"config_version"`
}

type ServiceDependency struct {
    ServiceName    string            `json:"service_name"`
    Version        string            `json:"version"`
    Endpoint       string            `json:"endpoint"`
    HealthCheck    string            `json:"health_check"`
    Timeout        time.Duration     `json:"timeout"`
    RetryPolicy    RetryPolicy       `json:"retry_policy"`
    CircuitBreaker CircuitBreakerConfig `json:"circuit_breaker"`
}

type RetryPolicy struct {
    MaxAttempts int           `json:"max_attempts"`
    BaseDelay   time.Duration `json:"base_delay"`
    MaxDelay    time.Duration `json:"max_delay"`
    Multiplier  float64       `json:"multiplier"`
}

type CircuitBreakerConfig struct {
    Enabled               bool          `json:"enabled"`
    FailureThreshold      int           `json:"failure_threshold"`
    RecoveryTimeout       time.Duration `json:"recovery_timeout"`
    HalfOpenMaxCalls      int           `json:"half_open_max_calls"`
    HalfOpenSuccessThreshold int        `json:"half_open_success_threshold"`
}

type ResourceConfig struct {
    CPU    ResourceLimits `json:"cpu"`
    Memory ResourceLimits `json:"memory"`
    Disk   ResourceLimits `json:"disk"`
}

type ResourceLimits struct {
    Request string `json:"request"`
    Limit   string `json:"limit"`
}

type NetworkConfig struct {
    Port            int               `json:"port"`
    HealthCheckPort int               `json:"health_check_port"`
    MetricsPort     int               `json:"metrics_port"`
    Protocol        string            `json:"protocol"`
    LoadBalancer    LoadBalancerConfig `json:"load_balancer"`
}

type LoadBalancerConfig struct {
    Algorithm    string        `json:"algorithm"`
    HealthCheck  string        `json:"health_check"`
    Timeout      time.Duration `json:"timeout"`
    MaxFails     int           `json:"max_fails"`
    FailTimeout  time.Duration `json:"fail_timeout"`
}

type MonitoringConfig struct {
    Metrics   MetricsConfig   `json:"metrics"`
    Logging   LoggingConfig   `json:"logging"`
    Tracing   TracingConfig   `json:"tracing"`
    Alerting  AlertingConfig  `json:"alerting"`
}

type MetricsConfig struct {
    Enabled    bool     `json:"enabled"`
    Endpoint   string   `json:"endpoint"`
    Interval   time.Duration `json:"interval"`
    Labels     map[string]string `json:"labels"`
}

type LoggingConfig struct {
    Level      string `json:"level"`
    Format     string `json:"format"`
    Output     string `json:"output"`
    Structured bool   `json:"structured"`
}

type TracingConfig struct {
    Enabled     bool    `json:"enabled"`
    SampleRate  float64 `json:"sample_rate"`
    Endpoint    string  `json:"endpoint"`
    ServiceName string  `json:"service_name"`
}

type AlertingConfig struct {
    Enabled   bool              `json:"enabled"`
    Rules     []AlertRule       `json:"rules"`
    Channels  []string          `json:"channels"`
}

type AlertRule struct {
    Name        string        `json:"name"`
    Condition   string        `json:"condition"`
    Threshold   float64       `json:"threshold"`
    Duration    time.Duration `json:"duration"`
    Severity    string        `json:"severity"`
}

type SecurityConfig struct {
    TLS            TLSConfig            `json:"tls"`
    Authentication AuthenticationConfig `json:"authentication"`
    Authorization  AuthorizationConfig  `json:"authorization"`
    RateLimit      RateLimitConfig      `json:"rate_limit"`
}

type TLSConfig struct {
    Enabled  bool   `json:"enabled"`
    CertFile string `json:"cert_file"`
    KeyFile  string `json:"key_file"`
    CAFile   string `json:"ca_file"`
}

type AuthenticationConfig struct {
    Enabled  bool              `json:"enabled"`
    Provider string            `json:"provider"`
    Config   map[string]string `json:"config"`
}

type AuthorizationConfig struct {
    Enabled bool              `json:"enabled"`
    Model   string            `json:"model"`
    Policies []string         `json:"policies"`
}

type RateLimitConfig struct {
    Enabled    bool          `json:"enabled"`
    Requests   int           `json:"requests"`
    Window     time.Duration `json:"window"`
    BurstSize  int           `json:"burst_size"`
}

// ConfigOrchestrator manages configurations across microservices
type ConfigOrchestrator struct {
    services        map[string]*ServiceConfig
    dependencies    map[string][]string
    updateChannels  map[string]chan *ServiceConfig
    globalConfig    map[string]interface{}
    mutex           sync.RWMutex
    subscribers     []chan ConfigUpdate
    running         bool
}

type ConfigUpdate struct {
    ServiceName string         `json:"service_name"`
    ConfigType  string         `json:"config_type"`
    OldConfig   *ServiceConfig `json:"old_config"`
    NewConfig   *ServiceConfig `json:"new_config"`
    Timestamp   time.Time      `json:"timestamp"`
}

func NewConfigOrchestrator() *ConfigOrchestrator {
    return &ConfigOrchestrator{
        services:       make(map[string]*ServiceConfig),
        dependencies:   make(map[string][]string),
        updateChannels: make(map[string]chan *ServiceConfig),
        globalConfig:   make(map[string]interface{}),
        subscribers:    make([]chan ConfigUpdate, 0),
    }
}

func (co *ConfigOrchestrator) RegisterService(config *ServiceConfig) {
    co.mutex.Lock()
    defer co.mutex.Unlock()
    
    oldConfig := co.services[config.ServiceName]
    config.LastUpdated = time.Now()
    config.ConfigVersion++
    
    co.services[config.ServiceName] = config
    co.updateChannels[config.ServiceName] = make(chan *ServiceConfig, 10)
    
    // Update dependency graph
    deps := make([]string, 0)
    for _, dep := range config.Dependencies {
        deps = append(deps, dep.ServiceName)
    }
    co.dependencies[config.ServiceName] = deps
    
    // Notify subscribers
    update := ConfigUpdate{
        ServiceName: config.ServiceName,
        ConfigType:  "service_registration",
        OldConfig:   oldConfig,
        NewConfig:   config,
        Timestamp:   time.Now(),
    }
    
    co.notifySubscribers(update)
    
    fmt.Printf("Registered service: %s v%s\n", config.ServiceName, config.Version)
}

func (co *ConfigOrchestrator) UpdateServiceConfig(serviceName string, updates map[string]interface{}) error {
    co.mutex.Lock()
    defer co.mutex.Unlock()
    
    config, exists := co.services[serviceName]
    if !exists {
        return fmt.Errorf("service not found: %s", serviceName)
    }
    
    oldConfig := *config
    
    // Apply updates to custom config
    for key, value := range updates {
        config.CustomConfig[key] = value
    }
    
    config.LastUpdated = time.Now()
    config.ConfigVersion++
    
    // Notify subscribers
    update := ConfigUpdate{
        ServiceName: serviceName,
        ConfigType:  "service_update",
        OldConfig:   &oldConfig,
        NewConfig:   config,
        Timestamp:   time.Now(),
    }
    
    co.notifySubscribers(update)
    
    // Propagate to dependent services
    co.propagateConfigChange(serviceName)
    
    fmt.Printf("Updated configuration for service: %s\n", serviceName)
    return nil
}

func (co *ConfigOrchestrator) propagateConfigChange(serviceName string) {
    // Find services that depend on the updated service
    for service, deps := range co.dependencies {
        for _, dep := range deps {
            if dep == serviceName {
                // Notify dependent service of configuration change
                if channel, exists := co.updateChannels[service]; exists {
                    select {
                    case channel <- co.services[serviceName]:
                        fmt.Printf("Notified %s of %s configuration change\n", service, serviceName)
                    default:
                        fmt.Printf("Failed to notify %s (channel full)\n", service)
                    }
                }
            }
        }
    }
}

func (co *ConfigOrchestrator) GetServiceConfig(serviceName string) (*ServiceConfig, error) {
    co.mutex.RLock()
    defer co.mutex.RUnlock()
    
    config, exists := co.services[serviceName]
    if !exists {
        return nil, fmt.Errorf("service not found: %s", serviceName)
    }
    
    return config, nil
}

func (co *ConfigOrchestrator) GetServiceDependencies(serviceName string) ([]*ServiceConfig, error) {
    co.mutex.RLock()
    defer co.mutex.RUnlock()
    
    deps, exists := co.dependencies[serviceName]
    if !exists {
        return nil, fmt.Errorf("service not found: %s", serviceName)
    }
    
    var dependencies []*ServiceConfig
    for _, depName := range deps {
        if depConfig, exists := co.services[depName]; exists {
            dependencies = append(dependencies, depConfig)
        }
    }
    
    return dependencies, nil
}

func (co *ConfigOrchestrator) ValidateConfiguration() []string {
    co.mutex.RLock()
    defer co.mutex.RUnlock()
    
    var issues []string
    
    // Check for circular dependencies
    if circularDeps := co.detectCircularDependencies(); len(circularDeps) > 0 {
        issues = append(issues, fmt.Sprintf("Circular dependencies detected: %v", circularDeps))
    }
    
    // Check for missing dependencies
    for serviceName, deps := range co.dependencies {
        for _, dep := range deps {
            if _, exists := co.services[dep]; !exists {
                issues = append(issues, fmt.Sprintf("Service %s depends on missing service: %s", serviceName, dep))
            }
        }
    }
    
    // Check for port conflicts
    portUsage := make(map[int][]string)
    for serviceName, config := range co.services {
        if config.Networking.Port > 0 {
            portUsage[config.Networking.Port] = append(portUsage[config.Networking.Port], serviceName)
        }
    }
    
    for port, services := range portUsage {
        if len(services) > 1 {
            issues = append(issues, fmt.Sprintf("Port %d is used by multiple services: %v", port, services))
        }
    }
    
    return issues
}

func (co *ConfigOrchestrator) detectCircularDependencies() []string {
    visited := make(map[string]bool)
    recursionStack := make(map[string]bool)
    var circular []string
    
    var dfs func(string) bool
    dfs = func(service string) bool {
        visited[service] = true
        recursionStack[service] = true
        
        for _, dep := range co.dependencies[service] {
            if !visited[dep] {
                if dfs(dep) {
                    return true
                }
            } else if recursionStack[dep] {
                circular = append(circular, fmt.Sprintf("%s -> %s", service, dep))
                return true
            }
        }
        
        recursionStack[service] = false
        return false
    }
    
    for service := range co.services {
        if !visited[service] {
            if dfs(service) {
                break
            }
        }
    }
    
    return circular
}

func (co *ConfigOrchestrator) Subscribe() chan ConfigUpdate {
    co.mutex.Lock()
    defer co.mutex.Unlock()
    
    updateChan := make(chan ConfigUpdate, 10)
    co.subscribers = append(co.subscribers, updateChan)
    return updateChan
}

func (co *ConfigOrchestrator) notifySubscribers(update ConfigUpdate) {
    for _, subscriber := range co.subscribers {
        select {
        case subscriber <- update:
        default:
            // Subscriber channel full, skip
        }
    }
}

func (co *ConfigOrchestrator) GetSystemOverview() map[string]interface{} {
    co.mutex.RLock()
    defer co.mutex.RUnlock()
    
    overview := map[string]interface{}{
        "total_services":    len(co.services),
        "total_dependencies": len(co.dependencies),
        "services":          make([]map[string]interface{}, 0),
    }
    
    for serviceName, config := range co.services {
        serviceInfo := map[string]interface{}{
            "name":            serviceName,
            "version":         config.Version,
            "environment":     config.Environment,
            "dependencies":    len(config.Dependencies),
            "last_updated":    config.LastUpdated,
            "config_version":  config.ConfigVersion,
        }
        overview["services"] = append(overview["services"].([]map[string]interface{}), serviceInfo)
    }
    
    return overview
}

func main() {
    fmt.Println("=== Configuration Microservices Orchestration Example ===")
    
    orchestrator := NewConfigOrchestrator()
    
    // Subscribe to configuration updates
    updates := orchestrator.Subscribe()
    go func() {
        for update := range updates {
            fmt.Printf("Config Update: %s - %s at %s\n", 
                update.ServiceName, update.ConfigType, 
                update.Timestamp.Format("15:04:05"))
        }
    }()
    
    // Register API Gateway service
    apiGateway := &ServiceConfig{
        ServiceName: "api-gateway",
        Version:     "1.2.0",
        Environment: "production",
        Dependencies: []ServiceDependency{
            {
                ServiceName: "auth-service",
                Version:     "1.0.0",
                Endpoint:    "http://auth-service:8081",
                HealthCheck: "/health",
                Timeout:     5 * time.Second,
                RetryPolicy: RetryPolicy{
                    MaxAttempts: 3,
                    BaseDelay:   100 * time.Millisecond,
                    MaxDelay:    1 * time.Second,
                    Multiplier:  2.0,
                },
                CircuitBreaker: CircuitBreakerConfig{
                    Enabled:                  true,
                    FailureThreshold:         5,
                    RecoveryTimeout:          30 * time.Second,
                    HalfOpenMaxCalls:         3,
                    HalfOpenSuccessThreshold: 2,
                },
            },
            {
                ServiceName: "user-service",
                Version:     "2.1.0",
                Endpoint:    "http://user-service:8082",
                HealthCheck: "/health",
                Timeout:     3 * time.Second,
                RetryPolicy: RetryPolicy{
                    MaxAttempts: 2,
                    BaseDelay:   50 * time.Millisecond,
                    MaxDelay:    500 * time.Millisecond,
                    Multiplier:  1.5,
                },
            },
        },
        Resources: ResourceConfig{
            CPU:    ResourceLimits{Request: "500m", Limit: "1000m"},
            Memory: ResourceLimits{Request: "512Mi", Limit: "1Gi"},
            Disk:   ResourceLimits{Request: "1Gi", Limit: "5Gi"},
        },
        Networking: NetworkConfig{
            Port:            8080,
            HealthCheckPort: 8090,
            MetricsPort:     9090,
            Protocol:        "HTTP",
            LoadBalancer: LoadBalancerConfig{
                Algorithm:   "round_robin",
                HealthCheck: "/health",
                Timeout:     30 * time.Second,
                MaxFails:    3,
                FailTimeout: 10 * time.Second,
            },
        },
        Monitoring: MonitoringConfig{
            Metrics: MetricsConfig{
                Enabled:  true,
                Endpoint: "/metrics",
                Interval: 15 * time.Second,
                Labels:   map[string]string{"service": "api-gateway", "tier": "gateway"},
            },
            Logging: LoggingConfig{
                Level:      "info",
                Format:     "json",
                Output:     "stdout",
                Structured: true,
            },
            Tracing: TracingConfig{
                Enabled:     true,
                SampleRate:  0.1,
                Endpoint:    "http://jaeger:14268/api/traces",
                ServiceName: "api-gateway",
            },
            Alerting: AlertingConfig{
                Enabled:  true,
                Rules: []AlertRule{
                    {
                        Name:      "high_response_time",
                        Condition: "avg_response_time > threshold",
                        Threshold: 1.0,
                        Duration:  5 * time.Minute,
                        Severity:  "warning",
                    },
                },
                Channels: []string{"slack", "email"},
            },
        },
        Security: SecurityConfig{
            TLS: TLSConfig{
                Enabled:  true,
                CertFile: "/etc/certs/server.crt",
                KeyFile:  "/etc/certs/server.key",
                CAFile:   "/etc/certs/ca.crt",
            },
            Authentication: AuthenticationConfig{
                Enabled:  true,
                Provider: "jwt",
                Config:   map[string]string{"issuer": "auth-service", "audience": "api-gateway"},
            },
            Authorization: AuthorizationConfig{
                Enabled:  true,
                Model:    "rbac",
                Policies: []string{"admin_policy", "user_policy"},
            },
            RateLimit: RateLimitConfig{
                Enabled:   true,
                Requests:  1000,
                Window:    1 * time.Minute,
                BurstSize: 100,
            },
        },
        CustomConfig: map[string]interface{}{
            "max_request_size": "10MB",
            "cors_enabled":     true,
            "allowed_origins":  []string{"https://app.example.com"},
        },
    }
    
    // Register services
    orchestrator.RegisterService(apiGateway)
    
    // Register auth service
    authService := &ServiceConfig{
        ServiceName: "auth-service",
        Version:     "1.0.0",
        Environment: "production",
        Dependencies: []ServiceDependency{
            {
                ServiceName: "database",
                Version:     "1.0.0",
                Endpoint:    "postgres://db:5432/auth",
                Timeout:     10 * time.Second,
            },
        },
        Resources: ResourceConfig{
            CPU:    ResourceLimits{Request: "200m", Limit: "500m"},
            Memory: ResourceLimits{Request: "256Mi", Limit: "512Mi"},
        },
        Networking: NetworkConfig{
            Port:            8081,
            HealthCheckPort: 8091,
            MetricsPort:     9091,
            Protocol:        "HTTP",
        },
        CustomConfig: map[string]interface{}{
            "jwt_expiry":    "24h",
            "refresh_token": true,
        },
    }
    
    orchestrator.RegisterService(authService)
    
    // Register user service
    userService := &ServiceConfig{
        ServiceName: "user-service",
        Version:     "2.1.0",
        Environment: "production",
        Dependencies: []ServiceDependency{
            {
                ServiceName: "database",
                Version:     "1.0.0",
                Endpoint:    "postgres://db:5432/users",
                Timeout:     10 * time.Second,
            },
        },
        Resources: ResourceConfig{
            CPU:    ResourceLimits{Request: "300m", Limit: "600m"},
            Memory: ResourceLimits{Request: "512Mi", Limit: "1Gi"},
        },
        Networking: NetworkConfig{
            Port:            8082,
            HealthCheckPort: 8092,
            MetricsPort:     9092,
            Protocol:        "HTTP",
        },
        CustomConfig: map[string]interface{}{
            "pagination_limit": 100,
            "cache_ttl":       "1h",
        },
    }
    
    orchestrator.RegisterService(userService)
    
    time.Sleep(100 * time.Millisecond)
    
    // Validate configuration
    fmt.Println("\n=== Configuration Validation ===")
    if issues := orchestrator.ValidateConfiguration(); len(issues) > 0 {
        fmt.Println("Configuration issues found:")
        for _, issue := range issues {
            fmt.Printf("  - %s\n", issue)
        }
    } else {
        fmt.Println("✓ All configurations are valid")
    }
    
    // Display system overview
    fmt.Println("\n=== System Overview ===")
    overview := orchestrator.GetSystemOverview()
    overviewJSON, _ := json.MarshalIndent(overview, "", "  ")
    fmt.Println(string(overviewJSON))
    
    // Update configuration
    fmt.Println("\n=== Configuration Updates ===")
    updates_config := map[string]interface{}{
        "max_request_size": "20MB",
        "feature_flags": map[string]bool{
            "new_auth_flow": true,
            "beta_features": false,
        },
    }
    
    if err := orchestrator.UpdateServiceConfig("api-gateway", updates_config); err != nil {
        fmt.Printf("Error updating configuration: %v\n", err)
    }
    
    time.Sleep(100 * time.Millisecond)
    
    // Show dependencies
    fmt.Println("\n=== Service Dependencies ===")
    deps, _ := orchestrator.GetServiceDependencies("api-gateway")
    fmt.Printf("API Gateway depends on %d services:\n", len(deps))
    for _, dep := range deps {
        fmt.Printf("  - %s v%s\n", dep.ServiceName, dep.Version)
    }
    
    fmt.Println("\n=== Microservices Orchestration Demo Complete ===")
}
```

This example demonstrates comprehensive microservices configuration  
orchestration with service registration, dependency management, configuration  
validation, propagation, and centralized coordination across distributed  
service architectures.  