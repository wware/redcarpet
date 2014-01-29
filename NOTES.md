Getting this working on Ubuntu 12.04
===

There are big problems with the `apt-get` installation of ruby, rubygems, etc. Follow
these instructions to do a correct current installation of Ruby stuff:

* http://stackoverflow.com/questions/9056008
* http://ryanbigg.com/2010/12/ubuntu-ruby-rvm-rails-and-you/

and you still need `gem install nokogiri; gem install rake-compiler`, which probably
needs to be done as root. Then you can `rake test`.

How this will work
---

The format for RDFa properties embedded in Markdown is going
to work like this. Source markdown:

    [xmlns]: example=http://example.com/
    [about]: example:subject
    Blah blah blah blah {%foaf:predicate rdf:Foobar} blah blah blah.
    [about]: end

should be rendered to Turtle as:

    @prefix example http://example.com/ .
    example:subject foaf:predicate rdf:Foobar .

should be rendered to HTML as something like (maybe span instead of div):

    <div xmlns:example="http://example.com/"
         about="example:subject">
    Blah blah blah blah <span property="foaf:predicate">rdf:Foobar</span>
    blah blah blah.
    </div>

Where the object should have a different appearance in the HTML,
we provide an RDFa version of the object, and a HTML version.

Source markdown:

    [xmlns]: example=http://example.com/
    [about]: example:subject
    Blah blah blah blah {%foaf:predicate rdf:Foobar::Foo Bar} blah blah blah.
    [about]: end

should be rendered to Turtle as:

    @prefix example http://example.com/ .
    example:subject foaf:predicate rdf:Foobar .

which should be rendered to HTML as something like

    <div xmlns:example="http://example.com/"
         about="example:subject">
    Blah blah blah blah <span property="foaf:predicate" content="rdf:Foobar">Foo Bar</span>
    blah blah blah.
    </div>

Maybe don't use labels
---

Instead of the `[xmlns]:` and `[about]:` syntax, it might make sense to use a `{%xmlns XXX}`
syntax of some sort. For `{%about [arg]}`, you open a span if there's an argument (closing
any previously unclosed span) and close the span when there is no argument.

    {%about example:me}Blah blah blah {dc:iq wicked smart} blah blah blah
    {%rdf:golfScore exemplary} blah blah blah {%vanity:looks godlike}.{%about}

However you do it, you'll need to hang onto the `xmlns` things for a document, applying them
to each opening `about` span you create. When you finish rendering the document, they need
to be freed. A `realloc`-ed string buffer would work since the format you'll ultimately want
is a space-separated string anyway. The string buffer should be freed in the "clean up"
portion of `sd_markdown_render` in `ext/redcarpet/markdown.c`, and that's also the file
where you'll be building the string.

Rendering HTML
---

In `ext/redcarpet/markdown.h`, I probably need one or two new span-level callbacks to `sd_callbacks`,
and update all the places affected by that.


More pieces to think about
---

I am only talking about a small subset of RDFa so far. As the
[Wikipedia article](http://en.wikipedia.org/wiki/RDFa#Essence) explains, there
are several more things that will need to be implemented. It would not do to submit a pull request
until they are all in place.

I have not yet talked about: rel, rev, src, href, resource, datatype, typeof.

> The essence of RDFa is to provide a set of attributes that can be used to carry
  metadata in an XML language (hence the 'a' in RDFa).  These attributes are:
  * __about__ - a URI or CURIE specifying the resource the metadata is about
  * __rel__, __rev__ - specifying a relationship and reverse-relationship with another resource, respectively
  * __src__, __href__, __resource__ - specifying the partner resource, are these synonyms for one another?
  * __property__ - specifying a property for the content of an element or the partner resource
  * __content__ - optional attribute that overrides the content of the element when using the property attribute
  * __datatype__ - optional attribute that specifies the datatype of text specified for use with the
    property attribute
  * __typeof__ - optional attribute that specifies the RDF type(s) of the subject or the partner resource
    (the resource that the metadata is about).
