# Django ORM

___

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=50)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):       
        return self.headline 

```
객체 생성하기

```python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

객체 변경사항 저장하기

```python
>>> b5.name = 'New name'
>>> b5.save()
```

FK ManyToMany필드 저장하기

```python
>>> from blog.models import Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

```python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

```python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

객체 불러오기

```python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

모든 객체 불러오기

```python
>>> all_entries = Entry.objects.all()
```

필터 체이닝

```python
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(pub_date__gte=datetime(2005, 1, 30)
... )
```

필터링된 쿼리셋은 고유하다

```python
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```

쿼리셋은 게으르다

```python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```


get으로 단일 객체 불러오기

```python
>>> one_entry = Entry.objects.get(pk=1)
```

객체 제한걸기

```python
>>> Entry.objects.all()[:5]
```

```python
>>> Entry.objects.all()[5:10]
```

```python
>>> Entry.objects.all()[:10:2]
```

```python
>>> Entry.objects.order_by('headline')[0]
```

```python
>>> Entry.objects.order_by('headline')[0:1].get()
```

필드 조회

```python
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
```

```sql
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

```python
>>> Entry.objects.filter(blog_id=4)
```

조회 조건

```python
exact
iexact
contains
icontains
in
gt
gte
lt
lte
startswith
istartswith
endswith
iendswith
range
year
month
day
week_day
hour
minute
second
isnull
search
regex
iregex
```

관계를 넘어서는 조회

```python
>>> Entry.objects.filter(blog__name='Beatles Blog')
```

```python
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```

```python
Blog.objects.filter(entry__authors__name='Lennon')
```

```python
Blog.objects.filter(entry__authors__name__isnull=True)
```

```python
Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)
```

관계를 넘어선 여러값 가져오기

```python
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
```

```python
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
```

필터는 모델의 필드를 참조할 수 있다

```python
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```

```python
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks')* 2)
```

```python
>>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
```

```python
>>> Entry.objects.filter(authors__name=F('blog__name'))
```

```python
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```

```python
>>> F('somefield').bitand(16)
```
pk 조회

```python
>>> Blog.objects.get(id__exact=14) # Explicit form
>>> Blog.objects.get(id=14) # __exact is implied
>>> Blog.objects.get(pk=14) # pk implies id__exact 
```

```python
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```

```python
>>> Entry.objects.filter(blog__id__exact=3) # Explicit form
>>> Entry.objects.filter(blog__id=3)        # __exact is implied
>>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact 
```
LIKE문에서 %기호 및 밑줄이스케이프
```python
>>> Entry.objects.filter(headline__contains='%')
```

```python
SELECT ... WHERE headline LIKE '%\%%'
```

캐싱과 쿼리셋

```python
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```

```python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
```

쿼리셋이 캐싱되지 않은 경우

```python
>>> queryset = Entry.objects.all()
>>> print queryset[5] # Queries the database
>>> print queryset[5] # Queries the database again 
```

```python
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print queryset[5] # Uses cache
>>> print queryset[5] # Uses cache 
```

```python
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
```

Q객체를 사용한 복잡한 조회

```python
from django.db.models import Q
Q(question__startswith='What')
```

```python
Q(question__startswith='Who') | Q(question__startswith='What')
```

```sql
WHERE question LIKE 'Who%' OR question LIKE 'What%'
```

```python
Q(question__startswith='Who') | ~Q(pub_date__year=2005)
```

```python
Poll.objects.get(
    Q(question__startswith='Who'),
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
```

```sql
SELECT * from polls WHERE question LIKE 'Who%' AND (pub_date = '2005-05-02' OR pub_da\
te = '2005-05-06') 
```

```python
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith='Who')
```

```python
# INVALID QUERY
Poll.objects.get(
    question__startswith='Who',
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)))
```

객체 비교하기

```python
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id 
```

```python
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name 
```

객체 삭제
```python
e.delete()
```

```python
Entry.objects.filter(pub_date__year=2005).delete()
```

```python
b = Blog.objects.get(pk=1)
# This will delete the Blog and all of its Entry objects.
b.delete()
```

```python
Entry.objects.all().delete()
```

모델 인스턴스 복사

```python
>>> blog = Blog(name='My blog', tagline='Blogging is easy')
>>> blog.save() # blog.pk == 1
>>> blog.pk = None
>>> blog.save() # blog.pk == 2 
```

```python
>>> class ThemeBlog(Blog):
...     theme = models.CharField(max_length=200)

>>> django_blog = ThemeBlog(name=’Django’, tagline=’Django is easy’, theme=’python’)
>>> django_blog.save() # django_blog.pk == 3
```

```python
>>> django_blog.pk = None
>>> django_blog.id = None
>>> django_blog.save() # django_blog.pk == 4 
```

```python
>>> entry = Entry.objects.all()[0] # some previous entry
>>> old_authors = entry.authors.all()
>>> entry.pk = None
>>> entry.save()
>>> entry.authors = old_authors # saves new many2many relations 
```

한번에 여러객체 업데이트

```python
# Update all the headlines with pub_date in 2007.
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
```

```python
>>> b = Blog.objects.get(pk=1)

# Change every Entry so that it belongs to this Blog.
>>> Entry.objects.all().update(blog=b)
```

```python
>>> b = Blog.objects.get(pk=1)

# Update all the headlines belonging to this Blog.
>>> Entry.objects.select_related().filter(blog=b).update  (headline='Everything is th\
e same')
```

```python
for item in my_queryset:
    item.save()
```

```python
>>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
```

```python
# THIS WILL RAISE A FieldError
>>> Entry.objects.update(headline=F('blog__name'))
```

관련된 객체

다대일

```python
>>> e = Entry.objects.get(id=2)
>>> e.blog # Returns the related Blog object.
```

```python
>>> e = Entry.objects.get(id=2)
>>> e.blog = some_blog
>>> e.save()
```

```python
>>> e = Entry.objects.get(id=2)
>>> e.blog = None
>>> e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"
```

```python
>>> e = Entry.objects.get(id=2)
>>> print(e.blog)  # Hits the database to retrieve the associated Blog.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
```

```python
>>> e = Entry.objects.select_related().get(id=2)
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
```


반대로 찾기

```python
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # Returns all Entry objects related to Blog.

# b.entry_set is a Manager that returns QuerySets.
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
```

```python
>>> b = Blog.objects.get(id=1)
>>> b.entries.all() # Returns all Entry objects related to Blog.

# b.entries is a Manager that returns QuerySets.
>>> b.entries.filter(headline__contains='Lennon')
>>> b.entries.count()
```

커스텀 역방향 매니저 사용

```python
from django.db import models

class Entry(models.Model):
    #...
    objects = models.Manager()  # Default Manager
    entries = EntryManager()    # Custom Manager

b = Blog.objects.get(id=1)
b.entry_set(manager='entries').all()
```

```python
b.entry_set(manager='entries').is_published()
```

추가 메서드

- add(obj1, obj2, ...) Adds the specified model objects to the related object set.
- create(**kwargs) Creates a new object, saves it and puts it in the related object set. Returns the newly created object.
- remove(obj1, obj2, ...) Removes the specified model objects from the related object set.
- clear() Removes all objects from the related object set.
- set(objs) Replace the set of related objects.


```python
b = Blog.objects.get(id=1)
b.entry_set = [e1, e2]
```

다대다

```python
>>> e = Entry.objects.get(id=3)
>>> e.authors.all() # Returns all Author objects for this Entry.
>>> e.authors.count()
>>> e.authors.filter(name__contains='John')
>>> a = Author.objects.get(id=5)
>>> a.entry_set.all() # Returns all Entry objects for this Author
```
일대일

```python
class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry)
    details = models.TextField()

ed = EntryDetail.objects.get(id=2)
ed.entry # Returns the related Entry object.
```

```python
e = Entry.objects.get(id=2)
e.entrydetail # returns the related EntryDetail object 
```

```python
e.entrydetail = ed 
```
관련객체에 대한 쿼리

```python
Entry.objects.filter(blog=b) # Query using object instance
Entry.objects.filter(blog=b.id) # Query using id from instance
Entry.objects.filter(blog=5) # Query using id directly 
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

