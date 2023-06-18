# SOLID Principles with Go Examples Every Developer Should Master (review and rewrite in python)

This is a review of https://towardsdev.com/solid-principles-with-go-examples-every-developer-should-master-6bc6f9f2b6ab



## Single Responsibility Principle (SRP)
```python=
class UserRepository:
    def __init__(self):
        self.db = "some db"
    def find_by_id(id):
        // Find user by ID
    def save(user):
        // save to database
```


## Open/Closed Principle (OCP)
```python=
    class UserService:
        def __init__(self):
            self.repo = UserRepository()
        def find_user_by_id(self,id):
            self.repo.find_byd_id(id)
        def save_user(self,user):
            self.repo.save(use)
```


## Liskov Substitution Principle (LSP)
```pytnon=
@dataclass
class User:
    id: int
    name: str
    email: str
    def send_email(self):
        // send email to user
        ...

class Admin(User):
    def send_email(self):
        // send email to admin
        ...
```

## Interface Segregation Principle (ISP)
In my opinon, role interface is the extreme
```python=
class UserService:
    def create(self):
        ...
    def update(self):
        ...
    def delete(self):
        ...
    def get_all_users(self):
        ...
```
```python=
class CreateUser:
    def create(self):
        ...
```

## Dependency Inversion Principle (DIP)

```python=
class Saver:
    def save(self):
        raise NotImplementedError

class DB(Saver):
    def save(self):
        //implementation
        ...

class UserService:
    def __init__(self):
        self.saver = Saver()
    def create_user(self,name):
        self.saver.save(name)
```

