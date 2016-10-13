# wp-demo02

**This is a simple REST api using RAML and MULE. To run this project you will need Anypoint Studio already installed.**

**Installation**

1. Import the project into your workspace within Anypoint Studio, and make sure the project settings have jdk 8 set as a JRE system library.
2. Ensure that the mockservice project is running (see https://github.com/wajid-poernomo/wp-demo-mockservice).
3. Run As -> Mule Application.


**Technical Specification:**

The following API calls are intended to aid discovery through collection-item pattern and hypermedia. Because REST is a constraints driven
approach, I have used http status codes to indicate validation (400), empty results (404), and server errors(500).

In terms of hypermedia - if a apps developer wanted to drill down to weather using the old SOAP service, 
they might need to make a call the service to get the countries, then a separate call to cities, and while at the same time store these as application state. 

Instead of this when they call:

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
Then when they navigate to the following, which has additional links to the weather:

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

In this way the "application state" is managed at the api/resource discovery level.

Usage of the collection-item pattern was used as a way of promoting reusable code in RAML, as well as a "discoverable api".
You can be examples here browsing the following Uris (there are hypermedia links for weather):

status 200:

http://localhost:8081/weatherApi/countries
http://localhost:8081/weatherApi/countries/Algeria
http://localhost:8081/weatherApi/cities
http://localhost:8081/weatherApi/cities/Tlemcen%20Zenata

Error code exsamples are as follows (404):

http://localhost:8081/weatherApi/cities/hobbiton
http://localhost:8081/weatherApi/countries/leggoland

validation error - city or country name less than 2 characters (400):

http://localhost:8081/weatherApi/cities/a
http://localhost:8081/weatherApi/countries/a
http://localhost:8081/weatherApi/weather?country=a&city=b

500s are covered in unit tests. 

**Unit Testing.**

To run the unit tests:

1. Go to /src/test/munit/api-apikit-test.xml and double click.
2. right click on the "message flow" and select "Run MUnit Suite".

These unit tests are based on the RAML file with coverage of the 400 (bad input), 404 (not found), and 500 (server error) error codes.


**Technical Commentary**

Working with a design first approach using RAML was excellent, and I was able to get more clarity of intention working in this way.

I also found the unit test scaffolding based on the RAML extremely useful.

One issue I had here was wanting more granularity in exception handling,
and the 500 is currently mapped to java.lang.Exception as a fall through unless a more specific
exception is thrown (as in the case of 404 and 400). This would definitely warrant more design going further.

As noted in the mock service commentary, from a design point of view being stricter around format, mime type and encoding for messages
flowing in and out of components, as well as separating into subflows, would be good design approaches based on my experience here.

I have found working the the CDATA responses and encoded strings, is that the mock data sometimes becomes decoded to xml 
when touching the api-apikit-test.xml file. This appears to be Anypoint studio trying to help, but actually creates incorrect results when 
running the mock responses through dataweave. 

As a unit testing note, I would probably try to avoid mocking the web service consumer component directly in the future, possibly with some
layer of indirection like sub-flows, partly for the difficulty in mocking the xml responses that I have noted here.
