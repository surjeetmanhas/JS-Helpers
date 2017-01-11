# JS-Helpers


1) String.fornat

if (!String.prototype.format) {
    String.prototype.format = function() {
      var args = arguments;
      return this.replace(/{(\d+)}/g, function(match, number) { 
        return typeof args[number] != 'undefined'
          ? args[number]
          : match
        ;
      });
    };
}

Usage:   "error ({0}) due to {1} or {2}".format(object.ref.depth.code, r1, r2)


2)  URL Builder..


build returns supplied path when NO parameters specified
build returns formatted url when ONE parameter specified
build returns formatted url when MULTIPLE parameters specified
build encodes parameter VALUE if necessary
build encodes parameter NAME if necessary
build trims off LEADING spaces of path
build trims off TRAILING spaces of path
build throws exception if path contains question mark

angular('myModule').service('urlBuilder',
  function() {
    var me = this;
 
    me.build = function(path, paramObject) {
 
      if (path.match(/\?/)) {
        throw new Error('invalid character in path');
      }
      paramObject = paramObject || {};
      var paramList = Object.keys(paramObject)
        .map(function(paramName) {
          // similar to angular's own $httpParamSerializer
          return "{0}={1}".format(
            encodeURIComponent(paramName),
            encodeURIComponent(paramObject[paramName]));
        });
 
      return paramList.length > 0
        ? "{0}?{1}".format(path.trim(), paramList.join('&'))
        : path.trim();
    };
  });
  
  
   "http://foo.com/bar?id="+id+"&term="+termValue       now becomes
 
 urlBuilder.build(
    "http://foo.com/bar", 
    { id: id, term: termValue }
)
   
   
   
   
   
  3) HTML Builder
  
build with NO content and NO attributes returns an empty element
build with content and NO attributes returns an element with content
build with NO content and ONE attribute returns an empty element with attribute
build with content and ONE attribute returns an element with attribute
build with content and NULL attribute returns an element with attribute having no value
build with content and UNDEFINED attribute returns an element with attribute having no value
build with content and EMPTY attribute returns an element with attribute having empty value
build with content and WHITESPACE attribute returns an element with attribute having whitespace value
build with NO content and MULTIPLE attributes returns an empty element with attributes
build with content and MULTIPLE attributes returns an empty element with attributes
build with reserved characters in content encodes them
build with NO content and self-closing flag ON returns an empty element
build with NO content and self-closing flag OFF returns an explicitly closed element
build with NO content and an attribute and self-closing flag OFF returns an explicitly closed element
build with attribute containing DOUBLE quotes uses single quote delimiters
build with attribute containing SINGLE quotes uses double quote delimiters
build throws exception when attribute value contains BOTH double quotes and single quotes
build throws exception when tag contains invalid characters
build throws exception when attribute name contains invalid characters
  
  angular('myModule').service('htmlBuilder',
  function() {
    'use strict';
 
    var me = this;
 
    me.build = function(tag, content, attributes, config) {
 
      if (tag.match(/[^a-z]/i)) {
        throw new Error('tag may only contain alphabetic characters');
      }
 
      // process optional arguments
      content = content ? me.htmlEscape(content) : '';
      attributes = attributes || {};
      var allowSelfClosing =
        !config || typeof config.allowSelfClosing === 'undefined'
        ? true
        : config.allowSelfClosing;
 
      // do the work
      var formatString = content || !allowSelfClosing
        ? '<{0}{2}>{1}</{0}>'
        : '<{0}{2}/>';
      var result = formatString.format(
        tag, content, buildAttributeString(attributes));
      return result;
    };
 
    me.htmlEscape = function(str) {
      return String(str)
        .replace(/&/g, '&')
        .replace(/"/g, '"')
        .replace(/'/g, ''')
        .replace(/</g, '<')
        .replace(/>/g, '>');
    };
 
    function buildAttributeString(attributes) {
      var attrList = Object.keys(attributes)
        .map(function(attrName) {
          if (attrName.match(/[ "'>/=]/)) {
            throw new Error(
              'invalid character in attribute name ({0})'
              .format(attrName));
          }
          var attrValue = attributes[attrName];
          if (typeof (attrValue) === 'undefined'
            || attrValue === null) {
            return ' ' + attrName;
          } else {
            if (typeof (attrValue) === 'string'
              && attrValue.match(/".*'|'.*"/)) {
              throw new Error(
                'invalid character in attribute value ({0})'
                .format(attrValue));
            }
            var quote = typeof (attrValue) === 'string'
              && attrValue.match(/"/) ? '\'' : '"';
            return ' {0}={2}{1}{2}' // note leading space
              .format(attrName, attrValue, quote);
          }
        });
      return attrList.join('');
    }
  });
  
  
  the above code checks to see what kind of quotes each attribute uses and accommodates either type. The code also accommodates an HTML peculiarity wherein, even though the syntax might allow something, the semantics do not: the most notorious example of this is the <script> tag, which must be written
  as  <script src="..."></script>  rather than   <script src="..."/> 
  
The builder service by default creates an empty (self-closing) element, but allows you to specify the former behavior via a parameter:

htmlBuilder.build(
    'script',
    undefined,
    { src: 'c:/path/to/my/script.js' },
    { allowSelfClosing: false }
);


By setting config.allowSelfClosing to false, you then get an explicit end tag even with no content.

Something like this…
  "<span class='"+myClass+"' + xyz='5'>" + myContent + "</span>"
  
  now becomes
  
  htmlBuilder.build(
    'span',
    myContent,
    {
        class: myClass, 
        xyz: 5
    }
)



4)  XML Builder

public string GetXyzAsXml()
{
  return xmlBuilder.build(
    "xyz",
    new AttributeList { new Attribute("name", Name) },
    ItemList.Select(pair =>
      xmlBuilder.build(
        "content",
        xmlBuilder.build("key", pair.Key),
        xmlBuilder.build("value", pair.Value)
      )
    ).ToList()
  );
}

The above provides the data, and provides the ordering, but does not deal with the syntax of the XML at all; that has been completely abstracted away into the XML builder. With an XML builder, no longer are you dealing with XML as raw strings. If you always use the xmlBuilder when you need a piece of XML, you have its safety net guaranteeing the string you get back is not a string of meaningless characters, but rather valid XML.



  
  
  
