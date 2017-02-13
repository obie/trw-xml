# The Basics

Sometimes you just want an XML representation of an object, and Active
Record models provide easy, automatic XML generation via the `to_xml`
method. Let's play with this method in the console and see what it can
do.

I'll fire up the console for my book-authoring sample application and
find an Active Record object to manipulate.

{lang=irb, linenos=off}
    >> User.find_by(login: 'obie')
     => #<User id: 8, login: "obie", email: "obie@example.com",
        crypted_password: "4a6046804fc4dc3183ad9012fbfee91c85723d8c",
        salt: "399754af1b01cf3d4b87da5478d82674b0438eb8",
        created_at: "2010-05-18 19:31:40", updated_at: "2010-05-18 19:31:40",
        remember_token: nil, remember_token_expires_at: nil,
        authorized_approver: true, client_id: nil, timesheets_updated_at: nil>

There we go, a `User` instance. Let's see that instance as its generic
XML representation.

{lang=irb, linenos=off}
    >> User.find_by(login: 'obie').to_xml
    => "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<user>\n
       <authorized-approver type=\"boolean\">true</authorized-approver>\n
       <salt>399754af1b01cf3d4b87da5478d82674b0438eb8</salt>\n
       <created-at type=\"datetime\">2010-05-18T19:31:40Z</created-at>\n
       <crypted-password>4a6046804fc4dc3183ad9012fbfee91c85723d8c
       </crypted-password>\n  <remember-token-expires-at type=\"datetime\"
       nil=\"true\"></remember-token-expires-at>\n
       <updated-at type=\"datetime\">2010-05-18T19:31:40Z</updated-at>\n
       <id type=\"integer\">8</id>\n  <client-id type=\"integer\"
       nil=\"true\"></client-id>\n  <remember-token nil=\"true\">
       </remember-token>\n <login>obie</login>\n
       <email>obie@example.com</email>\n  <timesheets-updated-at
      type=\"datetime\" nil=\"true\"></timesheets-updated-at>\n</user>\n"

Ugh, that's ugly. Ruby's `print` function might help us out here.

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <authorized-approver type="boolean">true</authorized-approver>
      <salt>399754af1b01cf3d4b87da5478d82674b0438eb8</salt>
      <created-at type="datetime">2010-05-18T19:31:40Z</created-at>

      <crypted-password>4a6046804fc4dc3183ad9012fbfee91c85723d8c
      </crypted-password>
      <remember-token-expires-at type="datetime" nil="true">
      </remember-token-expires-at>
      <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
      <id type="integer">8</id>
      <client-id type="integer" nil="true"></client-id>
      <remember-token nil="true"></remember-token>
      <login>obie</login>
      <email>obie@example.com</email>
      <timesheets-updated-at type="datetime" nil="true"></timesheets-updated-at>
    </user>

Much better! So what do we have here? Looks like a fairly
straightforward serialized representation of our `User` instance in XML.

## Customizing `to_xml` Output

The standard processing instruction is at the top, followed by an
element name corresponding to the class name of the object. The
properties are represented as subelements, with non-string data fields
including a `type` attribute. Mind you, this is the default behavior and
we can customize it with some additional parameters to the `to_xml`
method.

We'll strip down that XML representation of a user to just an email and
login using the `only` parameter. It's provided in a familiar options
hash, with the value of the `:only` parameter as an array:

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(only: [:email, :login])

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <login>obie</login>
      <email>obie@example.com</email>
    </user>

Following the familiar Rails convention, the `only` parameter is
complemented by its inverse, `except`, which will exclude the specified
properties. What if I want my user's email and login as a snippet of XML
that will be included in another document? Then let's get rid of that
pesky instruction too, using the `skip_instruct` parameter.

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(only: [:email, :login], skip_instruct: true)

{lang=xml}
    <user>
      <login>obie</login>
      <email>obie@example.com</email>
    </user>

We can change the root element in our XML representation of `User` and
the indenting from two to four spaces by using the `root` and `indent`
parameters respectively.

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(root: 'employee', indent: 4)

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <employee>
        <authorized-approver type="boolean">true</authorized-approver>
        <salt>399754af1b01cf3d4b87da5478d82674b0438eb8</salt>
        <created-at type="datetime">2010-05-18T19:31:40Z</created-at>
        <crypted-password>4a6046804fc4dc3183ad9012fbfee91c85723d8c</crypted-password>
        <remember-token-expires-at type="datetime" nil="true"></remember-token-expires-at>
        <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
        <id type="integer">8</id>
        <client-id type="integer" nil="true"></client-id>
        <remember-token nil="true"></remember-token>
        <login>obie</login>
        <email>obie@example.com</email>
        <timesheets-updated-at type="datetime" nil="true"></timesheets-updated-at>
    </employee>

By default Rails converts CamelCase and underscore attribute names to
dashes as in `created-at` and `client-id`. You can force underscore
attribute names by setting the `dasherize` parameter to `false`.

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(dasherize: false,
         only: [:created_at, :client_id])

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <created_at type="datetime">2010-05-18T19:31:40Z</created_at>
      <client_id type="integer" nil="true"></client_id>
    </user>

In the preceding output, the attribute type is included. This too can be
configured using the `skip_types` parameter.

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(skip_types: true,
         only: [:created_at, :client_id])

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <created-at>2010-05-18T19:31:40Z</created-at>
      <client-id nil="true"></client-id>
    </user>

## Associations and `to_xml`

So far we've only worked with a base Active Record and not with any of
its associations. What if we wanted an XML representation of not just a
book but also its associated chapters? Rails provides the `:include`
parameter for just this purpose. The `:include` parameter will also take
an array or associations to represent in XML.

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(include: :timesheets)

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <authorized-approver type="boolean">true</authorized-approver>
      <salt>399754af1b01cf3d4b87da5478d82674b0438eb8</salt>
      <created-at type="datetime">2010-05-18T19:31:40Z</created-at>
      <crypted-password>
        4a6046804fc4dc3183ad9012fbfee91c85723d8c
      </crypted-password>
      <remember-token-expires-at type="datetime"
      nil="true"></remember-token-expires-at>
      <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
      <id type="integer">8</id>
      <client-id type="integer" nil="true"></client-id>
      <remember-token nil="true"></remember-token>
      <login>obie</login>
      <email>obie@example.com</email>
      <timesheets-updated-at type="datetime" nil="true"></timesheets-updated-at>
      <timesheets type="array">
        <timesheet>
          <created-at type="datetime">2010-05-04T19:31:40Z</created-at>
          <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
          <lock-version type="integer">0</lock-version>
          <id type="integer">8</id>
          <user-id type="integer">8</user-id>
          <submitted type="boolean">true</submitted>
          <approver-id type="integer">7</approver-id>
        </timesheet>
        <timesheet>
          <created-at type="datetime">2010-05-18T19:31:40Z</created-at>
          <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
          <lock-version type="integer">0</lock-version>
          <id type="integer">9</id>
          <user-id type="integer">8</user-id>
          <submitted type="boolean">false</submitted>
          <approver-id type="integer" nil="true"></approver-id>
        </timesheet>
        <timesheet>
          <created-at type="datetime">2010-05-11T19:31:40Z</created-at>
          <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
          <lock-version type="integer">0</lock-version>
          <id type="integer">10</id>
          <user-id type="integer">8</user-id>
          <submitted type="boolean">false</submitted>
          <approver-id type="integer" nil="true"></approver-id>
        </timesheet>
      </timesheets>
    </user>

Rails has a much more useful `to_xml` method on core classes. For example,
arrays are easily serializable to XML, with element names
inferred from the name of the Ruby type:

{lang=irb, linenos=off}
    >> print ['cat', 'dog', 'ferret'].to_xml

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <strings type="array">
      <string>cat</string>
      <string>dog</string>
      <string>ferret</string>
    </strings>

If you have mixed types in the array, this is also reflected in the XML
output:

{lang=irb, linenos=off}
    >> print [3, 'cat', 'dog', :ferret].to_xml

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <objects type="array">
      <object type="integer">3</object>
      <object>cat</object>
      <object>dog</object>
      <object type="symbol">ferret</object>
    </objects>

To construct a more semantic structure, the `root` option on `to_xml`
triggers more expressive element names:

{lang=irb, linenos=off}
    >> print ['cat', 'dog', 'ferret'].to_xml(root: 'pets')

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <pets type="array">
      <pet>cat</pet>
      <pet>dog</pet>
      <pet>ferret</pet>
    </pets>

Ruby hashes are naturally representable in XML, with keys corresponding
to element names, and their values corresponding to element contents.
Rails automatically calls `to_s` on the values to get string values for
them:

{lang=irb, linenos=off}
    >> print({owners: ['Chad', 'Trixie'], pets: ['cat', 'dog', 'ferret'],
         id: 123}.to_xml(root: 'registry'))

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <registry>
      <pets type="array">
        <pet>cat</pet>
        <pet>dog</pet>
        <pet>ferret</pet>
      </pets>
      <owners type="array">
        <owner>Chad</owner>
        <owner>Trixie</owner>
      </owners>
      <id type="integer">123</id>
    </registry>

I> #### Josh G says...
I>
I> This simplistic serialization may not be appropriate for certain
I> interoperability contexts, especially if the output must pass XML Schema
I> (XSD) validation when the order of elements is often important. In Ruby
I> 1.9.x and 2.0, the `Hash` class uses insertion order. This may not be
I> adequate for producing output that matches an XSD. The section "The XML
I> Builder" will discuss `Builder::XmlMarkup` to address this situation.

The `:include` option of `to_xml` is not used on `Array` and `Hash`
objects.

## Advanced Usage

By default, Active Record's `to_xml` method only serializes persistent
attributes into XML. However, there are times when transient, derived,
or calculated values need to be serialized out into XML form as well.
For example, our `User` model has a method that returns only draft
timesheets:

{lang=ruby}
    class User < ActiveRecord::Base
      ...
      def draft_timesheets
        timesheets.draft
      end
      ...
    end

To include the result of this method when we serialize the XML, we use
the `:methods` parameter:

{lang=irb, linenos=off}
    >> print User.find_by(login: 'obie').to_xml(methods: :draft_timesheets)

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <id type="integer">8</id>
      ...
      <draft-timesheets type="array">
        <draft-timesheet>
          <created-at type="datetime">2010-05-18T19:31:40Z</created-at>
          <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
          <lock-version type="integer">0</lock-version>
          <id type="integer">9</id>
          <user-id type="integer">8</user-id>
          <submitted type="boolean">false</submitted>
          <approver-id type="integer" nil="true"></approver-id>
        </draft-timesheet>
        <draft-timesheet>
          <created-at type="datetime">2010-05-11T19:31:40Z</created-at>
          <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
          <lock-version type="integer">0</lock-version>
          <id type="integer">10</id>
          <user-id type="integer">8</user-id>
          <submitted type="boolean">false</submitted>
          <approver-id type="integer" nil="true"></approver-id>
        </draft-timesheet>
      </draft-timesheets>
    </user>

We could also set the `methods` parameter to an array of method names to
be called.

## Dynamic Runtime Attributes

In cases where we want to include extra elements unrelated to the object
being serialized, we can pass `to_xml` a block, or use the `:procs`
option.

If we are using the same logic applied to different `to_xml` calls, we
can construct lambdas ahead of time and use one or more of them in the
`:procs` option. They will be called with `to_xml`'s option hash,
through which we access the underlying `XmlBuilder`. (`XmlBuilder`
provides the principal means of XML generation in Rails.

{lang=irb, linenos=off}
    >> current_user = User.find_by(login: 'admin')
    >> generated_at = lambda { |opts| opts[:builder].tag!('generated-at',
         Time.now.utc.iso8601) }
    >> generated_by = lambda { |opts| opts[:builder].tag!('generated-by',
         current_user.email) }

    >> print(User.find_by(login: 'obie').to_xml(procs: [generated_at,
         generated_by]))

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      ...
      <id type="integer">8</id>
      <client-id type="integer" nil="true"></client-id>
      <remember-token nil="true"></remember-token>
      <login>obie</login>
      <email>obie@example.com</email>
      <timesheets-updated-at type="datetime" nil="true"></timesheets-updated-at>
      <generated-at>2010-05-18T19:33:49Z</generated-at>
      <generated-by>admin@example.com</generated-by>
    </user>

{lang=irb, linenos=off}
    >> print Timesheet.all.to_xml(procs: [generated_at, generated_by])

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <timesheets type="array">
      <timesheet>
        ...
        <id type="integer">8</id>
        <user-id type="integer">8</user-id>
        <submitted type="boolean">true</submitted>
        <approver-id type="integer">7</approver-id>
        <generated-at>2010-05-18T20:18:30Z</generated-at>
        <generated-by>admin@example.com</generated-by>
      </timesheet>
      <timesheet>
        ...
        <id type="integer">9</id>
        <user-id type="integer">8</user-id>
        <submitted type="boolean">false</submitted>
        <approver-id type="integer" nil="true"></approver-id>
        <generated-at>2010-05-18T20:18:30Z</generated-at>
        <generated-by>admin@example.com</generated-by>
      </timesheet>
      <timesheet>
        ...
        <id type="integer">10</id>
        <user-id type="integer">8</user-id>
        <submitted type="boolean">false</submitted>
        <approver-id type="integer" nil="true"></approver-id>
        <generated-at>2010-05-18T20:18:30Z</generated-at>
        <generated-by>admin@example.com</generated-by>
      </timesheet>
    </timesheets>

Note that the `:procs` are applied to each top-level resource in the
collection (or the single resource if the top level is not a
collection). Use the sample application to compare the output with the
output from the following:

{lang=irb, linenos=off}
    >> print User.all.to_xml(include: :timesheets, procs: [generated_at, generated_by])

To add custom elements only to the root node, `to_xml` will yield an
`XmlBuilder` instance when given a block:

{lang=irb, linenos=off}
    >> print(User.all.to_xml { |xml| xml.tag! 'generated-by', current_user.email })

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <users type="array">
      <user>...</user>
      <user>...</user>
      <generated-by>admin@example.com</generated-by>
    </users>

Unfortunately, both `:procs` and the optional block are hobbled by a
puzzling limitation: The record being serialized is not exposed to the
procs being passed in as arguments, so only data external to the object
may be added in this fashion.

To gain complete control over the XML serialization of Rails objects,
you need to override the `to_xml` method and implement it yourself.

## Overriding `to_xml`

Sometimes you need to do something out of the ordinary when trying to
represent data in XML form. In those situations you can create the XML
by hand.

{lang=ruby}
    class User < ActiveRecord::Base
      ...
      def to_xml(options = {}, &block)
       xml = options[:builder] || ::Builder::XmlMarkup.new(options)
       xml.instruct! unless options[:skip_instruct]
       xml.user do
         xml.tag!(:email, email)
       end
      end
      ...
    end

This would give the following result:

{lang=irb, linenos=off}
    >> print User.first.to_xml

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?><user><email>admin@example.com</email></user>

Of course, you could just go ahead and use good Object Oriented design
and use a class responsible for translating between your model and an
external representation.





