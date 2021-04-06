---
layout: post
title: Data Processing with Vue.js & nuxt
subtitle: Making graphs and charts from JSON
date: "2020-06-03"
---

I recently deployed a little project I've been working on for about a week.
A static site built with Nuxt.js (A vue.js based static-site generator). 

I did for 2 main reasons;

1. To play with and learn about Vue and Nuxt (and frontend stuff in general).
2. To play with ideas about data processing.

In this blog post I'll outline the tools and the processes I used. 
You can visit the live site [here](https:/joshburns.xyz/covid-data).

## The Data

First we need to look at the raw data. Our raw data comes from [Datahub](https://datahub.io/core/covid-19).
Datahub provides CSV and JSON files that we can download via cURL. Later-on we will get back to this. The 
data I will focus on here is the impact of the virus over time.

## The Framework

To build the website I used Nuxt.js a static site generator that uses Vue.js.
I also used [Vuetify](https://Vuetifyjs.com/en/getting-started/quick-start/)
Which is a material design framework for Vue.js. Vuetify gives us access to
certain components like [Data Tables](https://vuetifyjs.com/en/components/data-tables/)
and [Sparklines](https://vuetifyjs.com/en/components/sparklines/).

## Data Ingress
To get the data into the website I wrote a simple shell script that looks like this

```sh
#!/bin/sh

mkdir -p ./data
curl -L https://datahub.io/core/covid-19/r/0.json > data/time_series_combined.json
curl -L https://datahub.io/core/covid-19/r/3.json > data/worldwide_aggregate.json
```

This script uses cURL to fetch the JSON data from datahub and saves it. note the
single `>` operator. in most shell scripts people will use `>>` to **append** text
to a file. the single `>` denotes the command to **overwrite** the data with the text.

We can now leverage Vue.js / Javascript and import json directly into our Vue component.

```js
...
<script>
import timeData from '~/data/time_series_combined.json';
export default {
    computed: {
        timeseries: function () {
            return timeData;
        }
    }
}
</script>
```



## Generate a searchable datatable.

We now have a `timeseries` property we can use in our Vue component. Here is the full
source code for the page that uses this data on the live site. Vuetify's Data Table
has built-in search and pagination features as-well. With little effort we have a
nice presentable data collection.

```js
<template>
  <v-layout>
    <v-flex class="text-center">
        
      <v-card>
        <v-card-title>
          Cases Over Time by Country
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="mdi-database-search"
            label="Search"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-data-table
          :headers="headers"
          :items="timeseries"
          :search="search"
        >
          <template v-slot:no-results>
            <v-alert :value="true" color="error" icon="warning">
              Your search for "{{ search }}" found no results.
            </v-alert>
          </template>
        </v-data-table>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import timeData from '~/data/time_series_combined.json';
export default {
    data () {
        return {
            search: '',
            headers: [
                { text: 'Date', value: 'Date', align: 'left', sortable: true },
                { text: 'Country/Region', value: 'Country/Region', sortable: true },
                { text: 'Province/State', value: 'Province/State', sortable: true },
                { text: 'Latitude', value: 'Lat', sortable: true },
                { text: 'Longitude', value: 'Long', sortable: true },
                { text: 'Confirmed', value: 'Confirmed', sortable: true },
                { text: 'Recovered', value: 'Recovered', sortable: true },
                { text: 'Deaths', value: 'Deaths', sortable: true },
            ]
        }
    },
    computed: {
        timeseries: function () {
            return timeData;
        }
    }
}
</script>
```


## Sparklines

Sparklines work in a simillar fasion to data tables with a key difference.
They only use 1 data input, so just feeding it a JSON blob wont do much good.

So I handled this by using the computed propety as a generator and have it
loop through the data, filter out the element we wanted, push that value on
to an array, and move on. That looks like this

```js
<script>
import agdata from '~/data/worldwide_aggregate.json';
...
export default {
    data: () => ({
        ...
    }),
    computed: {
        value: function () {
            var retArr = []; 
            for (var j = 0; j < agdata.length; j++){
                var obj = agdata[j];
                var val = obj['Increase rate'];
                if(val != null) {
                    retArr.push(obj['Increase rate']);
                }
            }
            return retArr;
        }
    }
}
</script>
```
 Take note of the `val != null` condition in that loop. A sparkline
 requires a complete array of values. Otherwise it will error out.
 We now have a computed propety `value` that we can use in our 
 component like so
 
 ```js
 
 <template>
    <v-flex class="text-center">
        <v-card>
        <v-card-title>
            Increase Rate of Cases over Time
        </v-card-title>
        <v-sparkline
            :value="value"
            ...
        ></v-sparkline>
        <p>This sparkline shows the increase trend in COVID-19 Cases over the span on the outbreak.</p>
        </v-card>
    </v-flex>
</template>

<script>
import agdata from '~/data/worldwide_aggregate.json';
...
export default {
    data: () => ({
        ...
    }),
    computed: {
        value: function () {
            var retArr = []; 
            for (var j = 0; j < agdata.length; j++){
                var obj = agdata[j];
                var val = obj['Increase rate'];
                if(val != null) {
                    retArr.push(obj['Increase rate']);
                }
            }
            return retArr;
        }
    }
}
</script>
 
 ```

