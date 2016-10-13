# wp-demo02

**This is a simple REST api using RAML and MULE. To run this project you will need Anypoint Studio already installed.**

**Installation**

1. Import the project into your workspace within Anypoint Studio, and make sure the project settings have jdk 8 set as a JRE system library.
2. Ensure that the mockservice project is running (see https://github.com/wajid-poernomo/wp-demo-mockservice).
3. Run As -> Mule Application.


**Technical Specification:**

The following API is designed for discoverability and ease of use through the collection-item pattern and hypermedia. Because REST is a constraints driven
approach, I have used http status codes to indicate validation (400), empty results (404), and server errors(500).

The intention is to provide a REST based API interface/gateway for the older soap based web service http://www.webservicex.net/globalweather.asmx?wsdl.

**Hypermedia**

If a apps developer wanted to drill down to weather using the old SOAP service, they might need to make a call the service to get the countries, 
then a separate call to cities, and while at the same time managing the results as application state. 

Instead of this when they call:

http://localhost:8081/weatherApi/countries <br />

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
Then when they navigate to the following, there are additional links to the weather for the cities:

http://localhost:8081/weatherApi/cities/Finland <br />

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

In this way the "application state" is managed at the api/resource level.

**Collection-Item Pattern**

Usage of the collection-item pattern was used as a way of promoting reusable code in RAML, as well as a "discoverable api".
Some examples are provided here:

status 200:

http://localhost:8081/weatherApi/countries <br />
http://localhost:8081/weatherApi/countries/Algeria <br />
http://localhost:8081/weatherApi/cities  <br /> 
http://localhost:8081/weatherApi/cities/Tlemcen%20Zenata <br />

**Error Codes**

Error code examples are as follows (404):

http://localhost:8081/weatherApi/cities/hobbiton
http://localhost:8081/weatherApi/countries/leggoland

Validation error - city or country name less than 2 characters (400):

http://localhost:8081/weatherApi/cities/a <br />
http://localhost:8081/weatherApi/countries/a <br />
http://localhost:8081/weatherApi/weather?country=a&city=b <br />

500s are covered in unit tests. 


**Running Unit Tests.**

To run the unit tests:

1. Go to /src/test/munit/api-apikit-test.xml and double click.
2. right click on the "message flow" and select "Run MUnit Suite".

These unit tests are based on the RAML file with coverage of the 400 (bad input), 404 (not found), and 500 (server error) error codes.


**Technical Commentary**

Working with a design first approach using RAML was excellent, and I was able to get more clarity of intention working in this way.

I also found the unit test scaffolding based on the RAML extremely useful.

Because of the CDATA payloads, in addition to creating some issues both in terms of the mock web service, and the unit testing mocks, I found I needed to add routing based 
on string comparison to avoid issues with transformation.

In general being stricter around format and structure of messages flowing in and out of components, as well as separating into subflows, would be good design approaches 
to avoid these two issues. Also in terms of unit testing avoiding mocking the web service consumer directly might be a better practice going forwards.

One issue I had here was wanting more granularity in exception handling,
and the 500 is currently mapped to java.lang.Exception as a fall through unless a more specific
exception is thrown (as in the case of 404 and 400). This would definitely warrant more design going further.

Another issue have found mocking CDATA responses and encoded strings sometimes becomes decoded to xml 
when touching the api-apikit-test.xml file. This appears to be Anypoint studio trying to help, but actually creates incorrect results when 
running the mock responses through dataweave. Following the above design approaches going forwards would help guard against this. 