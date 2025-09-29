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