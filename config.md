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