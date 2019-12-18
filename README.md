# deserialize

A library to make deserialization easy. To get started, just run `pip install deserialize`

### How it used to be

Without the library, if you want to convert:

```json
{
    "a": 1,
    "b": 2
}
```

into a dedicated class, you had to do something like this:

```python
class MyThing:

    def __init__(self, a, b):
        self.a = a
        self.b = b

    @staticmethod
    def from_json(json_data):
        a_value = json_data.get("a")
        b_value = json_data.get("b")

        if a_value is None:
            raise Exception("'a' was None")
        elif b_value is None:
            raise Exception("'b' was None")
        elif type(a_value) != int:
            raise Exception("'a' was not an int")
        elif type(b_value) != int:
            raise Exception("'b' was not an int")

        return MyThing(a_value, b_value)

my_instance = MyThing.from_json(json_data)
```

### How it is now

With `deserialize` all you need to do is this:

```python
import deserialize

class MyThing:
    a: int
    b: int

my_instance = deserialize.deserialize(MyThing, json_data)
```

That's it. It will pull out all the data and set it for you type checking and even checking for null values.

If you want null values to be allowed though, that's easy too:

```python
from typing import Optional

class MyThing:
    a: Optional[int]
    b: Optional[int]
```

Now `None` is a valid value for these.

Types can be nested as deep as you like. For example, this is perfectly valid:

```python
class Actor:
    name: str
    age: int

class Episode:
    title: str
    identifier: st
    actors: List[Actor]

class Season:
    episodes: List[Episode]
    completed: bool

class TVShow:
    seasons: List[Season]
    creator: str
```

## Advanced Usage

### Custom Keys

It may be that you want to name your properties in your object something different to what is in the data. This can be for readability reasons, or because you have to (such as if your data item is named `__class__`). This can be handled too. Simply use the `key` annotation as follows:

```python
@deserialize.key("identifier", "id")
class MyClass:
    value: int
    identifier: str
```

This will now assign the data with the key `id` to the field `identifier`. You can have multiple annotations to override multiple keys.

### Ignored Keys

You may want some properties in your object that aren't loaded from disk, but instead created some other way. To do this, use the `ignore` decorator. Here's an example:

```python
@deserialize.ignore("identifier")
class MyClass:
    value: int
    identifier: str
```

When deserializing, the library will now ignore the `identifier` property.

### Parsers

Sometimes you'll want something in your object in a format that the data isn't in. For example, if you get the data:

```python
{
    "successful": True,
    "timestamp": 1543770752
}
```

You may want that to be represented as:

```python
class Result:
    successful: bool
    timestamp: datetime.datetime
```

By default, it will fail on this deserialization as the value in the data is not a timestamp. To correct this, use the `parser` decorator to tell it a function to use to parse the data. E.g.

```python
@deserialize.parser("timestamp", datetime.datetime.fromtimestamp)
class Result:
    successful: bool
    timestamp: datetime.datetime
```

This will now detect when handling the data for the _key_ `timestamp` and run it through the parser function supplied before assigning it to your new class instance.

The parser is run _before_ type checking is done. This means that if you had something like `Optional[datetime.datetime]`, you should ensure your parser can handle the value being `None`. Your parser will obviously need to return the type that you have declared on the property in order to work.


### Subclassing

Subclassing is supported. If you have a type `Shape` for example, which has a subclass `Rectangle`, any properties on `Shape` are supported if you try and decode some data into a `rectangle object.

### Raw Storage

It can sometimes be useful to keep a reference to the raw data that was used to construct an object. To do this, simply set the `raw_storage_mode` paramater to `RawStorageMode.root` or `RawStorageMode.all`. This will store the data in a parameter named `__deserialize_raw__` on the root object, or on all objects in the tree respectively.

### Defaults

Some data will come to you with fields missing. In these cases, a default is often known. To do this, simply decorate your class like this:

```python
@deserialize.default("value", 0)
class IntResult:
    successful: bool
    value: int
```

If you pass in data like `{"successful": True}` this will deserialize to a default value of `0` for `value`. Note, that this would not deserialize since `value` is not optional: `{"successful": True, "value": None}`.
