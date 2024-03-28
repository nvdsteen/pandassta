# pandassta: combining sensorthings and pandas 

`pandassta` package allows easy tools to interact with a [FROST-Server](https://www.iosb.fraunhofer.de/en/projects-and-products/frost-server.html) Sensorthings API, using pandas dataframes.
This package was developed within a quality assurance project, which is reflected in some specific functions.

## Installation

```shell
pip install pandassta
```

## Basic usage

### Building query

Different wrappers are available for some common queries, but custom queries can easily be constructed.
The code below builds a query to get the observations per datastream, with the observed properties of thing 1.

```python
obsprop = Entity(Entities.OBSERVEDPROPERTY)
obsprop.selection = [Properties.NAME, Properties.IOT_ID]

obs = Entity(Entities.OBSERVATIONS)
obs.settings = [Settings.COUNT("true"), Settings.TOP(0)]
obs.selection = [Properties.IOT_ID]

ds = Entity(Entities.DATASTREAMS)
ds.settings = [Settings.COUNT("true")]
ds.expand = [obsprop, obs]
ds.selection = [
    Properties.NAME,
    Properties.IOT_ID,
    Properties.DESCRIPTION,
    Properties.UNITOFMEASUREMENT,
    Entities.OBSERVEDPROPERTY,
]
thing = Entity(Entities.THINGS)
thing.id = 1
thing.selection = [Properties.NAME, Properties.IOT_ID, Entities.DATASTREAMS]
thing.expand = [ds]
query = Query(base_url=config.load_sta_url(), root_entity=thing)
query_http = query.build()
```

## Components
### General definitions: sta.py

Reflection of the sensorthings structure.

### Construction and execution of queries: sta_requests.py

Classes and function that allow or simplify the construction requests.

### General function to go from a json response to a pandas dataframe: df.py

Classes and functions to convert observations to a dataframe.

