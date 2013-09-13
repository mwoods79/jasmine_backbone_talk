# The Sweet Scent of Jasmine

!SLIDE

# Sweet Scent of Jasmine

### micah.woods@hashrocket.com
### @mwoods79

!SLIDE

# Sweet Scent of Jasmine

* Setup / Install with Rails
* Quickly run through Jasmine features
* Show some examples with Backbone
* Write a few tests

!SLIDE

# Teaspoon

``` ruby
group :development, :test do
  gem "teaspoon"
end
```

`$ rails g teaspoon:install`

`$ brew install phantomjs`

!SLIDE

# Teaspoon

http://localhost:3000/teaspoon

or

`$ teaspoon`

!SLIDE

# Jasmine Basics

``` javascript
describe("A suite", function() {
  var foo;

  beforeEach(function() {
    foo = 1;
  });

  afterEach(function() {
    foo = undefined;
  });

  it("set's foo to 1", function() {
    expect(foo).toBe(1);
  });
});
```

!SLIDE

# Strict Equality

``` javascript
it("contains spec with an expectation", function() {
  expect(true).toBe(true);
  expect(false).not.toBe(true);
});
```

!SLIDE

# Equality

``` javascript
it("works for simple literals and variables", function() {
  expect(12).toEqual(12);
});

it("should work for objects", function() {
  var foo = { a: 12, b: 34 };
  var bar = { a: 12, b: 34 };
  expect(foo).toEqual(bar);
});
```

!SLIDE

# Regex (Match)

``` javascript
it("The 'toMatch' matcher is for regular expressions", function() {
  var message = 'foo bar baz';

  expect(message).toMatch(/bar/);
  expect(message).toMatch('bar');
  expect(message).not.toMatch(/quux/);
});
```

!SLIDE

# Strict Undefined

``` javascript
it("The 'toBeDefined' matcher compares against `undefined`", function() {
  var a = { foo: 'foo' };
  expect(a.foo).toBeDefined();
  expect(a.bar).not.toBeDefined();
});
```

!SLIDE

# Others

``` javascript
expect(null).toBeNull();
expect({}).toBeTruthy();
expect(undefined).toBeFalsy();
expect(["bar", "baz"]).toContain('bar');
expect(e).toBeLessThan(pi);
expect(pi).toBeGreaterThan(e);

function bar() { /* Throws error */ };
expect(bar).toThrow();
```

!SLIDE

# Simple Spy's

``` javascript
var foo = {
  setBar: function(value) {
    this.bar = value;
  }
};

describe("A spy", function() {
  it("tracks that the spy was called", function() {
    spyOn(foo, 'setBar').andCallThrough();
    foo.setBar("baz")
    expect(foo.setBar).toHaveBeenCalled();
    expect(foo.bar).toBe("baz");
  });
});
```

!SLIDE

# Models

``` javascript
App.Models.Todo = (function(Model) {

  var Todo = Model.extend({
    urlRoot: "todo",
    defaults: { title: '', completed: false },
    toggle: function() {
      this.save({complete: !this.get("completed")});
    }
  });

  return Todo

})(Backbone.Model);
```

!SLIDE

``` javascript
describe("App.Models.Todo", function() {
  var todo;
  beforeEach(function() {
    todo = new App.Models.Todo();
  });

  it("has proper default values", function() {
    expect(todo.urlRoot).toBe('todo')
    expect(todo.defaults).toEqual({title: '', completed: false});
  });

  describe("#toggle()", function() {
    it("saves the model when toggled", function() {
      spyOn(todo, "save");
      todo.toggle();
      expect(todo.save).toHaveBeenCalledWith({completed: true});
    });
  });
});
```

!SLIDE

# Collections

``` javascript
App.Collections.Todos = (function(Collection) {
  var Todos = Collection.extend({
    url: 'todo'
    model: app.Todo,

    completed: function () {
      return this.where({completed: true});
    },

    remaining: function () {
      return this.where({completed: false});
    }
  });

  return Todos;
})(Backbone.Collection);
```

!SLIDE

``` javascript
describe("App.Collections.Todos", function() {
  var todos;
  beforeEach(function() {
    todos = new App.Collections.Todos();
  });

  it("has proper default values", function() {
    expect(todos.url).toBe('todo')
    expect(todos.model).toBe(App.Todo)
  });

  describe("filtering", function() {
    beforeEach(function() {
      spyOn(todos, "where")
    });

    it("#completed()", function() {
      todos.completed()
      expect(todos.where).toHaveBeenCalledWith({completed: true});
    });

    it("#remaining()", function() {
      todos.remaining()
      expect(todos.where).toHaveBeenCalledWith({completed: false});
    });
  });
});
```

!SLIDE

# Views

``` javascript
App.Views.TodoView = (function(View) {
  var TodoView = View.extend({
    tagName:  'li',

    template: JST['templates/item_template']

    events: {
      'click .toggle': 'toggleCompleted'
    },

    initialize: function () {
      this.bindModel();
    },

    bindModel: function() {
      this.listenTo(this.model, 'change', this.render);
    },

    render: function () {
      this.$el.html(this.template({model: this.model}));
      return this;
    },

    toggleCompleted: function () {
      this.model.toggle();
    }

  });

  return TodoView;
})(Backbone.View);
```

!SLIDE

``` javascript
describe("App.Views.TodoView", function() {
  var view, model;

  beforeEach(function() {
    model = new App.Models.Todo();
    view  = new App.Views.TodoView({model: model});
  });

  describe("defaults", function() {
    it("is an `li`", function() {
      expect(view.tagName).toBe('li')
    });

    it("uses the item template", function() {
      expect(view.template).toBe(JST['templates/item_template']);
    });

    it("#events", function() {
      expect(view.events).toEqual({
        'click .toggle': 'toggleCompleted'
      });
    });
  });

  describe("#toggleCompleted()", function() {
    it("toggles it's models completed state", function() {
      // why did I do this instead of checking that the models state changed?
      spyOn(model, "toggle");
      view.completed();
      expect(model.toggle).toHaveBeenCalled();
    });
  });

  describe("#initialize()", function() {
    it("listens to model `change`", function() {
      spyOn(App.TodoView.prototype, "listenTo");
      view = new App.TodoView({model: model});
      expect(view.listenTo).toHaveBeenCalledWith(model, 'change', view.render);
    });
  });
});
```
