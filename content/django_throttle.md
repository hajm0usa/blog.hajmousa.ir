+++
title = "Professional API Rate Limiting in Django REST Framework"
date = 2025-02-04
+++

## 🎯 Overview of Rate Limiting Strategies

Rate limiting prevents API abuse by restricting the number of requests a client can make in a given time period.

---

## 1. Django REST Framework Built-in Throttling

### Basic Implementation

**settings.py**
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',      # Anonymous users
        'user': '1000/hour',     # Authenticated users
        'burst': '60/min',       # Burst protection
        'sustained': '1000/day', # Daily limit
    }
}
```

### Built-in Throttle Classes

**1. AnonRateThrottle** - Limits anonymous requests by IP
**2. UserRateThrottle** - Limits authenticated users by user ID
**3. ScopedRateThrottle** - Limits based on scope (per-endpoint)

---

## 2. Custom Throttle Classes (Production-Ready)

### Custom User Throttle with Different Tiers

**throttles.py**
```python
from rest_framework.throttling import UserRateThrottle
from django.core.cache import cache


class PremiumUserThrottle(UserRateThrottle):
    """
    Higher rate limits for premium users.
    """
    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            # Check if user is premium
            if hasattr(request.user, 'is_premium') and request.user.is_premium:
                return None  # No throttling for premium users
        
        return super().get_cache_key(request, view)


class TieredUserThrottle(UserRateThrottle):
    """
    Different rate limits based on user tier.
    """
    def allow_request(self, request, view):
        if not request.user.is_authenticated:
            return super().allow_request(request, view)
        
        # Define rates per user tier
        user_tier = getattr(request.user, 'tier', 'free')
        
        tier_rates = {
            'free': '100/hour',
            'basic': '500/hour',
            'premium': '2000/hour',
            'enterprise': '10000/hour',
        }
        
        # Temporarily override rate
        self.rate = tier_rates.get(user_tier, '100/hour')
        self.num_requests, self.duration = self.parse_rate(self.rate)
        
        return super().allow_request(request, view)


class BurstRateThrottle(UserRateThrottle):
    """
    Prevents burst attacks - stricter short-term limits.
    """
    scope = 'burst'
    rate = '60/min'  # Max 60 requests per minute


class SustainedRateThrottle(UserRateThrottle):
    """
    Prevents sustained abuse - daily limits.
    """
    scope = 'sustained'
    rate = '1000/day'


class IPRateThrottle(UserRateThrottle):
    """
    Rate limit by IP address (useful for detecting DDoS).
    """
    def get_cache_key(self, request, view):
        return self.cache_format % {
            'scope': self.scope,
            'ident': self.get_ident(request)
        }
    
    def get_ident(self, request):
        """
        Get client IP, handling proxies correctly.
        """
        xff = request.META.get('HTTP_X_FORWARDED_FOR')
        if xff:
            return xff.split(',')[0].strip()
        
        remote_addr = request.META.get('REMOTE_ADDR')
        return remote_addr
```

---

## 3. Endpoint-Specific Rate Limiting

### Using ScopedRateThrottle

**settings.py**
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour',
        'login': '5/15min',          # Strict login limits
        'register': '3/hour',        # Prevent spam registrations
        'password_reset': '3/hour',  # Prevent abuse
        'order_create': '20/hour',   # Prevent order spam
        'payment': '10/hour',        # Strict payment limits
        'search': '200/hour',        # Higher for search
        'upload': '50/day',          # File upload limits
    }
}
```

**views.py**
```python
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.throttling import ScopedRateThrottle
from rest_framework.views import APIView


class LoginView(APIView):
    """Login with strict rate limiting."""
    throttle_classes = [ScopedRateThrottle]
    throttle_scope = 'login'
    
    def post(self, request):
        # Login logic
        pass


class RegisterView(APIView):
    """Registration with anti-spam protection."""
    throttle_classes = [ScopedRateThrottle]
    throttle_scope = 'register'
    
    def post(self, request):
        # Registration logic
        pass


class OrderViewSet(viewsets.ModelViewSet):
    """Orders with per-action throttling."""
    
    def get_throttles(self):
        """Different throttles per action."""
        if self.action == 'create':
            throttle_classes = [ScopedRateThrottle]
            return [throttle() for throttle in throttle_classes]
        return super().get_throttles()
    
    def create(self, request):
        self.throttle_scope = 'order_create'
        return super().create(request)


@api_view(['POST'])
@throttle_classes([ScopedRateThrottle])
def password_reset(request):
    """Password reset with throttling."""
    # Will use 'password_reset' scope from DEFAULT_THROTTLE_RATES
    pass
```

---

## 4. Advanced: Redis-Based Rate Limiting

### Using django-ratelimit (More Powerful)

**Installation:**
```bash
pip install django-ratelimit
```

**Usage:**
```python
from django_ratelimit.decorators import ratelimit
from django_ratelimit.exceptions import Ratelimited
from django.http import JsonResponse


# Decorator approach
@ratelimit(key='ip', rate='100/h', method='GET')
@ratelimit(key='user', rate='1000/h', method='POST')
def my_view(request):
    return JsonResponse({'status': 'ok'})


# Class-based view
from django.utils.decorators import method_decorator

class MyView(APIView):
    @method_decorator(ratelimit(key='user', rate='50/m'))
    def post(self, request):
        return Response({'status': 'ok'})


# Custom key function
def get_user_tier_key(group, request):
    """Rate limit based on user tier."""
    if not request.user.is_authenticated:
        return 'anon'
    
    tier = getattr(request.user, 'tier', 'free')
    return f'user:{request.user.id}:{tier}'


@ratelimit(key=get_user_tier_key, rate='100/h')
def tiered_view(request):
    pass


# Handle rate limit exceeded
def handler(request, exception):
    return JsonResponse({
        'error': 'Rate limit exceeded',
        'detail': 'Too many requests. Please try again later.'
    }, status=429)
```

---

## 5. Professional Implementation: Combined Approach

### Complete throttles.py

```python
"""
Professional rate limiting implementation.
"""
from rest_framework.throttling import SimpleRateThrottle, UserRateThrottle
from django.core.cache import cache
from django.conf import settings
import hashlib


class BaseThrottle(SimpleRateThrottle):
    """Base throttle with professional features."""
    
    def get_cache_key(self, request, view):
        """
        Generate cache key for throttling.
        Uses combination of: scope + identifier
        """
        if not self.allow_request(request, view):
            return None
        
        ident = self.get_ident(request)
        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }
    
    def get_ident(self, request):
        """
        Get client identifier (IP or User ID).
        Properly handles proxies and load balancers.
        """
        if request.user and request.user.is_authenticated:
            return f'user_{request.user.id}'
        
        # Get real IP behind proxy
        xff = request.META.get('HTTP_X_FORWARDED_FOR')
        if xff:
            ip = xff.split(',')[0].strip()
        else:
            ip = request.META.get('REMOTE_ADDR', '')
        
        return f'ip_{ip}'
    
    def throttle_failure(self):
        """Called when throttle limit is exceeded."""
        # Log the throttle event
        import logging
        logger = logging.getLogger('throttle')
        logger.warning(
            f'Throttle exceeded: {self.scope} - '
            f'Rate: {self.rate} - '
            f'Wait: {self.wait()}'
        )
        return False


class AnonThrottle(BaseThrottle):
    """Anonymous user throttling."""
    scope = 'anon'
    
    def get_cache_key(self, request, view):
        if request.user and request.user.is_authenticated:
            return None  # Don't throttle authenticated users here
        return super().get_cache_key(request, view)


class AuthUserThrottle(BaseThrottle):
    """Authenticated user throttling."""
    scope = 'user'
    
    def get_cache_key(self, request, view):
        if not request.user or not request.user.is_authenticated:
            return None  # Handled by AnonThrottle
        return super().get_cache_key(request, view)


class BurstThrottle(BaseThrottle):
    """
    Burst protection - prevents rapid fire requests.
    Applied to all users.
    """
    scope = 'burst'
    rate = '60/min'


class AdminExemptThrottle(UserRateThrottle):
    """
    Throttle that exempts admin users.
    """
    def allow_request(self, request, view):
        if request.user and request.user.is_staff:
            return True  # No throttling for admin
        return super().allow_request(request, view)


class TierBasedThrottle(BaseThrottle):
    """
    Rate limiting based on user subscription tier.
    """
    TIER_RATES = {
        'free': ('100', 'hour'),
        'basic': ('500', 'hour'),
        'pro': ('2000', 'hour'),
        'enterprise': ('10000', 'hour'),
    }
    
    def __init__(self):
        super().__init__()
        self.scope = 'tiered'
    
    def get_rate(self):
        """Get rate based on user tier."""
        # This will be set per request
        if not hasattr(self, 'tier_rate'):
            return '100/hour'  # Default
        return self.tier_rate
    
    def allow_request(self, request, view):
        # Determine user tier
        if not request.user or not request.user.is_authenticated:
            tier = 'free'
        else:
            tier = getattr(request.user, 'subscription_tier', 'free')
        
        # Set rate for this tier
        rate_num, rate_period = self.TIER_RATES.get(tier, ('100', 'hour'))
        self.tier_rate = f'{rate_num}/{rate_period}'
        self.rate = self.tier_rate
        self.num_requests, self.duration = self.parse_rate(self.rate)
        
        return super().allow_request(request, view)


class EndpointSpecificThrottle(BaseThrottle):
    """
    Different rates for different endpoints.
    """
    ENDPOINT_RATES = {
        'login': '5/15min',
        'register': '3/hour',
        'password_reset': '3/hour',
        'order_create': '20/hour',
        'payment': '10/hour',
        'search': '200/hour',
        'list': '100/hour',
        'retrieve': '200/hour',
    }
    
    def __init__(self):
        super().__init__()
    
    def allow_request(self, request, view):
        # Determine endpoint from view
        view_name = getattr(view, 'throttle_scope', None)
        if not view_name:
            # Fallback to action or basename
            view_name = getattr(view, 'action', 'default')
        
        # Get rate for this endpoint
        self.rate = self.ENDPOINT_RATES.get(view_name, '100/hour')
        self.scope = view_name
        self.num_requests, self.duration = self.parse_rate(self.rate)
        
        return super().allow_request(request, view)


class PerAPIKeyThrottle(BaseThrottle):
    """
    Rate limiting per API key (for integrations).
    """
    scope = 'api_key'
    
    def get_ident(self, request):
        """Use API key as identifier."""
        api_key = request.META.get('HTTP_X_API_KEY', '')
        if not api_key:
            api_key = request.query_params.get('api_key', '')
        
        if api_key:
            # Hash the API key for privacy
            return hashlib.sha256(api_key.encode()).hexdigest()[:16]
        
        # Fallback to IP
        return super().get_ident(request)


class DynamicRateThrottle(BaseThrottle):
    """
    Dynamically adjust rate limits based on server load.
    """
    scope = 'dynamic'
    
    def get_rate(self):
        """Adjust rate based on current server load."""
        # Check server metrics (CPU, memory, etc.)
        # This is a simplified example
        
        server_load = cache.get('server_load', 50)  # 0-100
        
        if server_load > 80:
            return '50/hour'  # Reduce load
        elif server_load > 60:
            return '100/hour'
        else:
            return '500/hour'  # Normal load
    
    def allow_request(self, request, view):
        self.rate = self.get_rate()
        self.num_requests, self.duration = self.parse_rate(self.rate)
        return super().allow_request(request, view)
```

---

## 6. Custom Response for Rate Limit Exceeded

### Custom Exception Handler

**exception_handlers.py**
```python
from rest_framework.views import exception_handler
from rest_framework.exceptions import Throttled
from rest_framework.response import Response
from datetime import datetime, timedelta


def custom_exception_handler(exc, context):
    """
    Custom exception handler for rate limiting.
    Returns helpful information when throttled.
    """
    response = exception_handler(exc, context)
    
    if isinstance(exc, Throttled):
        # Calculate retry time
        wait = exc.wait
        retry_after = datetime.now() + timedelta(seconds=wait)
        
        custom_response = {
            'error': 'rate_limit_exceeded',
            'message': 'Too many requests. Please slow down.',
            'details': {
                'throttle_type': exc.detail if hasattr(exc, 'detail') else 'general',
                'retry_after_seconds': int(wait),
                'retry_after_timestamp': retry_after.isoformat(),
                'retry_after_human': f'{int(wait // 60)} minutes' if wait > 60 else f'{int(wait)} seconds'
            }
        }
        
        response = Response(custom_response, status=429)
        
        # Add Retry-After header (standard)
        response['Retry-After'] = str(int(wait))
        
        # Add custom headers
        response['X-RateLimit-Limit'] = getattr(exc, 'num_requests', 'unknown')
        response['X-RateLimit-Remaining'] = 0
        response['X-RateLimit-Reset'] = retry_after.timestamp()
    
    return response
```

**settings.py**
```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'path.to.custom_exception_handler',
}
```

---

## 7. Rate Limit Headers (Professional)

### Add Rate Limit Info to All Responses

**middleware.py**
```python
from django.utils.deprecation import MiddlewareMixin
from django.core.cache import cache
from rest_framework.throttling import SimpleRateThrottle


class RateLimitHeadersMiddleware(MiddlewareMixin):
    """
    Add rate limit headers to all API responses.
    Following GitHub API conventions.
    """
    
    def process_response(self, request, response):
        # Only for API requests
        if not request.path.startswith('/api/'):
            return response
        
        # Get throttle instance (simplified)
        throttle = SimpleRateThrottle()
        throttle.scope = 'user' if request.user.is_authenticated else 'anon'
        
        # Get rate info
        rate = throttle.get_rate()
        if rate:
            num_requests, duration = throttle.parse_rate(rate)
            
            # Get current usage from cache
            ident = throttle.get_ident(request)
            cache_key = throttle.cache_format % {
                'scope': throttle.scope,
                'ident': ident
            }
            
            history = cache.get(cache_key, [])
            current_usage = len(history)
            
            # Add headers
            response['X-RateLimit-Limit'] = num_requests
            response['X-RateLimit-Remaining'] = max(0, num_requests - current_usage)
            response['X-RateLimit-Reset'] = int(
                (throttle.timer() + duration)
            )
        
        return response
```

---

## 8. Monitoring & Analytics

### Track Rate Limit Events

**signals.py**
```python
from django.dispatch import Signal, receiver
from django.core.cache import cache
import logging

# Custom signal for throttle events
throttle_exceeded = Signal()

logger = logging.getLogger('throttle')


@receiver(throttle_exceeded)
def log_throttle_event(sender, request, **kwargs):
    """Log rate limit violations."""
    user = request.user if request.user.is_authenticated else 'anonymous'
    ip = request.META.get('REMOTE_ADDR', 'unknown')
    path = request.path
    
    logger.warning(
        f'Rate limit exceeded - '
        f'User: {user} - '
        f'IP: {ip} - '
        f'Path: {path} - '
        f'Scope: {kwargs.get("scope", "unknown")}'
    )
    
    # Increment counter for analytics
    cache_key = f'throttle_events:{request.path}:count'
    cache.incr(cache_key, 1)
    cache.expire(cache_key, 86400)  # 24 hours


def get_throttle_stats():
    """Get throttle statistics for monitoring."""
    # This would be called by a monitoring endpoint
    stats = {
        'total_violations': cache.get('throttle_total_count', 0),
        'by_endpoint': {},
        'by_user': {},
    }
    return stats
```

---

## 9. Production Settings

### Complete settings.py Configuration

```python
# settings/production.py

REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'path.to.throttles.AnonThrottle',
        'path.to.throttles.AuthUserThrottle',
        'path.to.throttles.BurstThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        # General limits
        'anon': '100/hour',
        'user': '1000/hour',
        'burst': '60/min',
        
        # Authentication endpoints (strict)
        'login': '5/15min',
        'register': '3/hour',
        'password_reset': '3/hour',
        'email_verify': '5/hour',
        
        # Business operations
        'order_create': '20/hour',
        'payment': '10/hour',
        'cart_modify': '100/hour',
        
        # Read operations (more lenient)
        'search': '200/hour',
        'list': '500/hour',
        'retrieve': '1000/hour',
        
        # Resource intensive
        'upload': '50/day',
        'export': '10/day',
        'report_generate': '20/day',
        
        # API keys (for integrations)
        'api_key': '5000/hour',
    },
    'EXCEPTION_HANDLER': 'path.to.custom_exception_handler',
}

# Cache backend for throttling (Redis recommended)
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
            'PARSER_CLASS': 'redis.connection.HiredisParser',
            'CONNECTION_POOL_CLASS_KWARGS': {
                'max_connections': 50,
                'retry_on_timeout': True,
            },
        },
        'KEY_PREFIX': 'throttle',
        'TIMEOUT': 86400,  # 24 hours
    }
}

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'throttle_file': {
            'level': 'WARNING',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/var/log/django/throttle.log',
            'maxBytes': 1024 * 1024 * 10,  # 10MB
            'backupCount': 5,
        },
    },
    'loggers': {
        'throttle': {
            'handlers': ['throttle_file'],
            'level': 'WARNING',
            'propagate': False,
        },
    },
}
```

---

## 10. Testing Rate Limits

### test_throttling.py

```python
from django.test import TestCase
from rest_framework.test import APIClient
from rest_framework import status
from django.contrib.auth import get_user_model
from django.core.cache import cache

User = get_user_model()


class ThrottlingTest(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        cache.clear()  # Clear cache before each test
    
    def test_anonymous_throttle(self):
        """Test anonymous user rate limiting."""
        # Make requests up to limit
        for i in range(100):
            response = self.client.get('/api/products/')
            self.assertEqual(response.status_code, 200)
        
        # 101st request should be throttled
        response = self.client.get('/api/products/')
        self.assertEqual(response.status_code, 429)
        self.assertIn('retry_after_seconds', response.data['details'])
    
    def test_authenticated_throttle(self):
        """Test authenticated user has higher limits."""
        self.client.force_authenticate(user=self.user)
        
        # Authenticated users have 1000/hour limit
        for i in range(100):
            response = self.client.get('/api/products/')
            self.assertEqual(response.status_code, 200)
        
        # Should still work (under 1000 limit)
        response = self.client.get('/api/products/')
        self.assertEqual(response.status_code, 200)
    
    def test_burst_throttle(self):
        """Test burst protection."""
        self.client.force_authenticate(user=self.user)
        
        # Try to make 61 requests in one minute (limit is 60)
        for i in range(60):
            response = self.client.post('/api/search/', {'q': 'test'})
            self.assertEqual(response.status_code, 200)
        
        # 61st request should be throttled
        response = self.client.post('/api/search/', {'q': 'test'})
        self.assertEqual(response.status_code, 429)
    
    def test_login_throttle(self):
        """Test strict login throttling."""
        # Try 5 login attempts (limit)
        for i in range(5):
            response = self.client.post('/api/auth/login/', {
                'username': 'testuser',
                'password': 'wrong'
            })
        
        # 6th attempt should be throttled
        response = self.client.post('/api/auth/login/', {
            'username': 'testuser',
            'password': 'testpass123'
        })
        self.assertEqual(response.status_code, 429)
    
    def test_rate_limit_headers(self):
        """Test rate limit headers are present."""
        response = self.client.get('/api/products/')
        
        self.assertIn('X-RateLimit-Limit', response)
        self.assertIn('X-RateLimit-Remaining', response)
        self.assertIn('X-RateLimit-Reset', response)
    
    def test_admin_exempt(self):
        """Test admin users are exempt from throttling."""
        admin = User.objects.create_superuser(
            username='admin',
            password='admin123',
            email='admin@test.com'
        )
        self.client.force_authenticate(user=admin)
        
        # Make many requests - should not be throttled
        for i in range(200):
            response = self.client.get('/api/products/')
            self.assertEqual(response.status_code, 200)
```

---

## 11. Best Practices Summary

### ✅ DO:

1. **Use Different Rates for Different Endpoints**
   - Strict for auth endpoints (login, register)
   - Lenient for read operations
   - Moderate for write operations

2. **Implement Burst Protection**
   - Short-term limits (per minute)
   - Long-term limits (per hour/day)

3. **Exempt Admin Users**
   - Allow internal tools to work without limits
   - Monitoring systems need access

4. **Add Helpful Headers**
   - `X-RateLimit-Limit`: Total allowed
   - `X-RateLimit-Remaining`: Requests left
   - `X-RateLimit-Reset`: When limit resets
   - `Retry-After`: How long to wait

5. **Log Throttle Events**
   - Monitor for abuse patterns
   - Identify problematic endpoints
   - Track frequent offenders

6. **Use Redis for Production**
   - Much faster than database
   - Built for this use case
   - Handles expiration automatically

7. **Test Thoroughly**
   - Test all rate limit scenarios
   - Verify limits are enforced
   - Check headers are correct

### ❌ DON'T:

1. **Don't Use Same Limit for Everything**
   - Different endpoints have different needs
   - One size doesn't fit all

2. **Don't Forget Burst Protection**
   - Hourly limit might allow 1000 requests in 1 second
   - Add minute-based limits

3. **Don't Block Legitimate Users**
   - Set reasonable limits
   - Provide clear error messages
   - Tell them when they can retry

4. **Don't Use Database for Throttle Cache**
   - Too slow
   - Defeats the purpose
   - Use Redis or Memcached

5. **Don't Ignore Behind Proxy IPs**
   - Get real IP from X-Forwarded-For
   - Handle load balancers correctly

---

## 12. Monitoring Dashboard Example

**admin_views.py**
```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAdminUser
from rest_framework.response import Response
from django.core.cache import cache


@api_view(['GET'])
@permission_classes([IsAdminUser])
def throttle_stats(request):
    """
    Admin endpoint to view throttle statistics.
    GET /api/admin/throttle-stats/
    """
    stats = {
        'current_hour': {
            'total_requests': cache.get('requests:hour:count', 0),
            'throttled_requests': cache.get('throttled:hour:count', 0),
            'unique_ips': cache.get('unique_ips:hour', set()).__len__(),
        },
        'top_throttled_ips': get_top_throttled_ips(),
        'top_throttled_endpoints': get_top_throttled_endpoints(),
        'by_endpoint': get_stats_by_endpoint(),
    }
    
    return Response(stats)


def get_top_throttled_ips(limit=10):
    """Get IPs with most throttle violations."""
    # Implementation would query cache/logs
    pass


def get_top_throttled_endpoints(limit=10):
    """Get endpoints with most throttle violations."""
    # Implementation would query cache/logs
    pass
```
