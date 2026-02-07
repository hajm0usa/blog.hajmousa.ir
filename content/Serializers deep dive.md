+++
title="The Professional Django & Django REST Framework Handbook"
date=2026-01-28
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

#### Conditional Requests (ETags)

```python
from django.views.decorators.http
from django.views.decorators.http import etag, condition
from django.utils.decorators import method_decorator

def product_etag(request, pk):
    """Generate ETag based on product's updated_at."""
    try:
        product = Product.objects.get(pk=pk)
        return f'"{product.updated_at.timestamp()}"'
    except Product.DoesNotExist:
        return None

def product_last_modified(request, pk):
    """Return last modified time."""
    try:
        product = Product.objects.get(pk=pk)
        return product.updated_at
    except Product.DoesNotExist:
        return None

class ProductDetailView(APIView):
    @method_decorator(condition(etag_func=product_etag, last_modified_func=product_last_modified))
    def get(self, request, pk):
        product = get_object_or_404(Product, pk=pk)
        serializer = ProductSerializer(product)
        return Response(serializer.data)
```

#### Redis Caching

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# In view
from django.core.cache import cache
from django.core.cache.utils import make_template_fragment_key

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def retrieve(self, request, *args, **kwargs):
        product_id = kwargs.get('pk')
        cache_key = f'product_detail_{product_id}'
        
        # Try to get from cache
        cached_data = cache.get(cache_key)
        if cached_data:
            return Response(cached_data)
        
        # If not in cache, get from database
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        
        # Store in cache for 1 hour
        cache.set(cache_key, serializer.data, 60 * 60)
        
        return Response(serializer.data)
    
    def perform_update(self, serializer):
        instance = serializer.save()
        # Invalidate cache on update
        cache_key = f'product_detail_{instance.id}'
        cache.delete(cache_key)
    
    def perform_destroy(self, instance):
        # Invalidate cache on delete
        cache_key = f'product_detail_{instance.id}'
        cache.delete(cache_key)
        instance.delete()
```

#### Custom Cache Decorator

```python
from functools import wraps
from django.core.cache import cache

def cache_response(timeout=60 * 15, key_prefix='view'):
    """Cache API response."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(self, request, *args, **kwargs):
            # Build cache key
            cache_key = f"{key_prefix}:{request.path}:{request.GET.urlencode()}"
            
            # Check cache
            cached_response = cache.get(cache_key)
            if cached_response:
                return cached_response
            
            # Get response
            response = view_func(self, request, *args, **kwargs)
            
            # Cache successful responses
            if response.status_code == 200:
                cache.set(cache_key, response, timeout)
            
            return response
        return wrapper
    return decorator

class ProductViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    @cache_response(timeout=60 * 30, key_prefix='products_list')
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```

### Batch Operations

```python
class ProductBatchViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    @action(detail=False, methods=['post'])
    def batch_create(self, request):
        """Create multiple products at once."""
        serializer = self.get_serializer(data=request.data, many=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    
    @action(detail=False, methods=['patch'])
    def batch_update(self, request):
        """Update multiple products."""
        products_data = request.data
        updated_products = []
        
        for product_data in products_data:
            product_id = product_data.get('id')
            if not product_id:
                continue
            
            try:
                product = Product.objects.get(id=product_id)
                serializer = self.get_serializer(
                    product,
                    data=product_data,
                    partial=True
                )
                serializer.is_valid(raise_exception=True)
                serializer.save()
                updated_products.append(serializer.data)
            except Product.DoesNotExist:
                pass
        
        return Response(updated_products)
    
    @action(detail=False, methods=['delete'])
    def batch_delete(self, request):
        """Delete multiple products."""
        ids = request.data.get('ids', [])
        if not ids:
            return Response(
                {'error': 'No IDs provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        deleted_count, _ = Product.objects.filter(id__in=ids).delete()
        return Response({
            'deleted_count': deleted_count
        })
```

### File Upload Handling

```python
from rest_framework.parsers import MultiPartParser, FormParser, JSONParser

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    parser_classes = [MultiPartParser, FormParser, JSONParser]
    
    @action(detail=True, methods=['post'])
    def upload_image(self, request, pk=None):
        """Upload product image."""
        product = self.get_object()
        
        if 'image' not in request.FILES:
            return Response(
                {'error': 'No image provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        product.image = request.FILES['image']
        product.save()
        
        serializer = self.get_serializer(product)
        return Response(serializer.data)
    
    @action(detail=False, methods=['post'])
    def bulk_upload(self, request):
        """Upload multiple images."""
        files = request.FILES.getlist('images')
        product_id = request.data.get('product_id')
        
        if not files:
            return Response(
                {'error': 'No files provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        product = get_object_or_404(Product, id=product_id)
        
        uploaded_images = []
        for file in files:
            image = ProductImage.objects.create(
                product=product,
                image=file
            )
            uploaded_images.append({
                'id': image.id,
                'url': image.image.url
            })
        
        return Response({'images': uploaded_images})
```

#### Custom File Validators

```python
from django.core.exceptions import ValidationError

def validate_image_size(image):
    """Validate image file size (max 5MB)."""
    max_size = 5 * 1024 * 1024  # 5MB
    if image.size > max_size:
        raise ValidationError('Image size cannot exceed 5MB')

def validate_image_dimensions(image):
    """Validate image dimensions."""
    from PIL import Image
    img = Image.open(image)
    width, height = img.size
    
    if width < 200 or height < 200:
        raise ValidationError('Image must be at least 200x200 pixels')
    
    if width > 4000 or height > 4000:
        raise ValidationError('Image dimensions too large')

class Product(models.Model):
    image = models.ImageField(
        upload_to='products/%Y/%m/%d/',
        validators=[validate_image_size, validate_image_dimensions]
    )
```

#### File Upload with Progress

```python
from rest_framework.response import Response
from rest_framework.decorators import action
from django.core.files.uploadedfile import UploadedFile

class DocumentViewSet(viewsets.ModelViewSet):
    queryset = Document.objects.all()
    serializer_class = DocumentSerializer
    
    @action(detail=False, methods=['post'])
    def chunked_upload(self, request):
        """Handle large file uploads in chunks."""
        chunk = request.FILES.get('chunk')
        chunk_number = int(request.data.get('chunk_number', 0))
        total_chunks = int(request.data.get('total_chunks', 1))
        filename = request.data.get('filename')
        
        # Store chunks temporarily
        cache_key = f'upload_{request.user.id}_{filename}'
        
        if chunk_number == 0:
            # First chunk, initialize
            cache.set(cache_key, {'chunks': []}, timeout=3600)
        
        # Get stored data
        upload_data = cache.get(cache_key)
        upload_data['chunks'].append(chunk.read())
        cache.set(cache_key, upload_data, timeout=3600)
        
        # If all chunks received, combine and save
        if chunk_number == total_chunks - 1:
            full_file = b''.join(upload_data['chunks'])
            
            document = Document.objects.create(
                title=filename,
                file=ContentFile(full_file, name=filename),
                uploaded_by=request.user
            )
            
            # Clean up cache
            cache.delete(cache_key)
            
            serializer = self.get_serializer(document)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        
        return Response({
            'status': 'chunk_received',
            'chunk_number': chunk_number
        })
```

### Async Views (Django 4.1+)

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from asgiref.sync import sync_to_async
import asyncio

class AsyncProductView(APIView):
    async def get(self, request):
        """Async view for better performance."""
        # Fetch data asynchronously
        products = await sync_to_async(list)(
            Product.objects.all()
        )
        
        # Serialize asynchronously
        serializer = ProductSerializer(products, many=True)
        
        return Response(serializer.data)

class AsyncProductDetailView(APIView):
    async def get(self, request, pk):
        """Fetch product with related data asynchronously."""
        # Parallel async queries
        product_task = sync_to_async(Product.objects.get)(pk=pk)
        reviews_task = sync_to_async(list)(
            Review.objects.filter(product_id=pk)
        )
        
        product, reviews = await asyncio.gather(
            product_task,
            reviews_task
        )
        
        serializer = ProductDetailSerializer(product)
        return Response(serializer.data)
```

### Rate Limiting with Custom Logic

```python
from rest_framework.throttling import BaseThrottle
from django.core.cache import cache

class CustomRateThrottle(BaseThrottle):
    """
    Custom throttle with different rates based on user tier.
    """
    def allow_request(self, request, view):
        if request.user.is_authenticated:
            # Different rates for different user tiers
            if hasattr(request.user, 'subscription'):
                if request.user.subscription.tier == 'premium':
                    rate = 10000  # per day
                elif request.user.subscription.tier == 'basic':
                    rate = 1000
                else:
                    rate = 100
            else:
                rate = 100
        else:
            rate = 50  # Anonymous users
        
        # Check cache for request count
        cache_key = f'throttle_{self.get_ident(request)}'
        request_count = cache.get(cache_key, 0)
        
        if request_count >= rate:
            return False
        
        # Increment counter
        cache.set(cache_key, request_count + 1, 86400)  # 24 hours
        return True
    
    def get_ident(self, request):
        """Get identifier for the request."""
        if request.user.is_authenticated:
            return f'user_{request.user.id}'
        return self.get_ident_from_request(request)
    
    def wait(self):
        """Return time to wait before next request."""
        return 86400  # 24 hours

class EndpointSpecificThrottle(BaseThrottle):
    """Different rates for different endpoints."""
    
    RATES = {
        'expensive_operation': 10,  # per hour
        'normal_operation': 100,
        'read_only': 1000,
    }
    
    def allow_request(self, request, view):
        # Determine endpoint type
        endpoint_type = getattr(view, 'throttle_scope', 'normal_operation')
        rate = self.RATES.get(endpoint_type, 100)
        
        cache_key = f'throttle_{endpoint_type}_{request.user.id}'
        request_count = cache.get(cache_key, 0)
        
        if request_count >= rate:
            return False
        
        cache.set(cache_key, request_count + 1, 3600)  # 1 hour
        return True

# In ViewSet
class DataExportViewSet(viewsets.ViewSet):
    throttle_classes = [EndpointSpecificThrottle]
    throttle_scope = 'expensive_operation'
    
    def list(self, request):
        # Export operation limited to 10/hour
        pass
```

### Soft Deletes with Filtering

```python
class SoftDeleteModelViewSet(viewsets.ModelViewSet):
    """ViewSet that implements soft delete."""
    
    def get_queryset(self):
        """Exclude soft-deleted objects by default."""
        queryset = super().get_queryset()
        
        # Admin can see deleted items with ?include_deleted=true
        if self.request.user.is_staff:
            include_deleted = self.request.query_params.get('include_deleted', 'false')
            if include_deleted.lower() == 'true':
                return queryset
        
        return queryset.filter(is_deleted=False)
    
    def perform_destroy(self, instance):
        """Soft delete instead of hard delete."""
        instance.is_deleted = True
        instance.deleted_at = timezone.now()
        instance.deleted_by = self.request.user
        instance.save()
    
    @action(detail=True, methods=['post'], permission_classes=[IsAdminUser])
    def restore(self, request, pk=None):
        """Restore soft-deleted item."""
        instance = self.get_object()
        instance.is_deleted = False
        instance.deleted_at = None
        instance.deleted_by = None
        instance.save()
        
        serializer = self.get_serializer(instance)
        return Response(serializer.data)
    
    @action(detail=True, methods=['delete'], permission_classes=[IsAdminUser])
    def hard_delete(self, request, pk=None):
        """Permanently delete item."""
        instance = self.get_object()
        instance.delete()  # This calls the model's hard_delete if exists
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Audit Logging

```python
from django.contrib.contenttypes.models import ContentType
import json

class AuditLog(models.Model):
    """Model to track all changes."""
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    action = models.CharField(max_length=10)  # CREATE, UPDATE, DELETE
    changes = models.JSONField(default=dict)
    timestamp = models.DateTimeField(auto_now_add=True)
    ip_address = models.GenericIPAddressField(null=True)
    
    class Meta:
        ordering = ['-timestamp']
        indexes = [
            models.Index(fields=['content_type', 'object_id']),
            models.Index(fields=['user', 'timestamp']),
        ]

class AuditMixin:
    """Mixin to add audit logging to ViewSets."""
    
    def perform_create(self, serializer):
        instance = serializer.save()
        self._log_action('CREATE', instance, serializer.validated_data)
    
    def perform_update(self, serializer):
        # Get old data
        old_instance = self.get_object()
        old_data = self.get_serializer(old_instance).data
        
        # Save new data
        instance = serializer.save()
        new_data = self.get_serializer(instance).data
        
        # Calculate changes
        changes = self._get_changes(old_data, new_data)
        self._log_action('UPDATE', instance, changes)
    
    def perform_destroy(self, instance):
        data = self.get_serializer(instance).data
        instance.delete()
        self._log_action('DELETE', instance, data)
    
    def _log_action(self, action, instance, data):
        """Create audit log entry."""
        AuditLog.objects.create(
            user=self.request.user if self.request.user.is_authenticated else None,
            content_type=ContentType.objects.get_for_model(instance),
            object_id=instance.pk,
            action=action,
            changes=data,
            ip_address=self._get_client_ip()
        )
    
    def _get_changes(self, old_data, new_data):
        """Get dict of changed fields."""
        changes = {}
        for key in new_data:
            if key in old_data and old_data[key] != new_data[key]:
                changes[key] = {
                    'old': old_data[key],
                    'new': new_data[key]
                }
        return changes
    
    def _get_client_ip(self):
        """Extract client IP from request."""
        x_forwarded_for = self.request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = self.request.META.get('REMOTE_ADDR')
        return ip

class ProductViewSet(AuditMixin, viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

## Part 8: Testing

### Testing Models

```python
from django.test import TestCase
from django.core.exceptions import ValidationError
from decimal import Decimal

class ProductModelTest(TestCase):
    def setUp(self):
        """Run before each test method."""
        self.category = Category.objects.create(name='Electronics')
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
    
    def test_product_creation(self):
        """Test creating a product."""
        product = Product.objects.create(
            name='Test Product',
            price=Decimal('99.99'),
            cost=Decimal('50.00'),
            stock=10,
            category=self.category
        )
        
        self.assertEqual(product.name, 'Test Product')
        self.assertEqual(product.price, Decimal('99.99'))
        self.assertTrue(product.is_in_stock)
    
    def test_product_str_method(self):
        """Test string representation."""
        product = Product.objects.create(
            name='Test Product',
            price=Decimal('99.99'),
            category=self.category
        )
        self.assertEqual(str(product), 'Test Product')
    
    def test_product_validation(self):
        """Test model validation."""
        product = Product(
            name='Invalid Product',
            price=Decimal('10.00'),
            cost=Decimal('50.00'),  # Cost higher than price
            category=self.category
        )
        
        with self.assertRaises(ValidationError):
            product.full_clean()
    
    def test_soft_delete(self):
        """Test soft delete functionality."""
        product = Product.objects.create(
            name='Test Product',
            price=Decimal('99.99'),
            category=self.category
        )
        
        product.delete()  # Soft delete
        
        self.assertTrue(product.is_deleted)
        self.assertIsNotNone(product.deleted_at)
        
        # Product still exists in database
        self.assertTrue(Product.objects.filter(pk=product.pk).exists())
        
        # But not in active manager
        self.assertFalse(Product.active.filter(pk=product.pk).exists())
    
    def test_manager_methods(self):
        """Test custom manager methods."""
        # Create products
        Product.objects.create(name='Product 1', price=Decimal('100'), category=self.category)
        Product.objects.create(name='Product 2', price=Decimal('200'), category=self.category, is_deleted=True)
        
        # Test active manager
        self.assertEqual(Product.active.count(), 1)
        self.assertEqual(Product.objects.count(), 2)
```

### Testing Serializers

```python
from rest_framework.test import APITestCase
from django.urls import reverse

class ProductSerializerTest(APITestCase):
    def setUp(self):
        self.category = Category.objects.create(name='Electronics')
        self.product_data = {
            'name': 'Test Product',
            'price': '99.99',
            'cost': '50.00',
            'stock': 10,
            'category_id': self.category.id
        }
    
    def test_serializer_with_valid_data(self):
        """Test serializer with valid data."""
        serializer = ProductSerializer(data=self.product_data)
        self.assertTrue(serializer.is_valid())
        product = serializer.save()
        
        self.assertEqual(product.name, 'Test Product')
        self.assertEqual(product.price, Decimal('99.99'))
    
    def test_serializer_with_invalid_data(self):
        """Test serializer with invalid data."""
        invalid_data = self.product_data.copy()
        invalid_data['price'] = '-10.00'  # Negative price
        
        serializer = ProductSerializer(data=invalid_data)
        self.assertFalse(serializer.is_valid())
        self.assertIn('price', serializer.errors)
    
    def test_serializer_read_only_fields(self):
        """Test read-only fields are not writable."""
        product = Product.objects.create(**self.product_data)
        
        update_data = {'id': 999, 'name': 'Updated Name'}
        serializer = ProductSerializer(product, data=update_data, partial=True)
        
        self.assertTrue(serializer.is_valid())
        updated_product = serializer.save()
        
        self.assertNotEqual(updated_product.id, 999)  # ID didn't change
        self.assertEqual(updated_product.name, 'Updated Name')
    
    def test_nested_serializer(self):
        """Test nested serializer representation."""
        product = Product.objects.create(**self.product_data)
        serializer = ProductDetailSerializer(product)
        
        self.assertIn('category', serializer.data)
        self.assertEqual(serializer.data['category']['name'], 'Electronics')
    
    def test_serializer_method_field(self):
        """Test SerializerMethodField."""
        product = Product.objects.create(**self.product_data)
        
        # Create some reviews
        Review.objects.create(product=product, rating=5, comment='Great')
        Review.objects.create(product=product, rating=4, comment='Good')
        
        serializer = ProductDetailSerializer(product)
        self.assertIn('average_rating', serializer.data)
        self.assertEqual(serializer.data['average_rating'], 4.5)
```

### Testing Views and ViewSets

```python
from rest_framework.test import APIClient, APITestCase
from rest_framework import status
from django.urls import reverse

class ProductViewSetTest(APITestCase):
    def setUp(self):
        """Set up test client and data."""
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.admin = User.objects.create_superuser(
            username='admin',
            password='adminpass123'
        )
        
        self.category = Category.objects.create(name='Electronics')
        self.product = Product.objects.create(
            name='Test Product',
            price=Decimal('99.99'),
            cost=Decimal('50.00'),
            stock=10,
            category=self.category
        )
        
        self.list_url = reverse('product-list')
        self.detail_url = reverse('product-detail', kwargs={'pk': self.product.pk})
    
    def test_list_products_unauthenticated(self):
        """Test listing products without authentication."""
        response = self.client.get(self.list_url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_list_products_authenticated(self):
        """Test listing products with authentication."""
        self.client.force_authenticate(user=self.user)
        response = self.client.get(self.list_url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
    
    def test_create_product_unauthorized(self):
        """Test creating product without authentication."""
        data = {
            'name': 'New Product',
            'price': '149.99',
            'cost': '75.00',
            'stock': 5,
            'category_id': self.category.id
        }
        
        response = self.client.post(self.list_url, data)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_create_product_authorized(self):
        """Test creating product with authentication."""
        self.client.force_authenticate(user=self.admin)
        
        data = {
            'name': 'New Product',
            'price': '149.99',
            'cost': '75.00',
            'stock': 5,
            'category_id': self.category.id
        }
        
        response = self.client.post(self.list_url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Product.objects.count(), 2)
        self.assertEqual(response.data['name'], 'New Product')
    
    def test_retrieve_product(self):
        """Test retrieving a single product."""
        response = self.client.get(self.detail_url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['name'], 'Test Product')
    
    def test_update_product(self):
        """Test updating a product."""
        self.client.force_authenticate(user=self.admin)
        
        data = {'name': 'Updated Product', 'price': '109.99'}
        response = self.client.patch(self.detail_url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.product.refresh_from_db()
        self.assertEqual(self.product.name, 'Updated Product')
        self.assertEqual(self.product.price, Decimal('109.99'))
    
    def test_delete_product(self):
        """Test deleting a product."""
        self.client.force_authenticate(user=self.admin)
        
        response = self.client.delete(self.detail_url)
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertFalse(Product.active.filter(pk=self.product.pk).exists())
    
    def test_filter_products(self):
        """Test filtering products."""
        # Create products in different categories
        cat2 = Category.objects.create(name='Books')
        Product.objects.create(
            name='Book Product',
            price=Decimal('19.99'),
            cost=Decimal('10.00'),
            category=cat2
        )
        
        # Filter by category
        response = self.client.get(self.list_url, {'category': self.category.id})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
        self.assertEqual(response.data['results'][0]['category'], self.category.id)
    
    def test_search_products(self):
        """Test searching products."""
        Product.objects.create(
            name='Special Product',
            price=Decimal('29.99'),
            cost=Decimal('15.00'),
            category=self.category
        )
        
        response = self.client.get(self.list_url, {'search': 'Special'})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
        self.assertIn('Special', response.data['results'][0]['name'])
    
    def test_ordering_products(self):
        """Test ordering products."""
        Product.objects.create(
            name='Cheap Product',
            price=Decimal('9.99'),
            cost=Decimal('5.00'),
            category=self.category
        )
        
        # Order by price ascending
        response = self.client.get(self.list_url, {'ordering': 'price'})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        prices = [Decimal(p['price']) for p in response.data['results']]
        self.assertEqual(prices, sorted(prices))
    
    def test_custom_action(self):
        """Test custom action endpoint."""
        self.client.force_authenticate(user=self.user)
        
        url = reverse('product-favorite', kwargs={'pk': self.product.pk})
        response = self.client.post(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(self.user.favorites.filter(pk=self.product.pk).exists())
    
    def test_pagination(self):
        """Test pagination."""
        # Create many products
        for i in range(25):
            Product.objects.create(
                name=f'Product {i}',
                price=Decimal('10.00'),
                cost=Decimal('5.00'),
                category=self.category
            )
        
        response = self.client.get(self.list_url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('results', response.data)
        self.assertIn('count', response.data)
        self.assertIn('next', response.data)
        self.assertLessEqual(len(response.data['results']), 20)

    def test_throttling(self):
        """Test rate limiting."""
        # Make many requests quickly
        for i in range(100):
            response = self.client.get(self.list_url)
            if response.status_code == status.HTTP_429_TOO_MANY_REQUESTS:
                break
        
        # Should eventually get throttled
        self.assertEqual(response.status_code, status.HTTP_429_TOO_MANY_REQUESTS)
    
    def test_permissions(self):
        """Test permission classes."""
        # Regular user shouldn't be able to delete
        self.client.force_authenticate(user=self.user)
        response = self.client.delete(self.detail_url)
        
        self.assertIn(
            response.status_code,
            [status.HTTP_403_FORBIDDEN, status.HTTP_401_UNAUTHORIZED]
        )
        
        # Admin should be able to delete
        self.client.force_authenticate(user=self.admin)
        response = self.client.delete(self.detail_url)
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
```

### Testing Authentication

```python
class AuthenticationTest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.token_url = reverse('token_obtain_pair')
        self.refresh_url = reverse('token_refresh')
    
    def test_obtain_token(self):
        """Test obtaining JWT tokens."""
        data = {
            'username': 'testuser',
            'password': 'testpass123'
        }
        
        response = self.client.post(self.token_url, data)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('access', response.data)
        self.assertIn('refresh', response.data)
    
    def test_invalid_credentials(self):
        """Test login with invalid credentials."""
        data = {
            'username': 'testuser',
            'password': 'wrongpassword'
        }
        
        response = self.client.post(self.token_url, data)
        
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_refresh_token(self):
        """Test refreshing access token."""
        # Get tokens
        data = {'username': 'testuser', 'password': 'testpass123'}
        response = self.client.post(self.token_url, data)
        refresh_token = response.data['refresh']
        
        # Refresh
        refresh_data = {'refresh': refresh_token}
        response = self.client.post(self.refresh_url, refresh_data)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('access', response.data)
    
    def test_authenticated_request(self):
        """Test making request with JWT token."""
        # Get token
        data = {'username': 'testuser', 'password': 'testpass123'}
        response = self.client.post(self.token_url, data)
        token = response.data['access']
        
        # Make authenticated request
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {token}')
        response = self.client.get(reverse('product-list'))
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
```

### Testing with Fixtures

```python
class ProductWithFixturesTest(APITestCase):
    fixtures = ['categories.json', 'products.json', 'users.json']
    
    def test_with_fixture_data(self):
        """Test using fixture data."""
        # Data loaded from fixtures
        self.assertEqual(Product.objects.count(), 10)
        self.assertEqual(Category.objects.count(), 5)
        
        response = self.client.get(reverse('product-list'))
        self.assertEqual(response.status_code, status.HTTP_200_OK)
```

### Testing with Factory Boy

```python
# factories.py
import factory
from factory.django import DjangoModelFactory

class UserFactory(DjangoModelFactory):
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f'user{n}')
    email = factory.LazyAttribute(lambda obj: f'{obj.username}@example.com')
    password = factory.PostGenerationMethodCall('set_password', 'testpass123')

class CategoryFactory(DjangoModelFactory):
    class Meta:
        model = Category
    
    name = factory.Sequence(lambda n: f'Category {n}')
    slug = factory.LazyAttribute(lambda obj: obj.name.lower().replace(' ', '-'))

class ProductFactory(DjangoModelFactory):
    class Meta:
        model = Product
    
    name = factory.Sequence(lambda n: f'Product {n}')
    price = factory.Faker('pydecimal', left_digits=3, right_digits=2, positive=True)
    cost = factory.LazyAttribute(lambda obj: obj.price * Decimal('0.6'))
    stock = factory.Faker('pyint', min_value=0, max_value=100)
    category = factory.SubFactory(CategoryFactory)

class ReviewFactory(DjangoModelFactory):
    class Meta:
        model = Review
    
    product = factory.SubFactory(ProductFactory)
    user = factory.SubFactory(UserFactory)
    rating = factory.Faker('pyint', min_value=1, max_value=5)
    comment = factory.Faker('paragraph')

# test_with_factories.py
class ProductFactoryTest(APITestCase):
    def test_with_factories(self):
        """Test using factories."""
        # Create test data easily
        user = UserFactory()
        category = CategoryFactory(name='Electronics')
        product = ProductFactory(category=category, price=Decimal('99.99'))
        
        self.assertEqual(product.category.name, 'Electronics')
        self.assertEqual(product.price, Decimal('99.99'))
    
    def test_batch_create(self):
        """Test creating multiple objects."""
        products = ProductFactory.create_batch(10)
        self.assertEqual(len(products), 10)
        self.assertEqual(Product.objects.count(), 10)
    
    def test_with_traits(self):
        """Test using factory traits."""
        # Create product with specific state
        expensive_product = ProductFactory(price=Decimal('999.99'))
        cheap_product = ProductFactory(price=Decimal('9.99'))
        
        self.assertGreater(expensive_product.price, cheap_product.price)
```

### Integration Testing

```python
class OrderWorkflowIntegrationTest(APITestCase):
    """Test complete order workflow."""
    
    def setUp(self):
        self.client = APIClient()
        self.user = UserFactory()
        self.client.force_authenticate(user=self.user)
        
        self.product1 = ProductFactory(price=Decimal('50.00'), stock=10)
        self.product2 = ProductFactory(price=Decimal('30.00'), stock=5)
    
    def test_complete_order_workflow(self):
        """Test entire order process from creation to completion."""
        # Step 1: Create order
        order_data = {
            'items': [
                {'product_id': self.product1.id, 'quantity': 2},
                {'product_id': self.product2.id, 'quantity': 1}
            ]
        }
        
        response = self.client.post(reverse('order-list'), order_data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        order_id = response.data['id']
        
        # Step 2: Verify stock was reduced
        self.product1.refresh_from_db()
        self.product2.refresh_from_db()
        self.assertEqual(self.product1.stock, 8)
        self.assertEqual(self.product2.stock, 4)
        
        # Step 3: Process payment
        payment_url = reverse('order-process-payment', kwargs={'pk': order_id})
        payment_data = {
            'payment_method': 'credit_card',
            'amount': '130.00'
        }
        
        response = self.client.post(payment_url, payment_data, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Step 4: Verify order status changed
        order = Order.objects.get(id=order_id)
        self.assertEqual(order.status, 'paid')
        
        # Step 5: Ship order
        ship_url = reverse('order-ship', kwargs={'pk': order_id})
        ship_data = {'tracking_number': 'TRACK123'}
        
        response = self.client.post(ship_url, ship_data, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Step 6: Complete order
        complete_url = reverse('order-complete', kwargs={'pk': order_id})
        response = self.client.post(complete_url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Final verification
        order.refresh_from_db()
        self.assertEqual(order.status, 'completed')
        self.assertIsNotNone(order.completed_at)
    
    def test_order_cancellation_workflow(self):
        """Test order cancellation restores stock."""
        # Create order
        order_data = {
            'items': [
                {'product_id': self.product1.id, 'quantity': 3}
            ]
        }
        
        response = self.client.post(reverse('order-list'), order_data, format='json')
        order_id = response.data['id']
        
        original_stock = self.product1.stock
        self.product1.refresh_from_db()
        reduced_stock = self.product1.stock
        
        # Cancel order
        cancel_url = reverse('order-cancel', kwargs={'pk': order_id})
        response = self.client.post(cancel_url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Verify stock restored
        self.product1.refresh_from_db()
        self.assertEqual(self.product1.stock, original_stock)
        
        # Verify order status
        order = Order.objects.get(id=order_id)
        self.assertEqual(order.status, 'cancelled')
```

### Performance Testing

```python
from django.test.utils import override_settings
from django.db import connection
from django.test.utils import CaptureQueriesContext

class PerformanceTest(APITestCase):
    def setUp(self):
        # Create test data
        self.categories = CategoryFactory.create_batch(5)
        self.products = []
        for category in self.categories:
            self.products.extend(
                ProductFactory.create_batch(20, category=category)
            )
    
    def test_query_count(self):
        """Test that list view doesn't have N+1 query problem."""
        with CaptureQueriesContext(connection) as queries:
            response = self.client.get(reverse('product-list'))
            
            self.assertEqual(response.status_code, status.HTTP_200_OK)
            # Should be a small, fixed number of queries
            self.assertLess(len(queries), 5)
    
    def test_select_related_optimization(self):
        """Test that select_related reduces queries."""
        # Without optimization
        with CaptureQueriesContext(connection) as queries_before:
            products = Product.objects.all()
            for product in products[:10]:
                _ = product.category.name
            query_count_before = len(queries_before)
        
        # With optimization
        with CaptureQueriesContext(connection) as queries_after:
            products = Product.objects.select_related('category').all()
            for product in products[:10]:
                _ = product.category.name
            query_count_after = len(queries_after)
        
        self.assertLess(query_count_after, query_count_before)
    
    def test_prefetch_related_optimization(self):
        """Test prefetch_related for reverse relations."""
        # Create reviews
        for product in self.products[:10]:
            ReviewFactory.create_batch(5, product=product)
        
        # Without prefetch
        with CaptureQueriesContext(connection) as queries_before:
            products = Product.objects.all()[:10]
            for product in products:
                _ = list(product.reviews.all())
            query_count_before = len(queries_before)
        
        # With prefetch
        with CaptureQueriesContext(connection) as queries_after:
            products = Product.objects.prefetch_related('reviews').all()[:10]
            for product in products:
                _ = list(product.reviews.all())
            query_count_after = len(queries_after)
        
        self.assertLess(query_count_after, query_count_before)
    
    @override_settings(DEBUG=True)
    def test_response_time(self):
        """Test API response time."""
        import time
        
        start = time.time()
        response = self.client.get(reverse('product-list'))
        duration = time.time() - start
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertLess(duration, 1.0)  # Should respond in under 1 second
```

### Mock Testing

```python
from unittest.mock import patch, Mock

class ExternalServiceTest(APITestCase):
    """Test views that interact with external services."""
    
    @patch('myapp.services.payment_gateway.charge')
    def test_payment_processing(self, mock_charge):
        """Test payment processing with mocked gateway."""
        # Setup mock
        mock_charge.return_value = {
            'success': True,
            'transaction_id': 'TX123456'
        }
        
        # Make request
        self.client.force_authenticate(user=UserFactory())
        order = OrderFactory()
        
        url = reverse('order-process-payment', kwargs={'pk': order.id})
        data = {'payment_method': 'credit_card', 'amount': '100.00'}
        
        response = self.client.post(url, data, format='json')
        
        # Assertions
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        mock_charge.assert_called_once()
        
        # Verify call arguments
        call_args = mock_charge.call_args
        self.assertEqual(call_args[1]['amount'], Decimal('100.00'))
    
    @patch('myapp.services.email_service.send_email')
    def test_order_confirmation_email(self, mock_send_email):
        """Test order confirmation email is sent."""
        mock_send_email.return_value = True
        
        self.client.force_authenticate(user=UserFactory())
        order_data = {
            'items': [
                {'product_id': ProductFactory().id, 'quantity': 1}
            ]
        }
        
        response = self.client.post(reverse('order-list'), order_data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        mock_send_email.assert_called_once()
    
    @patch('requests.get')
    def test_external_api_call(self, mock_get):
        """Test calling external API."""
        # Mock external API response
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {
            'rate': 1.2,
            'currency': 'EUR'
        }
        mock_get.return_value = mock_response
        
        # Make request that triggers external API call
        response = self.client.get(reverse('currency-rate'), {'currency': 'EUR'})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['rate'], 1.2)
```

## Part 9: Security Best Practices

### Input Validation and Sanitization

```python
from django.core.validators import RegexValidator
from rest_framework import serializers
import bleach

class SecureProductSerializer(serializers.ModelSerializer):
    # Validate input format
    sku = serializers.CharField(
        max_length=50,
        validators=[
            RegexValidator(
                regex=r'^[A-Z0-9-]+$',
                message='SKU must contain only uppercase letters, numbers, and hyphens'
            )
        ]
    )
    
    # Sanitize HTML content
    description = serializers.CharField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'sku', 'description', 'price']
    
    def validate_description(self, value):
        """Sanitize HTML to prevent XSS."""
        allowed_tags = ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li']
        allowed_attributes = {}
        
        cleaned = bleach.clean(
            value,
            tags=allowed_tags,
            attributes=allowed_attributes,
            strip=True
        )
        
        return cleaned
    
    def validate_price(self, value):
        """Validate price is reasonable."""
        if value < 0:
            raise serializers.ValidationError("Price cannot be negative")
        if value > Decimal('1000000'):
            raise serializers.ValidationError("Price exceeds maximum allowed")
        return value
```

### SQL Injection Prevention

```python
# NEVER DO THIS - Vulnerable to SQL injection
def vulnerable_search(request):
    query = request.GET.get('q', '')
    # DON'T USE RAW SQL WITH USER INPUT
    products = Product.objects.raw(
        f"SELECT * FROM products WHERE name LIKE '%{query}%'"
    )
    return products

# CORRECT APPROACH - Use Django ORM
def safe_search(request):
    query = request.GET.get('q', '')
    # Django ORM automatically sanitizes
    products = Product.objects.filter(name__icontains=query)
    return products

# IF YOU MUST USE RAW SQL - Use parameterized queries
def safe_raw_query(request):
    query = request.GET.get('q', '')
    products = Product.objects.raw(
        "SELECT * FROM products WHERE name LIKE %s",
        [f'%{query}%']
    )
    return products
```

### CSRF Protection

```python
# settings.py
CSRF_COOKIE_SECURE = True  # Only send cookie over HTTPS
CSRF_COOKIE_HTTPONLY = True  # Prevent JavaScript access
CSRF_COOKIE_SAMESITE = 'Strict'  # Prevent CSRF attacks

# For API views using session authentication
from rest_framework.decorators import api_view
from django.views.decorators.csrf import csrf_protect

@api_view(['POST'])
@csrf_protect
def create_product(request):
    # CSRF token required for this view
    pass

# For JWT/Token auth, CSRF is not needed
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}
```

### Secure Password Handling

```python
# settings.py
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
]

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 12,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Custom password validator
from django.core.exceptions import ValidationError
from django.utils.translation import gettext as _

class CustomPasswordValidator:
    """Require special characters in passwords."""
    
    def validate(self, password, user=None):
        if not any(char in '!@#$%^&*()_+-=' for char in password):
            raise ValidationError(
                _("Password must contain at least one special character"),
                code='password_no_special',
            )
    
    def get_help_text(self):
        return _("Your password must contain at least one special character")
```

### Rate Limiting and DDoS Protection

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day',
        'login': '5/hour',  # Prevent brute force
    }
}

# Custom throttle for sensitive endpoints
from rest_framework.throttling import UserRateThrottle

class LoginRateThrottle(UserRateThrottle):
    rate = '5/hour'
    
class PasswordResetThrottle(UserRateThrottle):
    rate = '3/hour'

# In view
class LoginView(APIView):
    throttle_classes = [LoginRateThrottle]
    
    def post(self, request):
        # Login logic
        pass

# IP-based blocking for repeated failures
from django.core.cache import cache

class SecureLoginView(APIView):
    def post(self, request):
        ip = self.get_client_ip(request)
        cache_key = f'login_attempts_{ip}'
        
        attempts = cache.get(cache_key, 0)
        
        if attempts >= 5:
            return Response(
                {'error': 'Too many failed login attempts. Try again later.'},
                status=status.HTTP_429_TOO_MANY_REQUESTS
            )
        
        # Try authentication
        serializer = LoginSerializer(data=request.data)
        if serializer.is_valid():
            # Success - reset counter
            cache.delete(cache_key)
            return Response(serializer.data)
        else:
            # Failure - increment counter
            cache.set(cache_key, attempts + 1, timeout=3600)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

### Secure File Uploads

```python
from django.core.exceptions import ValidationError
from PIL import Image
import magic

def validate_file_extension(value):
    """Validate file extension."""
    import os
    ext = os.path.splitext(value.name)[1]
    valid_extensions = ['.pdf', '.doc', '.docx', '.jpg', '.png', '.jpeg']
    if not ext.lower() in valid_extensions:
        raise ValidationError('Unsupported file extension.')

def validate_file_size(value):
    """Validate file size (max 10MB)."""
    filesize = value.size
    if filesize > 10 * 1024 * 1024:
        raise ValidationError("File size exceeds 10MB limit")

def validate_image_file(value):
    """Validate that uploaded file is actually an image."""
    try:
        # Check using PIL
        img = Image.open(value)
        img.verify()
        
        # Check MIME type
        value.seek(0)
        mime = magic.from_buffer(value.read(1024), mime=True)
        if not mime.startswith('image/'):
            raise ValidationError('File is not a valid image')
        
        value.seek(0)
    except Exception:
        raise ValidationError('Invalid image file')

class Document(models.Model):
    file = models.FileField(
        upload_to='documents/%Y/%m/%d/',
        validators=[
            validate_file_extension,
            validate_file_size
        ]
    )
    uploaded_by = models.ForeignKey(User, on_delete=models.CASCADE)
    uploaded_at = models.DateTimeField(auto_now_add=True)

# In serializer
class DocumentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Document
        fields = ['id', 'file', 'uploaded_at']
    
    def validate_file(self, value):
        """Additional file validation."""
        # Check file name for path traversal
        if '..' in value.name or '/' in value.name:
            raise serializers.ValidationError("Invalid file name")
        
        return value

# Secure file serving
from django.http import FileResponse
from django.shortcuts import get_object_or_404

class DocumentDownloadView(APIView):
    permission_classes = [IsAuthenticated]
    
    def get(self, request, pk):
        document = get_object_or_404(Document, pk=pk)
        
        # Check permissions
        if document.uploaded_by != request.user and not request.user.is_staff:
            return Response(
                {'error': 'Permission denied'},
                status=status.HTTP_403_FORBIDDEN
            )
        
        # Serve file securely
        response = FileResponse(document.file.open('rb'))
        response['Content-Disposition'] = f'attachment; filename="{document.file.name}"'
        return response
```

### Preventing Mass Assignment

```python
class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserProfile
        fields = ['bio', 'avatar', 'phone']
        read_only_fields = ['is_verified', 'is_premium', 'balance']  # Critical fields
    
    def update(self, instance, validated_data):
        # Only update allowed fields
        for field in ['bio', 'avatar', 'phone']:
            if field in validated_data:
                setattr(instance, field, validated_data[field])
        
        instance.save()
        return instance

# In view
class UserProfileViewSet(viewsets.ModelViewSet):
    serializer_class = UserProfileSerializer
    permission_classes = [IsAuthenticated]
    
    def get_queryset(self):
        # Users can only access their own profile
        return UserProfile.objects.filter(user=self.request.user)
    
    def update(self, request, *args, **kwargs):
        # Prevent users from modifying other profiles
        instance = self.get_object()
        if instance.user != request.user:
            return Response(
                {'error': 'Permission denied'},
                status=status.HTTP_403_FORBIDDEN
            )
        
        return super().update(request, *args, **kwargs)
```

### Secure Headers

```python
# settings.py

# Security settings for production
SECURE_SSL_REDIRECT = True  # Redirect HTTP to HTTPS
SECURE_HSTS_SECONDS = 31536000  # HSTS header
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

SESSION_COOKIE_SECURE = True  # Only send session cookie over HTTPS
SESSION_COOKIE_HTTPONLY = True  # Prevent JavaScript access
SESSION_COOKIE_SAMESITE = 'Lax'

SECURE_CONTENT_TYPE_NOSNIFF = True  # Prevent MIME type sniffing
SECURE_BROWSER_XSS_FILTER = True  # Enable XSS filter
X_FRAME_OPTIONS = 'DENY'  # Prevent clickjacking

# CORS settings
CORS_ALLOWED_ORIGINS = [
    "https://yourdomain.com",
    "https://www.yourdomain.com",
]

CORS_ALLOW_CREDENTIALS = True

# CSP (Content Security Policy)
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'", "'unsafe-inline'", "https://cdn.example.com")
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
CSP_IMG_SRC = ("'self'", "data:", "https:")
CSP_FONT_SRC = ("'self'", "https:")
```

### API Key Management

```python
# models.py
import secrets

class APIKey(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    key = models.CharField(max_length=64, unique=True)
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
    last_used = models.DateTimeField(null=True, blank=True)
    is_active = models.BooleanField(default=True)
    
    def save(self, *args, **kwargs):
        if not self.key:
            self.key = secrets.token_urlsafe(48)
        super().save(*args, **kwargs)
    
    def __str__(self):
        return f"{self.name} - {self.key[:8]}..."

# Custom authentication
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed

class APIKeyAuthentication(BaseAuthentication):
    def authenticate(self, request):
        api_key = request.META.get('HTTP_X_API_KEY')
        
        if not api_key:
            return None
        
        try:
            key = APIKey.objects.select_related('user').get(
                key=api_key,
                is_active=True
            )
            key.last_used = timezone.now()
            key.save(update_fields=['last_used'])
            
            return (key.user, key)
        except APIKey.DoesNotExist:
            raise AuthenticationFailed('Invalid API key')

# In settings
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'myapp.authentication.APIKeyAuthentication',
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}
```

## Part 10: Performance Optimization

### Database Optimization

```python
# Use database indexes
class Product(models.Model):
    name = models.CharField(max_length=200, db_index=True)
    sku = models.CharField(max_length=50, unique=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['category', 'price']),
            models.Index(fields=['-created_at']),
            models.Index(fields=['name', 'category']),
        ]
        # Composite unique constraint
        constraints = [
            models.UniqueConstraint(
                fields=['name', 'category'],
                name='unique_product_per_category'
            )
        ]

# Database-level operations
from django.db.models import F

# Use F() expressions for atomic updates
Product.objects.filter(id=product_id).update(
    stock=F('stock') - quantity,
    sold_count=F('sold_count') + quantity
)

# Bulk operations
# Bad: Multiple queries
for product_data in products_data:
    Product.objects.create(**product_data)

# Good: Single query
Product.objects
# Good: Single query
Product.objects.bulk_create([
    Product(**product_data) for product_data in products_data
])

# Bulk update
products = Product.objects.filter(category_id=1)
for product in products:
    product.price = product.price * Decimal('1.1')

Product.objects.bulk_update(products, ['price'], batch_size=1000)

# Efficient bulk delete
Product.objects.filter(stock=0, is_active=False).delete()

# Update or Create
product, created = Product.objects.update_or_create(
    sku='SKU123',
    defaults={'price': Decimal('99.99'), 'stock': 10}
)
```

### Query Optimization Techniques

```python
class OptimizedProductViewSet(viewsets.ModelViewSet):
    serializer_class = ProductSerializer
    
    def get_queryset(self):
        """Optimize queries based on action."""
        queryset = Product.objects.all()
        
        if self.action == 'list':
            # Optimize list view
            queryset = queryset.select_related('category').only(
                'id', 'name', 'price', 'stock',
                'category__id', 'category__name'
            )
        
        elif self.action == 'retrieve':
            # Optimize detail view with all relations
            queryset = queryset.select_related(
                'category',
                'brand',
                'created_by'
            ).prefetch_related(
                'reviews',
                'reviews__user',
                'images',
                'tags'
            )
        
        return queryset
    
    def list(self, request, *args, **kwargs):
        """Optimized list with aggregations."""
        queryset = self.filter_queryset(self.get_queryset())
        
        # Annotate with calculated fields
        queryset = queryset.annotate(
            review_count=Count('reviews'),
            avg_rating=Avg('reviews__rating')
        )
        
        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        
        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)

# Using iterator() for large querysets
def export_products(request):
    """Export large dataset efficiently."""
    products = Product.objects.all().iterator(chunk_size=1000)
    
    for product in products:
        # Process one at a time without loading all into memory
        export_product_data(product)

# Database connection pooling
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '5432',
        'CONN_MAX_AGE': 600,  # Connection pooling
        'OPTIONS': {
            'connect_timeout': 10,
        }
    }
}

# Read replicas for scaling
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'HOST': 'primary-db',
    },
    'replica': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'HOST': 'replica-db',
    }
}

DATABASE_ROUTERS = ['myapp.routers.PrimaryReplicaRouter']

# routers.py
class PrimaryReplicaRouter:
    """Route reads to replica, writes to primary."""
    
    def db_for_read(self, model, **hints):
        return 'replica'
    
    def db_for_write(self, model, **hints):
        return 'default'
    
    def allow_relation(self, obj1, obj2, **hints):
        return True
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        return db == 'default'
```

### Serializer Optimization

```python
# Use source parameter to avoid extra queries
class OptimizedProductSerializer(serializers.ModelSerializer):
    category_name = serializers.CharField(source='category.name', read_only=True)
    brand_name = serializers.CharField(source='brand.name', read_only=True)
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'category_name', 'brand_name']

# Avoid serializing unnecessary data
class MinimalProductSerializer(serializers.ModelSerializer):
    """Lightweight serializer for mobile apps."""
    class Meta:
        model = Product
        fields = ['id', 'name', 'price']  # Only essential fields

# Conditional field inclusion
class FlexibleProductSerializer(serializers.ModelSerializer):
    reviews = ReviewSerializer(many=True, read_only=True)
    related_products = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'reviews', 'related_products']
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        request = self.context.get('request')
        if request:
            # Remove expensive fields if not explicitly requested
            include_reviews = request.query_params.get('include_reviews', 'false')
            if include_reviews.lower() != 'true':
                self.fields.pop('reviews', None)
            
            include_related = request.query_params.get('include_related', 'false')
            if include_related.lower() != 'true':
                self.fields.pop('related_products', None)
    
    def get_related_products(self, obj):
        # Only calculate if field is included
        related = Product.objects.filter(
            category=obj.category
        ).exclude(id=obj.id)[:5]
        return MinimalProductSerializer(related, many=True).data

# Optimize to_representation
class EfficientOrderSerializer(serializers.ModelSerializer):
    items = OrderItemSerializer(many=True)
    
    class Meta:
        model = Order
        fields = ['id', 'user', 'status', 'items', 'total']
    
    def to_representation(self, instance):
        """Cache related objects to avoid repeated queries."""
        # Prefetch all products at once
        if isinstance(instance, list):
            product_ids = set()
            for order in instance:
                product_ids.update(
                    order.items.values_list('product_id', flat=True)
                )
            products = {
                p.id: p for p in Product.objects.filter(id__in=product_ids)
            }
            # Store in context for child serializers
            self.context['cached_products'] = products
        
        return super().to_representation(instance)
```

### Caching Strategies

```python
# View-level caching with cache keys
from django.core.cache import cache
from django.utils.encoding import force_bytes
from hashlib import md5

class CachedProductViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def list(self, request, *args, **kwargs):
        # Generate cache key from query params
        query_params = request.query_params.dict()
        cache_key = self.get_cache_key('list', query_params)
        
        # Try to get from cache
        cached_response = cache.get(cache_key)
        if cached_response:
            return Response(cached_response)
        
        # Get from database
        response = super().list(request, *args, **kwargs)
        
        # Cache for 15 minutes
        cache.set(cache_key, response.data, 60 * 15)
        
        return response
    
    def retrieve(self, request, *args, **kwargs):
        cache_key = self.get_cache_key('detail', {'pk': kwargs.get('pk')})
        
        cached_response = cache.get(cache_key)
        if cached_response:
            return Response(cached_response)
        
        response = super().retrieve(request, *args, **kwargs)
        cache.set(cache_key, response.data, 60 * 30)
        
        return response
    
    def get_cache_key(self, action, params):
        """Generate consistent cache key."""
        params_str = str(sorted(params.items()))
        key_hash = md5(force_bytes(params_str)).hexdigest()
        return f'product_{action}_{key_hash}'

# Query result caching
from django.core.cache import cache

def get_featured_products():
    """Get featured products with caching."""
    cache_key = 'featured_products'
    products = cache.get(cache_key)
    
    if products is None:
        products = list(
            Product.objects.filter(is_featured=True)
            .select_related('category')
            .values('id', 'name', 'price', 'category__name')
        )
        cache.set(cache_key, products, 60 * 60)  # 1 hour
    
    return products

# Template fragment caching
from django.core.cache import cache
from django.template.loader import render_to_string

def render_product_card(product_id):
    cache_key = f'product_card_{product_id}'
    html = cache.get(cache_key)
    
    if html is None:
        product = Product.objects.select_related('category').get(id=product_id)
        html = render_to_string('product_card.html', {'product': product})
        cache.set(cache_key, html, 60 * 30)
    
    return html

# Cache invalidation
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver

@receiver([post_save, post_delete], sender=Product)
def invalidate_product_cache(sender, instance, **kwargs):
    """Invalidate cache when product changes."""
    # Clear specific product cache
    cache_key_detail = f'product_detail_{instance.id}'
    cache.delete(cache_key_detail)
    
    # Clear list cache
    cache.delete_pattern('product_list_*')
    
    # Clear featured products if applicable
    if instance.is_featured:
        cache.delete('featured_products')
    
    # Clear category cache
    cache.delete(f'category_{instance.category_id}_products')

# Use cache_page decorator
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator

class ProductListView(APIView):
    @method_decorator(cache_page(60 * 15))
    def get(self, request):
        products = Product.objects.all()
        serializer = ProductSerializer(products, many=True)
        return Response(serializer.data)
```

### Async Tasks with Celery

```python
# Install: pip install celery redis

# celery.py
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

app = Celery('myproject')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

# settings.py
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'UTC'

# tasks.py
from celery import shared_task
from django.core.mail import send_mail
from .models import Order, Product

@shared_task
def send_order_confirmation_email(order_id):
    """Send order confirmation email asynchronously."""
    try:
        order = Order.objects.get(id=order_id)
        send_mail(
            subject=f'Order Confirmation #{order.id}',
            message=f'Thank you for your order!',
            from_email='noreply@example.com',
            recipient_list=[order.user.email],
            fail_silently=False,
        )
        return f'Email sent for order {order_id}'
    except Order.DoesNotExist:
        return f'Order {order_id} not found'

@shared_task
def process_bulk_import(file_path):
    """Process large file import in background."""
    import csv
    
    with open(file_path, 'r') as file:
        reader = csv.DictReader(file)
        products = []
        
        for row in reader:
            products.append(Product(
                name=row['name'],
                price=row['price'],
                stock=row['stock']
            ))
            
            # Bulk create in batches
            if len(products) >= 1000:
                Product.objects.bulk_create(products)
                products = []
        
        # Create remaining
        if products:
            Product.objects.bulk_create(products)
    
    return 'Import completed'

@shared_task
def generate_sales_report(start_date, end_date):
    """Generate sales report asynchronously."""
    from django.db.models import Sum, Count
    
    orders = Order.objects.filter(
        created_at__gte=start_date,
        created_at__lte=end_date,
        status='completed'
    )
    
    report = {
        'total_orders': orders.count(),
        'total_revenue': orders.aggregate(Sum('total'))['total__sum'],
        'top_products': Product.objects.filter(
            orderitem__order__in=orders
        ).annotate(
            total_sold=Sum('orderitem__quantity')
        ).order_by('-total_sold')[:10].values('name', 'total_sold')
    }
    
    return report

@shared_task(bind=True, max_retries=3)
def charge_customer(self, order_id, amount):
    """Charge customer with retry logic."""
    try:
        order = Order.objects.get(id=order_id)
        # Payment gateway call
        result = payment_gateway.charge(
            customer=order.user.stripe_customer_id,
            amount=amount
        )
        
        if result['success']:
            order.payment_status = 'paid'
            order.save()
            return 'Payment successful'
        else:
            raise Exception('Payment failed')
    
    except Exception as exc:
        # Retry with exponential backoff
        raise self.retry(exc=exc, countdown=60 * (2 ** self.request.retries))

# In views
class OrderViewSet(viewsets.ModelViewSet):
    queryset = Order.objects.all()
    serializer_class = OrderSerializer
    
    def perform_create(self, serializer):
        order = serializer.save()
        
        # Trigger async task
        send_order_confirmation_email.delay(order.id)
        
        return order
    
    @action(detail=False, methods=['post'])
    def bulk_import(self, request):
        file = request.FILES.get('file')
        
        # Save file temporarily
        file_path = f'/tmp/{file.name}'
        with open(file_path, 'wb') as f:
            for chunk in file.chunks():
                f.write(chunk)
        
        # Start background task
        task = process_bulk_import.delay(file_path)
        
        return Response({
            'task_id': task.id,
            'status': 'processing'
        })
    
    @action(detail=False, methods=['get'])
    def task_status(self, request):
        task_id = request.query_params.get('task_id')
        
        from celery.result import AsyncResult
        result = AsyncResult(task_id)
        
        return Response({
            'task_id': task_id,
            'status': result.status,
            'result': result.result if result.ready() else None
        })

# Periodic tasks
from celery.schedules import crontab

app.conf.beat_schedule = {
    'cleanup-old-sessions': {
        'task': 'myapp.tasks.cleanup_old_sessions',
        'schedule': crontab(hour=2, minute=0),  # Run daily at 2 AM
    },
    'send-daily-summary': {
        'task': 'myapp.tasks.send_daily_summary',
        'schedule': crontab(hour=9, minute=0),
    },
    'update-product-rankings': {
        'task': 'myapp.tasks.update_product_rankings',
        'schedule': crontab(minute='*/30'),  # Every 30 minutes
    },
}
```

### Compression and Response Optimization

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.gzip.GZipMiddleware',  # Enable GZIP compression
    # ... other middleware
]

# Custom compression middleware
from django.utils.decorators import decorator_from_middleware
from django.middleware.gzip import GZipMiddleware

class ConditionalGZipMiddleware(GZipMiddleware):
    """Only compress responses over certain size."""
    
    def process_response(self, request, response):
        # Only compress if response is large enough
        if len(response.content) < 200:
            return response
        return super().process_response(request, response)

# Response streaming for large data
from rest_framework.response import Response
from rest_framework.renderers import JSONRenderer
import json

class StreamingProductView(APIView):
    """Stream large dataset to client."""
    
    def get(self, request):
        def generate_products():
            yield '{"products": ['
            
            products = Product.objects.all().iterator(chunk_size=100)
            first = True
            
            for product in products:
                if not first:
                    yield ','
                first = False
                
                data = ProductSerializer(product).data
                yield json.dumps(data)
            
            yield ']}'
        
        return StreamingHttpResponse(
            generate_products(),
            content_type='application/json'
        )

# Pagination optimization
from rest_framework.pagination import CursorPagination

class OptimizedCursorPagination(CursorPagination):
    """Cursor pagination for better performance on large datasets."""
    page_size = 50
    page_size_query_param = 'page_size'
    max_page_size = 100
    ordering = '-created_at'
    
    def get_paginated_response(self, data):
        return Response({
            'next': self.get_next_link(),
            'previous': self.get_previous_link(),
            'results': data,
            'count': None  # Don't count total (expensive on large tables)
        })
```

### Database Connection Optimization

```python
# Persistent connections
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'CONN_MAX_AGE': 600,  # Keep connections open for 10 minutes
    }
}

# Use pgbouncer for connection pooling (external tool)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '6432',  # pgbouncer port
        'OPTIONS': {
            'connect_timeout': 10,
        }
    }
}

# Close idle connections
from django.db import connection

def close_old_connections_middleware(get_response):
    def middleware(request):
        response = get_response(request)
        connection.close_if_unusable_or_obsolete()
        return response
    return middleware
```

## Part 11: Deployment and Production

### Environment Configuration

```python
# settings/base.py - Base settings for all environments
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third party
    'rest_framework',
    'rest_framework_simplejwt',
    'django_filters',
    'corsheaders',
    
    # Local apps
    'apps.users',
    'apps.products',
    'apps.orders',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myproject.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'myproject.wsgi.application'

# Database - will be overridden in environment-specific settings
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator'},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# DRF settings
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
    ],
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}

# settings/development.py
from .base import *

DEBUG = True

ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# Development database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# CORS for development
CORS_ALLOW_ALL_ORIGINS = True

# Email backend for development
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Disable caching in development
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}

# settings/production.py
from .base import *

DEBUG = False

ALLOWED_HOSTS = [
    'api.yourdomain.com',
    'www.yourdomain.com',
    'yourdomain.com',
]

# Security settings
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_HTTPONLY = True

# Production database with connection pooling
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT', '5432'),
        'CONN_MAX_AGE': 600,
        'OPTIONS': {
            'connect_timeout': 10,
            'options': '-c statement_timeout=30000'
        }
    }
}

# Redis cache
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': os.environ.get('REDIS_URL', 'redis://127.0.0.1:6379/1'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
            'SOCKET_CONNECT_TIMEOUT': 5,
            'SOCKET_TIMEOUT': 5,
            'CONNECTION_POOL_KWARGS': {'max_connections': 50}
        }
    }
}

# Celery
CELERY_BROKER_URL = os.environ.get('CELERY_BROKER_URL', 'redis://localhost:6379/0')
CELERY_RESULT_BACKEND = os.environ.get('CELERY_RESULT_BACKEND', 'redis://localhost:6379/0')

# Email settings
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST')
EMAIL_PORT = int(os.environ.get('EMAIL_PORT', 587))
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')
DEFAULT_FROM_EMAIL = os.environ.get('DEFAULT_FROM_EMAIL')

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': BASE_DIR / 'logs' / 'django.log',
            'maxBytes': 1024 * 1024 * 10,  # 10 MB
            'backupCount': 10,
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': False,
        },
        'myapp': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': False,
        },
    },
}

# Static files with WhiteNoise
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')

# Media files on S3
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')
AWS_S3_REGION_NAME = os.environ.get('AWS_S3_REGION_NAME', 'us-east-1')
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'
AWS_DEFAULT_ACL = 'private'
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=86400',
}

# CORS
CORS_ALLOWED_ORIGINS = [
    "https://yourdomain.com",
    "https://www.yourdomain.com",
]
CORS_ALLOW_CREDENTIALS = True

# settings/__init__.py
import os

environment = os.environ.get('DJANGO_ENV', 'development')

if environment == 'production':
    from .production import *
elif environment == 'testing':
    from .testing import *
else:
    from .development import *
```

### Docker Configuration

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    postgresql-client \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt /app/
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# Copy project
COPY . /app/

# Create directories
RUN mkdir -p /app/staticfiles /app/mediafiles /app/logs

# Collect static files
RUN python manage.py collectstatic --noinput

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Run gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--threads", "2", "myproject.wsgi:application"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  web:
    build: .
    command: gunicorn myproject.wsgi:application --bind 0.0.0.0:8000 --workers 4
    volumes:
      - .:/app
      - static_volume:/app/staticfiles
      - media_volume:/app/mediafiles
    ports:
      - "8000:8000"
    environment:
      - DJANGO_ENV=production
      - DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY}
      - DB_NAME=mydb
      - DB_USER=myuser
      - DB_PASSWORD=mypassword
      - DB_HOST=db
      - DB_PORT=5432
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  celery:
    build: .
    command: celery -A myproject worker -l info
    volumes:
      - .:/app
    environment:
      - DJANGO_ENV=production
      - DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY}
      - DB_NAME=mydb
      - DB_USER=myuser
      - DB_PASSWORD=mypassword
      - DB_HOST=db
      - DB_PORT=5432
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
    depends_on:
      - db
      - redis

  celery-beat:
    build: .
    command: celery -A myproject beat -l info
    volumes:
      - .:/app
    environment:
      - DJANGO_ENV=production
      - DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY}
      - DB_NAME=mydb
      - DB_USER=myuser
      - DB_PASSWORD=mypassword
      - DB_HOST=db
      - DB_PORT=5432
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
    depends_on:
      - db
      - redis

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - static_volume:/app/staticfiles
      - media_volume:/app/mediafiles
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    upstream django {
        server web:8000;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;

    server {
        listen 80;
        server_name yourdomain.com www.yourdomain.com;
        client_max_body_size 100M;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

        # Gzip compression
        gzip on;
        gzip_vary on;
        gzip_min_length 1024;
        gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/json;

        # Static files
        location /static/ {
            alias /app/staticfiles/;
            expires 30d;
            add_header Cache-Control "public, immutable";
        }

        # Media files
        location /media/ {
            alias /app/mediafiles/;
            expires 7d;
            add_header Cache-Control "public";
        }

        # API endpoints with rate limiting
        location /api/ {
            limit_req zone=api_limit burst=20 nodelay;
            
            proxy_pass http://django;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }

        # Login endpoint with stricter rate limiting
        location /api/token/ {
            limit_req zone=login_limit burst=3 nodelay;
            
            proxy_pass http://django;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Health check
        location /health/ {
            access_log off;
            proxy_pass http://django;
        }

        # Admin panel
        location /admin/ {
            proxy_pass http://django;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### CI/CD with GitHub Actions

```yaml
# .github/workflows/django.yml
name: Django CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  PYTHON_VERSION: '3.11'
  POSTGRES_DB: test_db
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: postgres

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: ${{ env.POSTGRES_DB }}
          POSTGRES_USER: ${{ env.POSTGRES_USER }}
          POSTGRES_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python ${{ env.PYTHON_VERSION }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
        cache: 'pip'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install coverage flake8 black isort
    
    - name: Run linting
      run: |
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
        black --check .
        isort --check-only .
    
    - name: Run migrations
      env:
        DJANGO_ENV: testing
        DJANGO_SECRET_KEY: test-secret-key
        DB_NAME: ${{ env.POSTGRES_DB }}
        DB_USER: ${{ env.POSTGRES_USER }}
        DB_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
        DB_HOST: localhost
        DB_PORT: 5432
      run: |
        python manage.py migrate
    
    - name: Run tests with coverage
      env:
        DJANGO_ENV: testing
        DJANGO_SECRET_KEY: test-secret-key
        DB_NAME: ${{ env.POSTGRES_DB }}
        DB_USER: ${{ env.POSTGRES_USER }}
        DB_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
        DB_HOST: localhost
        DB_PORT: 5432
      run: |
        coverage run --source='.' manage.py test
        coverage report
        coverage xml
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
        fail_ci_if_error: true

  security:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install bandit safety
    
    - name: Run security checks
      run: |
        bandit -r . -f json -o bandit-report.json || true
        safety check --json > safety-report.json || true
    
    - name: Upload security reports
      uses: actions/upload-artifact@v3
      with:
        name: security-reports
        path: |
          bandit-report.json
          safety-report.json

  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ${{ secrets.DOCKER_USERNAME }}/myapp:latest
          ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}
        cache-from: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/myapp:buildcache
        cache-to: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/myapp:buildcache,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.PRODUCTION_HOST }}
        username: ${{ secrets.PRODUCTION_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /opt/myapp
          docker-compose pull
          docker-compose up -d
          docker-compose exec -T web python manage.py migrate
          docker-compose exec -T web python manage.py collectstatic --noinput
```

### Health Checks and Monitoring

```python
# health/views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.db import connection
from django.core.cache import cache
import redis

class HealthCheckView(APIView):
    """
    Health check endpoint for monitoring.
    """
    permission_classes = []
    authentication_classes = []
    
    def get(self, request):
        health_status = {
            'status': 'healthy',
            'checks': {}
        }
        
        # Check database
        try:
            with connection.cursor() as cursor:
                cursor.execute("SELECT 1")
            health_status['checks']['database'] = 'healthy'
        except Exception as e:
            health_status['checks']['database'] = f'unhealthy: {str(e)}'
            health_status['status'] = 'unhealthy'
        
        # Check cache
        try:
            cache.set('health_check', 'ok', 10)
            result = cache.get('health_check')
            if result == 'ok':
                health_status['checks']['cache'] = 'healthy'
            else:
                raise Exception('Cache verification failed')
        except Exception as e:
            health_status['checks']['cache'] = f'unhealthy: {str(e)}'
            health_status['status'] = 'unhealthy'
        
        # Check Celery
        try:
            from myproject.celery import app
            inspect = app.control.inspect()
            stats = inspect.stats()
            if stats:
                health_status['checks']['celery'] = 'healthy'
            else:
                health_status['checks']['celery'] = 'no workers available'
        except Exception as e:
            health_status['checks']['celery'] = f'unhealthy: {str(e)}'
        
        # Return appropriate status code
        if health_status['status'] == 'healthy':
            return Response(health_status, status=status.HTTP_200_OK)
        else:
            return Response(health_status, status=status.HTTP_503_SERVICE_UNAVAILABLE)

class ReadinessCheckView(APIView):
    """
    Readiness check for Kubernetes/load balancers.
    """
    permission_classes = []
    authentication_classes = []
    
    def get(self, request):
        # Check if application is ready to serve traffic
        try:
            # Quick database check
            with connection.cursor() as cursor:
                cursor.execute("SELECT 1")
            
            return Response({'status': 'ready'}, status=status.HTTP_200_OK)
        except Exception:
            return Response({'status': 'not ready'}, status=status.HTTP_503_SERVICE_UNAVAILABLE)

class LivenessCheckView(APIView):
    """
    Liveness check for Kubernetes.
    """
    permission_classes = []
    authentication_classes = []
    
    def get(self, request):
        # Simple check to see if application is alive
        return Response({'status': 'alive'}, status=status.HTTP_200_OK)

# urls.py
urlpatterns = [
    path('health/', HealthCheckView.as_view(), name='health'),
    path('health/ready/', ReadinessCheckView.as_view(), name='readiness'),
    path('health/live/', LivenessCheckView.as_view(), name='liveness'),
]
```

### Logging and Monitoring

```python
# middleware/logging.py
import time
import json
import logging
from django.utils.deprecation import MiddlewareMixin

logger = logging.getLogger(__name__)

class RequestLoggingMiddleware(MiddlewareMixin):
    """Log all requests and responses."""
    
    def process_request(self, request):
        request._start_time = time.time()
        
        # Log request
        log_data = {
            'method': request.method,
            'path': request.path,
            'user': str(request.user) if request.user.is_authenticated else 'anonymous',
            'ip': self.get_client_ip(request),
            'user_agent': request.META.get('HTTP_USER_AGENT', ''),
        }
        
        logger.info(f"Request: {json.dumps(log_data)}")
    
    def process_response(self, request, response):
        if hasattr(request, '_start_time'):
            duration = time.time() - request._start_time
            
            log_data = {
                'method': request.method,
                'path': request.path,
                'status': response.status_code,
                'duration': f"{duration:.3f}s",
                'user': str(request.user) if request.user.is_authenticated else 'anonymous',
            }
            
            logger.info(f"Response: {json.dumps(log_data)}")
        
        return response
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip

class ErrorLoggingMiddleware(MiddlewareMixin):
    """Log all exceptions with context."""
    
    def process_exception(self, request, exception):
        log_data = {
            'exception': str(exception),
            'exception_type': type(exception).__name__,
            'method': request.method,
            'path': request.path,
            'user': str(request.user) if request.user.is_authenticated else 'anonymous',
            'body': request.body.decode('utf-8') if request.body else None,
        }
        
        logger.error(f"Exception: {json.dumps(log_data)}", exc_info=True)
        
        return None

# Integration with Sentry
# settings.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration
from sentry_sdk.integrations.celery import CeleryIntegration

if not DEBUG:
    sentry_sdk.init(
        dsn=os.environ.get('SENTRY_DSN'),
        integrations=[
            DjangoIntegration(),
            CeleryIntegration(),
        ],
        traces_sample_rate=0.1,  # 10% of transactions
        send_default_pii=True,
        environment=os.environ.get('DJANGO_ENV', 'production'),
    )
```

### Database Backups

```python
# management/commands/backup_db.py
from django.core.management.base import BaseCommand
from django.conf import settings
import subprocess
import os
from datetime import datetime
import boto3

class Command(BaseCommand):
    help = 'Backup database to S3'
    
    def add_arguments(self, parser):
        parser.add_argument(
            '--upload-to-s3',
            action='store_true',
            help='Upload backup to S3',
        )
    
    def handle(self, *args, **options):
        # Generate backup filename
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_file = f'backup_{timestamp}.sql'
        
        # Database configuration
        db_config = settings.DATABASES['default']
        
        # Create backup using pg_dump
        try:
            self.stdout.write('Creating database backup...')
            
            command = [
                'pg_dump',
                f"--dbname=postgresql://{db_config['USER']}:{db_config['PASSWORD']}@{db_config['HOST']}:{db_config['PORT']}/{db_config['NAME']}",
                '-f', backup_file
            ]
            
            subprocess.run(command, check=True)
            self.stdout.write(self.style.SUCCESS(f'Backup created: {backup_file}'))
            
            # Upload to S3 if requested
            if options['upload_to_s3']:
                self.upload_to_s3(backup_file)
            
            # Clean up old backups (keep last 7 days)
            self.cleanup_old_backups()
            
        except subprocess.CalledProcessError as e:
            self.stdout.write(self.style.ERROR(f'Backup failed: {e}'))
        except Exception as e:
            self.stdout.write(self.style.ERROR(f'Error: {e}'))
    
    def upload_to_s3(self, backup_file):
        """Upload backup to S3."""
        self.stdout.write('Uploading to S3...')
        
        s3 = boto3.client('s3')
        bucket_name = os.environ.get('BACKUP_BUCKET_NAME')
        
        with open(backup_file, 'rb') as f:
            s3.upload_fileobj(
                f,
                bucket_name,
                f'database-backups/{backup_file}'
            )
        
        self.stdout.write(self.style.SUCCESS('Upload completed'))
    
    def cleanup_old_backups(self):
        """Remove backups older than 7 days."""
        import glob
        from datetime import timedelta
        
        cutoff_date = datetime.now() - timedelta(days=7)
        
        for backup_file in glob.glob('backup_*.sql'):
            file_time = datetime.fromtimestamp(os.path.getmtime(backup_file))
            if file_time < cutoff_date:
                os.remove(backup_file)
                self.stdout.write(f'Removed old backup: {backup_file}')

# Automated backups with cron or Celery beat
# celery.py
from celery.schedules import crontab

app.conf.beat_schedule = {
    'backup-database-daily': {
        'task': 'myapp.tasks.backup_database',
        'schedule': crontab(hour=2, minute=0),  # 2 AM daily
    },
}

# tasks.py
@shared_task
def backup_database():
    """Automated database backup."""
    from django.core.management import call_command
    call_command('backup_db', '--upload-to-s3')
```

## Part 12: Advanced Topics and Best Practices

### Custom Management Commands

```python
# management/commands/seed_data.py
from django.core.management.base import BaseCommand
from django.db import transaction
from apps.products.models import Category, Product
from apps.users.models import User
import random

class Command(BaseCommand):
    help = 'Seed database with test data'
    
    def add_arguments(self, parser):
        parser.add_argument(
            '--products',
            type=int,
            default=100,
            help='Number of products to create',
        )
        parser.add_argument(
            '--users',
            type=int,
            default=10,
            help='Number of users to create',
        )
    
    @transaction.atomic
    def handle(self, *args, **options):
        self.stdout.write('Seeding database...')
        
        # Create categories
        categories = self.create_categories()
        self.stdout.write(self.style.SUCCESS(f'Created {len(categories)} categories'))
        
        # Create users
        users_count = self.create_users(options['users'])
        self.stdout.write(self.style.SUCCESS(f'Created {users_count} users'))
        
        # Create products
        products_count = self.create_products(options['products'], categories)
        self.stdout.write(self.style.SUCCESS(f'Created {products_count} products'))
        
        self.stdout.write(self.style.SUCCESS('Database seeding completed!'))
    
    def create_categories(self):
        categories = ['Electronics', 'Clothing', 'Books', 'Home & Garden', 'Sports']
        created_categories = []
        
        for name in categories:
            category, created = Category.objects.get_or_create(
                name=name,
                defaults={'slug': name.lower().replace(' ', '-')}
            )
            created_categories.append(category)
        
        return created_categories
    
    def create_users(self, count):
        for i in range(count):
            User.objects.get_or_create(
                username=f'user{i}',
                defaults={
                    'email': f'user{i}@example.com',
                    'first_name': f'User',
                    'last_name': f'{i}',
                }
            )
        return count
    
    def create_products(self, count, categories):
        from decimal import Decimal
        
        for i in range(count):
            Product.objects.get_or_create(
                name=f'Product {i}',
                defaults={
                    'price': Decimal(random.uniform(10, 1000)),
                    'cost': Decimal(random.uniform(5, 500)),
                    'stock': random.randint(0, 100),
                    'category': random.choice(categories),
                    'description': f'Description for product {i}',
                }
            )
        return count
```

### Custom Validators

```python
# validators.py
from django.core.exceptions import ValidationError
from django.utils.translation import gettext_lazy as _
import re

def validate_phone_number(value):
    """Validate phone number format."""
    pattern = r'^\+?1?\d{9,15}$'
    if not re.match(pattern, value):
        raise ValidationError(
            _('%(value)s is not a valid phone number'),
            params={'value': value},
        )

def validate_alphanumeric(value):
    """Validate that value contains only alphanumeric characters."""
    if not value.isalnum():
        raise ValidationError(
            _('This field must contain only letters and numbers')
        )

def validate_file_extension(value):
    """Validate file extension."""
    import os
    ext = os.path.splitext(value.name)[1]
    valid_extensions = ['.pdf', '.doc', '.docx', '.txt']
    if ext.lower() not in valid_extensions:
        raise ValidationError(
            _('Unsupported file extension. Allowed: %(extensions)s'),
            params={'extensions': ', '.join(valid_extensions)},
        )

class PasswordComplexityValidator:
    """Custom password validator for complexity requirements."""
    
    def validate(self, password, user=None):
        if len(password) < 12:
            raise ValidationError(
                _("Password must be at least 12 characters long."),
                code='password_too_short',
            )
        
        if not re.search(r'[A-Z]', password):
            raise ValidationError(
                _("Password must contain at least one uppercase letter."),
                code='password_no_upper',
            )
        
        if not re.search(r'[a-z]', password):
            raise ValidationError(
                _("Password must contain at least one lowercase letter."),
                code='password_no_lower',
            )
        
        if not re.search(r'[0-9]', password):
            raise ValidationError(
                _("Password must contain at least one digit."),
                code='password_no_digit',
            )
        
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
            raise ValidationError(
                _("Password must contain at least one special character."),
                code='password_no_special',
            )
    
    def get_help_text(self):
        return _(
            "Your password must contain at least 12 characters, including "
            "uppercase and lowercase letters, numbers, and special characters."
        )

# Usage in models
class UserProfile(models.Model):
    phone = models.CharField(
        max_length=15,
        validators=[validate_phone_number]
    )
    
    document = models.FileField(
        upload_to='documents/',
        validators=[validate_file_extension]
    )
```

### Custom Mixins

```python
# mixins.py
from rest_framework import status
from rest_framework.response import Response
from django.core.cache import cache

class CacheResponseMixin:
    """Mixin to cache responses."""
    cache_timeout = 60 * 15  # 15 minutes
    
    def get_cache_key(self):
        """Generate cache key based on view and query params."""
        view_name = self.__class__.__name__
        query_params = self.request.query_params.dict()
        params_str = str(sorted(query_params.items()))
        return f"{view_name}:{params_str}"
    
    def dispatch(self, request, *args, **kwargs):
        if request.method == 'GET':
            cache_key = self.get_cache_key()
            cached_response = cache.get(cache_key)
            
            if cached_response:
                return Response(cached_response)
            
            response = super().dispatch(request, *args, **kwargs)
            
            if response.status_code == 200:
                cache.set(cache_key, response.data, self.cache_timeout)
            
            return response
        
        return super().dispatch(request, *args, **kwargs)

class TimestampMixin:
    """Add timestamp fields to models."""
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True

class UserTrackingMixin:
    """Track which user created/updated records."""
    created_by = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        related_name='%(class)s_created'
    )
    updated_by = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        related_name='%(class)s_updated'
    )
    
    class Meta:
        abstract = True

class SerializerByActionMixin:
    """Use different serializers for different actions."""
    serializer_classes = {}
    
    def get_serializer_class(self):
        return self.serializer_classes.get(
            self.action,
            self.serializer_class
        )

class MultiplePermissionsMixin:
    """Use different permissions for different actions."""
    permissions_classes = {}
    
    def get_permissions(self):
        return [
            permission() for permission in self.permissions_classes.get(
                self.action,
                self.permission_classes
            )
        ]

# Usage
class ProductViewSet(
    SerializerByActionMixin,
    MultiplePermissionsMixin,
    viewsets.ModelViewSet
):
    queryset = Product.objects.all()
    
    serializer_class = ProductSerializer
    serializer_classes = {
        'list': ProductListSerializer,
        'retrieve': ProductDetailSerializer,
        'create': ProductCreateSerializer,
    }
    
    permission_classes = [IsAuthenticated]
    permissions_classes = {
        'list': [AllowAny],
        'retrieve': [AllowAny],
        'create': [IsAuthenticated, IsStaffUser],
        'update': [IsAuthenticated, IsOwner],
        'destroy': [IsAuthenticated, IsAdminUser],
    }
```

### API Documentation with drf-spectacular

```python
# Install: pip install drf-spectacular

# settings.py
INSTALLED_APPS = [
    # ...
    'drf_spectacular',
]

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

SPECTACULAR_SETTINGS = {
    'TITLE': 'Your API',
    'DESCRIPTION': 'Your API description',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
    'COMPONENT_SPLIT_REQUEST': True,
    'SCHEMA_PATH_PREFIX': r'/api/',
    'SWAGGER_UI_SETTINGS': {
        'deepLinking': True,
        'persistAuthorization': True,
        'displayOperationId': True,
    },
    'PREPROCESSING_HOOKS': [],
    'SERVE_PERMISSIONS': ['rest_framework.permissions.AllowAny'],
}

# urls.py
from drf_spectacular.views import (
    SpectacularAPIView,
    SpectacularRedocView,
    SpectacularSwaggerView
)

urlpatterns = [
    # API Schema
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # Swagger UI
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    # ReDoc
    path('api/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]

# Enhanced documentation in views
from drf_spectacular.utils import extend_schema, extend_schema_view, OpenApiParameter, OpenApiExample
from drf_spectacular.types import OpenApiTypes

@extend_schema_view(
    list=extend_schema(
        summary="List all products",
        description="Retrieve a paginated list of all products with optional filtering",
        parameters=[
            OpenApiParameter(
                name='category',
                type=OpenApiTypes.INT,
                location=OpenApiParameter.QUERY,
                description='Filter by category ID',
            ),
            OpenApiParameter(
                name='min_price',
                type=OpenApiTypes.FLOAT,
                location=OpenApiParameter.QUERY,
                description='Minimum price filter',
            ),
            OpenApiParameter(
                name='max_price',
                type=OpenApiTypes.FLOAT,
                location=OpenApiParameter.QUERY,
                description='Maximum price filter',
            ),
            OpenApiParameter(
                name='search',
                type=OpenApiTypes.STR,
                location=OpenApiParameter.QUERY,
                description='Search in product name and description',
            ),
        ],
        tags=['Products'],
    ),
    retrieve=extend_schema(
        summary="Get product details",
        description="Retrieve detailed information about a specific product",
        tags=['Products'],
    ),
    create=extend_schema(
        summary="Create a new product",
        description="Create a new product (requires authentication)",
        tags=['Products'],
        examples=[
            OpenApiExample(
                'Product Creation Example',
                value={
                    'name': 'Sample Product',
                    'price': '99.99',
                    'cost': '50.00',
                    'stock': 100,
                    'category_id': 1,
                    'description': 'Product description'
                },
                request_only=True,
            ),
        ],
    ),
    update=extend_schema(
        summary="Update a product",
        description="Update an existing product (requires authentication)",
        tags=['Products'],
    ),
    destroy=extend_schema(
        summary="Delete a product",
        description="Delete a product (requires admin privileges)",
        tags=['Products'],
    ),
)
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    @extend_schema(
        summary="Mark product as favorite",
        description="Add this product to user's favorites",
        request=None,
        responses={200: {'type': 'object', 'properties': {'status': {'type': 'string'}}}},
        tags=['Products'],
    )
    @action(detail=True, methods=['post'])
    def favorite(self, request, pk=None):
        """Mark product as favorite."""
        product = self.get_object()
        request.user.favorites.add(product)
        return Response({'status': 'product favorited'})

# Custom serializer fields documentation
from drf_spectacular.utils import extend_schema_field

class ProductDetailSerializer(serializers.ModelSerializer):
    @extend_schema_field(OpenApiTypes.FLOAT)
    def get_average_rating(self, obj):
        """Calculate average rating from reviews."""
        return obj.reviews.aggregate(Avg('rating'))['rating__avg']
    
    average_rating = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'average_rating']
```

### GraphQL Integration (Optional)

```python
# Install: pip install graphene-django

# settings.py
INSTALLED_APPS = [
    # ...
    'graphene_django',
]

GRAPHENE = {
    'SCHEMA': 'myproject.schema.schema',
    'MIDDLEWARE': [
        'graphene_django.debug.DjangoDebugMiddleware',
    ],
}

# schema.py
import graphene
from graphene_django import DjangoObjectType
from apps.products.models import Product, Category

class CategoryType(DjangoObjectType):
    class Meta:
        model = Category
        fields = '__all__'

class ProductType(DjangoObjectType):
    class Meta:
        model = Product
        fields = '__all__'

class Query(graphene.ObjectType):
    all_products = graphene.List(ProductType)
    product = graphene.Field(ProductType, id=graphene.Int())
    
    all_categories = graphene.List(CategoryType)
    category = graphene.Field(CategoryType, id=graphene.Int())
    
    def resolve_all_products(self, info, **kwargs):
        return Product.objects.all()
    
    def resolve_product(self, info, id):
        return Product.objects.get(pk=id)
    
    def resolve_all_categories(self, info, **kwargs):
        return Category.objects.all()
    
    def resolve_category(self, info, id):
        return Category.objects.get(pk=id)

class CreateProduct(graphene.Mutation):
    class Arguments:
        name = graphene.String(required=True)
        price = graphene.Decimal(required=True)
        category_id = graphene.Int(required=True)
    
    product = graphene.Field(ProductType)
    
    def mutate(self, info, name, price, category_id):
        category = Category.objects.get(pk=category_id)
        product = Product.objects.create(
            name=name,
            price=price,
            category=category
        )
        return CreateProduct(product=product)

class Mutation(graphene.ObjectType):
    create_product = CreateProduct.Field()

schema = graphene.Schema(query=Query, mutation=Mutation)

# urls.py
from graphene_django.views import GraphQLView

urlpatterns = [
    path('graphql/', GraphQLView.as_view(graphiql=True)),
]
```

### WebSocket Support with Django Channels

```python
# Install: pip install channels channels-redis

# settings.py
INSTALLED_APPS = [
    'daphne',  # Place at top
    # ...
    'channels',
]

ASGI_APPLICATION = 'myproject.asgi.application'

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [('127.0.0.1', 6379)],
        },
    },
}

# asgi.py
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
from channels.security.websocket import AllowedHostsOriginValidator

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

django_asgi_app = get_asgi_application()

from myapp import routing

application = ProtocolTypeRouter({
    "http": django_asgi_app,
    "websocket": AllowedHostsOriginValidator(
        AuthMiddlewareStack(
            URLRouter(
                routing.websocket_urlpatterns
            )
        )
    ),
})

# consumers.py
from channels.generic.websocket import AsyncJsonWebsocketConsumer
from channels.db import database_sync_to_async

class NotificationConsumer(AsyncJsonWebsocketConsumer):
    async def connect(self):
        self.user = self.scope["user"]
        
        if self.user.is_anonymous:
            await self.close()
        else:
            self.room_group_name = f'notifications_{self.user.id}'
            
            # Join room group
            await self.channel_layer.group_add(
                self.room_group_name,
                self.channel_name
            )
            
            await self.accept()
    
    async def disconnect(self, close_code):
        # Leave room group
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
    
    async def receive_json(self, content):
        """Receive message from WebSocket."""
        message_type = content.get('type')
        
        if message_type == 'ping':
            await self.send_json({'type': 'pong'})
    
    async def notification_message(self, event):
        """Send notification to WebSocket."""
        await self.send_json({
            'type': 'notification',
            'message': event['message'],
            'data': event.get('data', {})
        })

# routing.py
from django.urls import path
from . import consumers

websocket_urlpatterns = [
    path('ws/notifications/', consumers.NotificationConsumer.as_asgi()),
]

# Sending notifications
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

def send_notification_to_user(user_id, message, data=None):
    """Send notification to specific user via WebSocket."""
    channel_layer = get_channel_layer()
    async_to_sync(channel_layer.group_send)(
        f'notifications_{user_id}',
        {
            'type': 'notification_message',
            'message': message,
            'data': data or {}
        }
    )

# In views or signals
class OrderViewSet(viewsets.ModelViewSet):
    def perform_create(self, serializer):
        order = serializer.save()
        
        # Send real-time notification
        send_notification_to_user(
            order.user.id,
            'Order created successfully',
            {'order_id': order.id, 'total': str(order.total)}
        )
```

### Custom Middleware Examples

```python
# middleware/custom.py
import time
import logging
from django.utils.deprecation import MiddlewareMixin
from django.core.cache import cache
from django.http import JsonResponse

logger = logging.getLogger(__name__)

class APIVersionMiddleware(MiddlewareMixin):
    """Handle API versioning via header."""
    
    def process_request(self, request):
        # Get version from header or default to v1
        api_version = request.META.get('HTTP_API_VERSION', 'v1')
        request.api_version = api_version

class RateLimitMiddleware(MiddlewareMixin):
    """Simple rate limiting middleware."""
    
    def process_request(self, request):
        if request.path.startswith('/api/'):
            ip = self.get_client_ip(request)
            cache_key = f'rate_limit_{ip}'
            
            # Get request count
            request_count = cache.get(cache_key, 0)
            
            if request_count >= 100:  # 100 requests per minute
                return JsonResponse({
                    'error': 'Rate limit exceeded'
                }, status=429)
            
            # Increment counter
            cache.set(cache_key, request_count + 1, 60)
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip

class RequestIDMiddleware(MiddlewareMixin):
    """Add unique request ID to each request."""
    
    def process_request(self, request):
        import uuid
        request.id = str(uuid.uuid4())
    
    def process_response(self, request, response):
        if hasattr(request, 'id'):
            response['X-Request-ID'] = request.id
        return response

class DatabaseQueryCountMiddleware(MiddlewareMixin):
    """Log number of database queries per request (development only)."""
    
    def process_request(self, request):
        from django.db import reset_queries
        reset_queries()
    
    def process_response(self, request, response):
        from django.db import connection
        from django.conf import settings
        
        if settings.DEBUG:
            queries = len(connection.queries)
            if queries > 10:  # Warn if too many queries
                logger.warning(
                    f'High query count: {queries} queries for {request.path}'
                )
        
        return response

class MaintenanceModeMiddleware(MiddlewareMixin):
    """Enable maintenance mode."""
    
    def process_request(self, request):
        from django.conf import settings
        
        # Check if maintenance mode is enabled
        maintenance_mode = cache.get('maintenance_mode', False)
        
        if maintenance_mode and not request.user.is_staff:
            # Allow staff to access during maintenance
            return JsonResponse({
                'error': 'Site is under maintenance',
                'message': 'We will be back soon!'
            }, status=503)
```

### Custom Template Tags and Filters

```python
# templatetags/custom_tags.py
from django import template
from django.utils.safestring import mark_safe
import json

register = template.Library()

@register.filter
def currency(value):
    """Format value as currency."""
    try:
        return f"${float(value):,.2f}"
    except (ValueError, TypeError):
        return value

@register.filter
def percentage(value):
    """Format value as percentage."""
    try:
        return f"{float(value):.1f}%"
    except (ValueError, TypeError):
        return value

@register.simple_tag
def settings_value(name):
    """Get value from Django settings."""
    from django.conf import settings
    return getattr(settings, name, "")

@register.inclusion_tag('components/pagination.html')
def render_pagination(page_obj):
    """Render pagination component."""
    return {
        'page_obj': page_obj,
        'has_previous': page_obj.has_previous(),
        'has_next': page_obj.has_next(),
    }

@register.filter
def jsonify(value):
    """Convert Python object to JSON."""
    return mark_safe(json.dumps(value))

# Usage in templates
# {{ product.price|currency }}
# {{ discount|percentage }}
# {% settings_value "SITE_NAME" %}
# {% render_pagination page_obj %}
# {{ data|jsonify }}
```

### Database Optimization Tips

```python
# Use select_related for ForeignKey and OneToOne
products = Product.objects.select_related('category', 'brand').all()

# Use prefetch_related for ManyToMany and reverse ForeignKey
authors = Author.objects.prefetch_related('books').all()

# Custom prefetch with filtering
from django.db.models import Prefetch

active_reviews = Review.objects.filter(is_approved=True)
products = Product.objects.prefetch_related(
    Prefetch('reviews', queryset=active_reviews, to_attr='active_reviews')
)

# Use only() to load specific fields
products = Product.objects.only('id', 'name', 'price')

# Use defer() to exclude heavy fields
products = Product.objects.defer('description', 'long_text_field')

# Use values() for dictionaries (no model instances)
product_data = Product.objects.values('id', 'name', 'price')

# Use values_list() for tuples
product_names = Product.objects.values_list('name', flat=True)

# Aggregate functions
from django.db.models import Count, Sum, Avg, Max, Min

stats = Product.objects.aggregate(
    total_products=Count('id'),
    total_value=Sum('price'),
    avg_price=Avg('price'),
    max_price=Max('price'),
    min_price=Min('price')
)

# Annotate to add calculated fields
products = Product.objects.annotate(
    review_count=Count('reviews'),
    avg_rating=Avg('reviews__rating')
)

# Use F() for database-level operations
from django.db.models import F

# Increment stock atomically
Product.objects.filter(id=1).update(stock=F('stock') + 10)

# Compare fields
expensive_products = Product.objects.filter(price__gt=F('cost') * 2)

# Use Q objects for complex queries
from django.db.models import Q

products = Product.objects.filter(
    Q(category__name='Electronics') | Q(category__name='Computers'),
    Q(price__lt=1000) & Q(stock__gt=0)
)

# Subqueries
from django.db.models import OuterRef, Subquery

latest_review = Review.objects.filter(
    product=OuterRef('pk')
).order_by('-created_at')

products = Product.objects.annotate(
    latest_review_rating=Subquery(latest_review.values('rating')[:1])
)

# Bulk operations for better performance
products_to_create = [
    Product(name=f'Product {i}', price=i*10)
    for i in range(1000)
]
Product.objects.bulk_create(products_to_create, batch_size=500)

# Bulk update
products = Product.objects.all()
for product in products:
    product.price = product.price * 1.1

Product.objects.bulk_update(products, ['price'], batch_size=500)

# Database indexes
class Product(models.Model):
    name = models.CharField(max_length=200)
    sku = models.CharField(max_length=50)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    
    class Meta:
        indexes = [
            models.Index(fields=['name']),
            models.Index(fields=['sku']),
            models.Index(fields=['price', 'created_at']),
            models.Index(fields=['-created_at']),  # For descending order
        ]
        # Composite index for common queries
        index_together = [
            ['category', 'price'],
            ['category', 'created_at'],
        ]

# Use explain() to analyze queries
print(Product.objects.filter(price__gt=100).explain())

# Use iterator() for large querysets to save memory
for product in Product.objects.all().iterator(chunk_size=2000):
    process_product(product)

# Raw SQL when necessary (but use sparingly)
products = Product.objects.raw(
    'SELECT * FROM products WHERE price > %s',
    [100]
)

# Database transactions
from django.db import transaction

@transaction.atomic
def create_order_with_items(order_data, items_data):
    order = Order.objects.create(**order_data)
    
    for item_data in items_data:
        OrderItem.objects.create(order=order, **item_data)
        
        # Update stock
        product = item_data['product']
        product.stock = F('stock') - item_data['quantity']
        product.save()
    
    return order

# Savepoints for partial rollback
from django.db import transaction

@transaction.atomic
def complex_operation():
    # Some operations
    
    sid = transaction.savepoint()
    
    try:
        # Risky operations
        risky_operation()
        transaction.savepoint_commit(sid)
    except Exception:
        transaction.savepoint_rollback(sid)
        # Handle error
```

### Final Production Checklist

```python
# settings/production.py checklist

# 1. Security settings
DEBUG = False
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')  # Never hardcode
ALLOWED_HOSTS = ['yourdomain.com']

# 2. HTTPS settings
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# 3. Cookie security
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SAMESITE = 'Lax'

# 4. Security headers
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

# 5. Database optimization
CONN_MAX_AGE = 600
DATABASES['default']['OPTIONS'] = {
    'connect_timeout': 10,
}

# 6. Static and media files
STATIC_ROOT = BASE_DIR / 'staticfiles'
MEDIA_ROOT = BASE_DIR / 'mediafiles'
# Or use S3/CDN for production

# 7. Logging
LOGGING = {
    # Comprehensive logging configuration
}

# 8. Caching
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        # Redis configuration
    }
}

# 9. Email configuration
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
# SMTP settings

# 10. CORS if needed
CORS_ALLOWED_ORIGINS = ['https://yourdomain.com']

# 11. Rate limiting
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

# 12. Error monitoring (Sentry)
import sentry_sdk
sentry_sdk.init(dsn=os.environ.get('SENTRY_DSN'))

# 13. Admin security
ADMIN_URL = os.environ.get('ADMIN_URL', 'admin/')  # Obfuscate admin URL

# 14. Password validation
AUTH_PASSWORD_VALIDATORS = [
    # Strong password validators
]

# 15. Session security
SESSION_COOKIE_AGE = 3600  # 1 hour
SESSION_SAVE_EVERY_REQUEST = True

# 16. File upload limits
DATA_UPLOAD_MAX_MEMORY_SIZE = 10 * 1024 * 1024  # 10 MB
FILE_UPLOAD_MAX_MEMORY_SIZE = 10 * 1024 * 1024

# 17. Database backups
# Set up automated backups

# 18. Monitoring
# Set up health checks and monitoring

# 19. Performance
# Enable database query optimization
# Set up caching strategy
# Use CDN for static files

# 20. Documentation
# Keep API documentation up to date
```

## Conclusion

This comprehensive handbook covers everything a professional Django and Django REST Framework developer needs to know:

**Core Concepts:**

- Models with advanced patterns (abstract models, managers, custom methods)
- Serializers (nested, dynamic, validation)
- Views and ViewSets (generic views, custom actions)
- Routers and URL configuration

**Advanced Features:**

- Authentication and permissions (JWT, custom permissions)
- Caching strategies
- Async operations with Celery
- File uploads and handling
- Batch operations
- WebSocket support

**Best Practices:**

- Security (XSS, SQL injection, CSRF protection)
- Performance optimization (query optimization, caching, indexing)
- Testing (unit tests, integration tests, factories)
- Deployment (Docker, CI/CD, monitoring)

**Production Ready:**

- Environment configuration
- Logging and monitoring
- Health checks
- Database backups
- Error handling

This handbook provides working code examples that you can adapt to your projects. Remember that being a professional Django developer means continuously learning, staying updated with best practices, and writing maintainable, secure, and performant code.
