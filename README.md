# sqlalchemy-challenge
This is my Module 10 Challenge for my Data Analytics and Visualization Boot Camp. I was tasked with completing a climate analysis for Hawaii.


## Table of Contents
* [General Info](#general-information)
* [Technologies Used](#technologies-used)
* [Screenshots](#screenshots)
* [Setup](#setup)
* [Usage](#usage)
* [Project Status](#project-status)
* [Acknowledgements](#acknowledgements)
* [Contact](#contact)


## General Information
Using SqlAlchemy, I analyzed and explored the climate data for Hawaii. I retrieved the precipitation data for the previous 12 months, filtered the stations for the most active and completed a query for the previous 12 months temperature observations for the most active station. In addition, I plotted the precipitation and temperate data.
Afterwards, I used Flask API to create a climate app of the queries.


## Technologies Used
- SqlAlchemy
- Flask API
- Python
- Jupyter Notebook


## Screenshots
![1](https://user-images.githubusercontent.com/117790100/220489017-6c8a5ffd-eb6b-4c90-b152-061367ec90d9.png)
![2](https://user-images.githubusercontent.com/117790100/220489025-75ff3d05-1def-4222-834b-2a50c5c5c23b.png)
![3](https://user-images.githubusercontent.com/117790100/220489034-7405a933-4280-4b05-b536-f5f0163d7c7b.png)


## Setup
The climate data used in the analysis can be found in the resources folder.


## Usage
Please see the code below to analyze and explore the climate data through the climate app.

```
import numpy as np

import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
import statistics

from flask import Flask, jsonify, request

#################################################
# Database Setup
#################################################
engine = create_engine("sqlite:///hawaii.sqlite")

# Reflect database to model
Base = automap_base()
# Reflect the tables
Base.prepare(autoload_with=engine)

# Save references to tables
Measurements = Base.classes.measurement
Stations = Base.classes.station

#################################################
# Flask Setup
#################################################
app = Flask(__name__)


#################################################
# Flask Routes
#################################################

@app.route("/")
def index():
    """List all available api routes."""
    return """HAWAII CLIMATE<br>
    <br>
    All Measurements:
    <br><a href="/api/v1.0/measurements">Measurements (JSON)</a><br>
    <br>
    Precipitation for the last year:
    <br><a href="/api/v1.0/precipitation">Precipitation (JSON)</a><br>
    <br>
    Stations Data:   
    <br><a href="/api/v1.0/stations">Stations (JSON)</a><br>
    <br>
    Most Active Station Temperature Data for the last year:
    <br><a href="/api/v1.0/tobs">USC00519281 Tobs (JSON)</a><br>
    <br>
    Start Date and/or End Date (min, max, avg temperatures):<br>
    <br>
    /api/v1.0/temperature?startdate=yyyy-mm-dd&enddate=yyyy-mm-dd<p/>
    <form action='/api/v1.0/temperature' method='get'>\
    <label>Start Date:</label><br>\
    <input type='text' name='startdate' value='' placeholder='2010-01-01'><br>\
    <label>End Date:</label><br>\
    <input type='text' name='enddate' value='' placeholder='2010-01-01'><br>\
    <input type='submit' value='Submit'>\
    </form>
    """

# Route for All Measurements
@app.route("/api/v1.0/measurements")
def measurement():
    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Query all measurements
    results = session.query(Measurements.id, Measurements.station,
                            Measurements.date, Measurements.prcp, Measurements.tobs).all()

    session.close()

    # Create a dictionary from the row data and append to a list
    all_measurements = []
    for id, station, date, prcp, tobs in results:
        measurement_dict = {}
        measurement_dict["id"] = id
        measurement_dict["station"] = station
        measurement_dict["date"] = date
        measurement_dict["prcp"] = prcp
        measurement_dict["tobs"] = tobs
        all_measurements.append(measurement_dict)

    return jsonify(all_measurements)

# Route for previous 12 months Precipitation
@app.route("/api/v1.0/precipitation")
def precipitation():
    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Query all prcp
    results = session.query(Measurements.date, Measurements.prcp).all()

    session.close()

    # Create a dictionary from the row data and append to a list
    all_prcp = []
    for date, prcp in results:
        prcp_dict = {}
        prcp_dict[date] = prcp
        if date >= "2016-08-23":
            all_prcp.append(prcp_dict)
        else:
            continue

    return jsonify(all_prcp)


# Route for All Stations
@app.route("/api/v1.0/stations")
def station():
    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Query all stations
    results = session.query(Stations.id, Stations.station, Stations.name,
                            Stations.latitude, Stations.longitude, Stations.elevation).all()

    session.close()

    # Create a dictionary from the row data and append to a list
    all_stations = []
    for id, station, name, latitude, longitude, elevation in results:
        station_dict = {}
        station_dict["id"] = id
        station_dict["station"] = station
        station_dict["name"] = name
        station_dict["latitude"] = latitude
        station_dict["longitude"] = longitude
        station_dict["elevation"] = elevation
        all_stations.append(station_dict)

    return jsonify(all_stations)


# Route for previous 12 months Temperatures of most active station
@app.route("/api/v1.0/tobs")
def tobs():
    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Query all tobs
    results = session.query(Measurements.id, Measurements.station,
                            Measurements.date, Measurements.prcp, Measurements.tobs).all()

    session.close()

    # Create a dictionary from the row data and append to a list
    all_tobs = []
    for id, station, date, prcp, tobs in results:
        tobs_dict = {}
        tobs_dict["id"] = id
        tobs_dict["station"] = station
        tobs_dict["date"] = date
        tobs_dict["prcp"] = prcp
        tobs_dict["tobs"] = tobs
        if date >= "2016-08-23" and station == "USC00519281":
            all_tobs.append(tobs_dict)
        else:
            continue

    return jsonify(all_tobs)


# Route for startdate and/or enddate inputs with 'GET' method
@app.route("/api/v1.0/temperature", methods=['GET'])
def temperature():

    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Create requests for startdate and/or enddate input(s)
    startdate = request.args["startdate"]
    enddate = request.args["enddate"]

    # Print startdate and/or enddate to terminal
    print("\n=================================================")
    print(f"startdate: {startdate}    enddate: {enddate}")
    print("=================================================\n")

    # Return TMIN, TAVG, TMAX
    # Select statement
    sql = [func.min(Measurements.tobs), func.avg(
        Measurements.tobs), func.max(Measurements.tobs)]

    session.close()
    
    if not enddate:
        # Calculate TMIN, TAVG, TMAX for dates greater than start
        results = session.query(*sql).\
            filter(Measurements.date >= startdate).all()
    else:
        # Calculate TMIN, TAVG, TMAX with start and stop
        results = session.query(*sql).\
            filter(Measurements.date >= startdate).\
            filter(Measurements.date <= enddate).all()

    # Unravel results into a 1D array and convert to a list
    temps = list(np.ravel(results))
    
    # Results
    return jsonify(temps=temps)


if __name__ == '__main__':
    app.run(debug=True)

```


## Project Status
Project is complete and no longer being worked on.


## Acknowledgements
- Many thanks to instructional team and my tutor, David Chao.


## Contact
Created by Diane Guzman.


