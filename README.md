# vui-ad-hoc-alexa-recognizer

npm module that provides VUI (voice user interface) special ad-hoc recognizer
designed to parse raw text from the user and map it to the Alexa intents.  It
uses the two files (intents and utterances) that are used to configure Alexa skills.
This allows easy "middleware" implementation that can be placed between a Cortana
or Google assistant skill and the Alexa backend.
Additional configuration can be added through config file.
This can be used either by itself or as part of other vui-xxx projects.

# Repository
This module as well as related vui modules can be found here:
https://github.com/RationalAnimal

# Installation

```shell
npm install vui-ad-hoc-alexa-recognizer --save
```

# Summary

npm module that provides VUI (voice user interface) special ad-hoc recognizer
designed to parse raw text from the user and map it to the Alexa intents.  It
uses the two files (intents and utterances) that are used to configure Alexa skills.
This allows easy "middleware" implementation that can be placed between a Cortana
or Google assistant skill and the Alexa backend.
Additional configuration can be added through config file.
This can be used either by itself or as part of other vui-xxx projects.
It has two pieces of functionality: first, run it offline to generate a json file
that will be used in matching/parsing the text; second it will match the raw
text at run time using the generated json file.

# Usage

Imagine you already have an Alexa skill and you would like to port it to Cortana
or Google Assistant.  Here are examples of files that you will have for your
Alexa skill (these are NOT complete files, you can find the complete sample
files in the test directory):

````shell
> cat test/utterances.txt
````
````
TestIntent test
TestIntent test me
TestIntent test please
TestIntent	test pretty please
TestIntent       test pleeeeeeease
MinionIntent One of the minions is {MinionSlot}
MinionIntent {MinionSlot}
StateIntent {StateSlot}
StateIntent New England includes {StateSlot} as one of it's states
BlahIntent here is my number {BlahSlot}, use it wisely. And here is another one {BlehSlot}, don't squander it
BlahIntent here is {BlahSlot} and {BlehSlot}
AnotherIntent First is {SomeSlot} and then there is {SomeOtherSlot}
````
````shell
> cat test/intents.json
````
````json
{
  "intents": [
    {
      "intent": "TestIntent",
      "slots": []
    },
    {
      "intent": "BlahIntent",
      "slots": [
        {
          "name": "BlahSlot",
          "type": "AMAZON.NUMBER"
        },
        {
          "name": "BlehSlot",
          "type": "AMAZON.NUMBER"
        }
      ]
    },
    {
      "intent": "AnotherIntent",
      "slots": [
        {
          "name": "SomeSlot",
          "type": "SOME"
        },
        {
          "name": "SomeOtherSlot",
          "type": "SOMEOTHER"
        }
      ]
    },
    {
      "intent": "MinionIntent",
      "slots": [
        {
          "name": "MinionSlot",
          "type": "MINIONS"
        }
      ]
    },
    {
      "intent": "StateIntent",
      "slots": [
        {
          "name": "StateSlot",
          "type": "AMAZON.US_STATE"
        }
      ]
    }
  ]
}

````
and also here is an example of a custom slot type file:

````shell
> cat test/minions.txt
````
````
Bob
Steve
Stewart
````

## Generate recognizer.json file

The first step is to generate a run time file - recognizer.json.  This file has
all the information that is needed to parse user text later.  To create it, run
the generator, e.g.:

````shell
node generator.js -i test/intents.json -u test/utterances.txt -c test/config.json
````
 (the example intents.json, utterances.txt, and config.json files are included in the test directory)
This will produce a recognizer.json in the current directory.

For usage, simply run the generator without any arguments:

````shell
node generator.js
````

Note here that you should already have the intents.json and utterances.txt files
as these files are used to configure the Alexa skill.  Config.json is optional,
but highly recommended if you have custom slots that are used by themselves (not
within a large utterance).  This is where you can provide the values of your
custom slot types (either directly or by providing a file name to load them from)

## Parse user text

The second step is to use recognizer.json file at run time to parse the user
text and produce the output json that can be used to set the intent portion of
the request json.  For an example of how to use it, try:

````shell
node matcher.js "Bob"
````
which will produce:

````json
{
  "name": "MinionIntent",
  "slots": {
    "MinionSlot": {
      "name": "MinionSlot",
      "value": "Bob"
    }
  }
}
````
or

````shell
node matcher.js "here is four hundred eighty eight million three hundred fifty two thousand five hundred twelve and also six oh three five five five one two one two"
````
which will produce:

````json
{
  "name": "BlahIntent",
  "slots": {
    "BlahSlot": {
      "name": "BlahSlot",
      "value": "488352512"
    },
    "BlehSlot": {
      "name": "BlehSlot",
      "value": "6035551212"
    }
  }
}
````


````shell
node matcher.js "thirty five fifty one"
````
which will produce:

````json
{
  "name": "FourDigitIntent",
  "slots": {
    "FooSlot": {
      "name": "FooSlot",
      "value": "3551"
    }
  }
}
````

````shell
node matcher.js "sure"
````
which will produce:

````json
{
  "name": "AMAZON.YesIntent",
  "slots": {}
}
````

````shell
> node matcher.js "New England includes New Hampshire as one of its states"
````
which will produce:

````json
{
  "name": "StateIntent",
  "slots": {
    "StateSlot": {
      "name": "StateSlot",
      "value": "New Hampshire"
    }
  }
}
````

````shell
> node matcher.js "My first name is Jim"
````
which will produce:

````json
{
  "name": "FirstNameIntent",
  "slots": {
    "FirstNameSlot": {
      "name": "FirstNameSlot",
      "value": "Jim"
    }
  }
}

````

````shell
> node matcher.js "December thirty first nineteen ninety nine"
````
which will produce:

````json
{
  "name": "DateIntent",
  "slots": {
    "DateSlot": {
      "name": "DateSlot",
      "value": "1999-12-31"
    }
  }
}
````

````shell
> node matcher.js "lets do it on tuesday"
````
which will produce:

````json
{
  "name": "DayOfWeekIntent",
  "slots": {
    "DayOfWeekSlot": {
      "name": "DayOfWeekSlot",
      "value": "tuesday"
    }
  }
}
````


Please note that matcher.js is just a convenience and also serves as an example.
You will NOT be using it at run time (most likely, though some might find the use
for it).

You will probably deploy your code (that uses the parser) to some middleware
layer, like this:

````
Alexa  -------------->    Alexa   <-- middleware <---- Cortana
Skill  <-------------- AWS Lambda --> AWS Lambda ---->  Skill
````

Where in the middleware AWS Lambda (exposed via API Gateway) you will be able to
see the raw user text from Cortana (passed as the "message" field in the request),
then call it same way matcher.js does, get the resulting json and update the
intent in the request from "None" to the resulting json.  The backend Lambda can
then process it further.

Notice that this module currently is being written as a primarily stand alone
solution for the Alexa to Cortana (or Google Assistant) porting.  However, it
will be retrofitted later to fit into the vui-xxx framework.

### Matched custom slot values

There are differences between what kind of values different services may send.
Alexa appears to respect capitalization of the custom slot values (or it sends
lower case versions) while Cortana capitalizes the first letter under some
circumstances, but not always, and also adds a period at the end of utterances
or even other punctuation signs (I've seen entering a zip code changed from
"12345" to "Is 12345 ?"). To keep Cortana
behaving consistently, the returned matches use the capitalization of the custom
slot values supplied in the config file rather than what Cortana will send.
Thus, if your custom slot value (in the config file) is "petunia" then "petunia"
will be returned even if Cortana will send you "Petunia".  Note that the builtin
slot types are not yet configured to do the same (so for and AMAZON.Country you
could get "france" or "France").  This will be fixed in a later release, but for
now please use case insensitive conversions.

### Slot flags

In some cases you would like to match on a particular slot differently from the
standard algorithm.  For example, if you are trying to get the user's first name
you may way to match on ANYTHING the user says so that unusual names are matched.
In this case you can modify your utterances file to include special flags, e.g.:

````
FirstNameIntent My first name is {FirstNameSlot: INCLUDE_WILDCARD_MATCH, EXCLUDE_VALUES_MATCH }
````

These flags will be used in parsing.  There are only two sets of flags: to
include/exclude custom slot values in the matching pattern and to include/exclude
a wildcard in the matching pattern.  If you don't specify either of these, then
INCLUDE_VALUES_MATCH and EXCLUDE_WILDCARD_MATCH will be used as the default.  Also,
if you include by mistake both INCLUDE... and EXCLUDE... for the same flag, the
default value is (silently) going to be used.  If you are concerned you can look
at the generated recognizer.json to see how the flags were parsed.

It would typically not be usufull (at this time with only these two sets of flags)
to specify INCLUDE... for both or EXCLUDE... for both.  If you are going to include
wildcard then there is no reason to include values as well - it will only slow
down the parsing.  If you exclude both then it will be as if you removed that slot
from the utterance completely.

Also note that you have to be very careful when using wildcards.  For example,
imagine this utterance instead of the above example:

````
FirstNameIntent {FirstNameSlot:INCLUDE_WILDCARD_MATCH,EXCLUDE_VALUES_MATCH}
````

This will match on ANYTHING the user says.  So, DON'T use wildcards in "naked"
utterances (ones that use nothing but slots) unless you are absolutely SURE that
that is what you want. This is why these flags exist at the utterance level rather
than intent level.

Also, you should probably NOT specify wildcard matches on slots of many of the
built in intents, such as date or number - this will likely not end well and
it doesn't make sense (you've
been warned). In the future there will be other flags added, possibly specific to
particular built in slot types (e.g. I may add a flag to return only the female
first names from the AMAZON.US_FIRST_NAME type slot or numeric values within
a certain range from the AMAZON.NUMBER type slot).

IMPORTANT NOTE: The flags are currently being parsed, but NOT YET used (they are
ignored as of the current version 0.2.7).  The real parsing functionality will
be added in a day or two and this README file will be updated.

### Dollar values

If a service like Cortana passes a dollar value, e.g. $1000, it will be mapped
to "1000 dollars" as would be expected by an Alexa skill. (Note that if you
want to test it with matcher.js you have to either escape the $ character or
enclose the whole string in '' rather than "" to avoid command line handling
of $)

````shell
> node matcher.js 'the first price is $1000 and the second price is $525000'
````
which will produce:

````json
{
  "name": "PriceIntent",
  "slots": {
    "PriceOneSlot": {
      "name": "PriceOneSlot",
      "value": "1000"
    },
    "PriceTwoSlot": {
      "name": "PriceTwoSlot",
      "value": "525000"
    }
  }
}
````
Note that this is identical to:

````shell
> node matcher.js 'the first price is 1000 dollars and the second price is 525000 dollars'
````
which will produce:

````json
{
  "name": "PriceIntent",
  "slots": {
    "PriceOneSlot": {
      "name": "PriceOneSlot",
      "value": "1000"
    },
    "PriceTwoSlot": {
      "name": "PriceTwoSlot",
      "value": "525000"
    }
  }
}
````

## Optimizations

### Intent parsing order

You can pass to the matching call the name(s) of the intents that you want to
try to match first. (Currently it only supports custom intents, but that's not
a problem since built in intents are very fast).  Then this call
will likely execute much faster.  Since most of the time you know what the next
likely answers (i.e. utterances) are going to be, you can provide them to the
matching call.

````javascript
var result = _matchText("have you been to France", ["CountryIntent"]);
````

### Intent exclusion

In addition to the intent parsing order you can also pass a list of intents to
be excluded from the matching process.  This is useful if you have intents that
have very large sets of custom values and you are pretty sure you don't want to
parse then in a particular place in your skill (i.e. if you are in a flow that
does not include some intents then you should be able to exclude them from
parsing).

````javascript
var result = _matchText("have you been to France", ["CountryIntent"], ["FirstNameIntent"]);
````

## SoundEx support

SoundEx support is in the process of being added, however it's not yet enabled.
SoundEx will allow matching expressions that are not exact matches, but
approximate matches.

## Examples

Right now I am focusing on implementing new features and fixing bugs rather than
creating documentation and tutorials.  Using it is relatively easy, but if you need
to find more information than is listed here and don't want to dig through the
code then you may want to take a look at the test directory.  If you've installed
the npm module you will NOT have it, but you can go to the GitHub repo and
clone it to get the full distribution.  Then look at the unit tests to see what
you can parse and what kinds of results you will be getting. (The unit tests
  are not yet complete, but it's still a very simple and concise "reference").

## Alexa features supported

Currently, you can parse:
1. All Alexa built in intents
2. Utterances without slots
3. Utterances with custom slots
4. Utterances with these built in slot types:
  AMAZON.NUMBER, AMAZON.FOUR_DIGIT_NUMBER, AMAZON.DATE, AMAZON.US_STATE, AMAZON.US_FIRST_NAME, AMAZON.Country, AMAZON.WeekOfDay

More Amazon built in slot types are coming shortly
