**Python Models**



```
  Class Book(models.Model):

title = models.Charfield(max_length = 50)

rating = models.IntegerField()

From app_name import Book

  

harry_potter = Book(title=“Harry Potter 1”, rating=5)

harry_potter.save()
lor = Book(title=“Lord Of the Rings”, rating=4)

lor.save()

  

Book.objects.all()
```





  

