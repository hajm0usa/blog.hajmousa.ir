+++
title = "Deep Dive into `perform_create()` in Django REST Framework"
date = 2025-12-17
+++

## What is `perform_create()`?

**Definition:**
`perform_create()` is a hook method in DRF's `CreateModelMixin` that handles the actual saving of a new model instance after validation passes.

**The Default Implementation:**
```python
def perform_create(self, serializer):
    serializer.save()
```

That's it! Super simple. But this simplicity is its power.


## Understanding the Create Flow

### Complete Request Flow:

```
1. POST request arrives
   ↓
2. ViewSet.create() called (from CreateModelMixin)
   ↓
3. Serializer initialized with request.data
   ↓
4. serializer.is_valid() called
   ↓
5. Validation passes
   ↓
6. perform_create(serializer) called ← YOU OVERRIDE THIS
   ↓
7. serializer.save() called (inside perform_create)
   ↓
8. Model instance created and saved
   ↓
9. Response created with serializer.data
   ↓
10. 201 Created returned
```

**Where perform_create() Fits:**
- After validation succeeds
- Before response is generated
- Perfect timing to modify the save operation


## Why Override `perform_create()` Instead of `create()`?

### Comparison:

**Override `perform_create()`:**
- ✅ Simple, focused on save logic only
- ✅ DRF handles response automatically
- ✅ Less code to write
- ✅ Follows DRF conventions
- ✅ Easier to maintain
- ✅ Can still access request via self.request
- ❌ Can't customize response format
- ❌ Can't change status code easily

**Override `create()`:**
- ✅ Full control over everything
- ✅ Custom response format
- ✅ Custom status codes
- ✅ Complex multi-step logic
- ❌ More code to write
- ❌ Must handle response yourself
- ❌ Can break if not careful

**Rule of Thumb:**
- Need to modify what's saved? → `perform_create()`
- Need to modify what's returned? → `create()`


## Common Use Cases for `perform_create()`

### 1. Auto-Setting User from Request

**Most Common Pattern:**

```python
class OrderViewSet(viewsets.ModelViewSet):
    serializer_class = OrderSerializer
    
    def perform_create(self, serializer):
        serializer.save(user=self.request.user)
```

**Why This Works:**
- User not in POST data
- User comes from authentication
- Serializer doesn't need user in fields
- Auto-assigned securely

**Serializer Setup:**
```python
class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model = Order
        fields = ['shipping_address', 'payment_method']
        # User not included - added automatically!
```

**Alternative (Don't Do This):**
```python
# Bad: Trusting user_id from request
class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ['user', 'shipping_address']  # Security risk!

# User could send: {"user": 999, "shipping_address": "..."}
# Now they're creating orders for other users!
```


### 2. Setting Request-Related Data

```python
def perform_create(self, serializer):
    serializer.save(
        user=self.request.user,
        ip_address=self.request.META.get('REMOTE_ADDR'),
        user_agent=self.request.META.get('HTTP_USER_AGENT'),
        created_from='api'
    )
```

**Use Cases:**
- Fraud detection (IP, user agent)
- Analytics (where order came from)
- Audit trails
- Multi-tenancy (tenant from request)


### 3. Auto-Generating Values

```python
def perform_create(self, serializer):
    order_number = self.generate_order_number()
    serializer.save(
        order_number=order_number,
        status='pending'
    )

def generate_order_number(self):
    from django.utils import timezone
    year = timezone.now().year
    count = Order.objects.filter(created_at__year=year).count() + 1
    return f"ORD-{year}-{count:06d}"
```

**When to Use:**
- Sequential numbering
- Custom ID generation
- Computed initial values
- Business logic defaults

**Alternative: Model save()**
```python
# If logic should apply everywhere (API, admin, shell)
class Order(models.Model):
    def save(self, *args, **kwargs):
        if not self.order_number:
            self.order_number = generate_order_number()
        super().save(*args, **kwargs)
```

**Choose perform_create() when:**
- Logic is API-specific
- Depends on request data
- Different for different endpoints

**Choose Model save() when:**
- Logic is universal
- Should apply everywhere
- Core business rule


### 4. Setting Computed Values

```python
def perform_create(self, serializer):
    # Calculate totals before saving
    items_data = self.request.data.get('items', [])
    subtotal = sum(item['price'] * item['quantity'] for item in items_data)
    tax = subtotal * Decimal('0.10')
    total = subtotal + tax
    
    serializer.save(
        subtotal=subtotal,
        tax=tax,
        total=total
    )
```

**Use Cases:**
- Price calculations
- Aggregations
- Derived values
- Business logic computations


### 5. Setting Relationships

```python
def perform_create(self, serializer):
    # Get related object
    cart = self.request.user.cart
    
    serializer.save(
        user=self.request.user,
        cart=cart,
        item_count=cart.items.count()
    )
```

**Use Cases:**
- Related object from request
- Parent-child relationships
- Association with user's data


### 6. Conditional Logic Based on Request

```python
def perform_create(self, serializer):
    # Different behavior based on user type
    if self.request.user.is_premium:
        discount = Decimal('0.15')  # 15% for premium
    else:
        discount = Decimal('0.05')  # 5% for regular
    
    serializer.save(
        user=self.request.user,
        discount_applied=discount
    )
```

**Use Cases:**
- User tier logic
- Feature flags
- A/B testing
- Permission-based defaults


### 7. Setting Workflow State

```python
def perform_create(self, serializer):
    # Orders start as draft for enterprise customers
    if self.request.user.is_enterprise:
        initial_status = 'draft'
    else:
        initial_status = 'pending'
    
    serializer.save(
        user=self.request.user,
        status=initial_status,
        requires_approval=self.request.user.is_enterprise
    )
```


### 8. Multi-Tenancy

```python
def perform_create(self, serializer):
    # Get tenant from request (subdomain, header, etc.)
    tenant = self.request.tenant  # From custom middleware
    
    serializer.save(
        tenant=tenant,
        created_by=self.request.user
    )
```

**Use Cases:**
- SaaS applications
- Organization isolation
- Data segregation


## Advanced Patterns

### Pattern 1: Validation Before Save

```python
def perform_create(self, serializer):
    # Additional validation not in serializer
    user = self.request.user
    
    # Check user limits
    if user.orders.filter(status='pending').count() >= 5:
        raise ValidationError("Maximum 5 pending orders allowed")
    
    # Check cart
    cart = user.cart
    if cart.is_empty:
        raise ValidationError("Cart is empty")
    
    # Validate stock
    for item in cart.items.all():
        if item.quantity > item.book.stock:
            raise ValidationError(f"Insufficient stock for {item.book.title}")
    
    # All good, proceed
    serializer.save(user=user)
```

**When to Use:**
- Business rules beyond data validation
- Cross-model validation
- Runtime checks (stock, limits)
- State-dependent validation

**Alternative: Serializer validate()**
```python
# Use serializer.validate() for data-centric validation
# Use perform_create() for request-dependent validation
```


### Pattern 2: Transaction Wrapper

```python
from django.db import transaction

def perform_create(self, serializer):
    with transaction.atomic():
        # Create main object
        order = serializer.save(user=self.request.user)
        
        # Related operations
        cart = self.request.user.cart
        
        # Clone cart items
        for cart_item in cart.items.all():
            OrderItem.objects.create(
                order=order,
                book=cart_item.book,
                quantity=cart_item.quantity,
                price=cart_item.book.price
            )
        
        # Update stock
        for cart_item in cart.items.all():
            book = cart_item.book
            book.stock -= cart_item.quantity
            book.save()
        
        # Clear cart
        cart.clear()
```

**Critical for:**
- Multi-model operations
- Stock management
- Financial transactions
- Data consistency

**Important:**
```python
# serializer.save() returns the instance!
instance = serializer.save(user=self.request.user)
# You can use it for further operations
```


### Pattern 3: Trigger Side Effects

```python
def perform_create(self, serializer):
    # Save instance
    order = serializer.save(user=self.request.user)
    
    # Trigger async tasks (Celery)
    from .tasks import (
        send_order_confirmation,
        notify_warehouse,
        update_analytics
    )
    
    send_order_confirmation.delay(order.id)
    notify_warehouse.delay(order.id)
    update_analytics.delay(order.id)
    
    # Note: Instance already saved, tasks just use the ID
```

**Best Practices:**
- Save first, trigger tasks after
- Use instance ID in tasks (not whole object)
- Tasks should be idempotent
- Handle task failures gracefully


### Pattern 4: Logging and Auditing

```python
import logging

logger = logging.getLogger(__name__)

def perform_create(self, serializer):
    user = self.request.user
    
    # Log before creation
    logger.info(f"User {user.id} creating order", extra={
        'user_id': user.id,
        'ip': self.request.META.get('REMOTE_ADDR'),
        'data': serializer.validated_data
    })
    
    # Create
    order = serializer.save(user=user)
    
    # Log after creation
    logger.info(f"Order {order.order_number} created successfully", extra={
        'order_id': order.id,
        'user_id': user.id,
        'total': str(order.total)
    })
    
    # Create audit record
    AuditLog.objects.create(
        user=user,
        action='order_created',
        object_type='Order',
        object_id=order.id,
        ip_address=self.request.META.get('REMOTE_ADDR')
    )
```



### Pattern 5: Cache Invalidation

```python
from django.core.cache import cache

def perform_create(self, serializer):
    user = self.request.user
    
    # Create instance
    order = serializer.save(user=user)
    
    # Invalidate related caches
    cache.delete(f'user_orders:{user.id}')
    cache.delete(f'cart:{user.id}')
    cache.delete(f'order_count:{user.id}')
    
    # Clear user's cart cache
    if hasattr(user, 'cart'):
        user.cart.clear_cache()
```



### Pattern 6: Different Behavior for Different Actions

```python
def perform_create(self, serializer):
    """Handle regular creation"""
    serializer.save(
        user=self.request.user,
        status='pending'
    )

@action(detail=False, methods=['post'])
def create_draft(self, request):
    """Custom action for draft orders"""
    serializer = self.get_serializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    self.perform_create_draft(serializer)
    return Response(serializer.data, status=201)

def perform_create_draft(self, serializer):
    """Different logic for drafts"""
    serializer.save(
        user=self.request.user,
        status='draft',
        is_draft=True
    )
```



## Accessing Request Data

### Available in perform_create():

```python
def perform_create(self, serializer):
    # User
    user = self.request.user
    
    # Request method
    method = self.request.method  # Always 'POST' for create
    
    # Headers
    auth_header = self.request.META.get('HTTP_AUTHORIZATION')
    content_type = self.request.META.get('CONTENT_TYPE')
    
    # IP address
    ip = self.request.META.get('REMOTE_ADDR')
    
    # User agent
    user_agent = self.request.META.get('HTTP_USER_AGENT')
    
    # Query params
    source = self.request.query_params.get('source')
    
    # Custom request attributes (from middleware)
    tenant = getattr(self.request, 'tenant', None)
    
    serializer.save(
        user=user,
        ip_address=ip,
        source=source
    )
```



## Working with serializer.save()

### Understanding serializer.save()

**What it does:**
1. Calls `serializer.create(validated_data)` or `serializer.update(instance, validated_data)`
2. Returns the created/updated instance
3. Passes any kwargs to create/update method

**Passing Extra Data:**

```python
def perform_create(self, serializer):
    # These become part of validated_data in serializer.create()
    instance = serializer.save(
        user=self.request.user,
        created_by=self.request.user,
        tenant_id=5
    )
    
    # instance is the created Order object
    print(instance.id)  # Available immediately
```

**In Serializer:**

```python
class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model = Order
        fields = ['shipping_address', 'payment_method']
        # user, created_by, tenant_id not in fields!
    
    def create(self, validated_data):
        # validated_data now includes:
        # - shipping_address (from request)
        # - payment_method (from request)
        # - user (from perform_create)
        # - created_by (from perform_create)
        # - tenant_id (from perform_create)
        
        return Order.objects.create(**validated_data)
```



## Error Handling

### Raising Exceptions

```python
from rest_framework.exceptions import ValidationError, PermissionDenied

def perform_create(self, serializer):
    user = self.request.user
    
    # Check permissions
    if not user.can_create_orders:
        raise PermissionDenied("You don't have permission to create orders")
    
    # Check limits
    if user.orders.filter(status='pending').count() >= 10:
        raise ValidationError("Maximum pending orders reached")
    
    # Check cart
    cart = user.cart
    if not cart or cart.is_empty:
        raise ValidationError({
            'cart': 'Cart is empty or not found'
        })
    
    # Check stock
    out_of_stock = []
    for item in cart.items.all():
        if item.quantity > item.book.stock:
            out_of_stock.append(item.book.title)
    
    if out_of_stock:
        raise ValidationError({
            'stock': f"Out of stock: {', '.join(out_of_stock)}"
        })
    
    # All good
    serializer.save(user=user)
```

**Exception Types:**
- `ValidationError` → 400 Bad Request
- `PermissionDenied` → 403 Forbidden
- `NotFound` → 404 Not Found
- `AuthenticationFailed` → 401 Unauthorized



### Try-Except Pattern

```python
def perform_create(self, serializer):
    try:
        with transaction.atomic():
            order = serializer.save(user=self.request.user)
            
            # Complex operations
            self.clone_cart_to_order(order)
            self.reserve_stock(order)
            self.send_notifications(order)
            
    except InsufficientStockError as e:
        raise ValidationError(f"Stock error: {str(e)}")
    
    except PaymentError as e:
        raise ValidationError(f"Payment error: {str(e)}")
    
    except Exception as e:
        logger.error(f"Order creation failed: {str(e)}")
        raise ValidationError("Order creation failed. Please try again.")
```



## Testing perform_create()

### Unit Test Example

```python
from django.test import TestCase
from rest_framework.test import APIRequestFactory
from rest_framework.request import Request

class OrderViewSetTest(TestCase):
    def setUp(self):
        self.factory = APIRequestFactory()
        self.user = User.objects.create_user('test', 'test@test.com')
        self.viewset = OrderViewSet()
    
    def test_perform_create_sets_user(self):
        """Test that perform_create auto-sets user"""
        # Create request
        request = self.factory.post('/orders/')
        request.user = self.user
        
        # Setup viewset
        self.viewset.request = Request(request)
        
        # Create serializer
        data = {'shipping_address': '123 Main St'}
        serializer = OrderSerializer(data=data)
        serializer.is_valid()
        
        # Call perform_create
        self.viewset.perform_create(serializer)
        
        # Check result
        order = Order.objects.get()
        self.assertEqual(order.user, self.user)
        self.assertEqual(order.shipping_address, '123 Main St')
```

### Integration Test

```python
from rest_framework.test import APITestCase

class OrderCreationTest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user('test', 'test@test.com')
        self.client.force_authenticate(user=self.user)
    
    def test_create_order_with_user(self):
        """Test order creation via API"""
        data = {
            'shipping_address': '123 Main St',
            'payment_method': 'credit_card'
        }
        
        response = self.client.post('/api/orders/', data)
        
        self.assertEqual(response.status_code, 201)
        self.assertEqual(response.data['user'], self.user.id)
        
        # Verify database
        order = Order.objects.get()
        self.assertEqual(order.user, self.user)
```



## Common Mistakes

### ❌ Mistake 1: Modifying Validated Data Directly

```python
# Wrong!
def perform_create(self, serializer):
    serializer.validated_data['user'] = self.request.user  # Don't do this!
    serializer.save()
```

**Why wrong:**
- `validated_data` might be immutable
- Can cause unexpected behavior
- Not the DRF way

**Correct:**
```python
def perform_create(self, serializer):
    serializer.save(user=self.request.user)  # Pass as kwarg
```



### ❌ Mistake 2: Not Using Transactions

```python
# Wrong! No transaction
def perform_create(self, serializer):
    order = serializer.save(user=self.request.user)
    
    # If this fails, order is already created!
    self.clone_cart_items(order)
```

**Correct:**
```python
def perform_create(self, serializer):
    with transaction.atomic():
        order = serializer.save(user=self.request.user)
        self.clone_cart_items(order)
```



### ❌ Mistake 3: Heavy Processing in perform_create

```python
# Wrong! Blocks request
def perform_create(self, serializer):
    order = serializer.save(user=self.request.user)
    
    # Generate PDF (takes 5 seconds)
    generate_invoice_pdf(order)
    
    # Send 100 emails (takes 30 seconds)
    notify_all_admins(order)
```

**Correct:**
```python
def perform_create(self, serializer):
    order = serializer.save(user=self.request.user)
    
    # Queue async tasks
    generate_invoice_pdf.delay(order.id)
    notify_all_admins.delay(order.id)
```



### ❌ Mistake 4: Forgetting to Return or Not Caring About Return

```python
# perform_create doesn't return anything!
def perform_create(self, serializer):
    instance = serializer.save(user=self.request.user)
    return instance  # This does nothing!
```

**Why:**
- `perform_create()` return value is ignored by DRF
- Response is built from `serializer.data`
- If you need custom response, override `create()` instead



### ❌ Mistake 5: Not Handling Exceptions

```python
# Wrong! Unhandled exceptions
def perform_create(self, serializer):
    cart = self.request.user.cart  # What if cart doesn't exist?
    order = serializer.save(user=self.request.user)
    self.clone_cart(cart, order)  # What if this fails?
```

**Correct:**
```python
def perform_create(self, serializer):
    try:
        cart = self.request.user.cart
    except Cart.DoesNotExist:
        raise ValidationError("Cart not found")
    
    if cart.is_empty:
        raise ValidationError("Cart is empty")
    
    try:
        with transaction.atomic():
            order = serializer.save(user=self.request.user)
            self.clone_cart(cart, order)
    except Exception as e:
        logger.error(f"Failed: {e}")
        raise ValidationError("Order creation failed")
```



## Best Practices Summary

### ✅ DO:

1. **Use for simple modifications**
   - Setting user from request
   - Adding computed values
   - Setting initial state

2. **Keep it focused**
   - Should be about the save operation
   - Delegate complex logic to services

3. **Use transactions for multi-step operations**
   - Wrap in `transaction.atomic()`
   - Ensure data consistency

4. **Pass extra data via serializer.save() kwargs**
   - Clean and explicit
   - Type-safe

5. **Handle exceptions appropriately**
   - Validate before saving
   - Provide clear error messages

6. **Use async tasks for heavy operations**
   - Don't block the request
   - Queue with Celery

7. **Log important actions**
   - Audit trail
   - Debugging

### ❌ DON'T:

1. **Don't modify validated_data directly**
   - Use serializer.save() kwargs

2. **Don't put complex business logic here**
   - Move to service layer

3. **Don't forget transactions**
   - Multi-step operations need atomicity

4. **Don't block with heavy operations**
   - Use background tasks

5. **Don't try to customize response here**
   - Override `create()` instead

6. **Don't ignore errors**
   - Handle and report properly


## When to Move Beyond perform_create()

**Move to `create()` override when:**
- Need custom response format
- Multiple serializers involved
- Complex validation before creation
- Custom status codes
- Need to return different data

**Move to Service Layer when:**
- Logic is complex (> 20 lines)
- Logic is reusable (used in multiple places)
- Multiple models involved
- Complex business rules
- Need thorough testing

**Example:**
```python
# Simple: Use perform_create()
def perform_create(self, serializer):
    serializer.save(user=self.request.user)

# Medium: Still perform_create() with helper
def perform_create(self, serializer):
    with transaction.atomic():
        order = serializer.save(user=self.request.user)
        self.process_order(order)

# Complex: Move to service + override create()
def create(self, request):
    serializer = self.get_serializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    
    order = OrderService.create_from_cart(
        user=request.user,
        data=serializer.validated_data
    )
    
    response_serializer = OrderDetailSerializer(order)
    return Response(response_serializer.data, status=201)
```


`perform_create()` is simple but powerful. Master it for 90% of your creation needs, and know when to move beyond it for the other 10%!
