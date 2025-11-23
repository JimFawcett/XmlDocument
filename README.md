# XmlDocument

A lightweight, self-contained C++17 XML object model and parser for Windows. XmlDocument provides a simple hierarchy for representing XML documents in memory with a DOM-style interface for construction, traversal, and serialization.

[![Platform](https://img.shields.io/badge/platform-Windows%2010%2B-blue)](https://github.com/JimFawcett/XmlDocument)
[![C++17](https://img.shields.io/badge/C%2B%2B-17-orange)](https://github.com/JimFawcett/XmlDocument)
[![Compiler](https://img.shields.io/badge/compiler-MSVC-green)](https://github.com/JimFawcett/XmlDocument)

## Overview

XmlDocument is designed to be:
- **Small and predictable** - Minimal footprint with clear, understandable implementation
- **Self-contained** - No external dependencies beyond C++17 standard library
- **Easy to use** - Intuitive API suitable for both professional use and educational purposes
- **Composable** - Simple element hierarchy that's easy to extend

### Key Features

- **Complete XML tree model** with support for all standard node types
- **Construction APIs** for elements, attributes, text, comments, and processing instructions
- **Flexible search** operations based on tag names or custom predicates
- **Load/save** functionality for both strings and files
- **DOM-style interface** familiar to developers with XML experience
- **Modern C++17** implementation with smart pointer semantics

## Quick Start

### Basic Usage Example

```cpp
#include "XmlDocument.h"
#include <iostream>

int main() {
    // Create a new document
    XmlDocument doc;
    
    // Build XML structure
    auto root = doc.makeTaggedElement("library");
    doc.setRoot(root);
    
    auto book = doc.makeTaggedElement("book");
    book->addAttribute("id", "001");
    book->addAttribute("genre", "fiction");
    
    auto title = doc.makeTaggedElement("title");
    title->addChild(doc.makeTextElement("The Great Gatsby"));
    
    auto author = doc.makeTaggedElement("author");
    author->addChild(doc.makeTextElement("F. Scott Fitzgerald"));
    
    book->addChild(title);
    book->addChild(author);
    root->addChild(book);
    
    // Save to file
    doc.save("library.xml");
    
    // Print to console
    std::cout << doc.toString() << std::endl;
    
    return 0;
}
```

### Loading and Querying XML

```cpp
#include "XmlDocument.h"

// Load from file
XmlDocument doc;
if (doc.load("library.xml")) {
    // Find all book elements
    auto books = doc.find("book");
    
    // Search with custom predicate
    auto fictionBooks = doc.select([](auto elem) {
        auto attrs = elem->attributes();
        return attrs.find("genre") != attrs.end() && 
               attrs["genre"] == "fiction";
    });
    
    // Iterate through results
    for (auto& book : fictionBooks) {
        std::cout << "Found fiction book: " << book->value() << std::endl;
    }
}

// Parse from string
std::string xmlText = R"(
<catalog>
    <item id="1">Widget</item>
    <item id="2">Gadget</item>
</catalog>
)";

XmlDocument doc2(xmlText);
auto items = doc2.find("item");
```

## Architecture

XmlDocument uses a polymorphic element hierarchy with shared ownership semantics for safe memory management.

### Element Type Hierarchy

```
AbstractXmlElement (abstract base)
    ├── DocElement              (document root container)
    ├── TaggedElement            (standard XML elements)
    ├── TextElement              (text nodes)
    ├── CommentElement           (<!-- comments -->)
    ├── ProcInstrElement         (<?processing instructions?>)
    └── XmlDeclElement           (<?xml declarations?>)
```

### Element Types Reference

| Class | Purpose | Example |
|-------|---------|---------|
| **DocElement** | Top-level document container with optional prologue/epilogue | Holds root element and metadata |
| **TaggedElement** | Standard XML element with tag name, attributes, and children | `<book id="1">...</book>` |
| **TextElement** | Pure text content node | Text inside elements |
| **CommentElement** | XML comment | `<!-- This is a comment -->` |
| **ProcInstrElement** | Processing instruction with target and attributes | `<?xml-stylesheet type="text/css"?>` |
| **XmlDeclElement** | XML declaration | `<?xml version="1.0" encoding="UTF-8"?>` |

## API Reference

### Document Construction

```cpp
// Create empty document
XmlDocument();

// Parse from XML string
XmlDocument(const std::string& xmlText);

// Load from file
bool load(const std::string& filePath);

// Save to file
bool save(const std::string& filePath) const;

// Serialize to string
std::string toString() const;

// Access/set root element
std::shared_ptr<AbstractXmlElement> root() const;
void setRoot(std::shared_ptr<AbstractXmlElement> elem);
```

### Element Creation (Factory Methods)

```cpp
// Create tagged element
auto makeTaggedElement(const std::string& tag);

// Create text node
auto makeTextElement(const std::string& text);

// Create comment
auto makeCommentElement(const std::string& text);

// Create processing instruction
auto makeProcInstrElement(const std::string& target);

// Create XML declaration
auto makeXmlDeclarElement();
```

### Search Operations

```cpp
// Find elements by tag name
std::vector<std::shared_ptr<AbstractXmlElement>>
    find(const std::string& tag);

// Select elements using custom predicate
std::vector<std::shared_ptr<AbstractXmlElement>>
    select(bool(*predicate)(std::shared_ptr<AbstractXmlElement>));
```

### Element Manipulation

```cpp
// Child management
void addChild(std::shared_ptr<AbstractXmlElement> child);
void removeChild(std::shared_ptr<AbstractXmlElement> child);
std::vector<std::shared_ptr<AbstractXmlElement>> children() const;

// Attribute management
void addAttribute(const std::string& name, const std::string& value);
void removeAttribute(const std::string& name);
std::unordered_map<std::string, std::string> attributes() const;

// Get element content
std::string value() const;
```

## Project Structure

```
XmlDocument/
├── XmlDocument/         # Core XmlDocument class implementation
├── XmlElement/          # Element hierarchy (abstract + concrete types)
├── XmlElementParts/     # Parser support and helper components
├── XmlParser/           # XML parser and tokenizer
├── XmlHelpDemo/         # Demonstration programs showing usage
├── Utilities/           # General helper code
└── XmlDocument.sln      # Visual Studio solution file
```

## Building

### Requirements

- **Platform:** Windows 10 or later
- **Compiler:** MSVC (Visual Studio 2019 or newer)
- **Standard:** C++17
- **Dependencies:** None (standard library only)

### Build Instructions

1. Open `XmlDocument.sln` in Visual Studio
2. Select your desired configuration (Debug/Release)
3. Build the solution (Ctrl+Shift+B)
4. Run the demo project `XmlHelpDemo` to see examples

No external libraries or package managers required!

## Advanced Examples

### Working with Attributes

```cpp
XmlDocument doc;
auto element = doc.makeTaggedElement("person");

// Add attributes
element->addAttribute("id", "12345");
element->addAttribute("name", "John Doe");
element->addAttribute("age", "30");

// Access attributes
auto attrs = element->attributes();
if (attrs.find("name") != attrs.end()) {
    std::cout << "Name: " << attrs["name"] << std::endl;
}

// Remove attribute
element->removeAttribute("age");
```

### Creating Complex Documents

```cpp
XmlDocument doc;

// Create XML declaration
auto decl = doc.makeXmlDeclarElement();
decl->addAttribute("version", "1.0");
decl->addAttribute("encoding", "UTF-8");

// Build document structure
auto root = doc.makeTaggedElement("catalog");
doc.setRoot(root);

// Add comment
auto comment = doc.makeCommentElement("Product catalog for 2024");
root->addChild(comment);

// Add products
for (int i = 0; i < 3; i++) {
    auto product = doc.makeTaggedElement("product");
    product->addAttribute("id", std::to_string(i + 1));
    
    auto name = doc.makeTaggedElement("name");
    name->addChild(doc.makeTextElement("Product " + std::to_string(i + 1)));
    
    auto price = doc.makeTaggedElement("price");
    price->addAttribute("currency", "USD");
    price->addChild(doc.makeTextElement(std::to_string(19.99 * (i + 1))));
    
    product->addChild(name);
    product->addChild(price);
    root->addChild(product);
}

doc.save("catalog.xml");
```

### Custom Search Predicates

```cpp
// Find elements with specific attribute value
auto expensiveItems = doc.select([](auto elem) {
    auto attrs = elem->attributes();
    if (attrs.find("price") != attrs.end()) {
        double price = std::stod(attrs["price"]);
        return price > 50.0;
    }
    return false;
});

// Find elements by type
auto findTextNodes = [](auto elem) {
    return dynamic_cast<TextElement*>(elem.get()) != nullptr;
};
auto textNodes = doc.select(findTextNodes);

// Complex search combining multiple criteria
auto findSpecificBooks = [](auto elem) {
    if (elem->tag() != "book") return false;
    
    auto attrs = elem->attributes();
    bool hasISBN = attrs.find("isbn") != attrs.end();
    bool isFiction = attrs["genre"] == "fiction";
    
    return hasISBN && isFiction;
};
```

## Status and Roadmap

### Current Status

✅ Stable and functional for small to medium XML workloads  
✅ Complete element model and parser  
✅ Actively maintained for clarity and correctness  
✅ Educational and professional use ready  

### Known Limitations

- No DTD or XML Schema validation
- No XPath or advanced query language support
- Not optimized for extremely large documents (multi-GB)
- Windows-only (by design)

### Design Philosophy

This library is intentionally kept simple and focused. It provides essential XML document handling without the complexity of full-featured XML frameworks. This makes it ideal for:

- **Educational purposes** - Clear, readable implementation for learning
- **Embedded use** - Lightweight XML handling in larger applications
- **Tool development** - Quick XML processing without heavy dependencies
- **Prototyping** - Fast development with minimal setup

## Documentation

For detailed design notes and additional documentation, visit:

**[WebSite Documentation](https://jimfawcett.github.io/NewSite/Cpp/RepoCpp_XmlDocument.html)**

## Contributing

Contributions are welcome! Here's how you can help:

1. **Report Issues** - Found a bug? [Open an issue](https://github.com/JimFawcett/XmlDocument/issues)
2. **Suggest Features** - Have an idea? Discuss it in issues first
3. **Submit Pull Requests** - Fork, branch, code, and submit PRs
4. **Improve Documentation** - Help make the docs clearer

### Contribution Guidelines

- Follow existing code style and conventions
- Write clear commit messages
- Add tests for new functionality where appropriate
- Update documentation as needed
- Keep changes focused and atomic

## License

No explicit license is currently included in this repository. If you intend to use this code in your projects, please verify usage terms with the repository owner or request that an open-source license (MIT, Apache 2.0, etc.) be added.

## Author

**Jim Fawcett, Ph.D.**  
Emeritus Teaching Professor  
[JimFawcett.github.io](https://jimfawcett.github.io)

Originally developed for CSE687 - Object Oriented Design coursework at Syracuse University.

## Support

- **Issues:** [GitHub Issues](https://github.com/JimFawcett/XmlDocument/issues)
- **Documentation:** [https://JimFawcett.github.io/XmlDocument.html](https://JimFawcett.github.io/XmlDocument.html)
- **Pull Requests:** [GitHub Pull Requests](https://github.com/JimFawcett/XmlDocument/pulls)

## Acknowledgments

This project was developed as part of teaching materials for Object-Oriented Design courses. Special thanks to the students who provided feedback and testing throughout its development.

---

**Note:** This is a Windows-specific implementation using MSVC. For cross-platform XML handling in C++, consider libraries like pugixml, TinyXML-2, or RapidXML.
