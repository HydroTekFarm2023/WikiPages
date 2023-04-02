The Analytics page displays several sensor-graph components, each of which host a d3.js graph displaying sensor data for a set period of time (part of a select element in analytics). When the Analytics page is loaded, it tells varman to fetch and/or patch relevant sensor data and cache it locally. That data is eventually returned to analytics, which sends it off to individual sensor-graphs. The sensor-graphs use a downsampling algorithm to reduce the amount of points shown, then displays and styles the resulting graph.

### Analytics page

The timeframeOptions variable contains an array of objects which specify the timeframes available as options on the page. The values are in hours. Any changes made here must also be reflected in genXAxisTicks() in sensor-graph component ts, as it uses that information to generate the ticks on the graphs' X axes.

The ngOnInit() function takes care of device changes (aka when currentDeviceIndex or Type changes), and the onSelectChange() callback from the html takes care of timeframe changes. Both of these trigger fetchHistoricData() to fetch data.

### Variable Management Service: Overview

Note: historical-data-interface.ts (under models) contains interfaces for analytics_data, sensor_data, and sensor_details.

The only variables relevant to Analytics/Historic Data caching are analyticsDataArray and SENSOR_INTERVAL_SECONDS (which is either 10 or 30, depending on user tiers or something that do not exist at time of writing, but is set when calling getHistoricData). The analyticsDataArray is an array with {topicID, analyticsData} as elements, so that each device's topicID can be used to find their cached data. (analyticsData is type analytics_data.)

getHistoricData() is the only function that is called when new graphing data is requested. It returns an observable in all cases so that analytics page could subscribe to it. Upon function call, the situation is put into several cases:

Case 1: No current entry - the caching array does not have an element matching the topicID. A new empty element corresponding to it is created. This does NOT return and will continue to updateHistoricalDataStorage(). 

Case 2: Full requested data exists - caching array has all the data we want, from start to finish. In this case the relevant data slice from the caching array is returned.

Inside updateHistoricalDataStorage() (which means fetching from the server is required)

Case 3: No data at all is available for this device (typically follows case 1) - The entry exists but there is no content at all. A request is sent to fetch the data for the entire duration, cache it, then return the relevant slice.

Case 4, 5, 6 depend on which side(s) is/are missing from the existing cached data. This means that some cached data is available but is not Case 2 (more data is potentially available)

Case 4: Both sides are missing - TWO requests are sent to fetch the data, from the front and back, so that the data can be merged to form one continuous chunk for the entire requested duration.

Case 5: Earlier data is missing - A request is sent to fetch the data from the first requested date to the first sensor data time in the existing cache. This is rare in actual usage since new data should be added every 10/30 seconds, so it would typically be a case 4.

Case 6: Later data is missing - Similar to above, but this time fetching the data from the currently cached data's last timestamp to the last timestamp of the request. 

### Variable Management Service: Caching and processing

Whenever Cases 3/4/5/6 are involved, the cache is updated with additional data. In case 3, the current non-existent data (null) is set to the result of the fetching HTTP request directly.

In cases 4,5,6, the result(s) of the fetched data are combined with existing data to make a new analytics_data object, which will then be returned and cached. This is done with using spread operators and sending the result to makeAnalyticsData(), which is a function to simplify making the objects. Passing null as the parameter for that function creates a valid empty analytics_data object for special or empty cases.

Timestamps are 'rounded' to the nearest SENSOR_INTERVAL_SECONDS first thing when the getHistoricData function is called. An example is given in comments in the roundTimestamp() function.

Slicing is implemented in getSlice(). It mostly does the normal slicing function, but also includes cases where if the requested earliest time is earlier than the cache's earliest date, it uses the first element in the cache, and similarly for the latest requested/cached timestamps.

### Sensor-Graph component

This component takes data from a lot of bindings, which would hopefully become less as sensor classes are finished up.

ngOnInit() compiles the historical data (see downsampling section), and ngAfterViewInit() builds the graph display using that compiled data.

There are two setter @Input()s, one for timeframe and one for the main data array. These are triggered when their respective values change (timeframe: someone selected another option, data: data changed).

### Sensor-Graph component: Display

Some links are written in comments, where I referenced others' code to make ours. If you get stuck, please take a look online first.

Generally, what each line does should be apparent, and plenty of comments are left in to explain as well. The variable names defined in the function are mostly decorative, so that you can see what it does immediately.

The graph is built inside a svg element (the wrapper), which uses a viewBox to scale its contents when you zoom in/out or change resolution. The svg element has a g element inside it (the bounds) which contains the actual graph. Throughout the function, more elements are appended for each different part of the graph (lines, axes, etc).

A dimensions object is defined near the top of the file. Most importantly, the margin object inside it defines how many pixels are left on each side of the graph, so that the graph is offset by a calculated amount below and displayed.

The aspect ratio is defined in dimensions (width and height) and in the html file (in the viewBox attributes). To change the aspect ratio properly, make sure both values are changed to match, and consider also changing margin values to match if that is applicable.

X-Axis: genXAxisTicks() generates specific X-Axis ticks according to the time range of the graph based on a switch statement. Generally the timespan is split into 6 or 7 portions evenly.

Y-Axis scaling: The Y-Axis scaling is modified by a variable called YSCALE_SPACE_PCT. The Y-Axis is scaled so that there is that amount of space above AND below the data's max and min points.

X/Y Axis text processing: See end of buildGraph(). The text is transformed and scaled to make them look nicer.

### Sensor-Graph component: Downsampling

Use downsampleCompileData() instead of compileData() ! the latter does not work I think.

This function takes a targetPts param. This is the number of points you WANT to have on the graph. The function then takes the _historicalData and checks the number of data points inside that. It generates a THEORETICAL number of data points inside the timeframe (not the ACTUAL amount of data) and uses that to calculate a ratio (Theoretical amt. pts. / target amt. pts).

 If there is less data than desired, it just puts everything on the graph (ratio is set to 1). Otherwise, the ratio is rounded and used to average every \[ratio\]th points together, so that each point in the resulting dataset would represent \[ratio\] points. This is not an exact algorithm, since you cannot average every 3.5 data points or anything decimal (it doesn't make sense). 

For the first point of every 'group' of \[ratio\] pts, the timestamp is stored. After every group's values are added, they are averaged by dividing the total by the counter value (i), and pushing that into the resulting array. The remaining group of points after the for loop ends is also averaged in the same way but separately, since they may not have exactly \[ratio\] points.

### Known Issues

Noncontinuous data: This entire system was built with the assumption that sensor data would be in one continuous chunk. I could not change it to accomodate non-continuous data since I was asked to do this very late and would not have the time.

Alarm lines: Attempted to use the sensors' alarm_min/_max values to draw lines on the graph, but there were display issues. The code is still there but commented out.

Hope this was useful. -Tom