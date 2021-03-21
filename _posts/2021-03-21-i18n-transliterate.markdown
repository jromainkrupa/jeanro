---
layout: post
author: jean-romain krupa
title:  "How to match 2 strings with special characters (I18n.transliterate)"
date:   2021-02-23 11:21:51 +0100
categories: jekyll update
---
# How to match 2 strings with special characters (I18n.transliterate)

Last week during the project week at [Le Wagon Lyon](https://www.lewagon.com/fr/lyon) I discovered a new tip I wanted to share.  The idea was to match 2 strings, the first with special characters, the second without. It is a problem I already encounter with the french language and our "é,  è, ô, ê, à,  â...". For example when you scrap things from the web and try to match them with your DB.

```ruby
"Hôpital".something     = "Hopital"
"Ærøskøbing".something  = "AEroskobing"
"Jürgen".something      = "Jurgen"
"Jürgen".something(:de) = "Juergen"

# => We want to create "something" that return true for all of these
```

The first intuition would be to use something like this:

```ruby
"Hôpital".gsub(/ô/,'o')
# => Hopital

"Ærøskøbing".gsub(/Æ/,'AE').gsub(/ø/, 'o')
# => "AEroskobing"
```

It is not super efficient and clearly not scalable. An Idea would be to store all the combinaisons in a hash to be reusable. A hash like below:

```ruby
DEFAULT_APPROXIMATIONS = {
          "À"=>"A", "Á"=>"A", "Â"=>"A", "Ã"=>"A", "Ä"=>"A", "Å"=>"A", "Æ"=>"AE",
          "Ç"=>"C", "È"=>"E", "É"=>"E", "Ê"=>"E", "Ë"=>"E", "Ì"=>"I", "Í"=>"I",
          "Î"=>"I", "Ï"=>"I", "Ð"=>"D", "Ñ"=>"N", "Ò"=>"O", "Ó"=>"O", "Ô"=>"O",
          "Õ"=>"O", "Ö"=>"O", "×"=>"x", "Ø"=>"O", "Ù"=>"U", "Ú"=>"U", "Û"=>"U",
          "Ü"=>"U", "Ý"=>"Y", "Þ"=>"Th", "ß"=>"ss", "à"=>"a", "á"=>"a", "â"=>"a",
          "ã"=>"a", "ä"=>"a", "å"=>"a", "æ"=>"ae", "ç"=>"c", "è"=>"e", "é"=>"e",
          "ê"=>"e", "ë"=>"e", "ì"=>"i", "í"=>"i", "î"=>"i", "ï"=>"i", "ð"=>"d",
          "ñ"=>"n", "ò"=>"o", "ó"=>"o", "ô"=>"o", "õ"=>"o", "ö"=>"o", "ø"=>"o",
          "ù"=>"u", "ú"=>"u", "û"=>"u", "ü"=>"u", "ý"=>"y", "þ"=>"th", "ÿ"=>"y",
          "Ā"=>"A", "ā"=>"a", "Ă"=>"A", "ă"=>"a", "Ą"=>"A", "ą"=>"a", "Ć"=>"C",
          "ć"=>"c", "Ĉ"=>"C", "ĉ"=>"c", "Ċ"=>"C", "ċ"=>"c", "Č"=>"C", "č"=>"c",
          "Ď"=>"D", "ď"=>"d", "Đ"=>"D", "đ"=>"d", "Ē"=>"E", "ē"=>"e", "Ĕ"=>"E",
          "ĕ"=>"e", "Ė"=>"E", "ė"=>"e", "Ę"=>"E", "ę"=>"e", "Ě"=>"E", "ě"=>"e",
          "Ĝ"=>"G", "ĝ"=>"g", "Ğ"=>"G", "ğ"=>"g", "Ġ"=>"G", "ġ"=>"g", "Ģ"=>"G",
          "ģ"=>"g", "Ĥ"=>"H", "ĥ"=>"h", "Ħ"=>"H", "ħ"=>"h", "Ĩ"=>"I", "ĩ"=>"i",
          "Ī"=>"I", "ī"=>"i", "Ĭ"=>"I", "ĭ"=>"i", "Į"=>"I", "į"=>"i", "İ"=>"I",
          "ı"=>"i", "Ĳ"=>"IJ", "ĳ"=>"ij", "Ĵ"=>"J", "ĵ"=>"j", "Ķ"=>"K", "ķ"=>"k",
          "ĸ"=>"k", "Ĺ"=>"L", "ĺ"=>"l", "Ļ"=>"L", "ļ"=>"l", "Ľ"=>"L", "ľ"=>"l",
          "Ŀ"=>"L", "ŀ"=>"l", "Ł"=>"L", "ł"=>"l", "Ń"=>"N", "ń"=>"n", "Ņ"=>"N",
          "ņ"=>"n", "Ň"=>"N", "ň"=>"n", "ŉ"=>"'n", "Ŋ"=>"NG", "ŋ"=>"ng",
          "Ō"=>"O", "ō"=>"o", "Ŏ"=>"O", "ŏ"=>"o", "Ő"=>"O", "ő"=>"o", "Œ"=>"OE",
          "œ"=>"oe", "Ŕ"=>"R", "ŕ"=>"r", "Ŗ"=>"R", "ŗ"=>"r", "Ř"=>"R", "ř"=>"r",
          "Ś"=>"S", "ś"=>"s", "Ŝ"=>"S", "ŝ"=>"s", "Ş"=>"S", "ş"=>"s", "Š"=>"S",
          "š"=>"s", "Ţ"=>"T", "ţ"=>"t", "Ť"=>"T", "ť"=>"t", "Ŧ"=>"T", "ŧ"=>"t",
          "Ũ"=>"U", "ũ"=>"u", "Ū"=>"U", "ū"=>"u", "Ŭ"=>"U", "ŭ"=>"u", "Ů"=>"U",
          "ů"=>"u", "Ű"=>"U", "ű"=>"u", "Ų"=>"U", "ų"=>"u", "Ŵ"=>"W", "ŵ"=>"w",
          "Ŷ"=>"Y", "ŷ"=>"y", "Ÿ"=>"Y", "Ź"=>"Z", "ź"=>"z", "Ż"=>"Z", "ż"=>"z",
          "Ž"=>"Z", "ž"=>"z"
        }
```



Then we could build our 'something' method to **transliterate** our strings thanks to the hash.

```ruby
def transliterate(string)
  # creates an array of letters
  result = string.split('')

  # we iterate other each letters and our hash
  result.each_with_index do |letter,  index|
    DEFAULT_APPROXIMATIONS.each do |key, value|
      if letter.include?(key.to_s)
        # we replace the letter if it's a match
        result[index] = value
      end
    end
  end
  result.join
end

transliterate("Hôpital")
# => "Hopital"
transliterate("Ærøskøbing")
# => "AEroskobing"
transliterate("Jürgen")
# => "Jurgen"
transliterate("Jürgen", locale: :de)
# => wrong number of arguments
```

Our little method works fine for our 3 first examples, let's start to into our last example. we could add rules to our hash by doing something like this.

```ruby
def add(hash)
  hash.each do |key, value|
    DEFAULT_APPROXIMATIONS[key.to_s] = value.to_s
  end
end
```

We could now improve our transliterate method if a rule is passed:

```ruby
def transliterate(string, rule=nil)
  add(rule) if rule
  result = string.split('')

  result.each_with_index do |letter,  index|
    DEFAULT_APPROXIMATIONS.each do |key, value|
      if letter.include?(key.to_s)
        result[index] = value
      end
    end
  end
  result.join
end

p transliterate("Hôpital")
# "Hopital"
p transliterate("Ærøskøbing")
# "AEroskobing"
p transliterate("Jürgen")
# "ecaeuao"
p transliterate("Jürgen", {'ü'=>'ue'})
# "Juergen"
```

Well it roughly (very roughly ) what I18n.transliterate do.

```ruby
I18n.transliterate("Hôspitalisé")
# => "Hospitalise"
I18n.transliterate("Ærøskøbing")
=> "AEroskobing"
```

Pretty cool no ?

The magic happens [here](https://github1s.com/ruby-i18n/i18n/blob/HEAD/lib/i18n/backend/transliterator.rb) to see the full code of the transliterate method.

Thanks to [Guillaume Briday](https://guillaumebriday.fr/) who showed me that.
