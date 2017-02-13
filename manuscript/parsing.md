# Parsing

Ruby has a full-featured XML library named Nokogiri and it is very powerful. However, if you have basic parsing needs, such as parsing responses from web services, you can use the simple XML parsing capability built into Rails.

## Turning XML into Hashes

Rails lets you turn arbitrary snippets of XML markup into Ruby hashes,
with the `from_xml` method that it adds to the `Hash` class.

To demonstrate, we'll throw together a string of simplistic XML and turn
it into a hash:

{lang=irb, linenos=off}
    >> xml = <<-XML
    <pets>
      <cat>Franzi</cat>
      <dog>Susie</dog>
      <horse>Red</horse>
    </pets>
    XML

{lang=irb}
    >> Hash.from_xml(xml)
    => {"pets"=>{"cat"=>"Franzi", "dog"=>"Susie", "horse"=>"Red"}}

There are no options for `from_xml`. You can also pass it an `IO`
object:

{lang=irb, linenos=off}
    >> Hash.from_xml(File.new('pets.xml'))
    => {"pets"=>{"cat"=>"Franzi", "dog"=>"Susie", "horse"=>"Red"}}

## Typecasting

Typecasting is done by using a `type` attribute in the XML elements. For
example, here's the auto-generated XML for a `User` object.

{lang=irb, linenos=off}
    >> print User.first.to_xml

{lang=xml}
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
      <authorized-approver type="boolean">true</authorized-approver>
      <salt>034fbec79d0ca2cd7d892f205d56ea95174ff557</salt>
      <created-at type="datetime">2010-05-18T19:31:40Z</created-at>
      <crypted-password>98dfc463d9122a1af0a5dc817601de437c69f365
      </crypted-password>
      <remember-token-expires-at type="datetime" nil="true" />
      <updated-at type="datetime">2010-05-18T19:31:40Z</updated-at>
      <id type="integer">7</id>
      <client-id type="integer" nil="true" />
      <remember-token nil="true" />
      <login>admin</login>
      <email>admin@example.com</email>
      <timesheets-updated-at type="datetime" nil="true" />
    </user>

As part of the `to_xml` method, Rails sets attributes called `type` that
identify the class of the value being serialized. If we take this XML
and feed it to the `from_xml` method, Rails will typecast the strings to
their corresponding Ruby objects:

{lang=irb, linenos=off}
    >> Hash.from_xml(User.first.to_xml)
    => {"user"=>{"salt"=>"034fbec79d0ca2cd7d892f205d56ea95174ff557",
        "authorized_approver"=>true,
        "created_at"=>Tue May 18 19:31:40 UTC 2010, "remember_token_expires_at"=>nil,
        "crypted_password"=>"98dfc463d9122a1af0a5dc817601de437c69f365",
        "updated_at"=>Tue May 18 19:31:40 UTC 2010, "id"=>7, "client_id"=>nil,
        "remember_token"=>nil, "login"=>"admin", "timesheets_updated_at"=>nil,
        "email"=>"admin@example.com"}}