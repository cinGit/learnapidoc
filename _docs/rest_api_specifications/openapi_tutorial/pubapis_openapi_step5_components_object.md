---
title: "OpenAPI tutorial step 5: The components object"
permalink: /pubapis_openapi_step5_components_object.html
course: "Documenting REST APIs"
sidebar: docapis
weight: 8.265
section: restapispecifications
path1: /restapispecifications.html
---

{% include workflow_map.html step="5" map="content/openapi_tutorial_map.html"  %}

In the previous step ([OpenAPI tutorial step 4: The paths object](pubapis_openapi_step4_paths_object.html), when you described the [`requestBody`](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#requestBodyObject) and [`responses` object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#responsesObject) objects, you used a [`schema`](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject) object to describe the model for the request or response. The `schema` refers to the fields, values, and structure (for example, the hierarchy and nesting of the various objects) of a JSON or YAML object. (See [What is a schema?](https://spacetelescope.github.io/understanding-json-schema/about.html#what-is-a-schema) for more details.) It's common to use a reference pointer for the `schema` object that points to more details in the `components` object.

{% if site.format == "web" %}
* TOC
{:toc}
{% endif %}

## Reasons to use the components object

Describing the schema of complex responses can be one of the more challenging aspects of the OpenAPI spec. While our Mashape weather API is simple, the response from the `weatherdata` endpoint is complex. Although you can define the schema directly in the `requestBody` or `responses` object, you typically don't list it there for two reasons:

* You might want to re-use parts of the schema in other requests or responses. It's common to have the same object, such as `units` or `days`, appear in multiple places in an API. OpenAPI allows you to re-use these same definitions in multiple places without manually copying and pasting the content.
* You don't want to clutter up your `paths` object with too many request and response details, since the `paths` object is already somewhat complex with several levels of objects.

Instead of listing the schema for your requests and responses in the `paths` object, for more complex schemas (or for schemas that are re-used in multiple operations or paths), you typically use a [reference object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#referenceObject), that is, a reference pointer (`$ref`), that refers to a specific definition in the [`components` object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#componentsObject).

Think of the `components` object like an appendix where the re-usable details are provided. If multiple parts of your spec have the same schema, you point each of these references to the same object in your `components` object, and in so doing you single source the content.

## Objects in components

You can store a lot of different re-usable objects in the `components` object. The [`components` object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#componentsObject) can contain these objects:

* `schemas`
* `responses`
* `parameters`
* `examples`
* `requestBodies`
* `headers`
* `securitySchemes`
* `links`
* `callbacks`

## Storing re-used parameters in components {#reused_parameters}

In the Mashape Weather API, the `lat` and `lng` parameters remain the same for each of the paths. Rather than duplicate this same description in each path, we can store the object in the `components`.

In the `path` object, rather than listing the parameter details, we simply include a `$ref` pointer that refers to the location in `components` that contains these details:

```yaml
parameters:
      - $ref: '#/components/parameters/latParam'
      - $ref: '#/components/parameters/lngParam'
```

Notice that the `parameters` are stored under `components/parameters`. Now let's define the parameters one time:

```yaml
components:
  parameters:
    latParam:
      in: query
      description: "Latitude coordinates."
      required: true
      style: form
      explode: false
      schema:
        type: string
      example: "37.3708698"
    lngParam:
      in: query
      description: "Longitude coordinates."
      required: true
      style: form
      explode: false
      schema:
        type: string
      example: "-122.037593"
```

{: .tip}
See [Using $ref](https://swagger.io/docs/specification/using-ref/) for more details on this standard JSON reference property.

## The weatherdata response

For the `weatherdata` response, although we're not re-using objects across multiple `response` objects in our API, the response is so lengthy and complex, it's going to be better organized under `components`.

If you recall in the previous step ([OpenAPI tutorial step 4: The paths object](pubapis_openapi_step4_paths_object.html), the `responses` object for the `weatherdata` endpoint looked like this:

```yaml
responses:
  200:
    description: Successful operation
    content:
      application/json:
        schema:
          description: Successful operation
          $ref: '#/components/schemas/WeatherdataResponse'
```

The `$ref` points to a definition stored in the `components` object. Before we describe the response in the `components` object, let's look at the `weatherdata` response in detail. This response contains multiple nested objects at various levels.

```json
{
  "query": {
    "count": 1,
    "created": "2017-10-26T02:04:45Z",
    "lang": "en-US",
    "results": {
      "channel": {
        "units": {
          "distance": "km",
          "pressure": "mb",
          "speed": "km/h",
          "temperature": "C"
        },
        "title": "Yahoo! Weather - Sunnyvale, CA, US",
        "link": "http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91990359/",
        "description": "Yahoo! Weather for Sunnyvale, CA, US",
        "language": "en-us",
        "lastBuildDate": "Wed, 25 Oct 2017 07:04 PM PDT",
        "ttl": "60",
        "location": {
          "city": "Sunnyvale",
          "country": "United States",
          "region": " CA"
        },
        "wind": {
          "chill": "82",
          "direction": "305",
          "speed": "17.70"
        },
        "atmosphere": {
          "humidity": "33",
          "pressure": "33999.36",
          "rising": "0",
          "visibility": "25.91"
        },
        "astronomy": {
          "sunrise": "7:26 am",
          "sunset": "6:18 pm"
        },
        "image": {
          "title": "Yahoo! Weather",
          "width": "142",
          "height": "18",
          "link": "http://weather.yahoo.com",
          "url": "http://l.yimg.com/a/i/brand/purplelogo//uh/us/news-wea.gif"
        },
        "item": {
          "title": "Conditions for Sunnyvale, CA, US at 06:00 PM PDT",
          "lat": "37.377499",
          "long": "-122.04686",
          "link": "http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91990359/",
          "pubDate": "Wed, 25 Oct 2017 06:00 PM PDT",
          "condition": {
            "code": "32",
            "date": "Wed, 25 Oct 2017 06:00 PM PDT",
            "temp": "28",
            "text": "Sunny"
          },
          "forecast": [
            {
              "code": "32",
              "date": "25 Oct 2017",
              "day": "Wed",
              "high": "31",
              "low": "15",
              "text": "Sunny"
            },
            {
              "code": "32",
              "date": "26 Oct 2017",
              "day": "Thu",
              "high": "30",
              "low": "17",
              "text": "Sunny"
            },
            {
              "code": "32",
              "date": "27 Oct 2017",
              "day": "Fri",
              "high": "30",
              "low": "15",
              "text": "Sunny"
            },
            {
              "code": "34",
              "date": "28 Oct 2017",
              "day": "Sat",
              "high": "25",
              "low": "12",
              "text": "Mostly Sunny"
            },
            {
              "code": "30",
              "date": "29 Oct 2017",
              "day": "Sun",
              "high": "21",
              "low": "11",
              "text": "Partly Cloudy"
            },
            {
              "code": "28",
              "date": "30 Oct 2017",
              "day": "Mon",
              "high": "20",
              "low": "11",
              "text": "Mostly Cloudy"
            },
            {
              "code": "30",
              "date": "31 Oct 2017",
              "day": "Tue",
              "high": "21",
              "low": "10",
              "text": "Partly Cloudy"
            },
            {
              "code": "30",
              "date": "01 Nov 2017",
              "day": "Wed",
              "high": "21",
              "low": "10",
              "text": "Partly Cloudy"
            },
            {
              "code": "30",
              "date": "02 Nov 2017",
              "day": "Thu",
              "high": "18",
              "low": "10",
              "text": "Partly Cloudy"
            },
            {
              "code": "28",
              "date": "03 Nov 2017",
              "day": "Fri",
              "high": "18",
              "low": "10",
              "text": "Mostly Cloudy"
            }
          ],
          "description": "<![CDATA[<img src=\"http://l.yimg.com/a/i/us/we/52/32.gif\"/>\n<BR />\n<b>Current Conditions:</b>\n<BR />Sunny\n<BR />\n<BR />\n<b>Forecast:</b>\n<BR /> Wed - Sunny. High: 31Low: 15\n<BR /> Thu - Sunny. High: 30Low: 17\n<BR /> Fri - Sunny. High: 30Low: 15\n<BR /> Sat - Mostly Sunny. High: 25Low: 12\n<BR /> Sun - Partly Cloudy. High: 21Low: 11\n<BR />\n<BR />\n<a href=\"http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91990359/\">Full Forecast at Yahoo! Weather</a>\n<BR />\n<BR />\n<BR />\n]]>",
          "guid": {
            "isPermaLink": "false"
          }
        }
      }
    }
  }
}
```

There are a couple of ways to go about describing this. You could create one long description like this:

```yaml
components:
  schemas:
    WeatherdataResponse:
      title: weatherdata response
      type: object
      properties:
        query:
          type: object
          properties:
            count:
              type: integer
              description: The number of items (rows) returned -- specifically, the number of sub-elements in the results property
              format: int32
              example: 1
            created:
              type: string
              description: The date and time the response was created
              example: 6/14/2017 2:30:14 PM
            lang:
              type: string
              description: The locale for the response
              example: en-US
            results:
              type: object
              properties:
                channel:
                  type: object
                  properties:
                    units:
                      description: Units for various aspects of the forecast. Note that the default is to use metric formats for the units (Celsius, kilometers, millibars, kilometers per hour).
                      Units:
                        description: Units for various aspects of the forecast. Note that the default is to use metric formats for the units (Celsius, kilometers, millibars, kilometers per hour).
                        type: object
                        properties:
                          distance:
                            type: string
                            description: Units for distance, mi for miles or km for kilometers
                            example: km
                          pressure:
                            type: string
                            description: Units of barometric pressure, "in" for pounds per square inch or "mb" for millibars.
                            example: mb
                          speed:
                            type: string
                            description: Units of speed, "mph" for miles per hour or "kph" for kilometers per hour.
                            example: km/h
                          temperature:
                            type: string
                            description: Degree units, "f" for Fahrenheit or "c" for Celsius.
                            example: C

                    title:
                      type: string
                      description: The title of the feed, which includes the location city
                      example: Yahoo! Weather - Singapore, South West, SG

                    link:
                      type: string
                      description: The URL for the Weather page of the forecast for this location
                      example: http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91792352/

                    description:
                      type: string
                      description: The overall description of the feed including the location
                      example: Yahoo! Weather for Singapore, South West, SG

                    language:
                      type: string
                      description: The language of the weather forecast
                      example: en-us

                    lastBuildDate:
                      type: string
                      description: The last time the feed was updated. The format is in the date format defined by [RFC822 Section 5](http://www.rfc-editor.org/rfc/rfc822.txt).
                      example: Wed, 14 Jun 2017 10:30 PM SGT

                    ttl:
                      type: string
                      description: Time to Live -- how long in minutes this feed should be cached.
                      example: 60

                    location:
                      description: The location of this forecast
                      type: object
                      properties:
                        city:
                          type: string
                          description: City name
                          example: Singapore
                        country:
                          type: string
                          description: Two-character country code
                          example: Singapore
                        region:
                          type: string
                          description: State, territory, or region, if given
                          example: South West

                    wind:
                      description: Forecast information about wind
                      type: object
                      properties:
                        chill:
                          type: integer
                          description: Wind chill in degrees
                          format: int32
                          example: 81
                        direction:
                          type: integer
                          description: Wind direction, in degrees
                          format: int32
                          example: 158
                        speed:
                          type: integer
                          description: Wind speed, in the units specified in the speed attribute of the units element (mph or kph)
                          format: int32

                    atmosphere:
                      description: Forecast information about current atmospheric pressure, humidity, and visibility
                      type: object
                      properties:
                        humidity:
                          type: integer
                          description: humidity, in percent
                          format: int32
                          example: 87
                        pressure:
                          type: integer
                          description: Barometric pressure, in the units specified by the pressure attribute of the yweather:units element ("in" or "mb").
                          format: int32
                        rising:
                          type: integer
                          description: 'State of the barometric pressure: steady (0), rising (1), or falling (2).'
                          format: int32
                          example: 0
                        visibility:
                          type: integer
                          description: Visibility, in the units specified by the distance attribute of the units element ("mi" or "km"). Note that the visibility is specified as the actual value * 100. For example, a visibility of 16.5 miles will be specified as 1650. A visibility of 14 kilometers will appear as 1400.
                          format: int32

                    astronomy:
                      description: Forecast information about current astronomical conditions
                      type: object
                      properties:
                        sunrise:
                          type: string
                          description: Today's sunrise time. The time is a string in a local time format of "h:mm am/pm"
                          example: 6:59 am
                        sunset:
                          type: string
                          description: Today's sunset time. The time is a string in a local time format of "h:mm am/pm"
                          example: 7:11 pm

                    image:
                      description: The image used to identify this feed
                      type: object
                      properties:
                        title:
                          type: string
                          description: The title of the image.
                          example: Yahoo! Weather
                        width:
                          type: string
                          description: The width of the image, in pixels
                          example: 142
                        height:
                          type: string
                          description: The height of the image, in pixels
                          example: 18
                        link:
                          type: string
                          description: Description of link
                          example: http://weather.yahoo.com
                        url:
                          type: string
                          description: The URL of Yahoo! Weather
                          example: http://l.yimg.com/a/i/brand/purplelogo//uh/us/news-wea.gif

                    item:
                      description: The item element contains current conditions and forecast for the given location. There are multiple weather forecast elements for today and tomorrow.
                      type: object
                      properties:
                        title:
                          type: string
                          description: The forecast title and time.
                          example: Conditions for Singapore, South West, SG at 09:00 PM SGT
                        lat:
                          type: string
                          description: The latitude of the location.
                          example: 1.33464
                        long:
                          type: string
                          description: The longitude of the location.
                          example: 103.726471
                        link:
                          type: string
                          description: The Yahoo! Weather URL for this forecast.
                          example: http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91792352/
                        pubDate:
                          type: string
                          description: The date and time this forecast was posted, in the date format defined by [RFC822 Section 5](http://www.rfc-editor.org/rfc/rfc822.txt).
                          example: Wed, 14 Jun 2017 09:00 PM SGT
                        condition:
                          description: The current weather conditions
                          type: object
                          properties:
                            code:
                              type: integer
                              description: The condition code for this forecast. You could use this code to choose a text description or image for the forecast. The possible values for this element are described in the [Condition Codes](https://developer.yahoo.com/weather/).documentation.html.
                              format: int32
                              example: 33
                            date:
                              type: string
                              description: The current date and time for which this forecast applies. The date is in RFC822 Section 5 format.
                              example: Wed, 14 Jun 2017 09:00 PM SGT
                            temp:
                              type: integer
                              description: The current temperature, in the units specified by the units element.
                              format: int32
                              example: 26
                            text:
                              type: string
                              description: A textual description of conditions
                              example: Mostly Clear


                        forecast:
                          description: The item element contains current conditions and forecast for the given location. There are multiple weather forecast elements for today and tomorrow.
                          type: array
                          items:
                            type: object
                            properties:
                              code:
                                type: integer
                                description: The condition code for this forecast. You could use this code to choose a text description or image for the forecast. The possible values for this element are described in the [Condition Codes](https://developer.yahoo.com/weather/documentation.html#codes).
                                format: int32
                                example: 4
                              date:
                                type: string
                                description: The date to which this forecast applies. The date is in 'dd Mmm yyyy' format, for example '30 Nov 2005'
                                example: 14 Jun 2017
                              day:
                                type: string
                                description: Day of the week to which this forecast applies. Possible values are Mon Tue Wed Thu Fri Sat Sun.
                                example: Wed
                              high:
                                type: integer
                                description: The forecasted high temperature for this day, in the units specified by the units element.
                                format: int32
                                example: 28
                              low:
                                type: integer
                                description: The forecasted low temperature for this day, in the units specified by the units element.
                                format: int32
                                example: 25
                              text:
                                type: string
                                description: A textual description of conditions
                                example: Thunderstorms
                            description: The weather forecast for a specific day. The item element contains multiple forecast elements for today and tomorrow.
                          Guid:
                            title: guid
                            type: object
                            properties:
                              isPermaLink:
                                type: string
                                description: The attribute isPermaLink is false.
                                example: false
                            description: Unique identifier for the forecast, made up of the location ID, the date, and the time.
                        description:
                          type: string
                          description: A simple summary of the current conditions and tomorrow's forecast, in HTML format, including a link to Yahoo! Weather for the full forecast.
                          example: "<![CDATA[<img src=\"http:\/\/l.yimg.com\/a\/i\/us\/we\/52\/4.gif\"\/>\n<BR \/>\n<b>Current Conditions:<\/b>\n<BR \/>Thunderstorms\n<BR \/>\n<BR \/>\n<b>Forecast:<\/b>\n<BR \/> Fri - Thunderstorms. High: 30Low: 25\n<BR \/> Sat - Thunderstorms. High: 28Low: 25\n<BR \/> Sun - Thunderstorms. High: 28Low: 25\n<BR \/> Mon - Thunderstorms. High: 28Low: 25\n<BR \/> Tue - Thunderstorms. High: 28Low: 25\n<BR \/>\n<BR \/>\n<a href=\"http:\/\/us.rd.yahoo.com\/dailynews\/rss\/weather\/Country__Country\/*https:\/\/weather.yahoo.com\/country\/state\/city-91792352\/\">Full Forecast at Yahoo! Weather<\/a>\n<BR \/>\n<BR \/>\n(provided by <a href=\"http:\/\/www.weather.com\" >The Weather Channel<\/a>)\n<BR \/>\n]]>"
                        guid:
                          type: string
                          description: Unique identifier for the forecast, made up of the location ID, the date, and the time.
                          format: uuid
```

One challenge is that it's difficult to keep all the levels straight. With so many nested objects, it's dizzying and confusing. Additionally, it's easy to make mistakes. Worst of all, you can't re-use the individual objects. This undercuts one of the main reasons for storing this object in `components` in the first place.

Another approach is to make each object its own entity in the `components`. Whenever an object contains an object, add a `$ref` value that points to the new object. This way objects remain shallow and you won't get lost in a sea of confusing sublevels.

```yaml
components:
  schemas:
    WeatherdataResponse:
      title: weatherdata response
      type: object
      properties:
        query:
          $ref: '#/components/schemas/Query'
    Query:
      title: query
      type: object
      properties:
        count:
          type: integer
          description: The number of items (rows) returned -- specifically, the number of sub-elements in the results property
          format: int32
          example: 1
        created:
          type: string
          description: The date and time the response was created
          example: 6/14/2017 2:30:14 PM
        lang:
          type: string
          description: The locale for the response
          example: en-US
        results:
          $ref: '#/components/schemas/Results'
    Results:
      title: results
      type: object
      properties:
        channel:
          $ref: '#/components/schemas/Channel'
    Channel:
      title: channel
      type: object
      properties:
        units:
          description: Units for various aspects of the forecast. Note that the default is to use metric formats for the units (Celsius, kilometers, millibars, kilometers per hour).
          $ref: '#/components/schemas/Units'
        title:
          type: string
          description: The title of the feed, which includes the location city
          example: Yahoo! Weather - Singapore, South West, SG
        link:
          type: string
          description: The URL for the Weather page of the forecast for this location
          example: http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91792352/
        description:
          type: string
          description: The overall description of the feed including the location
          example: Yahoo! Weather for Singapore, South West, SG
        language:
          type: string
          description: The language of the weather forecast
          example: en-us
        lastBuildDate:
          type: string
          description: The last time the feed was updated. The format is in the date format defined by [RFC822 Section 5](http://www.rfc-editor.org/rfc/rfc822.txt).
          example: Wed, 14 Jun 2017 10:30 PM SGT
        ttl:
          type: string
          description: Time to Live -- how long in minutes this feed should be cached.
          example: 60
        location:
          description: The location of this forecast
          $ref: '#/components/schemas/Location'
        wind:
          description: Forecast information about wind
          $ref: '#/components/schemas/Wind'
        atmosphere:
          description: Forecast information about current atmospheric pressure, humidity, and visibility
          $ref: '#/components/schemas/Atmosphere'
        astronomy:
          description: Forecast information about current astronomical conditions
          $ref: '#/components/schemas/Astronomy'
        image:
          description: The image used to identify this feed
          $ref: '#/components/schemas/Image'
        item:
          description: The item element contains current conditions and forecast for the given location. There are multiple weather forecast elements for today and tomorrow.
          $ref: '#/components/schemas/Item'
    Units:
      title: units
      type: object
      properties:
        distance:
          type: string
          description: Units for distance, mi for miles or km for kilometers
          example: km
        pressure:
          type: string
          description: Units of barometric pressure, "in" for pounds per square inch or "mb" for millibars.
          example: mb
        speed:
          type: string
          description: Units of speed, "mph" for miles per hour or "kph" for kilometers per hour.
          example: km/h
        temperature:
          type: string
          description: Degree units, "f" for Fahrenheit or "c" for Celsius.
          example: C
      description: Units for various aspects of the forecast. Note that the default is to use metric formats for the units (Celsius, kilometers, millibars, kilometers per hour).
    Location:
      title: location
      type: object
      properties:
        city:
          type: string
          description: City name
          example: Singapore
        country:
          type: string
          description: Two-character country code
          example: Singapore
        region:
          type: string
          description: State, territory, or region, if given
          example: South West
      description: The location of this forecast
    Wind:
      title: wind
      type: object
      properties:
        chill:
          type: integer
          description: Wind chill in degrees
          format: int32
          example: 81
        direction:
          type: integer
          description: Wind direction, in degrees
          format: int32
          example: 158
        speed:
          type: integer
          description: Wind speed, in the units specified in the speed attribute of the units element (mph or kph)
          format: int32
      description: Forecast information about wind
    Atmosphere:
      title: atmosphere
      type: object
      properties:
        humidity:
          type: integer
          description: humidity, in percent
          format: int32
          example: 87
        pressure:
          type: integer
          description: Barometric pressure, in the units specified by the pressure attribute of the yweather:units element ("in" or "mb").
          format: int32
        rising:
          type: integer
          description: 'State of the barometric pressure: steady (0), rising (1), or falling (2).'
          format: int32
          example: 0
        visibility:
          type: integer
          description: Visibility, in the units specified by the distance attribute of the units element ("mi" or "km"). Note that the visibility is specified as the actual value * 100. For example, a visibility of 16.5 miles will be specified as 1650. A visibility of 14 kilometers will appear as 1400.
          format: int32
      description: Forecast information about current atmospheric pressure, humidity, and visibility
    Astronomy:
      title: astronomy
      type: object
      properties:
        sunrise:
          type: string
          description: Today's sunrise time. The time is a string in a local time format of "h:mm am/pm"
          example: 6:59 am
        sunset:
          type: string
          description: Today's sunset time. The time is a string in a local time format of "h:mm am/pm"
          example: 7:11 pm
      description: Forecast information about current astronomical conditions
    Image:
      title: image
      type: object
      properties:
        title:
          type: string
          description: The title of the image.
          example: Yahoo! Weather
        width:
          type: string
          description: The width of the image, in pixels
          example: 142
        height:
          type: string
          description: The height of the image, in pixels
          example: 18
        link:
          type: string
          description: Description of link
          example: http://weather.yahoo.com
        url:
          type: string
          description: The URL of Yahoo! Weather
          example: http://l.yimg.com/a/i/brand/purplelogo//uh/us/news-wea.gif
      description: The image used to identify this feed
    Item:
      title: item
      type: object
      properties:
        title:
          type: string
          description: The forecast title and time.
          example: Conditions for Singapore, South West, SG at 09:00 PM SGT
        lat:
          type: string
          description: The latitude of the location.
          example: 1.33464
        long:
          type: string
          description: The longitude of the location.
          example: 103.726471
        link:
          type: string
          description: The Yahoo! Weather URL for this forecast.
          example: http://us.rd.yahoo.com/dailynews/rss/weather/Country__Country/*https://weather.yahoo.com/country/state/city-91792352/
        pubDate:
          type: string
          description: The date and time this forecast was posted, in the date format defined by [RFC822 Section 5](http://www.rfc-editor.org/rfc/rfc822.txt).
          example: Wed, 14 Jun 2017 09:00 PM SGT
        condition:
          description: The current weather conditions
          $ref: '#/components/schemas/Condition'
        forecast:
          type: array
          items:
            $ref: '#/components/schemas/ForecastArray'
          description: ''
        description:
          type: string
          description: A simple summary of the current conditions and tomorrow's forecast, in HTML format, including a link to Yahoo! Weather for the full forecast.
          example: "<![CDATA[<img src=\"http:\/\/l.yimg.com\/a\/i\/us\/we\/52\/4.gif\"\/>\n<BR \/>\n<b>Current Conditions:<\/b>\n<BR \/>Thunderstorms\n<BR \/>\n<BR \/>\n<b>Forecast:<\/b>\n<BR \/> Fri - Thunderstorms. High: 30Low: 25\n<BR \/> Sat - Thunderstorms. High: 28Low: 25\n<BR \/> Sun - Thunderstorms. High: 28Low: 25\n<BR \/> Mon - Thunderstorms. High: 28Low: 25\n<BR \/> Tue - Thunderstorms. High: 28Low: 25\n<BR \/>\n<BR \/>\n<a href=\"http:\/\/us.rd.yahoo.com\/dailynews\/rss\/weather\/Country__Country\/*https:\/\/weather.yahoo.com\/country\/state\/city-91792352\/\">Full Forecast at Yahoo! Weather<\/a>\n<BR \/>\n<BR \/>\n(provided by <a href=\"http:\/\/www.weather.com\" >The Weather Channel<\/a>)\n<BR \/>\n]]>"
        guid:
          type: string
          description: Unique identifier for the forecast, made up of the location ID, the date, and the time.
          format: uuid
      description: The item element contains current conditions and forecast for the given location. There are multiple weather forecast elements for today and tomorrow.
    Condition:
      title: condition
      type: object
      properties:
        code:
          type: integer
          description: The condition code for this forecast. You could use this code to choose a text description or image for the forecast. The possible values for this element are described in the [Condition Codes](https://developer.yahoo.com/weather/).documentation.html.
          format: int32
          example: 33
        date:
          type: string
          description: The current date and time for which this forecast applies. The date is in RFC822 Section 5 format.
          example: Wed, 14 Jun 2017 09:00 PM SGT
        temp:
          type: integer
          description: The current temperature, in the units specified by the units element.
          format: int32
          example: 26
        text:
          type: string
          description: A textual description of conditions
          example: Mostly Clear
      description: The current weather conditions
    ForecastArray:
      title: forecast array
      type: object
      properties:
        code:
          type: integer
          description: The condition code for this forecast. You could use this code to choose a text description or image for the forecast. The possible values for this element are described in the [Condition Codes](https://developer.yahoo.com/weather/documentation.html#codes).
          format: int32
          example: 4
        date:
          type: string
          description: The date to which this forecast applies. The date is in 'dd Mmm yyyy' format, for example '30 Nov 2005'
          example: 14 Jun 2017
        day:
          type: string
          description: Day of the week to which this forecast applies. Possible values are Mon Tue Wed Thu Fri Sat Sun.
          example: Wed
        high:
          type: integer
          description: The forecasted high temperature for this day, in the units specified by the units element.
          format: int32
          example: 28
        low:
          type: integer
          description: The forecasted low temperature for this day, in the units specified by the units element.
          format: int32
          example: 25
        text:
          type: string
          description: A textual description of conditions
          example: Thunderstorms
      description: The weather forecast for a specific day. The item element contains multiple forecast elements for today and tomorrow.
    Guid:
      title: guid
      type: object
      properties:
        isPermaLink:
          type: string
          description: The attribute isPermaLink is false.
          example: false
      description: Unique identifier for the forecast, made up of the location ID, the date, and the time.
```

Swagger UI displays each object in `components` in a section called `Models` at the end of your Swagger UI display. If you decided to consolidate all schemas into a single object, without using the `$ref` property to point to new objects, then you will see just one object in Models. To see this display, go to the [Sample Swagger UI display](learnapidoc/assets/files/swagger/), and in the **Explore** field at the top, paste in this URL: `/learnapidoc/docs/rest_api_specifications/openapi_weather_single_component.yml`.

Because I want to re-use objects, I'm going to use the latter method in `components`, where each object contains `$ref` pointers to any sub-object it contains. As a result, the Models section looks like this:

<img src="/learnapidoc/images/swaggerui_models_broken_out.png" />

## Reason for models in the first place

I'm not really sure why the Models section appears at all in the Swagger UI display. Apparently, this section was added by popular request because the online Swagger Editor showed the display, and many users asked for it to be incorporated into Swagger UI.

You don't need this Models section in Swagger UI because both the request and response sections of Swagger UI provide a "Model" link that lets the user toggle to this view. For example:

<img src="/learnapidoc/images/models_options_in_responses.png" />

## Hiding the Models section

You might confuse users by including the Models section. Currently, there isn't a [Swagger UI parameter](https://github.com/swagger-api/swagger-ui#parameters) to hide the Models section. To hide Models, simply add this style to the Swagger UI page:

```html
<style>
section.models {
    display: none;
}
</style>
```

## Describing a schema

For most of the sections in `components`, you follow the same object descriptions as detailed in the rest of the spec. However, when describing the `schema`, you use standard keywords and terms from the [JSON Schema ](https://tools.ietf.org/html/draft-wright-json-schema-00). In other words, you aren't merely using the OpenAPI spec. Here the OpenAPI spec feeds into the larger JSON definitions and description language for describing JSON models.

See the [schema object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject) for details. To describe your JSON objects, you might use the following keywords:

* `title`
* `multipleOf`
* `maximum`
* `exclusiveMaximum`
* `minimum`
* `exclusiveMinimum`
* `maxLength`
* `minLength`
* `pattern`
* `maxItems`
* `minItems`
* `uniqueItems`
* `maxProperties`
* `minProperties`
* `required`
* `enum`
* `type`
* `allOf`
* `oneOf`
* `anyOf`
* `not`
* `items`
* `properties`
* `additionalProperties`
* `description`
* `format`
* `default`

Besides looking in the [schema object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject) in the OpenAPI spec for details, you can also read the [JSON Schema](https://tools.ietf.org/html/draft-wright-json-schema-00).

Additionally, you can look at some example schemas. You can view [3.0 examples here](https://github.com/OAI/OpenAPI-Specification/tree/master/examples/v3.0). I typically find an object that resembles what I'm trying to represent and mimic the same properties and structure. The `schema` object in 3.0 differs slightly from the schema object in 2.0 &mdash; see this [post on Nordic APIs](https://nordicapis.com/whats-new-in-openapi-3-0/#jsonandotherschema) for details on what changed. However, example schemas from [2.0 specs](https://github.com/OAI/OpenAPI-Specification/tree/master/examples/v2.0) (which are a lot more abundant) would probably also be helpful as long as you just look at the schema definitions, because a lot has changed from 2.0 to 3.0 in the spec.