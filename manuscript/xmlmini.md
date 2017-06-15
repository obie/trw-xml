# XmlMini

The `ActiveSupport::XmlMini` module contains code that allows Rails to serialize/deserialize and parse XML using a number of different libraries.

* JDOM (requires JRuby)
* LibXML (fast native XML parser)
* Nokogiri (requires `nokogiri` gem)

If you're doing anything of significance with XML in your application, you should definitely use the super-fast native `libxml` parser. Install the binaries (instructions vary depending on platform) then the Ruby binding:

{lang=ruby, linenos=off}
    gem 'libxml-ruby', '=0.9.7'

Set `XmlMini` to use `libxml` in `application.rb` or an initializer.

{lang=ruby, linenos=off}
    XmlMini.backend = 'LibXML'

## Constants

The `TYPE_NAMES` constant holds a mapping of Ruby types to their representation when serialized as XML.

{lang=ruby}
    TYPE_NAMES = {
      "Symbol"     => "symbol",
      "Fixnum"     => "integer",
      "Bignum"     => "integer",
      "BigDecimal" => "decimal",
      "Float"      => "float",
      "TrueClass"  => "boolean",
      "FalseClass" => "boolean",
      "Date"       => "date",
      "DateTime"   => "dateTime",
      "Time"       => "dateTime",
      "Array"      => "array",
      "Hash"       => "hash"
    }

The `FORMATTING` constant holds a mapping of lambdas that define how Ruby values are serialized to strings for representation in XML.

{lang=ruby}
    FORMATTING = {
      "symbol"   => Proc.new { |symbol| symbol.to_s },
      "date"     => Proc.new { |date| date.to_s(:db) },
      "dateTime" => Proc.new { |time| time.xmlschema },
      "binary"   => Proc.new { |binary| ::Base64.encode64(binary) },
      "yaml"     => Proc.new { |yaml| yaml.to_yaml }
    }

The `PARSING` constant holds a mapping of lambdas used to deserialize values stored in XML back into Ruby objects.

{lang=ruby}
      PARSING = {
        "symbol"       => Proc.new { |symbol| symbol.to_sym },
        "date"         => Proc.new { |date| ::Date.parse(date) },
        "datetime"     => Proc.new {
          |time| Time.xmlschema(time).utc rescue ::DateTime.parse(time).utc },
        "integer"      => Proc.new { |integer| integer.to_i },
        "float"        => Proc.new { |float|   float.to_f },
        "decimal"      => Proc.new { |number|  BigDecimal(number) },
        "boolean"      => Proc.new {
          |boolean| %w(1 true).include?(boolean.strip) },
        "string"       => Proc.new { |string|  string.to_s },
        "yaml"         => Proc.new { |yaml|    YAML::load(yaml) rescue yaml },
        "base64Binary" => Proc.new { |bin|     ::Base64.decode64(bin) },
        "binary"       => Proc.new { |bin, entity| _parse_binary(bin, entity) },
        "file"         => Proc.new { |file, entity| _parse_file(file, entity) }
      }

{lang=ruby}
      PARSING.update(
        "double"   => PARSING["float"],
        "dateTime" => PARSING["datetime"]
      )
