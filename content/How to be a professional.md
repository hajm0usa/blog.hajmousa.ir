+++
title = "The Professional Django & Django REST Framework Handbook"
date = 2025-12-09
+++
## Introduction

This handbook is your comprehensive guide to becoming a professional Django and Django REST Framework (DRF) developer. We'll cover everything from fundamental concepts to advanced patterns that separate junior developers from seasoned professionals.

## Part 1: Django Fundamentals

### Understanding Django's Philosophy

Django follows the "batteries included" philosophy, providing everything you need out of the box. Professional developers understand these core principles:

- **Don't Repeat Yourself (DRY)**: Write reusable code
- **Convention over Configuration**: Follow Django's patterns
- **Explicit is better than implicit**: Clear, readable code wins
- **Loose coupling, high cohesion**: Components should be independent but focused

### Project Structure

A professional Django project structure looks like this:

```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/
│   ├── users/
│   ├── products/
│   └── orders/
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   └── production.txt
├── static/
├── media/
├── templates/
└── tests/
```

## Part 2: Models - The Foundation

### Model Design Principles

Professional model design considers:

1. **Normalization vs Denormalization**: Balance database efficiency with query performance
2. **Single Responsibility**: Each model should represent one concept
3. **Relationships**: Use appropriate relationship types
4. **Database Constraints**: Enforce data integrity at the database level

### Essential Model Patterns

#### Abstract Base Models

Create reusable model behaviors:

```python
from django.db import models
from django.utils import timezone

class TimeStampedModel(models.Model):
    """Abstract model providing created and modified timestamps."""
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True
        ordering = ['-created_at']

class SoftDeleteModel(models.Model):
    """Abstract model providing soft delete functionality."""
    is_deleted = models.BooleanField(default=False)
    deleted_at = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        abstract = True
    
    def delete(self, using=None, keep_parents=False):
        """Soft delete instead of actual deletion."""
        self.is_deleted = True
        self.deleted_at = timezone.now()
        self.save()
    
    def hard_delete(self):
        """Actually delete the record."""
        super().delete()
```

#### Using Model Managers

Custom managers encapsulate query logic:

```python
from django.db import models

class ActiveManager(models.Manager):
    """Manager that returns only non-deleted records."""
    def get_queryset(self):
        return super().get_queryset().filter(is_deleted=False)

class Product(TimeStampedModel, SoftDeleteModel):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    
    # Multiple managers
    objects = models.Manager()  # Default manager
    active = ActiveManager()     # Custom manager
    
    class Meta:
        db_table = 'products'
        indexes = [
            models.Index(fields=['name']),
            models.Index(fields=['-created_at']),
        ]
    
    def __str__(self):
        return self.name
    
    @property
    def is_in_stock(self):
        return self.stock > 0
```

#### Model Methods vs Properties

```python
class Order(TimeStampedModel):
    user = models.ForeignKey('users.User', on_delete=models.CASCADE)
    status = models.CharField(max_length=20, choices=[
        ('pending', 'Pending'),
        ('processing', 'Processing'),
        ('completed', 'Completed'),
        ('cancelled', 'Cancelled'),
    ])
    
    # Property for simple calculations
    @property
    def total_amount(self):
        return sum(item.subtotal for item in self.items.all())
    
    # Method for actions
    def cancel(self):
        if self.status in ['pending', 'processing']:
            self.status = 'cancelled'
            self.save()
            # Trigger any necessary side effects
            self.refund_payment()
            self.send_cancellation_email()
        else:
            raise ValueError("Cannot cancel completed orders")
    
    def refund_payment(self):
        # Payment refund logic
        pass
    
    def send_cancellation_email(self):
        # Email logic
        pass
```

#### Advanced Field Types

```python
from django.contrib.postgres.fields import ArrayField, JSONField
from django.core.validators import MinValueValidator, MaxValueValidator

class Product(models.Model):
    # Text fields with validation
    name = models.CharField(
        max_length=200,
        db_index=True,
        help_text="Product name"
    )
    slug = models.SlugField(unique=True, max_length=200)
    
    # Numeric fields with validators
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        validators=[MinValueValidator(0)]
    )
    discount_percentage = models.IntegerField(
        default=0,
        validators=[MinValueValidator(0), MaxValueValidator(100)]
    )
    
    # PostgreSQL-specific fields
    tags = ArrayField(
        models.CharField(max_length=50),
        blank=True,
        default=list
    )
    metadata = models.JSONField(default=dict, blank=True)
    
    # File fields
    image = models.ImageField(upload_to='products/%Y/%m/%d/')
    
    # Choices using TextChoices (Django 3.0+)
    class CategoryChoices(models.TextChoices):
        ELECTRONICS = 'ELEC', 'Electronics'
        CLOTHING = 'CLTH', 'Clothing'
        BOOKS = 'BOOK', 'Books'
    
    category = models.CharField(
        max_length=4,
        choices=CategoryChoices.choices,
        default=CategoryChoices.ELECTRONICS
    )
```

### Relationships Done Right

#### One-to-Many

```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,  # Delete books when author is deleted
        related_name='books'  # Access books via author.books.all()
    )
```

#### Many-to-Many with Through Model

```python
class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField(
        'Course',
        through='Enrollment',
        related_name='students'
    )

class Course(models.Model):
    name = models.CharField(max_length=100)

class Enrollment(models.Model):
    """Through model with additional fields."""
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_date = models.DateField(auto_now_add=True)
    grade = models.CharField(max_length=2, blank=True)
    
    class Meta:
        unique_together = ['student', 'course']
```

#### One-to-One

```python
class User(AbstractUser):
    pass

class Profile(models.Model):
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile'
    )
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True)
    
    def __str__(self):
        return f"{self.user.username}'s profile"
```

### Query Optimization

Professional developers write efficient queries:

#### Select Related and Prefetch Related

```python
# Bad: N+1 query problem
books = Book.objects.all()
for book in books:
    print(book.author.name)  # Hits database for each book!

# Good: Use select_related for ForeignKey
books = Book.objects.select_related('author').all()
for book in books:
    print(book.author.name)  # No additional queries

# Prefetch related for reverse ForeignKey and ManyToMany
authors = Author.objects.prefetch_related('books').all()
for author in authors:
    for book in author.books.all():  # Uses prefetched data
        print(book.title)
```

#### Advanced Prefetching

```python
from django.db.models import Prefetch

# Prefetch with filtering
active_orders = Order.objects.filter(status='active')
customers = Customer.objects.prefetch_related(
    Prefetch('orders', queryset=active_orders, to_attr='active_orders')
)

# Multi-level prefetching
orders = Order.objects.prefetch_related(
    'items__product',
    'items__product__category'
).select_related('user')
```

#### Using only() and defer()

```python
# Load only specific fields
products = Product.objects.only('id', 'name', 'price')

# Defer loading of heavy fields
products = Product.objects.defer('description', 'metadata')
```

#### Aggregation and Annotation

```python
from django.db.models import Count, Sum, Avg, F, Q, Value
from django.db.models.functions import Coalesce

# Simple aggregation
from django.db.models import Count
Author.objects.aggregate(total_books=Count('books'))

# Annotation adds calculated fields to each object
authors = Author.objects.annotate(
    book_count=Count('books'),
    total_sales=Sum('books__sales')
).filter(book_count__gte=3)

# F expressions for database-level operations
Product.objects.update(price=F('price') * 1.1)  # 10% price increase

# Complex annotations
orders = Order.objects.annotate(
    item_count=Count('items'),
    total=Sum(F('items__quantity') * F('items__price')),
    discounted_total=Coalesce(
        Sum(F('items__quantity') * F('items__price') * (100 - F('discount')) / 100),
        Value(0)
    )
)

# Conditional aggregation
from django.db.models import Case, When

Product.objects.aggregate(
    expensive_count=Count(Case(When(price__gte=100, then=1))),
    cheap_count=Count(Case(When(price__lt=100, then=1)))
)
```

### Model Validation

```python
from django.core.exceptions import ValidationError
from django.core.validators import validate_email

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    cost = models.DecimalField(max_digits=10, decimal_places=2)
    
    def clean(self):
        """Model-level validation."""
        if self.price < self.cost:
            raise ValidationError({
                'price': 'Price cannot be less than cost'
            })
        
        if self.price < 0:
            raise ValidationError('Price must be positive')
    
    def save(self, *args, **kwargs):
        """Call clean before saving."""
        self.full_clean()
        super().save(*args, **kwargs)
```

### Signals - Use Sparingly

Signals can create hidden dependencies. Use them wisely:

```python
from django.db.models.signals import post_save, pre_delete
from django.dispatch import receiver

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """Create profile when user is created."""
    if created:
        Profile.objects.create(user=instance)

@receiver(pre_delete, sender=Order)
def restore_stock(sender, instance, **kwargs):
    """Restore product stock when order is deleted."""
    for item in instance.items.all():
        item.product.stock += item.quantity
        item.product.save()
```

**Professional Tip**: Consider using model methods or service layers instead of signals when possible. Signals make code harder to trace and test.

## Part 3: Django REST Framework Serializers

Serializers are the heart of DRF, converting complex data types to Python datatypes and vice versa.

### Basic Serializers

#### ModelSerializer

```python
from rest_framework import serializers

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'stock', 'created_at']
        read_only_fields = ['id', 'created_at']
```

#### Explicit Field Definition

```python
class ProductSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    name = serializers.CharField(max_length=200)
    price = serializers.DecimalField(max_digits=10, decimal_places=2)
    stock = serializers.IntegerField(min_value=0)
    
    def create(self, validated_data):
        return Product.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        instance.name = validated_data.get('name', instance.name)
        instance.price = validated_data.get('price', instance.price)
        instance.stock = validated_data.get('stock', instance.stock)
        instance.save()
        return instance
```

### Advanced Serializer Patterns

#### Nested Serializers

```python
class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ['id', 'name', 'email']

class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(read_only=True)
    author_id = serializers.IntegerField(write_only=True)
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'author_id', 'published_date']
```

#### SerializerMethodField

```python
class OrderSerializer(serializers.ModelSerializer):
    total_amount = serializers.SerializerMethodField()
    customer_name = serializers.SerializerMethodField()
    status_display = serializers.CharField(source='get_status_display', read_only=True)
    
    class Meta:
        model = Order
        fields = ['id', 'customer_name', 'status', 'status_display', 'total_amount']
    
    def get_total_amount(self, obj):
        return sum(item.subtotal for item in obj.items.all())
    
    def get_customer_name(self, obj):
        return obj.user.get_full_name()
```

#### Different Serializers for Different Actions

```python
class ProductListSerializer(serializers.ModelSerializer):
    """Lightweight serializer for list views."""
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'stock']

class ProductDetailSerializer(serializers.ModelSerializer):
    """Detailed serializer with relationships."""
    category = CategorySerializer(read_only=True)
    reviews = ReviewSerializer(many=True, read_only=True)
    average_rating = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = [
            'id', 'name', 'description', 'price', 'stock',
            'category', 'reviews', 'average_rating', 'created_at'
        ]
    
    def get_average_rating(self, obj):
        return obj.reviews.aggregate(Avg('rating'))['rating__avg']

class ProductCreateSerializer(serializers.ModelSerializer):
    """Serializer optimized for creation."""
    class Meta:
        model = Product
        fields = ['name', 'description', 'price', 'stock', 'category_id']
```

#### Dynamic Fields

```python
class DynamicFieldsSerializer(serializers.ModelSerializer):
    """Serializer that allows dynamic field selection."""
    
    def __init__(self, *args, **kwargs):
        # Extract fields parameter before passing to parent
        fields = kwargs.pop('fields', None)
        exclude = kwargs.pop('exclude', None)
        
        super().__init__(*args, **kwargs)
        
        if fields is not None:
            # Drop fields not specified
            allowed = set(fields)
            existing = set(self.fields)
            for field_name in existing - allowed:
                self.fields.pop(field_name)
        
        if exclude is not None:
            for field_name in exclude:
                self.fields.pop(field_name, None)

class ProductSerializer(DynamicFieldsSerializer):
    class Meta:
        model = Product
        fields = '__all__'

# Usage in view
serializer = ProductSerializer(products, many=True, fields=['id', 'name', 'price'])
```

### Validation

#### Field-Level Validation

```python
class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ['name', 'price', 'cost', 'stock']
    
    def validate_price(self, value):
        """Validate individual field."""
        if value <= 0:
            raise serializers.ValidationError("Price must be positive")
        if value > 1000000:
            raise serializers.ValidationError("Price too high")
        return value
    
    def validate_name(self, value):
        """Check for duplicate names."""
        if Product.objects.filter(name__iexact=value).exists():
            raise serializers.ValidationError("Product with this name already exists")
        return value
```

#### Object-Level Validation

```python
class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ['name', 'price', 'cost', 'stock']
    
    def validate(self, data):
        """Cross-field validation."""
        if data.get('price', 0) < data.get('cost', 0):
            raise serializers.ValidationError({
                'price': 'Price cannot be less than cost'
            })
        
        if data.get('stock', 0) < 0:
            raise serializers.ValidationError({
                'stock': 'Stock cannot be negative'
            })
        
        return data
```

#### Custom Validators

```python
def validate_future_date(value):
    """Validator function for future dates."""
    if value <= timezone.now().date():
        raise serializers.ValidationError("Date must be in the future")

class EventSerializer(serializers.ModelSerializer):
    event_date = serializers.DateField(validators=[validate_future_date])
    
    class Meta:
        model = Event
        fields = ['name', 'event_date', 'location']
```

### Handling Relationships

#### Writable Nested Serializers

```python
class OrderItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = OrderItem
        fields = ['product_id', 'quantity', 'price']

class OrderSerializer(serializers.ModelSerializer):
    items = OrderItemSerializer(many=True)
    
    class Meta:
        model = Order
        fields = ['id', 'user', 'status', 'items']
    
    def create(self, validated_data):
        items_data = validated_data.pop('items')
        order = Order.objects.create(**validated_data)
        
        for item_data in items_data:
            OrderItem.objects.create(order=order, **item_data)
        
        return order
    
    def update(self, instance, validated_data):
        items_data = validated_data.pop('items', None)
        
        # Update order fields
        instance.status = validated_data.get('status', instance.status)
        instance.save()
        
        # Handle items update
        if items_data is not None:
            # Remove existing items
            instance.items.all().delete()
            # Create new items
            for item_data in items_data:
                OrderItem.objects.create(order=instance, **item_data)
        
        return instance
```

#### PrimaryKeyRelatedField, SlugRelatedField

```python
class BookSerializer(serializers.ModelSerializer):
    # Use author's ID
    author_id = serializers.PrimaryKeyRelatedField(
        queryset=Author.objects.all(),
        source='author'
    )
    
    # Use author's slug
    author_slug = serializers.SlugRelatedField(
        slug_field='slug',
        queryset=Author.objects.all(),
        source='author'
    )
    
    # Use custom string representation
    author_name = serializers.StringRelatedField(source='author')
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'author_id', 'author_slug', 'author_name']
```

#### HyperlinkedModelSerializer

```python
class ProductSerializer(serializers.HyperlinkedModelSerializer):
    category = serializers.HyperlinkedRelatedField(
        view_name='category-detail',
        read_only=True
    )
    
    class Meta:
        model = Product
        fields = ['url', 'id', 'name', 'price', 'category']
        extra_kwargs = {
            'url': {'view_name': 'product-detail'}
        }
```

### Context and Request Access

```python
class ProductSerializer(serializers.ModelSerializer):
    is_favorited = serializers.SerializerMethodField()
    can_edit = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'is_favorited', 'can_edit']
    
    def get_is_favorited(self, obj):
        request = self.context.get('request')
        if request and request.user.is_authenticated:
            return obj.favorited_by.filter(id=request.user.id).exists()
        return False
    
    def get_can_edit(self, obj):
        request = self.context.get('request')
        if request and request.user.is_authenticated:
            return request.user.has_perm('products.change_product')
        return False

# In view, pass context
serializer = ProductSerializer(product, context={'request': request})
```

### Performance Optimization

#### to_representation Optimization

```python
class OptimizedOrderSerializer(serializers.ModelSerializer):
    items = OrderItemSerializer(many=True, read_only=True)
    
    class Meta:
        model = Order
        fields = ['id', 'user', 'status', 'items', 'total']
    
    def to_representation(self, instance):
        """Optimize by prefetching related data."""
        # This runs once for the queryset, not per instance
        self.fields['items'].child.context['products'] = {
            p.id: p for p in Product.objects.filter(
                id__in=instance.items.values_list('product_id', flat=True)
            )
        }
        return super().to_representation(instance)
```

## Part 4: Views and ViewSets

### Function-Based Views (FBV)

While class-based views are more common in DRF, FBVs have their place:

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from rest_framework import status

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def product_list(request):
    """List products or create new product."""
    if request.method == 'GET':
        products = Product.objects.all()
        serializer = ProductSerializer(products, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = ProductSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def product_detail(request, pk):
    """Retrieve, update or delete a product."""
    try:
        product = Product.objects.get(pk=pk)
    except Product.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)
    
    if request.method == 'GET':
        serializer = ProductSerializer(product)
        return Response(serializer.data)
    
    elif request.method == 'PUT':
        serializer = ProductSerializer(product, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### APIView - Class-Based Views

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404

class ProductListView(APIView):
    """
    List all products or create a new product.
    """
    permission_classes = [IsAuthenticated]
    
    def get(self, request):
        products = Product.objects.all()
        
        # Filtering
        category = request.query_params.get('category')
        if category:
            products = products.filter(category=category)
        
        # Ordering
        ordering = request.query_params.get('ordering', '-created_at')
        products = products.order_by(ordering)
        
        serializer = ProductSerializer(products, many=True, context={'request': request})
        return Response(serializer.data)
    
    def post(self, request):
        serializer = ProductSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(created_by=request.user)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class ProductDetailView(APIView):
    """
    Retrieve, update or delete a product instance.
    """
    
    def get_object(self, pk):
        return get_object_or_404(Product, pk=pk)
    
    def get(self, request, pk):
        product = self.get_object(pk)
        serializer = ProductSerializer(product)
        return Response(serializer.data)
    
    def put(self, request, pk):
        product = self.get_object(pk)
        serializer = ProductSerializer(product, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def patch(self, request, pk):
        product = self.get_object(pk)
        serializer = ProductSerializer(product, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request, pk):
        product = self.get_object(pk)
        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Generic Views

Generic views provide common patterns:

```python
from rest_framework import generics
from rest_framework.permissions import IsAuthenticatedOrReadOnly

class ProductListView(generics.ListCreateAPIView):
    """List and create products."""
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    filterset_fields = ['category', 'price']
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at', 'name']
    
    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)

class ProductDetailView(generics.RetrieveUpdateDestroyAPIView):
    """Retrieve, update, or delete a product."""
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
```

#### All Generic View Classes

- `CreateAPIView` - POST only
- `ListAPIView` - GET list only
- `RetrieveAPIView` - GET single object only
- `DestroyAPIView` - DELETE only
- `UpdateAPIView` - PUT/PATCH only
- `ListCreateAPIView` - GET list, POST
- `RetrieveUpdateAPIView` - GET, PUT, PATCH
- `RetrieveDestroyAPIView` - GET, DELETE
- `RetrieveUpdateDestroyAPIView` - GET, PUT, PATCH, DELETE

### ViewSets - The Professional Choice

ViewSets combine logic for multiple related views:

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response

class ProductViewSet(viewsets.ModelViewSet):
    """
    ViewSet for viewing and editing products.
    
    Automatically provides:
    - list: GET /products/
    - create: POST /products/
    - retrieve: GET /products/{id}/
    - update: PUT /products/{id}/
    - partial_update: PATCH /products/{id}/
    - destroy: DELETE /products/{id}/
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    filterset_fields = ['category', 'price']
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at']
    
    def get_queryset(self):
        """Customize queryset based on action."""
        queryset = super().get_queryset()
        
        if self.action == 'list':
            # Optimize list queries
            queryset = queryset.select_related('category')
        elif self.action == 'retrieve':
            # Optimize detail queries
            queryset = queryset.prefetch_related('reviews', 'reviews__user')
        
        return queryset
    
    def get_serializer_class(self):
        """Use different serializers for different actions."""
        if self.action == 'list':
            return ProductListSerializer
        elif self.action == 'retrieve':
            return ProductDetailSerializer
        return ProductSerializer
    
    def perform_create(self, serializer):
        """Hook called before saving."""
        serializer.save(created_by=self.request.user)
    
    def perform_update(self, serializer):
        """Hook called before updating."""
        serializer.save(updated_by=self.request.user)
    
    # Custom actions
    @action(detail=True, methods=['post'])
    def favorite(self, request, pk=None):
        """Mark product as favorite."""
        product = self.get_object()
        request.user.favorites.add(product)
        return Response({'status': 'product favorited'})
    
    @action(detail=True, methods=['post'])
    def unfavorite(self, request, pk=None):
        """Remove product from favorites."""
        product = self.get_object()
        request.user.favorites.remove(product)
        return Response({'status': 'product unfavorited'})
    
    @action(detail=False, methods=['get'])
    def featured(self, request):
        """Get featured products."""
        featured = self.get_queryset().filter(is_featured=True)
        serializer = self.get_serializer(featured, many=True)
        return Response(serializer.data)
    
    @action(detail=True, methods=['get'])
    def reviews(self, request, pk=None):
        """Get product reviews."""
        product = self.get_object()
        reviews = product.reviews.all()
        serializer = ReviewSerializer(reviews, many=True)
        return Response(serializer.data)
```

#### ReadOnlyModelViewSet

For read-only endpoints:

```python
class CategoryViewSet(viewsets.ReadOnlyModelViewSet):
    """
    ViewSet that only provides list and retrieve actions.
    """
    queryset = Category.objects.all()
    serializer_class = CategorySerializer
```

#### Custom ViewSet

```python
from rest_framework import viewsets, mixins

class CreateListRetrieveViewSet(
    mixins.CreateModelMixin,
    mixins.ListModelMixin,
    mixins.RetrieveModelMixin,
    viewsets.GenericViewSet
):
    """
    ViewSet that provides create, list, and retrieve actions.
    No update or delete.
    """
    pass

class ProductViewSet(CreateListRetrieveViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

### Pagination

```python
from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination, CursorPagination

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    pagination_class = StandardResultsSetPagination

# In settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20
}
```

#### Cursor Pagination for Better Performance

```python
class ProductCursorPagination(CursorPagination):
    page_size = 20
    ordering = '-created_at'  # Must have ordering
    cursor_query_param = 'cursor'

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    pagination_class = ProductCursorPagination
```

### Filtering, Searching, Ordering

```python
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [
        DjangoFilterBackend,
        filters.SearchFilter,
        filters.OrderingFilter
    ]
    
    # Simple filtering
    filterset_fields = ['category', 'price', 'stock']
    
    # Search
    search_fields = ['name', 'description', 'category__name']
    
    # Ordering
    ordering_fields = ['price', 'created_at', 'name']
    ordering = ['-created_at']  # Default ordering
```

#### Custom Filters

```python
import django_filters

class ProductFilter(django_filters.FilterSet):
    min_price = django_filters.NumberFilter(field_name='price', lookup_expr='gte')
    max_price = django_filters.NumberFilter(field_name='price', lookup_expr='lte')
    name = django_filters.CharFilter(lookup_expr='icontains')
    in_stock = django_filters.BooleanFilter(method='filter_in_stock')
    
    class Meta:
        model = Product
        fields = ['category', 'name']
    
    def filter_in_stock(self, queryset, name, value):
        if value:
            return queryset.filter(stock__gt=0)
        return queryset.filter(stock=0)

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filterset_class = ProductFilter
```

### Throttling

```python
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle

class BurstRateThrottle(UserRateThrottle):
    rate = '60/min'

class SustainedRateThrottle(UserRateThrottle):
    rate = '1000/day'

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    throttle_classes = [BurstRateThrottle, SustainedRateThrottle]

# In settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```

### Exception Handling

```python
from rest_framework.views import exception_handler
from rest_framework.response import Response

def custom_exception_handler(exc, context):
    """Custom exception handler."""
    response = exception_handler(exc, context)
    
    if response is not None:
        # Add custom error format
        custom_response = {
            'error': {
                'status_code': response.status_code,
                'message': response.data,
                'detail': str(exc)
            }
        }
        response.data = custom_response
    
    return response

# In settings.py
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'myapp.utils.custom_exception_handler'
}
```

#### Raising Custom Exceptions

```python
from rest_framework.exceptions import APIException
from rest_framework import status

class ServiceUnavailable(APIException):
    status_code = status.HTTP_503_SERVICE_UNAVAILABLE
    default_detail = 'Service temporarily unavailable, try again later.'
    default_code = 'service_unavailable'

class InsufficientStock(APIException):
    status_code = status.HTTP_400_BAD_REQUEST
    default_detail = 'Not enough stock available.'
    default_code = 'insufficient_stock'

# In view
def create_order(self, request):
    product = Product.objects.get(id=request.data['product_id'])
    quantity = request.data['quantity']
    
    if product.stock < quantity:
        raise InsufficientStock({
            'available': product.stock,
            'requested': quantity
        })
```

## Part 5: Routers

Routers automatically generate URL patterns for ViewSets.

### Simple Router

```python
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register(r'products', ProductViewSet, basename='product')
router.register(r'categories', CategoryViewSet, basename='category')

urlpatterns = router.urls
```

### Default Router (with API Root)

```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')
router.register(r'categories', CategoryViewSet, basename='category')
router.register(r'orders', OrderViewSet, basename='order')

# URLs generated:
# GET /products/ - list
# POST /products/ - create
# GET /products/{id}/ - retrieve
# PUT /products/{id}/ - update
# PATCH /products/{id}/ - partial_update
# DELETE /products/{id}/ - destroy
# GET / - api root (DefaultRouter only)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

### Custom Actions and Routes

```python
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    # detail=True means URL includes {id}
    # URL: /products/{id}/favorite/
    @action(detail=True, methods=['post'], url_path='favorite')
    def mark_favorite(self, request, pk=None):
        product = self.get_object()
        request.user.favorites.add(product)
        return Response({'status': 'favorited'})
    
    # detail=False means no {id} in URL
    # URL: /products/featured/
    @action(detail=False, methods=['get'])
    def featured(self, request):
        featured = self.get_queryset().filter(is_featured=True)
        serializer = self.get_serializer(featured, many=True)
        return Response(serializer.data)
    
    # Custom URL path and name
    @action(
        detail=True,
        methods=['post'],
        url_path='add-review',
        url_name='add-review'
    )
    def add_review(self, request, pk=None):
        product = self.get_object()
        serializer = ReviewSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(product=product, user=request.user)
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

### Nested Routers

For nested resources, use `drf-nested-routers`:

```python
from rest_framework_nested import routers

router = routers.SimpleRouter()
router.register(r'authors', AuthorViewSet, basename='author')
router.register(r'publishers', PublisherViewSet, basename='publisher')

# Nested router for author's books
authors_router = routers.NestedSimpleRouter(router, r'authors', lookup='author')
authors_router.register(r'books', BookViewSet, basename='author-books')

# URLs generated:
# /authors/ - list authors
# /authors/{author_pk}/ - author detail
# /authors/{author_pk}/books/ - author's books list
# /authors/{author_pk}/books/{id}/ - specific book

urlpatterns = [
    path('api/', include(router.urls)),
    path('api/', include(authors_router.urls)),
]
```

### Custom Router Configuration

```python
from rest_framework.routers import Route, DynamicRoute, SimpleRouter

class CustomRouter(SimpleRouter):
    routes = [
        # List route
        Route(
            url=r'^{prefix}{trailing_slash}$',
            mapping={'get': 'list', 'post': 'create'},
            name='{basename}-list',
            detail=False,
            initkwargs={'suffix': 'List'}
        ),
        # Detail route
        Route(
            url=r'^{prefix}/{lookup}{trailing_slash}$',
            mapping={
                'get': 'retrieve',
                'put': 'update',
                'patch': 'partial_update',
                'delete': 'destroy'
            },
            name='{basename}-detail',
            detail=True,
            initkwargs={'suffix': 'Instance'}
        ),
        # Dynamic routes for custom actions
        DynamicRoute(
            url=r'^{prefix}/{lookup}/{url_path}{trailing_slash}$',
            name='{basename}-{url_name}',
            detail=True,
            initkwargs={}
        ),
    ]
```

### Multiple API Versions

```python
from rest_framework.routers import DefaultRouter

# Version 1
router_v1 = DefaultRouter()
router_v1.register(r'products', ProductV1ViewSet, basename='product')

# Version 2
router_v2 = DefaultRouter()
router_v2.register(r'products', ProductV2ViewSet, basename='product')

urlpatterns = [
    path('api/v1/', include(router_v1.urls)),
    path('api/v2/', include(router_v2.urls)),
]
```

## Part 6: Authentication and Permissions

### Authentication Classes

#### Token Authentication

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework.authtoken',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ]
}

# Run migration
# python manage.py migrate

# Create tokens for users
from rest_framework.authtoken.models import Token

token = Token.objects.create(user=user)
print(token.key)
```

#### Obtain Token View

```python
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('api/token/', obtain_auth_token, name='api_token_auth'),
]

# Custom token view
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.authtoken.models import Token
from rest_framework.response import Response

class CustomAuthToken(ObtainAuthToken):
    def post(self, request, *args, **kwargs):
        serializer = self.serializer_class(
            data=request.data,
            context={'request': request}
        )
        serializer.is_valid(raise_exception=True)
        user = serializer.validated_data['user']
        token, created = Token.objects.get_or_create(user=user)
        return Response({
            'token': token.key,
            'user_id': user.pk,
            'email': user.email,
            'username': user.username
        })
```

#### JWT Authentication (using djangorestframework-simplejwt)

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework_simplejwt',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}

from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'ROTATE_REFRESH_TOKENS': False,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    'AUTH_HEADER_TYPES': ('Bearer',),
}

# urls.py
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

#### Custom JWT Claims

```python
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
from rest_framework_simplejwt.views import TokenObtainPairView

class CustomTokenObtainPairSerializer(TokenObtainPairSerializer):
    @classmethod
    def get_token(cls, user):
        token = super().get_token(user)
        
        # Add custom claims
        token['username'] = user.username
        token['email'] = user.email
        token['is_staff'] = user.is_staff
        
        return token

class CustomTokenObtainPairView(TokenObtainPairView):
    serializer_class = CustomTokenObtainPairSerializer
```

#### Session Authentication

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ]
}
```

#### Multiple Authentication Methods

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

### Permission Classes

#### Built-in Permissions

```python
from rest_framework.permissions import (
    IsAuthenticated,
    IsAuthenticatedOrReadOnly,
    AllowAny,
    IsAdminUser,
    DjangoModelPermissions,
    DjangoObjectPermissions
)

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def get_permissions(self):
        """Different permissions for different actions."""
        if self.action in ['list', 'retrieve']:
            permission_classes = [AllowAny]
        elif self.action == 'create':
            permission_classes = [IsAuthenticated]
        else:
            permission_classes = [IsAdminUser]
        return [permission() for permission in permission_classes]
```

#### Custom Permissions

```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """
    message = 'You must be the owner to perform this action.'
    
    def has_object_permission(self, request, view, obj):
        # Read permissions for any request
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Write permissions only for owner
        return obj.owner == request.user

class IsAuthorOrReadOnly(permissions.BasePermission):
    """
    Custom permission for blog posts.
    """
    def has_permission(self, request, view):
        # Allow authenticated users to create
        if request.method == 'POST':
            return request.user and request.user.is_authenticated
        return True
    
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user

class IsPremiumUser(permissions.BasePermission):
    """Check if user has premium subscription."""
    
    def has_permission(self, request, view):
        return (
            request.user and
            request.user.is_authenticated and
            hasattr(request.user, 'subscription') and
            request.user.subscription.is_active
        )
```

#### Combining Permissions

```python
from rest_framework.permissions import BasePermission, SAFE_METHODS

class IsOwnerOrAdmin(BasePermission):
    """
    Permission that allows owners or admins to edit.
    """
    def has_object_permission(self, request, view, obj):
        # Admin can do anything
        if request.user and request.user.is_staff:
            return True
        
        # Owner can edit
        if hasattr(obj, 'owner'):
            return obj.owner == request.user
        
        return False

class CanPublishPost(BasePermission):
    """Complex business logic permission."""
    
    def has_permission(self, request, view):
        # Must be authenticated
        if not request.user or not request.user.is_authenticated:
            return False
        
        # Check if user can publish based on their role
        if request.method == 'POST' and view.action == 'publish':
            return request.user.has_perm('blog.can_publish')
        
        return True
    
    def has_object_permission(self, request, view, obj):
        if view.action == 'publish':
            # Only author or editor can publish
            return (
                obj.author == request.user or
                request.user.groups.filter(name='Editors').exists()
            )
        return True

# Using multiple permissions
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrAdmin, CanPublishPost]
```

#### Object-Level Permissions with Django Guardian

```python
# Install: pip install django-guardian

# settings.py
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'guardian.backends.ObjectPermissionBackend',
]

# In view
from guardian.shortcuts import assign_perm, get_objects_for_user

class DocumentViewSet(viewsets.ModelViewSet):
    serializer_class = DocumentSerializer
    permission_classes = [IsAuthenticated]
    
    def get_queryset(self):
        """Return only documents user has permission to view."""
        return get_objects_for_user(
            self.request.user,
            'documents.view_document',
            klass=Document
        )
    
    def perform_create(self, serializer):
        document = serializer.save(owner=self.request.user)
        # Assign permissions to creator
        assign_perm('view_document', self.request.user, document)
        assign_perm('change_document', self.request.user, document)
        assign_perm('delete_document', self.request.user, document)
```

## Part 7: Advanced Patterns

### Versioning

#### URL Path Versioning

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
    'VERSION_PARAM': 'version',
}

# urls.py
urlpatterns = [
    path('api/<str:version>/products/', ProductListView.as_view()),
]

# In view
class ProductListView(APIView):
    def get(self, request, version):
        if request.version == 'v1':
            serializer_class = ProductSerializerV1
        else:
            serializer_class = ProductSerializerV2
        
        products = Product.objects.all()
        serializer = serializer_class(products, many=True)
        return Response(serializer.data)
```

#### Accept Header Versioning

```python
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.AcceptHeaderVersioning',
}

# Client sends: Accept: application/json; version=1.0
```

#### Using Version in ViewSets

```python
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    
    def get_serializer_class(self):
        if self.request.version == 'v1':
            return ProductSerializerV1
        return ProductSerializerV2
    
    def get_queryset(self):
        queryset = super().get_queryset()
        
        if self.request.version == 'v2':
            # v2 includes more related data
            queryset = queryset.prefetch_related('reviews')
        
        return queryset
```

### Caching

#### View-Level Caching

```python
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page
from django.views.decorators.vary import vary_on_cookie

class ProductListView(APIView):
    @method_decorator(cache_page(60 * 15))  # Cache for 15 minutes
    def get(self, request):
        products = Product.objects.all()
        serializer = ProductSerializer(products, many=True)
        return Response(serializer.data)

# With ViewSet
class ProductViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    @method_decorator(cache_page(60 * 15))
    @method_decorator(vary_on_cookie)
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```
