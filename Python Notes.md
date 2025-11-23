**Python Models**



```
  Class Book(models.Model):

title = models.Charfield(max_length = 50)

rating = models.IntegerField(validators=[MinValueValidator(1), MaxValueValidator(5)])

author = models.CharField(null=True,max_length=100)

is_bestselling = models.BooleanField(default=False)

From app_name import Book

  

harry_potter = Book(title=“Harry Potter 1”, rating=5)

harry_potter.save()
lor = Book(title=“Lord Of the Rings”, rating=4)

lor.save()

python3 manage.py makemigrations

  

Book.objects.all()

//output it nicely

def __str__(self):
	return f"{self.title} ({self.rating})"
```

Model : models.py
View : views.py
Controller



  

