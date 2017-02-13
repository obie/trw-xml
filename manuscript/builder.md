# Builder

`Builder::XmlMarkup` is the class used internally by Rails when it needs
to generate XML. When `to_xml` is not enough and you need to generate
custom XML, you will use `Builder` instances directly. Fortunately, the
Builder API is one of the most powerful Ruby libraries available and is
very easy to use, once you get the hang of it.

The API documentation says: "All (well, almost all) methods sent to an
`XmlMarkup` object will be translated to the equivalent XML markup. Any
method with a block will be treated as an XML markup tag with nested
markup in the block."

That is a very concise way of describing how `Builder` works, but it is
easier to understand with some examples, again taken from `Builder`'s
API documentation. The `xm` variable is a `Builder::XmlMarkup` instance:

{lang=ruby}
    xm.em("emphasized") # => <em>emphasized</em>
    xm.em { xm.b("emp & bold") }    # => <em><b>emph &amp; bold</b></em>

    xm.a("foo", "href"=>"http://foo.org")
                                    # => <a href="http://foo.org">foo</a>

    xm.div { br }                   # => <div><br/></div>

    xm.target("name"=>"foo", "option"=>"bar")
                                    # => <target name="foo" option="bar"/>

    xm.instruct!                    # <?xml version="1.0" encoding="UTF-8"?>

    xm.html {                       # <html>
      xm.head {                     #   <head>
        xm.title("History")         #     <title>History</title>
      }                             #   </head>
      xm.body {                     #   <body>
        xm.comment! "HI"            #     <!-- HI -->
        xm.h1("Header")             #     <h1>Header</h1>
        xm.p("paragraph")           #     <p>paragraph</p>
      }                             #   </body>
    }                               # </html>

## Rendering XML Requests

A common use for `Builder::XmlBuilder` is to render XML in response to a
request. Previously we talked about overriding `to_xml` on Active Record
to generate our custom XML. Another way, though not as recommended, is
to use an XML template.

We could alter our `UsersController#show` method to use an XML template
by changing it from:

{lang=ruby}
    def UsersController < ApplicationController
      ...
      def show
        @user = User.find(params[:id])
        respond_to do |format|
          format.html
          format.xml { render xml: @user.to_xml }
        end
      end
      ...
    end

to

{lang=ruby}
    def UsersController < ApplicationController
      ...
      def show
        @user = User.find(params[:id])
        respond_to do |format|
          format.html
          format.xml
        end
      end
      ...
    end

Now Rails will look for a file called `show.xml.builder` in the
`app/views/users` directory. That file contains
`Builder::XmlMarkup` code like

{lang=ruby}
    xml.user {                                # <user>
      xml.email @user.email                   #   <email>...</email>
      xml.timesheets {                        #   <timesheets>
        @user.timesheets.each { |timesheet|   #
          xml.timesheet {                     #     <timesheet>
            xml.draft timesheet.submitted?    #       <draft>true</draft>
          }                                   #     </timesheet>
        }                                     #
      }                                       #   </timesheets>
    }                                         # </user>

In this view the variable `xml` is an instance of `Builder::XmlMarkup`.
Just as in views, we have access to the instance variables we set in our
controller, in this case `@user`. Using the `Builder` in a view can
provide a convenient way to generate XML.