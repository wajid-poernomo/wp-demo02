# wp-demo02
**This is a simple REST api using RAML and MULE. To run this project you will need Anypoint Studio already installed.**

1. Import the project into your workspace within Anypoint Studio.
2. Ensure that the mockservice project is running (see other repository).
3. Run As -> Mule Application.

**Design and implementation notes:**

The following API calls are intended to aid discovery through collection-item pattern and hypermedia. In addition to this,
to work develop in a constraints based restful way, I have used http status codes to indicate validation (400), empty results (404), 
and server errors(500).

Some examples of "discoverable API" can be seen browsing the following Uris (there are hypermedia links for weather):

with staus 200:

http://localhost:8081/weatherApi/countries
http://localhost:8081/weatherApi/countries/Algeria
http://localhost:8081/weatherApi/cities
http://localhost:8081/weatherApi/cities/Tlemcen%20Zenata

for 404:

http://localhost:8081/weatherApi/cities/hobbiton
http://localhost:8081/weatherApi/countries/leggoland

validation error - city or country name less than 2 characters (400):

http://localhost:8081/weatherApi/cities/a
http://localhost:8081/weatherApi/countries/a
http://localhost:8081/weatherApi/weather?country=a&city=b

500s are covered in unit tests. One issue I had here was wanting more granularity in exception handling,
and the 500 is currently mapped to java.lang.Exception as a fall through unless a more specific
exception is thrown (as in the case of 404 and 400). This would definitely warrant more design going further.

In a sense this is hypermedia as the engine of application state (HATEOAS) - if a apps developer wanted to drill down to weather, 
they might need to make a call to get countries, then get cities, and store these as application state. Instead of this when they call:

http://localhost:8081/weatherApi/countries

they get a response with links to the cities in those countries:
```json
{
  "countries": [
    {
      "name": "Algeria",
      "description": "Algeria - a great country to be.",
      "link": "~/weatherApi/cities/Algeria"
    },
    {
      "name": "Tunisia",
      "description": "Tunisia - a great country to be.",
      "link": "~/weatherApi/cities/Tunisia"
    },
    {
      "name": "Finland",
      "description": "Finland - a great country to be.",
      "link": "~/weatherApi/cities/Finland"
    },
    {
      "name": "Italy",
      "description": "Italy - a great country to be.",
      "link": "~/weatherApi/cities/Italy"
    },
    {
      "name": "United States",
      "description": "United States - a great country to be.",
      "link": "~/weatherApi/cities/United States"
    }
  ]
}
```
Then when they navigate to:

http://localhost:8081/weatherApi/cities/Finland

```json
{
  "cities": [
    {
      "name": "Ahtari",
      "description": "Ahtari - a great city to be.",
      "link": "~/weatherApi/weather?country=Finland&city=Ahtari"
    },
    {
      "name": "Ivalo",
      "description": "Ivalo - a great city to be.",
      "link": "~/weatherApi/weather?country=Finland&city=Ivalo"
    }
  ]
}
```
They have links to the weather for each of the cities.

**Unit Testing.**

To run the unit tests:

1. Go to /src/test/munit/api-apikit-test.xml and double click.
2. right click on the "message flow" and select "Run MUnit Suite".

These unit tests are based on the RAML file with coverage of the 400 (bad input), 404 (not found), and 500 (server error) error codes.

One issue I have found working the the CDATA responses and encoded strings, is that the mock data sometimes becomes decoded to xml 
touching the api-apikit-test.xml file. This appears to be Anypoint studio trying to help, but actually creates incorrect results when running 
the mock responses through dataweave. I've spent some time experimenting with setting mime/types and encoding but currently I have avoid 
auto-saving after I have a working setup.

As a unit testing note, I would probably try to avoid mocking the web service consumer component directly in the future, possibly with some
layer of indirection like sub-flows, partly for the difficulty in mocking the xml responses that I have noted here.

