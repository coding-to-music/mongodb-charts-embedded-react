# React Website with Embedded MongoDB Charts

https://www.mongodb.com/developer/how-to/mongodb-charts-embedding-sdk-react/

https://github.com/mongodb-developer/mongodb-charts-embedded-react

This [website](https://react-charts-npwaa-qynhm.mongodbstitch.com/) is an example of how you can embed charts built with MongoDB Charts into a React website with dynamic filters.

![Website screenshot](https://mongodb-devhub-cms.s3.us-west-1.amazonaws.com/website_2255a0a81c.png)

This repository is part of the [second blog post](https://www.mongodb.com/developer/how-to/mongodb-charts-embedding-sdk-react/) of this series of two blog posts.

The [first blog post](https://www.mongodb.com/developer/how-to/react-query-rest-api-realm/) talks about MongoDB Realm + React + REST API + Axios.

# Final Website

The final result has been deployed in MongoDB Realm Static Hosting.

https://react-charts-npwaa-qynhm.mongodbstitch.com/

to fix build problem
"TypeError: MiniCssExtractPlugin is not a constructor"

https://stackoverflow.com/questions/70715794/typeerror-minicssextractplugin-is-not-a-constructor

https://github.com/facebook/create-react-app/issues/11930

```java
npm i -D --save-exact mini-css-extract-plugin@2.4.5

rm -rf package-lock.json && rm -rf node_modules && npm cache clean --force && npm install
```

# Start the Beast

```java
npm install
npm start
```

# Author

Maxime Beugnet <maxime@mongodb.com>


## Introduction
In the previous blog post of this series, we created a React website that was retrieving a list of countries using Axios and a REST API hosted in MongoDB Realm.

In this blog post, we will continue to build on this foundation and create a dashboard with COVID-19 charts, built with MongoDB Charts and embedded in a React website with the MongoDB Charts Embedding SDK.

To add some spice in the mix, we will use our list of countries to create a dynamic filter so we can filter all the COVID-19 charts by country.

You can see the final result here that I hosted in a MongoDB Realm application using the static hosting feature available.

![Final website](https://user-images.githubusercontent.com/3156358/149651964-13790efa-4813-4f8e-bebe-b3b9cc25f6c6.png)

## Prerequisites
The code of this project is available on GitHub in this repository.

```java
git clone git@github.com:mongodb-developer/mongodb-charts-embedded-react.git
```

To run this project, you will need node and npm in a recent version. Here is what I'm currently using:

```java
node -v 
# v14.17.1
npm -v
# 8.0.0
```

You can run the project locally like so:

```java
cd mongodb-realm-react-charts
npm install
npm start
```

In the next sections of this blog post, I will explain what we need to do to make this project work.

### Create a MongoDB Charts Dashboard
Before we can actually embed our charts in our custom React website, we need to create them in MongoDB Charts.

Here is the link to the dashboard I created for this website. It looks like this.

### Dashboard in MongoDB Charts

If you want to use the same data as me, check out this blog post about the Open Data COVID-19 Project and especially this section to duplicate the data in your own cluster in MongoDB Atlas.

As you can see in the dashboard, my charts are not filtered by country here. You can find the data of all the countries in the four charts I created.

![Enable the Filtering and the Embedding](https://user-images.githubusercontent.com/3156358/149651949-b445a017-bca4-4f37-8210-2bdfcb1a1018.png)

To enable the filtering when I'm embedding my charts in my website, I must tell MongoDB Charts which field(s) I will be able to filter by, based on the fields available in my collection. Here, I chose to filter by a single field, country, and I chose to enable the unauthenticated access for this public blog post (see below).

![Embed menu in MongoDB Charts](https://user-images.githubusercontent.com/3156358/149651934-8c75cc8e-48b2-4c3b-8318-e7c2a00e7147.png)
In the User Specified Filters field, I added country and chose to use the JavaScript SDK option instead of the iFrame alternative that is less convenient to use for a React website with dynamic filters.

![Embedding Menu in my Charts](https://user-images.githubusercontent.com/3156358/149651899-ec430bd5-1a2e-4094-8fa6-5067ae294c3b.png)

For each of the four charts, I need to retrieve the Charts Base URL (unique for a dashboard) and the Charts IDs.

Now that we have everything we need, we can go into the React code.

## React Website

## MongoDB Charts Embedding SDK
First things first: We need to install the MongoDB Charts Embedding SDK in our project.

```java
npm i @mongodb-js/charts-embed-dom
```

It's already done in the project I provided above but it's not if you are following from the first blog post.

## React Project
My React project is made with just two function components: Dashboard and Chart.

The index.js root of the project is just calling the Dashboard function component.

```java
import React from 'react';
import ReactDOM from 'react-dom';
import Dashboard from "./Dashboard";
ReactDOM.render(<React.StrictMode>
  <Dashboard/>
</React.StrictMode>, document.getElementById('root'));
```

The Dashboard is the central piece of the project:

```java
import './Dashboard.css';
import {useEffect, useState} from "react";
import axios from "axios";
import Chart from "./Chart";
const Dashboard = () => {
  const url = 'https://webhooks.mongodb-stitch.com/api/client/v2.0/app/covid-19-qppza/service/REST-API/incoming_webhook/metadata';
  const [countries, setCountries] = useState([]);
  const [selectedCountry, setSelectedCountry] = useState("");
  const [filterCountry, setFilterCountry] = useState({});
  function getRandomInt(max) {
    return Math.floor(Math.random() * max);
  }
  useEffect(() => {
    axios.get(url).then(res => {
      setCountries(res.data.countries);
      const randomCountryNumber = getRandomInt(res.data.countries.length);
      let randomCountry = res.data.countries[randomCountryNumber];
      setSelectedCountry(randomCountry);
      setFilterCountry({"country": randomCountry});
    })
  }, [])
  useEffect(() => {
    if (selectedCountry !== "") {
      setFilterCountry({"country": selectedCountry});
    }
  }, [selectedCountry])
  return <div className="App">
    <h1 className="title">MongoDB Charts</h1>
    <h2 className="title">COVID-19 Dashboard with Filters</h2>
    <div className="form">
      {countries.map(c => <div className="elem" key={c}>
        <input type="radio" name="country" value={c} onChange={() => setSelectedCountry(c)} checked={c === selectedCountry}/>
        <label htmlFor={c} title={c}>{c}</label>
      </div>)}
    </div>
    <div className="charts">
      <Chart height={'600px'} width={'800px'} filter={filterCountry} chartId={'6e3cc5ef-2be2-421a-b913-512c80f492b3'}/>
      <Chart height={'600px'} width={'800px'} filter={filterCountry} chartId={'be3faa53-220c-438f-aed8-3708203b0a67'}/>
      <Chart height={'600px'} width={'800px'} filter={filterCountry} chartId={'7ebbba33-a92a-46a5-9e80-ba7e8f3b13de'}/>
      <Chart height={'600px'} width={'800px'} filter={filterCountry} chartId={'64f3435e-3b83-4478-8bbc-02a695c1a8e2'}/>
    </div>
  </div>
};
export default Dashboard;
```

It's responsible for a few things:

Line 17 - Retrieve the list of countries from the REST API using Axios (cf previous blog post).
Lines 18-22 - Select a random country in the list for the initial value.
Lines 22 & 26 - Update the filter when a new value is selected (randomly or manually).
Line 32 counties.map(...) - Use the list of countries to build a list of radio buttons to update the filter.
Line 32 <Charts .../> x4 - Call the Chart component one time for each chart with the appropriate props, including the filter and the Chart ID.
As you may have noticed here, I'm using the same filter fitlerCountry for all the Charts, but nothing prevents me from using a custom filter for each Chart.

You may also have noticed a very minimalistic CSS file Dashboard.css. Here it is:

```java
.title {
  text-align: center;
}
.form {
  border: solid black 1px;
}
.elem {
  overflow: hidden;
  display: inline-block;
  width: 150px;
  height: 20px;
}
.charts {
  text-align: center;
}
.chart {
  border: solid #589636 1px;
  margin: 5px;
  display: inline-block;
}
```

The Chart component looks like this:

```java
import React, {useEffect, useRef, useState} from 'react';
import ChartsEmbedSDK from "@mongodb-js/charts-embed-dom";
const Chart = ({filter, chartId, height, width}) => {
  const sdk = new ChartsEmbedSDK({baseUrl: 'https://charts.mongodb.com/charts-open-data-covid-19-zddgb'});
  const chartDiv = useRef(null);
  const [rendered, setRendered] = useState(false);
  const [chart] = useState(sdk.createChart({chartId: chartId, height: height, width: width, theme: "dark"}));
  useEffect(() => {
    chart.render(chartDiv.current).then(() => setRendered(true)).catch(err => console.log("Error during Charts rendering.", err));
  }, [chart]);
  useEffect(() => {
    if (rendered) {
      chart.setFilter(filter).catch(err => console.log("Error while filtering.", err));
    }
  }, [chart, filter, rendered]);
  return <div className="chart" ref={chartDiv}/>;
};
export default Chart;
```

The Chart component isn't doing much. It's just responsible for rendering the Chart once when the page is loaded and reloading the chart if the filter is updated to display the correct data (thanks to React).

Note that the second useEffect (with the chart.setFilter(filter) call) shouldn't be executed if the chart isn't done rendering. So it's protected by the rendered state that is only set to true once the chart is rendered on the screen.

And voilà! If everything went as planned, you should end up with a (not very) beautiful website like this one.

Conclusion
In this blog post, your learned how to embed MongoDB Charts into a React website using the MongoDB Charts Embedding SDK.

We also learned how to create dynamic filters for the charts using useEffect().

We didn't learn how to secure the Charts with an authentication token, but you can learn how to do that in this documentation.

If you have questions, please head to our developer community website where the MongoDB engineers and the MongoDB community will help you build your next big idea with MongoDB.