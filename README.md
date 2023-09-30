# New syntax `Trailing Method` 


## abstract

This pep proposes a new syntax named `trailing compound statement` for instantiating 
complex objects and organized setting attributes.

`Trailing compound statement`, or namely `trailing block`, is an indent code block as well as 
variable scope right after an object is instantiated. The local variables within such 
block are caught and pass to the dunder method `__trailing__` of the forward object.

`Trailing block` syntax may in the first glance seems the `with` statement without `with` keyword itself. 
However, the purpose and what focus on are significantly different. `Trailing block` has nothing to 
do with ***defer*** usage. It is designed to handling complex works listed below, with a clear syntax 
and nice scope arranging.  

`Trailing block` is highly inspired by `JSX syntax` of `ReactJS` and `SwiftUI`.


## motivation 

One may feel the coding works in the following scenarios are tedious, less clear and not so pythonic.


### instantiate ***Container*** objects along with its attributes & children

***Container*** objects here refers to the object
+ that only provides a container or frame after constructing;
+ that has many optional attributes;
+ that can has recursively children.


For example, if a developer is constructing a html document with purely python code, instead of 
together with templates, aiming to represent a valid html document:

```html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Website</title>
    <link rel="stylesheet" href="./style.css">
    <link rel="icon" href="./favicon.ico" type="image/x-icon">
  </head>
  <body>
    <h1>Welcome to My Website</h1>
    <ol>
        <li><span>first</span></li>
        <li>second</li>
        <li>third</li>
        <li>fourth</li>
    </ol>
    <button onclick="function(){ console.log('Hello!') }">Greet</button>
    <button onclick="function(){ console.log('Hello again!') }">Greet more</button>
  </body>
</html>

```

one may write these code:

```python

class BaseHTMLElement:
    ...
    
    def addChild(self, child: Self):
        ...
    

html = HTMLRootElement(lang='en')

head = HeadElement(title='My Website')

meta = HeadMetaElement()
meta.charset = 'UTF-8'
meta.viewport = {
    'width': 'device-width',
    'initial_scale': 1.0,
}
head.addChild(meta)

link0 = LinkElement(rel='stylesheet', href='./style.css')
link1 = LinkElement(rel='icon', href='./favicon.ico', type='image/x-icon')
head.addChild(link0)
head.addChild(link1)

body = BodyElement()

h1 = H1Element(inner_text='Welcome to My Website')
body.addChild(h1)

ol = OrderedListElement()

li0 = ListItemElement()
span = SpanElement(inner_text='first')
li0.addChild(span)

li1 = ListItemElement(inner_text='second')
li2 = ListItemElement(inner_text='third')
li3 = ListItemElement(inner_text='fourth')

ol.addChild(li1)
ol.addChild(li2)
ol.addChild(li3)
body.addChild(ol)

def on_click0():
    print('Hello')
    
def on_click1():
    print('Hello again')

button0 = ButtonElement(value='Greet', on_click=on_click0)
button1 = ButtonElement(value='Greet more', on_click=on_click1)

body.addChild(button0)
body.addChild(button1)
```

There are three main issues:
+ too may `addChild` calls;
+ the relationship of parent, child and grandchild cannot be seen clearly;
+ too many temporary child instantiating within the same scope, and one have to put every object a unique name.

With `trailing block` syntax, the code can be much clear:

```python

class BaseHTMLElement:
    ...
    
    def addChild(self, child: Self):
        ...

    def __trailing__(self, *trailer_args, **local_var_kwargs):
        for trailer in trailer_args:
            if isinstance(trail, BaseHTMLElement):
                self.addChild(trailer)

        for var_name in local_var_kwargs:
            var_value = local_var_kwargs[var_name]

            if hasattr(self, var_name):
                setattr(self, var_name, var_value)


html = HTMLRootElement(lang='en'):  # trailing block begins
    HeadElement(title='My Website'):  # sub trailing block
        HeadMetaElement():
            charset = 'UTF-8'
            viewport = {
                'width': 'device-width',
                'initial_scale': 1.0,
            }

        LinkElement():
            rel = 'stylesheet'
            href = './style.css'
        LinkElement():
            rel = 'icon'
            href = './favicon.ico'
            type = 'image/x-icon'

    BodyElement():  # sibling trailing block
        H1Element():
            inner_text = 'Welcome to My Website'

    OrderedListElement():
        ListItemElement():
            SpanElement(inner_text='first'):
                pass
        ListItemElement(inner_text='second'):pass
        ListItemElement(inner_text='third'):pass
        ListItemElement(inner_text='fourth'):pass

    ButtonElement(value='Greet'):
        def on_click():
            print('Hello')

    ButtonElement(value='Greet more'):
        def on_click():
            print('Hello again')
```

Of course, just coding with native `html` saves more works, and python has not to be 
a substitution for writing web pages. The point is, since `html` is the 
most developed DSL for UI design, with the mimic ability as html, python can natively be potent  
as a DSL for UI design, along with the self-speaking GPL ability as `javascript`. Leave the domain of 
web development and turn to GUI development on desktop & mobile, say, one use `PyQt` to develop a GUI app:

...

