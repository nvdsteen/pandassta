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


### Step by step tutorial 

Lets assume you want to obtain the air temperature and water temperature measured between 2023-03-10 00:00 and 2023-03-11 10:00. 

- Get list of things
	- Imports
	  
	  ```python
	  from pandassta.sta_requests import Config, Entity, Entities, Query, Properties
	  from pandassta.sta_requests import set_sta_url, get_request, response_datastreams_to_df
	  ```
	- Config
	  
	  ```python
	  config = Config()
	  set_sta_url("https://sensors.naturalsciences.be/sta/v1.1")
	  
	  thing = Entity(Entities.THINGS) #not structly needed in this step, but needed later
	  ```
	- Get json
	  
	  ```python
	  # if `thing` is not defined 
	  # query = Query(config.load_sta_url(), root_entity=Entities.THINGS)
	  query = Query(config.load_sta_url(), root_entity=thing)
	  q_url = query.build() # if needed
	  response = get_request(query)
	  ```
- Get list of datastreams
	- why not datastreams directly?
	- using application https://sensors.naturalsciences.be/sensorthings-data/
	- Using pandassta
	  
	  ```python
	  thing.id = 1
	  ds = Entity(Entities.DATASTREAMS)
	  thing.selection = [ Entities.DATASTREAMS ]
	  thing.expand = [ ds ]
	  
	  response = get_request(query)
	  ```
- Get the relevant data/observations.
    In this example, datastreams 7749 and 7767 were selected, but multiple datastreams give the air or water temperature!
	- define the filter
	  
	  ```python
	  filter_ds = f"{Properties.IOT_ID} in (7749, 7767)"
	  filter_obs = f"overlaps({Properties.PHENOMENONTIME}, 2023-03-10T00:00Z/2023-03-11T10:00Z)"
	  ds.filter = filter_ds
	  obs = Entity(Entities.OBSERVATIONS)
	  
	  obs.filter = filter_obs
	  
	  # # INCLUDING feature of interest! (coordinates)
	  # foi = Entity(Entities.FEATUREOFINTEREST)
	  # foi.selection = [Properties.COORDINATES, Properties.IOT_ID]
	  # obs.expand = [foi]
	  
	  ds.expand = [obs]
	  
	  response = get_request(query)
	  ```
- Data to a pandas dataframe
	- call pandassta method and verify dataframe
	  
	  ```python
	  df = response_datastreams_to_df(response[1])
	  df.head()
	  ```
		- output:

            |   | @iot.selfLink                                                                                          | @iot.id    | phenomenonTime       | resultTime | result | resultQuality | observation_type | observed_property_id | units             | feature_id | long | lat |
            |---|--------------------------------------------------------------------------------------------------------|------------|----------------------|------------|--------|---------------|------------------|----------------------|-------------------|------------|------|-----|
            | 0 | [Link](https://sensors.naturalsciences.be/sta/v1.1/Ob...)                                              | 1155244072 | 2023-03-10 01:04:00 | None       | 9.8811 | 2             | NaN              | None                 | Degrees Celsius   | None       | None | None|
            | 1 | [Link](https://sensors.naturalsciences.be/sta/v1.1/Ob...)                                              | 1155246938 | 2023-03-10 01:10:00 | None       | 9.8618 | 2             | NaN              | None                 | Degrees Celsius   | None       | None | None|
            | 2 | [Link](https://sensors.naturalsciences.be/sta/v1.1/Ob...)                                              | 1155251749 | 2023-03-10 01:20:03 | None       | 9.7390 | 2             | NaN              | None                 | Degrees Celsius   | None       | None | None|
            | 3 | [Link](https://sensors.naturalsciences.be/sta/v1.1/Ob...)                                              | 1155256547 | 2023-03-10 01:30:06 | None       | 9.7692 | 2             | NaN              | None                 | Degrees Celsius   | None       | None | None|
            | 4 | [Link](https://sensors.naturalsciences.be/sta/v1.1/Ob...)                                              | 1155261355 | 2023-03-10 01:40:08 | None       | 9.7360 | 2             | NaN              | None                 | Degrees Celsius   | None       | None | None|

		  
## Components
### General definitions: sta.py

Reflection of the sensorthings structure.

### Construction and execution of queries: sta_requests.py

Classes and function that allow or simplify the construction requests.

### General function to go from a json response to a pandas dataframe: df.py

Classes and functions to convert observations to a dataframe.

