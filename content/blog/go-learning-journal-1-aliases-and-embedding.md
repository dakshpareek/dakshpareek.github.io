---
title: "Go Learning Journal 1: Aliases and Embedding"
date: 2025-02-04
tags: ["Go", "Notes"]
draft: false
---

These are raw learning notes from my journey with Go, documenting key concepts and examples for future reference. I reason with LLMs to understand some concepts better and sometimes copy paste the responses of that in my notes. These are not original thoughts.

## Alias in Go

An alias is a new name for a type. Alias is declared using the keyword `type` followed by the alias name and the type name.

```go
type Image struct {
	id int
	name string
}

type Photo = Image
```

- The alias has same name and methods as the type it is aliasing.
- The alias can be assigned to a variable of the original type.
```go
var photo := Photo{123, "desktop.png"}
var img Img = photo
```

- An alias is just a new name for that type. If we want to create new methods or change fields in aliased type, then we must need to add them to the original type.
- Wo can alias a type from another module. There is one drawback to an alias in another package: we cannot use an alias to refer to the unexported methods and fields of the original type.
- Two kinds of exported identifiers can’t have alternate names. The first is a packagelevel variable. The second is a field in a struct. Once you choose a name for an exported struct field, there’s no way to create an alternate name.

Package-level Variables:
```go
package mypackage

// Exported variable (starts with capital letter)
var MyVariable = 42  // This name cannot have an alternate name
```

Struct Fields:
```go
package mypackage

type Person struct {
    Name string    // This exported field name cannot have an alternate name
    Age  int      // This exported field name cannot have an alternate name
}
```

---

- Command `go mod tidy` is used to fix the module dependencies. It scans your source code and synchronizes the go.mod and go.sum files with your module’s source code, adding and removing module references. It also makes sure that the indirect comments are correct.

---

- **go run** builds and executes a program in one step
```go
go run file
```

  The go run command builds the binary, executes the binary from that temporary directory, and then deletes the binary after your program finishes.

---

1. **What is Embedding?**
- Embedding allows you to include files (like text files, images, or any other files) directly into your Go binary (executable)
- These files become part of your program itself, rather than being separate files on the disk

2. **Why use Embedding?**
- Single Binary: Your program becomes one single file that contains everything it needs
- Easy Distribution: You don't need to worry about shipping additional files with your program
- Security: Files can't be accidentally modified or deleted as they're part of the program

3. **Difference from Regular File Operations:**

```go
// Regular File Reading (Traditional way)
data, err := os.ReadFile("config.txt")  // Reads from disk
// File must exist on the computer

// Embedding (New way)
//go:embed config.txt
var configData string  // File is part of the program
// No need for external file
```

4. **Simple Example:**
```go
package main

import (
    _ "embed"
    "fmt"
)

//go:embed hello.txt
var content string

func main() {
    // content already contains the text from hello.txt
    fmt.Println(content)
}
```

5. **Real-world Examples:**
- Embedding HTML templates in a web server
- Embedding default configuration files
- Embedding images or assets in a GUI application

6. **Types of Embedding:**
```go
// Single file as string
//go:embed file.txt
var data string

// Single file as bytes
//go:embed file.bin
var data []byte

// Multiple files or directories
//go:embed templates/*
var files embed.FS
```

7. **Key Benefits:**
- No external dependencies
- Guaranteed file availability
- Better deployment process
- Protection against accidental file modifications

Think of embedding like creating a zip file that's permanently part of your program, rather than keeping files separate on disk.

Example of why it's useful:
```go
// Without embedding
MyApp/
  ├── myapp (executable)
  ├── templates/
  ├── config.txt
  └── data.json

// With embedding
MyApp/
  └── myapp (executable with everything inside it)
```

This makes distribution and deployment much simpler and more reliable.

**Approach 1: Using File System (Traditional)**
```go
// main.go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Reads quotes.txt from disk
    data, err := os.ReadFile("quotes.txt")
    if err != nil {
        fmt.Println("Error: quotes.txt must be in same directory!")
        return
    }
    fmt.Println(string(data))
}
```

Directory structure:
```
myproject/
├── main.go
└── quotes.txt

After build:
myproject/
├── main     (binary/executable)
└── quotes.txt
```
You need to distribute BOTH files - the binary and quotes.txt

**Approach 2: Using Embedding**
```go
// main.go
package main

import (
    _ "embed"
    "fmt"
)

//go:embed quotes.txt
var quotes string

func main() {
    // quotes already contains the content of quotes.txt
    fmt.Println(quotes)
}
```

Directory structure after build:
```
myproject/
└── main     (binary/executable with quotes.txt content inside it)
```
You only need to distribute the binary file - it contains everything!

Key differences:
1. With filesystem approach:
   - If quotes.txt is missing, program fails
   - Need to distribute both files
   - Files can be accidentally modified/deleted

2. With embedding:
   - Everything is in one file
   - Can't lose the quotes file
   - Simpler distribution
   - Files can't be modified (they're part of the program)

This is particularly useful when distributing your program to others or deploying to servers - you don't have to worry about managing multiple files!

Best Practices for Using Embedding:
1. Use embedding for:
   - Default configurations
   - Static assets
   - Templates
   - Documentation
   - Default resources

2. Consider filesystem for:
   - User-specific configurations
   - Dynamic content
   - Files that need runtime modifications
   - Large files that might need updates

3. Hybrid Approach:
   - Embed defaults
   - Allow filesystem overrides
   - This gives flexibility while maintaining reliability
