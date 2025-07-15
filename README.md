# HybridBuffer Storage Interface

A pluggable storage interface for the HybridBuffer library, providing extensible storage backend capabilities.

## Overview

This package defines the core storage interface for HybridBuffer, enabling developers to create custom storage backends that can be seamlessly integrated into the HybridBuffer ecosystem. The storage interface provides a unified API for various storage mechanisms, from local filesystems to cloud storage services.

## Features

- **Pluggable Architecture**: Clean interface for custom storage backend development
- **Extensible Design**: Easy to implement new storage capabilities
- **Type-Safe**: Strongly typed interface for reliable storage integration
- **Performance Focused**: Minimal overhead design for high-performance applications
- **Multiple Access Patterns**: Support for sequential and random access patterns

## Installation

```bash
go get schneider.vip/hybridbuffer/storage
```

## Interface

```go
package storage

import "io"

// Backend defines the minimal interface for external storage backends
type Backend interface {
    // Create creates a new storage location and returns a writer
    Create() (io.WriteCloser, error)
    
    // Open opens an existing storage location for reading
    Open() (io.ReadCloser, error)
    
    // Remove removes the storage location
    Remove() error
}

// BackendReaderAt extends Backend with random access capabilities
type BackendReaderAt interface {
    Backend
    
    // OpenReaderAt opens the storage location for random access reading
    OpenReaderAt() (io.ReaderAt, error)
}

// BackendSeeker extends Backend with seeking capabilities
type BackendSeeker interface {
    Backend
    
    // OpenSeeker opens the storage location for seeking operations
    OpenSeeker() (io.ReadSeeker, error)
}
```

## Usage

### Implementing Custom Storage Backend

```go
package main

import (
    "io"
    "schneider.vip/hybridbuffer"
    "schneider.vip/hybridbuffer/storage"
)

type CustomStorage struct {
    // Your custom fields
}

func (s *CustomStorage) Create() (io.WriteCloser, error) {
    // Create and return a writer for your storage
    return &customWriter{storage: s}, nil
}

func (s *CustomStorage) Open() (io.ReadCloser, error) {
    // Open and return a reader for your storage
    return &customReader{storage: s}, nil
}

func (s *CustomStorage) Remove() error {
    // Clean up your storage
    return nil
}

// Optional: Implement ReaderAt for random access
func (s *CustomStorage) OpenReaderAt() (io.ReaderAt, error) {
    return &customReaderAt{storage: s}, nil
}
```

### Using with HybridBuffer

```go
package main

import (
    "schneider.vip/hybridbuffer"
    "schneider.vip/hybridbuffer/storage"
)

func main() {
    // Create your custom storage
    custom := &CustomStorage{}
    
    // Create HybridBuffer with custom storage
    buf := hybridbuffer.New(
        hybridbuffer.WithStorage(func() storage.Backend { return custom }),
    )
    defer buf.Close()
    
    // Use the buffer - storage will be used automatically when needed
    buf.WriteString("Hello, World!")
}
```

## Available Storage Backends

The HybridBuffer ecosystem provides several ready-to-use storage implementations:

- **[Filesystem](../hybridbuffer-storage-filesystem)**: Local file system storage with optional encryption
- **[Redis](../hybridbuffer-storage-redis)**: Redis-based storage for distributed scenarios
- **[S3](../hybridbuffer-storage-s3)**: Amazon S3 compatible storage for cloud deployments

## Storage Factory Pattern

For dynamic storage backend selection:

```go
package main

import (
    "schneider.vip/hybridbuffer"
    "schneider.vip/hybridbuffer/storage"
)

type StorageFactory interface {
    CreateBackend() (storage.Backend, error)
}

func main() {
    factory := &MyStorageFactory{}
    
    buf := hybridbuffer.NewBuilder().
        WithStorageFactory(factory).
        Build()
    defer buf.Close()
}
```

## Performance Characteristics

- **Sequential Access**: Optimized for streaming operations
- **Random Access**: Efficient ReaderAt implementation for supported backends
- **Seeking**: Full seeking support for compatible storage systems
- **Minimal Overhead**: Direct I/O operations with minimal abstraction cost

## Error Handling

All storage operations return proper error types that can be handled appropriately:

```go
backend := &MyStorageBackend{}

writer, err := backend.Create()
if err != nil {
    log.Printf("Failed to create storage: %v", err)
    return
}
defer writer.Close()
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.