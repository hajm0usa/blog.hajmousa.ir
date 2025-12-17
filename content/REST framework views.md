+++
title = "The Complete Guide to Django REST Framework Views"
date = 2025-12-17
+++

## Introduction

Django REST Framework (DRF) provides multiple approaches to building API views, each offering different levels of abstraction and flexibility. Understanding when and how to use each approach is crucial for building maintainable, scalable APIs.

### The View Hierarchy

```
Function-Based Views (@api_view)
    ↓
APIView (Base class)
    ↓
GenericAPIView (Adds queryset/serializer support)
    ↓
Mixins + GenericAPIView
    ↓
Concrete Generic Views (ListAPIView, etc.)
    ↓
ViewSets (ModelViewSet, etc.)
```


## Function-Based Views

Function-Based Views (FBVs) are the simplest way to create API endpoints. They use the `@api_view` decorator to transform regular Django views into DRF views.

### Basic Example

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def product_list(request):
    """
    List all products or create a new product.
    """
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
    """
    Retrieve, update or delete a product.
    """
    try:
        product = Product.objects.get(pk=pk)
    except Product.DoesNotExist:
        return Response(
            {'error': 'Product not found'}, 
            status=status.HTTP_404_NOT_FOUND
        )
    
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

### Advanced FBV with Query Parameters

```python
@api_view(['GET'])
def search_products(request):
    """
    Search products with filters.
    """
    queryset = Product.objects.all()
    
    # Filter by category
    category = request.query_params.get('category', None)
    if category:
        queryset = queryset.filter(category__slug=category)
    
    # Filter by price range
    min_price = request.query_params.get('min_price', None)
    max_price = request.query_params.get('max_price', None)
    if min_price:
        queryset = queryset.filter(price__gte=min_price)
    if max_price:
        queryset = queryset.filter(price__lte=max_price)
    
    # Search by name
    search = request.query_params.get('search', None)
    if search:
        queryset = queryset.filter(name__icontains=search)
    
    # Pagination
    from rest_framework.pagination import PageNumberPagination
    paginator = PageNumberPagination()
    paginator.page_size = 20
    result_page = paginator.paginate_queryset(queryset, request)
    
    serializer = ProductSerializer(result_page, many=True)
    return paginator.get_paginated_response(serializer.data)
```

### When to Use FBVs

**Pros:**

- Simple and explicit
- Easy to understand for beginners
- Great for one-off endpoints
- Full control over logic
- Easy to test

**Cons:**

- More boilerplate code
- Repetitive for standard CRUD operations
- Harder to share code between views

**Best for:**

- Custom business logic endpoints
- Webhooks
- Complex data processing
- One-off integrations

---

## Class-Based Views (APIView)

APIView is the base class for all DRF views. It provides request parsing, authentication, permissions, and throttling.

### Basic APIView Example

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import IsAuthenticated

class ProductList(APIView):
    """
    List all products or create a new product.
    """
    permission_classes = [IsAuthenticated]
    
    def get(self, request, format=None):
        products = Product.objects.all()
        serializer = ProductSerializer(products, many=True)
        return Response(serializer.data)
    
    def post(self, request, format=None):
        serializer = ProductSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class ProductDetail(APIView):
    """
    Retrieve, update or delete a product.
    """
    
    def get_object(self, pk):
        try:
            return Product.objects.get(pk=pk)
        except Product.DoesNotExist:
            return None
    
    def get(self, request, pk, format=None):
        product = self.get_object(pk)
        if not product:
            return Response(
                {'error': 'Product not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )
        serializer = ProductSerializer(product)
        return Response(serializer.data)
    
    def put(self, request, pk, format=None):
        product = self.get_object(pk)
        if not product:
            return Response(
                {'error': 'Product not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )
        serializer = ProductSerializer(product, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request, pk, format=None):
        product = self.get_object(pk)
        if not product:
            return Response(
                {'error': 'Product not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )
        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Advanced APIView with Custom Logic

```python
from django.db import transaction
from rest_framework.exceptions import ValidationError

class CheckoutView(APIView):
    """
    Handle complex checkout process.
    """
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        user = request.user
        cart_items = request.data.get('cart_items', [])
        shipping_address = request.data.get('shipping_address')
        payment_method = request.data.get('payment_method')
        
        # Validate cart
        if not cart_items:
            raise ValidationError({'cart_items': 'Cart is empty'})
        
        # Validate address
        if not shipping_address:
            raise ValidationError({'shipping_address': 'Shipping address is required'})
        
        try:
            with transaction.atomic():
                # Create order
                order = Order.objects.create(
                    user=user,
                    shipping_address=shipping_address,
                    status='pending'
                )
                
                # Create order items and calculate total
                total = 0
                for item in cart_items:
                    product = Product.objects.select_for_update().get(
                        id=item['product_id']
                    )
                    
                    # Check stock
                    if product.stock < item['quantity']:
                        raise ValidationError({
                            'stock': f'Insufficient stock for {product.name}'
                        })
                    
                    # Update stock
                    product.stock -= item['quantity']
                    product.save()
                    
                    # Create order item
                    order_item = OrderItem.objects.create(
                        order=order,
                        product=product,
                        quantity=item['quantity'],
                        price=product.price
                    )
                    
                    total += product.price * item['quantity']
                
                order.total = total
                order.save()
                
                # Process payment (simplified)
                payment = self.process_payment(order, payment_method)
                
                if payment['success']:
                    order.status = 'confirmed'
                    order.save()
                    
                    # Clear user's cart
                    Cart.objects.filter(user=user).delete()
                    
                    # Send confirmation email (async recommended)
                    # send_order_confirmation_email.delay(order.id)
                    
                    serializer = OrderSerializer(order)
                    return Response({
                        'message': 'Order created successfully',
                        'order': serializer.data
                    }, status=status.HTTP_201_CREATED)
                else:
                    raise ValidationError({'payment': payment['error']})
        
        except Product.DoesNotExist:
            return Response(
                {'error': 'Product not found'},
                status=status.HTTP_404_NOT_FOUND
            )
        except Exception as e:
            return Response(
                {'error': str(e)},
                status=status.HTTP_400_BAD_REQUEST
            )
    
    def process_payment(self, order, payment_method):
        """
        Process payment through payment gateway.
        This is a simplified example.
        """
        # Integration with Stripe, PayPal, etc.
        return {'success': True}
```

### When to Use APIView

**Pros:**

- More organized than FBVs
- Better code reusability with class methods
- Easier to add authentication/permissions at class level
- Good for complex logic

**Cons:**

- Still requires writing standard CRUD logic
- More verbose than generic views
- Need to handle common patterns manually

**Best for:**

- Complex business logic (checkout, payments)
- Multi-step processes
- Custom validation logic
- Endpoints that don't follow REST patterns


## Generic Views

Generic views provide pre-built functionality for common patterns. They combine GenericAPIView with mixins.

### Available Generic Views

```python
from rest_framework import generics

# List views
generics.ListAPIView              # GET (list)
generics.CreateAPIView            # POST
generics.ListCreateAPIView        # GET (list) + POST

# Detail views
generics.RetrieveAPIView          # GET (detail)
generics.UpdateAPIView            # PUT/PATCH
generics.DestroyAPIView           # DELETE
generics.RetrieveUpdateAPIView    # GET + PUT/PATCH
generics.RetrieveDestroyAPIView   # GET + DELETE
generics.RetrieveUpdateDestroyAPIView  # GET + PUT/PATCH + DELETE
```

### Basic Generic View Example

```python
from rest_framework import generics
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter

class ProductList(generics.ListCreateAPIView):
    """
    List all products or create a new product.
    """
    queryset = Product.objects.filter(is_active=True)
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    
    # Filtering
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['category', 'brand', 'is_featured']
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at', 'name']
    ordering = ['-created_at']
    
    def perform_create(self, serializer):
        """
        Custom logic when creating a product.
        """
        serializer.save(created_by=self.request.user)


class ProductDetail(generics.RetrieveUpdateDestroyAPIView):
    """
    Retrieve, update or delete a product.
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    
    def perform_update(self, serializer):
        """
        Custom logic when updating a product.
        """
        serializer.save(updated_by=self.request.user)
```

### Advanced Generic View with Custom Queryset

```python
class UserOrderList(generics.ListAPIView):
    """
    List orders for the authenticated user.
    """
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend, OrderingFilter]
    filterset_fields = ['status', 'created_at']
    ordering_fields = ['created_at', 'total']
    
    def get_queryset(self):
        """
        Filter orders by current user.
        """
        return Order.objects.filter(user=self.request.user).select_related(
            'user'
        ).prefetch_related(
            'items__product'
        )


class FeaturedProductList(generics.ListAPIView):
    """
    List featured products only.
    """
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]
    pagination_class = None  # Disable pagination for featured products
    
    def get_queryset(self):
        return Product.objects.filter(
            is_featured=True,
            is_active=True,
            stock__gt=0
        ).order_by('-created_at')[:10]
```

### Multiple Serializers in Generic Views

```python
class ProductDetail(generics.RetrieveUpdateDestroyAPIView):
    """
    Use different serializers for different actions.
    """
    queryset = Product.objects.all()
    
    def get_serializer_class(self):
        if self.request.method == 'GET':
            return ProductDetailSerializer  # More detailed
        return ProductSerializer  # Less detailed for updates
    
    def get_permissions(self):
        if self.request.method == 'GET':
            return [AllowAny()]
        return [IsAuthenticated(), IsAdminUser()]
```

### When to Use Generic Views

**Pros:**

- Much less code than APIView
- Built-in pagination, filtering, searching
- Easy to customize with hooks
- Standard REST patterns

**Cons:**

- Less flexible than APIView
- Can be confusing for complex customizations
- Need to understand the mixin system

**Best for:**

- Standard CRUD operations
- List/detail views
- Resource-based APIs
- When you need filtering/searching/pagination


## ViewSets and Routers

ViewSets combine the logic for multiple related views into a single class. They work with Routers for automatic URL configuration.

### ModelViewSet

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response

class ProductViewSet(viewsets.ModelViewSet):
    """
    A viewset for viewing and editing product instances.
    Provides: list, create, retrieve, update, partial_update, destroy
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['category', 'brand']
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at']
    
    def get_queryset(self):
        """
        Optionally filter by query parameters.
        """
        queryset = Product.objects.filter(is_active=True)
        
        # Filter by price range
        min_price = self.request.query_params.get('min_price')
        max_price = self.request.query_params.get('max_price')
        
        if min_price:
            queryset = queryset.filter(price__gte=min_price)
        if max_price:
            queryset = queryset.filter(price__lte=max_price)
        
        return queryset.select_related('category', 'brand')
    
    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)
    
    @action(detail=True, methods=['post'])
    def add_to_cart(self, request, pk=None):
        """
        Custom action to add product to cart.
        URL: /products/{id}/add_to_cart/
        """
        product = self.get_object()
        quantity = request.data.get('quantity', 1)
        
        cart_item, created = CartItem.objects.get_or_create(
            user=request.user,
            product=product,
            defaults={'quantity': quantity}
        )
        
        if not created:
            cart_item.quantity += quantity
            cart_item.save()
        
        return Response({
            'message': 'Product added to cart',
            'quantity': cart_item.quantity
        })
    
    @action(detail=True, methods=['post'])
    def add_review(self, request, pk=None):
        """
        Add a review for a product.
        URL: /products/{id}/add_review/
        """
        product = self.get_object()
        
        # Check if user has purchased this product
        has_purchased = OrderItem.objects.filter(
            order__user=request.user,
            product=product,
            order__status='delivered'
        ).exists()
        
        if not has_purchased:
            return Response(
                {'error': 'You must purchase this product to review it'},
                status=status.HTTP_403_FORBIDDEN
            )
        
        serializer = ReviewSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(user=request.user, product=product)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['get'])
    def featured(self, request):
        """
        List featured products.
        URL: /products/featured/
        """
        featured = self.queryset.filter(is_featured=True)[:10]
        serializer = self.get_serializer(featured, many=True)
        return Response(serializer.data)
```

### ReadOnlyModelViewSet

```python
class CategoryViewSet(viewsets.ReadOnlyModelViewSet):
    """
    A viewset that provides only 'list' and 'retrieve' actions.
    """
    queryset = Category.objects.filter(is_active=True)
    serializer_class = CategorySerializer
    permission_classes = [AllowAny]
    
    @action(detail=True, methods=['get'])
    def products(self, request, pk=None):
        """
        Get all products in this category.
        URL: /categories/{id}/products/
        """
        category = self.get_object()
        products = Product.objects.filter(category=category, is_active=True)
        
        # Apply pagination
        page = self.paginate_queryset(products)
        if page is not None:
            serializer = ProductSerializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        
        serializer = ProductSerializer(products, many=True)
        return Response(serializer.data)
```

### Custom ViewSet

```python
class OrderViewSet(viewsets.ModelViewSet):
    """
    ViewSet for managing orders.
    """
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend, OrderingFilter]
    filterset_fields = ['status']
    ordering = ['-created_at']
    
    def get_queryset(self):
        """
        Users can only see their own orders.
        """
        if self.request.user.is_staff:
            return Order.objects.all()
        return Order.objects.filter(user=self.request.user)
    
    def create(self, request):
        """
        Override create to handle order creation logic.
        """
        # Use the checkout logic from earlier
        return Response({'message': 'Use /checkout/ endpoint'})
    
    def update(self, request, pk=None):
        """
        Prevent updating completed orders.
        """
        order = self.get_object()
        if order.status in ['completed', 'cancelled']:
            return Response(
                {'error': 'Cannot update completed or cancelled orders'},
                status=status.HTTP_400_BAD_REQUEST
            )
        return super().update(request, pk)
    
    @action(detail=True, methods=['post'])
    def cancel(self, request, pk=None):
        """
        Cancel an order.
        URL: /orders/{id}/cancel/
        """
        order = self.get_object()
        
        if order.status not in ['pending', 'confirmed']:
            return Response(
                {'error': 'Can only cancel pending or confirmed orders'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        with transaction.atomic():
            # Restore stock
            for item in order.items.all():
                product = item.product
                product.stock += item.quantity
                product.save()
            
            # Update order status
            order.status = 'cancelled'
            order.save()
        
        return Response({'message': 'Order cancelled successfully'})
    
    @action(detail=False, methods=['get'])
    def statistics(self, request):
        """
        Get order statistics for the user.
        URL: /orders/statistics/
        """
        orders = self.get_queryset()
        
        stats = {
            'total_orders': orders.count(),
            'pending': orders.filter(status='pending').count(),
            'completed': orders.filter(status='completed').count(),
            'cancelled': orders.filter(status='cancelled').count(),
            'total_spent': orders.filter(
                status='completed'
            ).aggregate(total=Sum('total'))['total'] or 0
        }
        
        return Response(stats)
```

### Router Configuration

```python
# urls.py
from rest_framework.routers import DefaultRouter
from django.urls import path, include

router = DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')
router.register(r'categories', CategoryViewSet, basename='category')
router.register(r'orders', OrderViewSet, basename='order')

urlpatterns = [
    path('api/', include(router.urls)),
]

# This automatically creates:
# GET    /api/products/              - list
# POST   /api/products/              - create
# GET    /api/products/{id}/         - retrieve
# PUT    /api/products/{id}/         - update
# PATCH  /api/products/{id}/         - partial_update
# DELETE /api/products/{id}/         - destroy
# GET    /api/products/featured/     - custom action
# POST   /api/products/{id}/add_to_cart/ - custom action
```

### When to Use ViewSets

**Pros:**

- Most concise code
- Automatic URL routing
- Easy to add custom actions
- Perfect for REST APIs
- Great for standard resources

**Cons:**

- Can be harder to understand for beginners
- Less obvious what URLs are created
- Over-engineering for simple cases

**Best for:**

- Standard REST resources
- CRUD operations
- APIs with many similar resources
- When you want automatic URL routing


## Mixins

Mixins provide reusable functionality that can be combined with GenericAPIView.

### Available Mixins

```python
from rest_framework import mixins, generics

# Individual mixins
mixins.ListModelMixin        # list()
mixins.CreateModelMixin      # create()
mixins.RetrieveModelMixin    # retrieve()
mixins.UpdateModelMixin      # update(), partial_update()
mixins.DestroyModelMixin     # destroy()
```

### Creating Custom Views with Mixins

```python
class ProductList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    """
    Manually combine mixins for custom behavior.
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
    
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)


class ProductDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    generics.GenericAPIView):
    """
    Retrieve and update only (no delete).
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
    
    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)
    
    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)
```

### Custom Mixin

```python
class AuditMixin:
    """
    Custom mixin to add audit fields.
    """
    def perform_create(self, serializer):
        serializer.save(
            created_by=self.request.user,
            created_ip=self.get_client_ip()
        )
    
    def perform_update(self, serializer):
        serializer.save(
            updated_by=self.request.user,
            updated_ip=self.get_client_ip()
        )
    
    def get_client_ip(self):
        x_forwarded_for = self.request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = self.request.META.get('REMOTE_ADDR')
        return ip


class ProductViewSet(AuditMixin, viewsets.ModelViewSet):
    """
    Product viewset with audit trail.
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```


## Custom Actions

Custom actions allow you to add non-CRUD endpoints to ViewSets.

### Action Decorators

```python
from rest_framework.decorators import action

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    @action(detail=True, methods=['post'])
    def like(self, request, pk=None):
        """
        Detail action: requires a specific product ID.
        URL: /products/{id}/like/
        """
        product = self.get_object()
        product.likes += 1
        product.save()
        return Response({'likes': product.likes})
    
    @action(detail=False, methods=['get'])
    def trending(self, request):
        """
        List action: works on the entire collection.
        URL: /products/trending/
        """
        trending = self.queryset.order_by('-views')[:10]
        serializer = self.get_serializer(trending, many=True)
        return Response(serializer.data)
    
    @action(
        detail=True,
        methods=['post'],
        permission_classes=[IsAuthenticated],
        url_path='add-to-wishlist'  # Custom URL path
    )
    def add_to_wishlist(self, request, pk=None):
        """
        URL: /products/{id}/add-to-wishlist/
        """
        product = self.get_object()
        wishlist, created = Wishlist.objects.get_or_create(
            user=request.user,
            product=product
        )
        if created:
            return Response({'message': 'Added to wishlist'})
        return Response({'message': 'Already in wishlist'})
    
    @action(detail=False, methods=['post'], url_path='bulk-update')
    def bulk_update(self, request):
        """
        Update multiple products at once.
        URL: /products/bulk-update/
        """
        product_updates = request.data.get('products', [])
        
        updated_count = 0
        for update in product_updates:
            try:
                product = Product.objects.get(id=update['id'])
                for key, value in update.items():
                    if key != 'id':
                        setattr(product, key, value)
                product.save()
                updated_count += 1
            except Product.DoesNotExist:
                continue
        
        return Response({
            'message': f'Updated {updated_count} products'
        })
```


## Best Practices for E-commerce APIs

### 1. Proper Permission Classes

```python
from rest_framework.permissions import BasePermission

class IsOwnerOrReadOnly(BasePermission):
    """
    Custom permission to only allow owners to edit.
    """
    def has_object_permission(self, request, view, obj):
        if request.method in SAFE_METHODS:
            return True
        return obj.user == request.user


class IsAdminOrReadOnly(BasePermission):
    """
    Admin can edit, everyone can read.
    """
    def has_permission(self, request, view):
        if request.method in SAFE_METHODS:
            return True
        return request.user and request.user.is_staff


class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def get_permissions(self):
        if self.action in ['create', 'update', 'partial_update', 'destroy']:
            return [IsAdminOrReadOnly()]
        return [AllowAny()]
```

### 2. Pagination

```python
from rest_framework.pagination import PageNumberPagination

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100


class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    pagination_class = StandardResultsSetPagination
```

### 3. Filtering and Search

```python
from django_filters import rest_framework as filters

class ProductFilter(filters.FilterSet):
    min_price = filters.NumberFilter(field_name='price', lookup_expr='gte')
    max_price = filters.NumberFilter(field_name='
```
