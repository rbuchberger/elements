**This gem is very young and untested. Use with caution.**

# Elements

This is a teeny-tiny gem that builds nicely formatted HTML using sane, clean, readable Ruby. I use
it for writing jekyll plugins, but you can use it anywhere. It's ~100 lines of plain Ruby; it has no
dependencies.

Have you ever tried to build HTML with string concatenation and interpolation? At first it seems
simple, but once you start accounting for all the what-ifs, the indentation, the closing tags, and
the spaces you only need sometimes, it turns into a horrible mess.

The problem, of course, is that building long, complex, varying blocks of text with string
concatenation and interpolation is fragile, unreadable, and painful. You know this, but you're not
going to write an entirely new class just to spit out 10 lines of HTML, so you hammer through it and
end up with code like this:

```ruby
picture_tag = "<picture>\n"\
                      "#{source_tags}"\
                      "#{markdown_escape * 4}<img src=\"#{url}#{instance['source_default'][:generated_src]}\" #{html_attr_string}>\n"\
"#{markdown_escape * 2}</picture>\n"
```

or this: 
```ruby
    def build_li(this_item_data, icon_location, label)
      li = "  <li#{@attributes['li']}>"
      if this_item_data && this_item_data['url']
        li << "<a href=\"#{this_item_data['url']}\"#{@attributes['a']}>"
      end
      li << build_image_tag(icon_location)
      li << label
      li << '</a>' if this_item_data['url']
      li << "</li>\n"
    end
```

That is why I wrote this gem. Here's a demo:

```ruby
p = TagPair.new 'p'
puts p.to_s
# <p></p>

p.add_attributes {class: 'stumpy mopey grumpy', id: 'the-ugly-one'}
puts p.to_s
# <p class="stumpy mopey grumpy", id="the-ugly-one"></p>

p.add_attributes {class: 'slimy'}
puts p.to_s
# <p class="stumpy mopey grumpy slimy", id="the-ugly-one"></p>

p.add_content 'Bippity Boppity Boo!'
puts p.to_s
# <p class="stumpy mopey grumpy slimy", id="the-ugly-one">Bippity Boppity Boo!</p>

p.add_content TagPair.new 'a', content: 'Link!', attributes: {href: 'awesome-possum.com'}
puts p.to_s
# <p class="stumpy mopey grumpy slimy", id="the-ugly-one">Bippity Boppity Boo!<a href="awesome-possum.com">Link!</a></p>

div = p.add_parent TagPair.new 'div'
puts div.to_s
# <div>
#   <p class="stumpy mopey grumpy slimy", id="the-ugly-one">Bippity Boppity Boo!<a href="awesome-possum.com">Link!</a></p>
# </div>

div.add_content Tag.new 'img', attributes: {src: 'happy-puppy.jpg'}
puts div.to_s
# <div>
#   <p class="stumpy mopey grumpy slimy", id="the-ugly-one">Bippity Boppity Boo!<a href="awesome-possum.com">Link!</a></p>
#   <img src="happy-puppy.jpg">
# </div>

```
It doesn't do much to validate your HTML. Put garbage in and you'll get garbage out. That said, it
tries not to be too particular about how you use it. Arguments are pretty flexible about types, you
can build things in pretty much any order or direction, and so on.

## Installation

 - coming eventually. It'll be a gem and a require.
 
## Usage

So we're on the same page, here's the terminology I'm using:
```
<p class="stumpy">Hello</p>
|a|       b      |  c  | d |
```
- a: element
- b: attributes
- c: content
- d: closing tag

There are 2 classes: `Tag` is the base class, and `TagPair` inherits from it. A `Tag` is a
self-closing tag, meaning it has no content and no closing tag. A `TagPair` is the other kind.

### Tag Properties:

 - element:
     **String**, mandatory. What kind of tag it is; such as 'img' or 'hr'
 - attributes:
     **Hash** `{symbol: array || string}`, optional. Example: `{class: 'myclass plaid'}` 
 - newline: 
     **Boolean**. Whether to include a line break after the closing tag. Defaults to true.

### Tag Methods (that you care about)

`Tag.new(element, attributes: {}, newline: true)`
- Make a new tag.

`.to_s`
- The big one. Returns your HTML as a string, nondestructively.

`.reset_attributes(hash)`
- Deletes and overwrites the attributes. Destructive.

`.add_attributes(hash)`
- Appends them instead of overwriting. Destructive.

`.element` - returns the element type

`.add_parent(TagPair)`
returns supplied TagPair, with self added as a child. Nondestructive.

`attr_reader: .attributes and .element`

`attr_accessor: .newline`

### TagPair Properties:

 `TagPair` Inherits all of `Tag`'s properties and methods, but adds content and a closing tag.
 - content:
     **Array**, optional, containing anything (but probably just strings and Tags). Child elements
     are not rendered until calling `.to_s` on the parent, meaning you can access and modify them
     after defining a parent.

### TagPair Methods (that you care about)

`TagPair.new(element, attributes: {}, newline: true, content: [] || string || anything that knows.to_s)`
 - You can initialize it with content. 

`add_content(array, string, or anything else)`

`attr_accessor: content`
- You can modify the content array directly if you like.

`.to_s`
- Just like `Tag`. It splits opening and closing tags to their own lines if the stringified content
    are longer than 80 characters, or if it includes a line break somewhere. Indentation is
    hardcoded at two spaces. If you want that to be configurable, pull requests are welcome. 

## Contributing

I would really love some help on this one. The basic functionality is there, but there are surely
loads of bugs, and I haven't written any tests. I'm considering introducing 'preset' functionality,
where you can do something like `ElementFactory.p` to get a paragraph tag, but it's a bit of work
and I'm not convinced it's any easier than `TagPair.new 'p'`

For pull requests, I've been using rubocop with the default settings and would appreciate if you did
the same.

https://github.com/rbuchberger/elements

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
