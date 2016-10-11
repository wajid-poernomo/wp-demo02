# wp-demo02
This is a simple REST api using RAML and MULE. To run this project you will need Anypoint Studio already installed.

1. Import the project into your workspace within Anypoint Studio.
2. Ensure that the mockservice project is running (see other repository).
3. Run As -> Mule Application.

Using postman, you can hit the API's resources, which aid discovery through collection-item pattern and hypermedia.

examples:

http://localhost:8081/weatherApi/countries
http://localhost:8081/weatherApi/countries/Algeria

and 

http://localhost:8081/weatherApi/cities
which outputs:
{
  "cities": [
    {
      "name": "Tlemcen Zenata",
      "description": "Tlemcen Zenata - a great city to be.",
      "link": "~/weatherApi/weather?country=Algeria&city=Tlemcen Zenata"
    },    
    ....
 }
