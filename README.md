# Note on painless file upload in AngularJS and Django REST application

A common pattern when creating an application from scratch in Angular and Django is when there's a need of a form with any file upload feature. Let's say profile edition with avatar upload or an gallery with multiple file upload with extra preview for better UX. Angular does not provide support for file upload by default. Developers over internet suggest many different solutions, but personally none of them solve the issue completely.

### Endpoint setup

First, lets create sample Django REST application - model, serializer and view.

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

This will simply provide an endpoint for uploading images with some extra ```name``` parameter to the server. As you can see ```MultipartParser``` is being used in the view additionaly.

> ```MultipartParser``` is a parser for multipart form data, which may include file data. It parses the incoming bytestream as a multipart encoded form, and returns a DataAndFiles object.

## Angular project setup

Second, we will create a form with a file input, controller and resource service.

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
        Foo.update(foo).then (newFoo) ->
            $scope.newFoo = newFoo

    $scope.updateImage = (imageField) ->
      reader = new FileReader()
      reader.onloadend = -> $scope.preview = reader.result
      reader.readAsDataURL imageField.files[0]

      $scope.form.avatar = imageField.files[0]

    return @
```
```coffeescript
angular.module 'fu'
  .service 'Foo', ($resource) ->
    
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

## That's it!
Questions? Feel free to leave a review or ask questions.
