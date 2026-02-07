+++
title = "Parsers in Django REST Framework Explained"
date = 2026-02-07
+++

## 🎯 What Are Parsers?

**Parsers** determine how DRF interprets the incoming request data (request body) based on the `Content-Type` header.

Think of parsers as **translators** that convert raw HTTP request data into Python data structures that your views can use.

---

## 1. The Problem Parsers Solve

### Without Parsers

When a client sends data to your API, it comes as raw bytes:

```
POST /api/products/ HTTP/1.1
Content-Type: application/json

{"name": "Book", "price": "19.99"}
```

**Question:** How does Django know this is JSON and not XML, form data, or a file?

**Answer:** Parsers!

---

## 2. How Parsers Work

### Flow Diagram

```
Client Request
    ↓
Content-Type Header (e.g., "application/json")
    ↓
DRF checks available parsers
    ↓
Matching parser processes request.data
    ↓
Converts to Python dict/list
    ↓
Available as request.data in your view
```

---

## 3. Built-in Parsers

### JSONParser (Default)

**Handles:** `Content-Type: application/json`

**Converts:**
```json
{"name": "Book", "price": 19.99}
```

**To:**
```python
request.data = {
    'name': 'Book',
    'price': 19.99
}
```

**Example:**
```python
# Client sends:
POST /api/products/
Content-Type: application/json

{"name": "Django Book", "price": "29.99"}

# In your view:
def create(self, request):
    print(request.data)  # {'name': 'Django Book', 'price': '29.99'}
    name = request.data['name']  # 'Django Book'
```

---

### FormParser

**Handles:** `Content-Type: application/x-www-form-urlencoded`

This is regular HTML form data (like traditional form submissions).

**Converts:**
```
name=Book&price=19.99
```

**To:**
```python
request.data = {
    'name': 'Book',
    'price': '19.99'
}
```

**Example:**
```python
# Client sends:
POST /api/products/
Content-Type: application/x-www-form-urlencoded

name=Django+Book&price=29.99

# In your view:
def create(self, request):
    print(request.data)  # {'name': 'Django Book', 'price': '29.99'}
```

---

### MultiPartParser

**Handles:** `Content-Type: multipart/form-data`

**Used for:** File uploads and form data combined

**Converts:**
```
------WebKitFormBoundary
Content-Disposition: form-data; name="name"

Django Book
------WebKitFormBoundary
Content-Disposition: form-data; name="image"; filename="book.jpg"
Content-Type: image/jpeg

[binary image data]
------WebKitFormBoundary--
```

**To:**
```python
request.data = {
    'name': 'Django Book'
}
request.FILES = {
    'image': <InMemoryUploadedFile: book.jpg>
}
```

**Example:**
```python
# Client sends:
POST /api/products/
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

# In your view:
def create(self, request):
    print(request.data)   # {'name': 'Django Book'}
    print(request.FILES)  # {'image': <UploadedFile>}
    
    image = request.FILES['image']
    print(image.name)     # 'book.jpg'
    print(image.size)     # 25600 (bytes)
```

---

### FileUploadParser

**Handles:** Raw file upload (entire request body is the file)

**Converts:** Raw binary data → file object

**Example:**
```python
# Client sends:
PUT /api/products/django-book/upload/
Content-Type: image/jpeg
Content-Disposition: attachment; filename="book.jpg"

[raw binary data]

# In your view:
def upload(self, request, filename=None):
    print(request.data)  # <UploadedFile: book.jpg>
    file = request.data['file']
```

**Use case:** Simple file upload without form data

---

## 4. Configuring Parsers

### Global Configuration (settings.py)

```python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.FormParser',
        'rest_framework.parsers.MultiPartParser',
    ]
}
```

**This means:** Your API accepts JSON, form data, and file uploads by default.

---

### Per-ViewSet Configuration

```python
from rest_framework.parsers import JSONParser, MultiPartParser, FormParser


class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    # Only accept JSON
    parser_classes = [JSONParser]
```

**Result:** This viewset only accepts `Content-Type: application/json`

---

### Per-Action Configuration

```python
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    # Default: JSON only
    parser_classes = [JSONParser]
    
    @action(
        detail=True,
        methods=['post'],
        parser_classes=[MultiPartParser, FormParser]  # Override for this action
    )
    def upload_image(self, request, pk=None):
        """This action accepts file uploads."""
        image = request.FILES['image']
        # ... handle upload
```

**Result:**
- `POST /api/products/` → Accepts only JSON
- `POST /api/products/1/upload_image/` → Accepts file uploads

---

## 5. Real-World Examples

### Example 1: Product Creation (JSON)

```python
from rest_framework.parsers import JSONParser


class ProductViewSet(viewsets.ModelViewSet):
    parser_classes = [JSONParser]  # Only JSON
    
    def create(self, request):
        # Client must send:
        # Content-Type: application/json
        # {"name": "Book", "price": 19.99}
        
        print(request.data)  # {'name': 'Book', 'price': 19.99}
        serializer = ProductSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=201)
```

**Client request:**
```bash
curl -X POST http://localhost:8000/api/products/ \
  -H "Content-Type: application/json" \
  -d '{"name": "Django Book", "price": 29.99}'
```

---

### Example 2: Image Upload (MultiPart)

```python
from rest_framework.parsers import MultiPartParser, FormParser


class ProductViewSet(viewsets.ModelViewSet):
    # Accept both JSON and file uploads
    parser_classes = [JSONParser, MultiPartParser, FormParser]
    
    @action(detail=True, methods=['post'])
    def upload_image(self, request, pk=None):
        # Client must send:
        # Content-Type: multipart/form-data
        
        print(request.data)   # {'alt_text': 'Product image'}
        print(request.FILES)  # {'image': <UploadedFile>}
        
        if 'image' not in request.FILES:
            return Response({'error': 'No image provided'}, status=400)
        
        product = self.get_object()
        product.image = request.FILES['image']
        product.save()
        
        return Response({'message': 'Image uploaded'})
```

**Client request:**
```bash
curl -X POST http://localhost:8000/api/products/1/upload_image/ \
  -H "Content-Type: multipart/form-data" \
  -F "image=@book.jpg" \
  -F "alt_text=Product image"
```

---

### Example 3: Mixed Data + Files

```python
from rest_framework.parsers import MultiPartParser, FormParser, JSONParser


class OrderViewSet(viewsets.ModelViewSet):
    parser_classes = [MultiPartParser, FormParser, JSONParser]
    
    def create(self, request):
        # Can accept:
        # 1. Pure JSON
        # 2. Form data
        # 3. Form data + files
        
        print(f"Content-Type: {request.content_type}")
        print(f"Data: {request.data}")
        print(f"Files: {request.FILES}")
        
        serializer = OrderSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        # Handle optional receipt image
        if 'receipt' in request.FILES:
            order = serializer.save(receipt=request.FILES['receipt'])
        else:
            order = serializer.save()
        
        return Response(serializer.data, status=201)
```

**Client can send either:**

**Option 1: JSON**
```bash
curl -X POST http://localhost:8000/api/orders/ \
  -H "Content-Type: application/json" \
  -d '{"items": [1, 2, 3], "total": 99.99}'
```

**Option 2: Form + File**
```bash
curl -X POST http://localhost:8000/api/orders/ \
  -F "items=1,2,3" \
  -F "total=99.99" \
  -F "receipt=@receipt.pdf"
```

---

## 6. Custom Parser

### Create Custom XML Parser

```python
from rest_framework.parsers import BaseParser
from rest_framework.exceptions import ParseError
import xml.etree.ElementTree as ET


class XMLParser(BaseParser):
    """
    Custom parser for XML data.
    """
    media_type = 'application/xml'
    
    def parse(self, stream, media_type=None, parser_context=None):
        """
        Parse XML request body.
        
        Input:
        <product>
            <name>Book</name>
            <price>19.99</price>
        </product>
        
        Output:
        {'name': 'Book', 'price': '19.99'}
        """
        try:
            data = stream.read().decode('utf-8')
            tree = ET.fromstring(data)
            
            # Convert XML to dict
            result = {}
            for child in tree:
                result[child.tag] = child.text
            
            return result
            
        except Exception as e:
            raise ParseError(f'XML parse error: {str(e)}')


# Use in ViewSet
class ProductViewSet(viewsets.ModelViewSet):
    parser_classes = [JSONParser, XMLParser]  # Accept JSON or XML
    
    def create(self, request):
        # Works with both:
        # Content-Type: application/json
        # Content-Type: application/xml
        
        print(request.data)  # Always a Python dict
        # JSON: {'name': 'Book', 'price': 19.99}
        # XML:  {'name': 'Book', 'price': '19.99'}
```

---

## 7. Parser Priority

### Order Matters!

```python
parser_classes = [JSONParser, FormParser, MultiPartParser]
```

**DRF tries parsers in order:**
1. First, try JSONParser
2. If fails (or Content-Type doesn't match), try FormParser
3. If fails, try MultiPartParser
4. If all fail, return 415 Unsupported Media Type

---

### Content Negotiation

```python
# Client specifies what they're sending
POST /api/products/
Content-Type: application/json  # <-- Parser selected based on this

{"name": "Book"}
```

**DRF matches `Content-Type` to parser's `media_type`:**
- `application/json` → JSONParser
- `application/x-www-form-urlencoded` → FormParser
- `multipart/form-data` → MultiPartParser

---

## 8. Common Patterns

### Pattern 1: API Accepts JSON Only (Strict)

```python
class ProductViewSet(viewsets.ModelViewSet):
    parser_classes = [JSONParser]  # Only JSON
    
    # Client MUST send: Content-Type: application/json
    # Otherwise: 415 Unsupported Media Type
```

**Use when:** Building a pure REST API for SPAs, mobile apps

---

### Pattern 2: File Upload Endpoints

```python
class ProductViewSet(viewsets.ModelViewSet):
    parser_classes = [JSONParser]  # Default: JSON only
    
    @action(detail=True, methods=['post'], parser_classes=[MultiPartParser])
    def upload_image(self, request, pk=None):
        # This specific action accepts file uploads
        pass
```

**Use when:** Most endpoints are JSON, but some need file uploads

---

### Pattern 3: Flexible API (Accept Multiple Formats)

```python
class ProductViewSet(viewsets.ModelViewSet):
    parser_classes = [JSONParser, FormParser, MultiPartParser]
    
    # Accepts:
    # - application/json
    # - application/x-www-form-urlencoded
    # - multipart/form-data
```

**Use when:** Supporting various clients (web forms, mobile apps, etc.)

---

## 9. Debugging Parsers

### Check What's Being Parsed

```python
class ProductViewSet(viewsets.ModelViewSet):
    def create(self, request):
        # Debug information
        print(f"Content-Type: {request.content_type}")
        print(f"Parser used: {request.parser_context['parser']}")
        print(f"Parsed data: {request.data}")
        print(f"Files: {request.FILES}")
        print(f"Raw body (first 100 bytes): {request.body[:100]}")
        
        # Your logic here
        pass
```

**Output:**
```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary
Parser used: <MultiPartParser>
Parsed data: {'name': 'Book', 'price': '19.99'}
Files: {'image': <InMemoryUploadedFile: book.jpg>}
```

---

## 10. Common Issues & Solutions

### ❌ Issue 1: 415 Unsupported Media Type

**Error:**
```json
{
  "detail": "Unsupported media type \"application/xml\" in request."
}
```

**Cause:** Client sent `Content-Type` that no parser handles

**Solution:**
```python
# Add appropriate parser
parser_classes = [JSONParser, XMLParser]  # Now supports XML
```

---

### ❌ Issue 2: Files Not Appearing in request.FILES

**Problem:**
```python
def upload_image(self, request):
    print(request.FILES)  # Empty!
```

**Cause:** Missing MultiPartParser

**Solution:**
```python
@action(detail=True, methods=['post'], parser_classes=[MultiPartParser, FormParser])
def upload_image(self, request):
    print(request.FILES)  # Now has files!
```

---

### ❌ Issue 3: JSON Not Parsing

**Problem:**
```python
# Client sends valid JSON but request.data is empty
```

**Cause:** Missing JSONParser or wrong Content-Type

**Solution:**
```python
# In ViewSet
parser_classes = [JSONParser]  # Add this

# Client must send:
# Content-Type: application/json  # Not text/plain or missing!
```

---

## 11. Quick Reference

| Parser | Content-Type | Use Case | request.data | request.FILES |
|--------|--------------|----------|--------------|---------------|
| JSONParser | `application/json` | REST APIs, SPAs | dict/list | Empty |
| FormParser | `application/x-www-form-urlencoded` | HTML forms | dict | Empty |
| MultiPartParser | `multipart/form-data` | File uploads | dict | dict of files |
| FileUploadParser | Any | Raw file upload | file object | file object |

---

## Summary

**Parsers = Content-Type handlers**

```python
# Think of it like this:
if Content-Type == "application/json":
    use JSONParser
elif Content-Type == "multipart/form-data":
    use MultiPartParser
elif Content-Type == "application/x-www-form-urlencoded":
    use FormParser
else:
    return 415 Unsupported Media Type
```

**Key Points:**
- Parsers convert raw request data → Python structures
- Match based on `Content-Type` header
- Configure globally or per-view
- **Always add MultiPartParser for file uploads!**
- Order matters in `parser_classes`

**Golden Rule for Images:**
```python
# Always use these for file uploads:
parser_classes = [MultiPartParser, FormParser]
```
