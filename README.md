# Notes on painless file uploads in AngularJS with Django REST applications.

When creating an application from scratch using AngularJS with Django we usuall need a form with a file upload feature. Let's say, profile edit with avatar upload or a gallery with multiple file uploads with extra preview for better UX. Angular does not provide support for file upload by default. Developers over the Internet created few guidelines with preferable solutions, but in my opinion none explain the issue completely.

### Endpoint setup

First, let's create a sample Django REST application - model, serializer and view.

```python
# models.py
class Foo(models.Model):
    name = models.CharField(max_length=20)
    image = models.ImageField(upload_to=settings.IMAGES_PATH)
```

```python
# serializers.py
class FooSerializer(ModelSerializer):
    class Meta:
        model = Foo
```
```python        
# views.py
from rest_framework.parsers import MultiPartParser

class FooCreateView(CreateViewAPI):
    parser_classes = (MultiPartParser,)
    serializer_class = FooSerializer
```

This will simply provide an endpoint for uploading images with some extra ```name``` parameter to the server. As you can see ```MultipartParser``` is also used in the view.

> ```MultipartParser``` is a parser for multipart form data which may include some file data. It parses the incoming bytestream as a multipart encoded form and returns a DataAndFiles object.

## Angular project setup

Second, we will create a form with a file input, controller and a resource factory.

```html
<p ng-if="newFoo">
    <p>Uploaded foo image named "{{ newFoo.name }}": </p>
    <p><img ng-src="{{ newFoo.image}}" /></p>
</p>

<form ng-controller="UploadFormController" ng-submit="submit(form)">
    <div ng-if="preview">
        Image preview: <img ng-src="{{ preview }}" />
    </div>
    
    <input type="file" ng-model="form.image" onchange="angular.element(this).scope().updateImage(this)" />
    <input type="text" ng-model="form.name" />
    <input type="submit" />
</form>
```
```coffeescript
angular.module 'application'
  .controller 'UploadFormController', (Foo) ->
    $scope.form = {}

    $scope.submit = (foo) -> 
        $scope.newFoo = Foo.update(foo)

    $scope.updateImage = (imageField) ->
      reader = new FileReader()
      reader.onloadend = -> $scope.preview = reader.result
      reader.readAsDataURL imageField.files[0]

      $scope.form.avatar = imageField.files[0]

    return @
```
```coffeescript
angular.module 'fu'
  .factory 'Foo', ($resource) ->
    
    return $resource 'http://endpoint/', null,
      update:
        method: 'POST'
        transformRequest: (data) ->
          formData = new FormData()
          formData.append key, data[key] for key in Object.keys data
          return formData
        headers:
          'Content-Type': undefined
```

## This is it! That simple.
**No external packages. No tons of code. No pain.**

Questions? Feel free to leave a review or ask.
