# Django REST Framework Serializers

## Navigation
- [What Is a Serializer?](#what-is-a-serializer)
- [Serializer vs ModelSerializer](#serializer-vs-modelserializer)
- [Common Serializer Types](#common-serializer-types)
- [Nested & ListSerializers](#nested--listserializers)
- [Custom Field & Validation](#custom-field--validation)
- [Usage Examples](#usage-examples)

## What Is a Serializer?
Convert Django model instances (or querysets) to JSON and back.  
- Handle validation  
- Control output fields  

## Serializer vs ModelSerializer
- **Serializer**: manual field definition  
- **ModelSerializer**: auto-generates fields from a model’s `Meta`

## Common Serializer Types
```python
from rest_framework import serializers
from .models import Post

# 1. Basic Serializer
class PostBasicSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=200)
    content = serializers.CharField()

    def create(self, validated_data):
        return Post.objects.create(**validated_data)

# 2. ModelSerializer
class PostModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id','title','content','author','published']

# 3. HyperlinkedModelSerializer
class PostHyperlinkSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Post
        fields = ['url','id','title','author']
        extra_kwargs = {
          'url': {'view_name': 'post-detail', 'lookup_field': 'pk'}
        }
```

## Nested & ListSerializers
```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ['id','post','body']

class PostWithCommentsSerializer(serializers.ModelSerializer):
    comments = CommentSerializer(many=True, read_only=True)
    class Meta:
        model = Post
        fields = ['id','title','comments']
```

## Custom Field & Validation
```python
class UppercaseTitleField(serializers.CharField):
    def to_representation(self, value):
        return value.upper()

class PostValidatorSerializer(serializers.ModelSerializer):
    title = UppercaseTitleField()

    class Meta:
        model = Post
        fields = ['title','content']

    def validate_content(self, value):
        if 'spam' in value.lower():
            raise serializers.ValidationError("No spam allowed")
        return value
```

## Usage Examples
```python
# In a viewset
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostModelSerializer

# Manual serialization
post = Post.objects.first()
data = PostBasicSerializer(post).data
```
