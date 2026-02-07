+++
title = "Professional ImageField Management in Django REST Framework"
date = 2026-02-07
+++

## 🎯 Complete Guide: Models, Serializers, ViewSets, and Storage

---

## 1. Model Design

### Single Image (Simple)

```python
from django.db import models
from django.core.validators import FileExtensionValidator
import os
import uuid


def product_image_path(instance, filename):
    """
    Generate upload path for product images.
    Format: products/2024/01/15/uuid-filename.jpg
    """
    # Get file extension
    ext = filename.split('.')[-1]
    
    # Generate unique filename
    filename = f"{uuid.uuid4()}.{ext}"
    
    # Organize by date
    from datetime import datetime
    date_path = datetime.now().strftime('%Y/%m/%d')
    
    return os.path.join('products', date_path, filename)


class Product(models.Model):
    """Product with single image."""
    
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    
    # Single image field
    image = models.ImageField(
        upload_to=product_image_path,
        null=True,
        blank=True,
        validators=[
            FileExtensionValidator(
                allowed_extensions=['jpg', 'jpeg', 'png', 'webp']
            )
        ],
        help_text='Product image (max 5MB, formats: JPG, PNG, WebP)'
    )
    
    # Optional: Store image dimensions
    image_width = models.PositiveIntegerField(null=True, blank=True, editable=False)
    image_height = models.PositiveIntegerField(null=True, blank=True, editable=False)
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    def save(self, *args, **kwargs):
        """Store image dimensions on save."""
        if self.image:
            from PIL import Image
            try:
                img = Image.open(self.image)
                self.image_width, self.image_height = img.size
            except Exception:
                pass
        
        super().save(*args, **kwargs)
    
    def delete(self, *args, **kwargs):
        """Delete image file when model is deleted."""
        if self.image:
            # Delete the file from storage
            storage = self.image.storage
            if storage.exists(self.image.name):
                storage.delete(self.image.name)
        
        super().delete(*args, **kwargs)
```

---

### Multiple Images (Professional)

```python
class Product(models.Model):
    """Product model without direct image field."""
    
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    def get_primary_image(self):
        """Get the primary/featured image."""
        return self.images.filter(is_primary=True).first()
    
    def get_all_images(self):
        """Get all images ordered by position."""
        return self.images.all().order_by('order')


class ProductImage(models.Model):
    """
    Separate model for product images.
    Allows multiple images per product.
    """
    
    product = models.ForeignKey(
        Product,
        on_delete=models.CASCADE,
        related_name='images'
    )
    
    image = models.ImageField(
        upload_to='products/%Y/%m/%d/',
        validators=[
            FileExtensionValidator(
                allowed_extensions=['jpg', 'jpeg', 'png', 'webp']
            )
        ]
    )
    
    # Thumbnails (auto-generated)
    thumbnail = models.ImageField(
        upload_to='products/thumbnails/%Y/%m/%d/',
        null=True,
        blank=True,
        editable=False
    )
    
    medium = models.ImageField(
        upload_to='products/medium/%Y/%m/%d/',
        null=True,
        blank=True,
        editable=False
    )
    
    # Metadata
    alt_text = models.CharField(
        max_length=200,
        blank=True,
        help_text='Alternative text for accessibility'
    )
    
    caption = models.CharField(max_length=200, blank=True)
    
    is_primary = models.BooleanField(
        default=False,
        help_text='Set as main product image'
    )
    
    order = models.PositiveIntegerField(
        default=0,
        help_text='Display order (lower numbers first)'
    )
    
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['order', 'uploaded_at']
        indexes = [
            models.Index(fields=['product', 'is_primary']),
            models.Index(fields=['product', 'order']),
        ]
    
    def __str__(self):
        return f"Image for {self.product.name}"
    
    def save(self, *args, **kwargs):
        """Generate thumbnails on save."""
        # If this is set as primary, unset all others
        if self.is_primary:
            ProductImage.objects.filter(
                product=self.product,
                is_primary=True
            ).exclude(pk=self.pk).update(is_primary=False)
        
        # Save first to get the image file
        super().save(*args, **kwargs)
        
        # Generate thumbnails after saving
        if self.image and not self.thumbnail:
            self.generate_thumbnails()
    
    def generate_thumbnails(self):
        """Generate thumbnail and medium-sized images."""
        from PIL import Image
        from io import BytesIO
        from django.core.files.uploadedfile import InMemoryUploadedFile
        import sys
        
        try:
            # Open original image
            img = Image.open(self.image)
            
            # Convert RGBA to RGB if necessary
            if img.mode in ('RGBA', 'LA', 'P'):
                background = Image.new('RGB', img.size, (255, 255, 255))
                background.paste(img, mask=img.split()[-1] if img.mode == 'RGBA' else None)
                img = background
            
            # Generate thumbnail (200x200)
            thumb_size = (200, 200)
            img.thumbnail(thumb_size, Image.Resampling.LANCZOS)
            
            thumb_io = BytesIO()
            img.save(thumb_io, format='JPEG', quality=85)
            
            thumb_file = InMemoryUploadedFile(
                thumb_io,
                None,
                f'thumb_{self.image.name.split("/")[-1]}',
                'image/jpeg',
                sys.getsizeof(thumb_io),
                None
            )
            
            self.thumbnail.save(
                f'thumb_{self.image.name.split("/")[-1]}',
                thumb_file,
                save=False
            )
            
            # Generate medium size (800x800)
            img = Image.open(self.image)  # Reopen original
            if img.mode in ('RGBA', 'LA', 'P'):
                background = Image.new('RGB', img.size, (255, 255, 255))
                background.paste(img, mask=img.split()[-1] if img.mode == 'RGBA' else None)
                img = background
            
            medium_size = (800, 800)
            img.thumbnail(medium_size, Image.Resampling.LANCZOS)
            
            medium_io = BytesIO()
            img.save(medium_io, format='JPEG', quality=90)
            
            medium_file = InMemoryUploadedFile(
                medium_io,
                None,
                f'medium_{self.image.name.split("/")[-1]}',
                'image/jpeg',
                sys.getsizeof(medium_io),
                None
            )
            
            self.medium.save(
                f'medium_{self.image.name.split("/")[-1]}',
                medium_file,
                save=False
            )
            
            # Save with thumbnails
            super().save(update_fields=['thumbnail', 'medium'])
            
        except Exception as e:
            print(f"Error generating thumbnails: {e}")
    
    def delete(self, *args, **kwargs):
        """Delete all image files when model is deleted."""
        # Delete original
        if self.image:
            storage = self.image.storage
            if storage.exists(self.image.name):
                storage.delete(self.image.name)
        
        # Delete thumbnail
        if self.thumbnail:
            storage = self.thumbnail.storage
            if storage.exists(self.thumbnail.name):
                storage.delete(self.thumbnail.name)
        
        # Delete medium
        if self.medium:
            storage = self.medium.storage
            if storage.exists(self.medium.name):
                storage.delete(self.medium.name)
        
        super().delete(*args, **kwargs)
```

---

## 2. Serializers

### Single Image Serializer

```python
from rest_framework import serializers
from django.conf import settings


class ProductSerializer(serializers.ModelSerializer):
    """
    Serializer for product with single image.
    """
    
    # Return absolute URL for image
    image_url = serializers.SerializerMethodField()
    
    # Accept image upload
    image = serializers.ImageField(
        max_length=None,
        use_url=True,
        allow_null=True,
        required=False
    )
    
    class Meta:
        model = Product
        fields = [
            'id',
            'name',
            'slug',
            'image',
            'image_url',
            'image_width',
            'image_height',
            'created_at',
        ]
        read_only_fields = ['id', 'image_width', 'image_height', 'created_at']
    
    def get_image_url(self, obj):
        """Return absolute URL for image."""
        if obj.image:
            request = self.context.get('request')
            if request:
                return request.build_absolute_uri(obj.image.url)
            return obj.image.url
        return None
    
    def validate_image(self, value):
        """Validate image file."""
        if value:
            # Check file size (5MB limit)
            if value.size > 5 * 1024 * 1024:
                raise serializers.ValidationError(
                    "Image file too large. Maximum size is 5MB."
                )
            
            # Check image dimensions
            from PIL import Image
            try:
                img = Image.open(value)
                width, height = img.size
                
                # Minimum dimensions
                if width < 200 or height < 200:
                    raise serializers.ValidationError(
                        "Image dimensions too small. Minimum 200x200 pixels."
                )
                
                # Maximum dimensions
                if width > 5000 or height > 5000:
                    raise serializers.ValidationError(
                        "Image dimensions too large. Maximum 5000x5000 pixels."
                    )
                
            except Exception as e:
                raise serializers.ValidationError(f"Invalid image file: {str(e)}")
        
        return value
```

---

### Multiple Images Serializer

```python
class ProductImageSerializer(serializers.ModelSerializer):
    """
    Serializer for individual product images.
    """
    
    # Absolute URLs for all image sizes
    image_url = serializers.SerializerMethodField()
    thumbnail_url = serializers.SerializerMethodField()
    medium_url = serializers.SerializerMethodField()
    
    class Meta:
        model = ProductImage
        fields = [
            'id',
            'image',
            'image_url',
            'thumbnail_url',
            'medium_url',
            'alt_text',
            'caption',
            'is_primary',
            'order',
            'uploaded_at',
        ]
        read_only_fields = ['id', 'uploaded_at', 'thumbnail_url', 'medium_url']
    
    def get_image_url(self, obj):
        if obj.image:
            request = self.context.get('request')
            if request:
                return request.build_absolute_uri(obj.image.url)
            return obj.image.url
        return None
    
    def get_thumbnail_url(self, obj):
        if obj.thumbnail:
            request = self.context.get('request')
            if request:
                return request.build_absolute_uri(obj.thumbnail.url)
            return obj.thumbnail.url
        return None
    
    def get_medium_url(self, obj):
        if obj.medium:
            request = self.context.get('request')
            if request:
                return request.build_absolute_uri(obj.medium.url)
            return obj.medium.url
        return None
    
    def validate_image(self, value):
        """Validate uploaded image."""
        if value:
            # File size check
            if value.size > 5 * 1024 * 1024:
                raise serializers.ValidationError("Maximum file size is 5MB")
            
            # File type check (check magic bytes, not just extension)
            from PIL import Image
            try:
                img = Image.open(value)
                img.verify()  # Verify it's actually an image
                
                # Check format
                if img.format.lower() not in ['jpeg', 'jpg', 'png', 'webp']:
                    raise serializers.ValidationError(
                        "Unsupported format. Use JPG, PNG, or WebP"
                    )
                
            except Exception as e:
                raise serializers.ValidationError(f"Invalid image: {str(e)}")
        
        return value


class ProductWithImagesSerializer(serializers.ModelSerializer):
    """
    Product serializer with nested images.
    """
    
    images = ProductImageSerializer(many=True, read_only=True)
    primary_image = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = [
            'id',
            'name',
            'slug',
            'description',
            'price',
            'primary_image',
            'images',
            'created_at',
        ]
    
    def get_primary_image(self, obj):
        """Get primary image data."""
        primary = obj.get_primary_image()
        if primary:
            return ProductImageSerializer(primary, context=self.context).data
        return None


class ProductCreateSerializer(serializers.ModelSerializer):
    """
    Serializer for creating products.
    Images are uploaded separately via ProductImageViewSet.
    """
    
    class Meta:
        model = Product
        fields = ['name', 'slug', 'description', 'price']
```

---

## 3. ViewSets

### Single Image ViewSet

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.parsers import MultiPartParser, FormParser
from rest_framework.permissions import IsAuthenticatedOrReadOnly


class ProductViewSet(viewsets.ModelViewSet):
    """
    Product ViewSet with image upload support.
    """
    
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    lookup_field = 'slug'
    
    # Important: Add parsers for file upload
    parser_classes = [MultiPartParser, FormParser]
    
    @action(
        detail=True,
        methods=['post', 'put'],
        parser_classes=[MultiPartParser, FormParser]
    )
    def upload_image(self, request, slug=None):
        """
        Upload or update product image.
        
        POST /api/products/{slug}/upload_image/
        
        Form data:
        - image: Image file
        """
        product = self.get_object()
        
        # Check if image is provided
        if 'image' not in request.FILES:
            return Response(
                {'error': 'No image provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Delete old image if exists
        if product.image:
            product.image.delete(save=False)
        
        # Save new image
        product.image = request.FILES['image']
        
        try:
            product.save()
        except Exception as e:
            return Response(
                {'error': f'Failed to save image: {str(e)}'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        serializer = self.get_serializer(product)
        return Response(serializer.data)
    
    @action(detail=True, methods=['delete'])
    def delete_image(self, request, slug=None):
        """
        Delete product image.
        
        DELETE /api/products/{slug}/delete_image/
        """
        product = self.get_object()
        
        if not product.image:
            return Response(
                {'error': 'No image to delete'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Delete the image file
        product.image.delete(save=True)
        
        return Response(
            {'message': 'Image deleted successfully'},
            status=status.HTTP_204_NO_CONTENT
        )
```

---

### Multiple Images ViewSet

```python
from django.db import transaction


class ProductImageViewSet(viewsets.ModelViewSet):
    """
    ViewSet for managing product images.
    
    Endpoints:
    - POST   /api/products/{slug}/images/          - Upload new image
    - GET    /api/products/{slug}/images/          - List all images
    - GET    /api/products/{slug}/images/{id}/     - Get specific image
    - PATCH  /api/products/{slug}/images/{id}/     - Update image metadata
    - DELETE /api/products/{slug}/images/{id}/     - Delete image
    - POST   /api/products/{slug}/images/bulk_upload/ - Upload multiple
    - POST   /api/products/{slug}/images/{id}/set_primary/ - Set as primary
    """
    
    serializer_class = ProductImageSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    parser_classes = [MultiPartParser, FormParser]
    
    def get_queryset(self):
        """Filter images by product slug."""
        product_slug = self.kwargs.get('product_slug')
        return ProductImage.objects.filter(
            product__slug=product_slug
        ).select_related('product')
    
    def get_product(self):
        """Get product from URL."""
        from django.shortcuts import get_object_or_404
        product_slug = self.kwargs.get('product_slug')
        return get_object_or_404(Product, slug=product_slug)
    
    def create(self, request, product_slug=None):
        """
        Upload a new image for the product.
        
        POST /api/products/{slug}/images/
        
        Form data:
        - image: Image file (required)
        - alt_text: Alternative text (optional)
        - caption: Caption (optional)
        - is_primary: Set as primary image (optional, default: false)
        - order: Display order (optional, default: 0)
        """
        product = self.get_product()
        
        if 'image' not in request.FILES:
            return Response(
                {'error': 'No image file provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Create serializer with product
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        # Save with product
        serializer.save(product=product)
        
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    
    def partial_update(self, request, product_slug=None, pk=None):
        """
        Update image metadata (not the image file itself).
        
        PATCH /api/products/{slug}/images/{id}/
        
        Body:
        - alt_text
        - caption
        - is_primary
        - order
        """
        image = self.get_object()
        serializer = self.get_serializer(
            image,
            data=request.data,
            partial=True
        )
        serializer.is_valid(raise_exception=True)
        serializer.save()
        
        return Response(serializer.data)
    
    @action(detail=False, methods=['post'])
    def bulk_upload(self, request, product_slug=None):
        """
        Upload multiple images at once.
        
        POST /api/products/{slug}/images/bulk_upload/
        
        Form data:
        - images[]: Multiple image files
        """
        product = self.get_product()
        
        # Get all uploaded files
        images = request.FILES.getlist('images')
        
        if not images:
            return Response(
                {'error': 'No images provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Maximum 10 images at once
        if len(images) > 10:
            return Response(
                {'error': 'Maximum 10 images allowed per upload'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        uploaded_images = []
        errors = []
        
        with transaction.atomic():
            for idx, image_file in enumerate(images):
                serializer = self.get_serializer(data={
                    'image': image_file,
                    'order': idx
                })
                
                if serializer.is_valid():
                    serializer.save(product=product)
                    uploaded_images.append(serializer.data)
                else:
                    errors.append({
                        'file': image_file.name,
                        'errors': serializer.errors
                    })
        
        return Response({
            'uploaded': len(uploaded_images),
            'images': uploaded_images,
            'errors': errors
        }, status=status.HTTP_201_CREATED if uploaded_images else status.HTTP_400_BAD_REQUEST)
    
    @action(detail=True, methods=['post'])
    def set_primary(self, request, product_slug=None, pk=None):
        """
        Set this image as the primary product image.
        
        POST /api/products/{slug}/images/{id}/set_primary/
        """
        image = self.get_object()
        
        # Unset all other primary images for this product
        ProductImage.objects.filter(
            product=image.product,
            is_primary=True
        ).update(is_primary=False)
        
        # Set this as primary
        image.is_primary = True
        image.save()
        
        serializer = self.get_serializer(image)
        return Response(serializer.data)
    
    @action(detail=False, methods=['post'])
    def reorder(self, request, product_slug=None):
        """
        Reorder images.
        
        POST /api/products/{slug}/images/reorder/
        
        Body:
        {
            "order": [3, 1, 2, 4]  // Array of image IDs in desired order
        }
        """
        product = self.get_product()
        order = request.data.get('order', [])
        
        if not isinstance(order, list):
            return Response(
                {'error': 'order must be a list of image IDs'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        with transaction.atomic():
            for idx, image_id in enumerate(order):
                ProductImage.objects.filter(
                    id=image_id,
                    product=product
                ).update(order=idx)
        
        # Return updated list
        images = self.get_queryset()
        serializer = self.get_serializer(images, many=True)
        return Response(serializer.data)
```

---

## 4. URL Configuration

### Nested Router for Product Images

```python
from rest_framework_nested import routers
from django.urls import path, include

# Main router
router = routers.DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')

# Nested router for product images
products_router = routers.NestedDefaultRouter(
    router,
    r'products',
    lookup='product'
)
products_router.register(
    r'images',
    ProductImageViewSet,
    basename='product-images'
)

urlpatterns = [
    path('', include(router.urls)),
    path('', include(products_router.urls)),
]
```

**Installation:**
```bash
pip install drf-nested-routers
```

**Generated URLs:**
```
GET    /api/products/                           - List products
POST   /api/products/                           - Create product
GET    /api/products/{slug}/                    - Get product
PATCH  /api/products/{slug}/                    - Update product
DELETE /api/products/{slug}/                    - Delete product

GET    /api/products/{slug}/images/             - List images
POST   /api/products/{slug}/images/             - Upload image
GET    /api/products/{slug}/images/{id}/        - Get image
PATCH  /api/products/{slug}/images/{id}/        - Update metadata
DELETE /api/products/{slug}/images/{id}/        - Delete image

POST   /api/products/{slug}/images/bulk_upload/ - Bulk upload
POST   /api/products/{slug}/images/{id}/set_primary/ - Set primary
POST   /api/products/{slug}/images/reorder/     - Reorder images
```

---

## 5. Storage Configuration

### Local Development (settings.py)

```python
# Media files configuration
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# For development
if DEBUG:
    # Serve media files in development
    from django.conf.urls.static import static
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

### Production: AWS S3 Storage

**Installation:**
```bash
pip install django-storages boto3
```

**settings/production.py:**
```python
# AWS S3 Configuration
AWS_ACCESS_KEY_ID = env('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = env('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = env('AWS_STORAGE_BUCKET_NAME')
AWS_S3_REGION_NAME = env('AWS_S3_REGION_NAME', default='us-east-1')

AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=86400',  # 1 day
}
AWS_DEFAULT_ACL = 'public-read'
AWS_S3_FILE_OVERWRITE = False
AWS_QUERYSTRING_AUTH = False

# Media files
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/'
```

---

### Production: DigitalOcean Spaces

```python
# DigitalOcean Spaces (S3-compatible)
AWS_ACCESS_KEY_ID = env('SPACES_KEY')
AWS_SECRET_ACCESS_KEY = env('SPACES_SECRET')
AWS_STORAGE_BUCKET_NAME = env('SPACES_BUCKET')
AWS_S3_ENDPOINT_URL = f'https://{env("SPACES_REGION")}.digitaloceanspaces.com'
AWS_S3_REGION_NAME = env('SPACES_REGION')
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.{env("SPACES_REGION")}.cdn.digitaloceanspaces.com'

AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=86400',
}
AWS_DEFAULT_ACL = 'public-read'

DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/'
```

---

## 6. Frontend Integration Examples

### Upload Single Image (JavaScript)

```javascript
// Upload product image
async function uploadProductImage(slug, imageFile) {
    const formData = new FormData();
    formData.append('image', imageFile);
    
    const response = await fetch(`/api/products/${slug}/upload_image/`, {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${token}`
        },
        body: formData
    });
    
    return await response.json();
}

// Usage
const fileInput = document.querySelector('#image-input');
fileInput.addEventListener('change', async (e) => {
    const file = e.target.files[0];
    const result = await uploadProductImage('django-book', file);
    console.log(result);
});
```

---

### Upload Multiple Images

```javascript
// Bulk upload
async function bulkUploadImages(slug, imageFiles) {
    const formData = new FormData();
    
    // Append multiple files
    imageFiles.forEach((file) => {
        formData.append('images', file);
    });
    
    const response = await fetch(`/api/products/${slug}/images/bulk_upload/`, {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${token}`
        },
        body: formData
    });
    
    return await response.json();
}

// Usage
const multiFileInput = document.querySelector('#multi-image-input');
multiFileInput.addEventListener('change', async (e) => {
    const files = Array.from(e.target.files);
    const result = await bulkUploadImages('django-book', files);
    console.log(result);
});
```

---

### React Component Example

```jsx
import React, { useState } from 'react';
import axios from 'axios';

function ProductImageUpload({ productSlug }) {
    const [images, setImages] = useState([]);
    const [uploading, setUploading] = useState(false);
    
    const handleFileChange = (e) => {
        setImages(Array.from(e.target.files));
    };
    
    const handleUpload = async () => {
        if (images.length === 0) return;
        
        const formData = new FormData();
        images.forEach(image => {
            formData.append('images', image);
        });
        
        setUploading(true);
        
        try {
            const response = await axios.post(
                `/api/products/${productSlug}/images/bulk_upload/`,
                formData,
                {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                        'Authorization': `Bearer ${token}`
                    }
                }
            );
            
            console.log('Upload successful:', response.data);
            setImages([]);
            
        } catch (error) {
            console.error('Upload failed:', error);
        } finally {
            setUploading(false);
        }
    };
    
    return (
        <div>
            <input
                type="file"
                multiple
                accept="image/*"
                onChange={handleFileChange}
            />
            <button
                onClick={handleUpload}
                disabled={uploading || images.length === 0}
            >
                {uploading ? 'Uploading...' : `Upload ${images.length} images`}
            </button>
        </div>
    );
}
```

---

## 7. Image Optimization (Celery Task)

```python
# tasks.py
from celery import shared_task
from PIL import Image
from io import BytesIO
from django.core.files.uploadedfile import InMemoryUploadedFile
import sys


@shared_task
def optimize_product_image(image_id):
    """
    Optimize product image in background.
    - Compress image
    - Generate thumbnails
    - Convert to WebP
    """
    from products.models import ProductImage
    
    try:
        product_image = ProductImage.objects.get(id=image_id)
        
        # Open original
        img = Image.open(product_image.image)
        
        # Convert RGBA to RGB
        if img.mode in ('RGBA', 'LA', 'P'):
            background = Image.new('RGB', img.size, (255, 255, 255))
            background.paste(img, mask=img.split()[-1] if img.mode == 'RGBA' else None)
            img = background
        
        # Optimize and save as WebP (smaller file size)
        output = BytesIO()
        img.save(output, format='WebP', quality=85, optimize=True)
        output.seek(0)
        
        # Save optimized version
        product_image.image.save(
            product_image.image.name.replace('.jpg', '.webp').replace('.png', '.webp'),
            InMemoryUploadedFile(
                output,
                'ImageField',
                product_image.image.name,
                'image/webp',
                sys.getsizeof(output),
                None
            ),
            save=True
        )
        
        # Generate thumbnails
        product_image.generate_thumbnails()
        
        return f"Optimized image {image_id}"
        
    except Exception as e:
        return f"Failed to optimize image {image_id}: {str(e)}"


# In your serializer or viewset
def create(self, validated_data):
    instance = super().create(validated_data)
    
    # Queue optimization task
    optimize_product_image.delay(instance.id)
    
    return instance
```

---

## 8. Testing

```python
# tests/test_images.py
from django.test import TestCase
from django.core.files.uploadedfile import SimpleUploadedFile
from rest_framework.test import APIClient
from rest_framework import status
from PIL import Image
from io import BytesIO


class ProductImageTest(TestCase):
    
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user('test', 'test@test.com', 'pass')
        self.client.force_authenticate(user=self.user)
        
        self.product = Product.objects.create(
            name='Test Product',
            slug='test-product',
            price=Decimal('19.99')
        )
    
    def create_test_image(self):
        """Create a test image file."""
        file = BytesIO()
        image = Image.new('RGB', (100, 100), color='red')
        image.save(file, 'PNG')
        file.seek(0)
        return SimpleUploadedFile(
            'test.png',
            file.read(),
            content_type='image/png'
        )
    
    def test_upload_single_image(self):
        """Test uploading a single image."""
        image = self.create_test_image()
        
        response = self.client.post(
            f'/api/products/{self.product.slug}/upload_image/',
            {'image': image},
            format='multipart'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.product.refresh_from_db()
        self.assertTrue(self.product.image)
    
    def test_upload_multiple_images(self):
        """Test bulk upload."""
        images = [self.create_test_image() for _ in range(3)]
        
        response = self.client.post(
            f'/api/products/{self.product.slug}/images/bulk_upload/',
            {'images': images},
            format='multipart'
        )
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['uploaded'], 3)
        self.assertEqual(ProductImage.objects.filter(product=self.product).count(), 3)
    
    def test_set_primary_image(self):
        """Test setting primary image."""
        image1 = ProductImage.objects.create(
            product=self.product,
            image=self.create_test_image(),
            is_primary=True
        )
        image2 = ProductImage.objects.create(
            product=self.product,
            image=self.create_test_image()
        )
        
        response = self.client.post(
            f'/api/products/{self.product.slug}/images/{image2.id}/set_primary/'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        image1.refresh_from_db()
        image2.refresh_from_db()
        
        self.assertFalse(image1.is_primary)
        self.assertTrue(image2.is_primary)
    
    def test_image_validation_file_size(self):
        """Test file size validation."""
        # Create oversized file (mock)
        large_file = SimpleUploadedFile(
            'large.png',
            b'x' * (6 * 1024 * 1024),  # 6MB
            content_type='image/png'
        )
        
        response = self.client.post(
            f'/api/products/{self.product.slug}/images/',
            {'image': large_file},
            format='multipart'
        )
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
```

---

## 9. Best Practices Summary

### ✅ DO:

1. **Use separate model for multiple images** (ProductImage)
2. **Generate thumbnails** automatically
3. **Validate file size and format**
4. **Use absolute URLs** in serializers
5. **Delete files** when model is deleted
6. **Use unique filenames** (UUID)
7. **Organize by date** (upload_to with date path)
8. **Add parsers** to ViewSet (`MultiPartParser`, `FormParser`)
9. **Use cloud storage** in production (S3, Spaces)
10. **Optimize images** (compress, WebP format)

### ❌ DON'T:

1. **Don't store multiple images** in single ImageField
2. **Don't forget file cleanup** on delete
3. **Don't skip validation** (size, format, dimensions)
4. **Don't use default upload_to** (organize files properly)
5. **Don't commit media files** to git (add to `.gitignore`)
6. **Don't serve media** from Django in production
7. **Don't allow unlimited uploads** (set max count)
8. **Don't process images** synchronously (use Celery for large files)

This is the **professional, production-ready** way to handle images in Django REST Framework! 🚀
