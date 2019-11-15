---
title: Connecting a Highcharts Graph to a SharePoint List
date: 2019-11-15 10:32:58
tags:
- SharePoint
- Highcharts
---

![Giphy Lynch](/images/giphyLynch.gif)

## the need
I a SharePoint tool to report critical incidents that occur in programs across the agency. A critical incident happens, a user fills out the InfoPath form, a SharePoint workflow routes the information to necessary parties and eveyone is informed as to what happened in a matter of minutes. That's great, but when looking for trends across programs and within the programs themselves, we were relying on importing the information from the critical incident SharePoint list to Excel and doing the data analysis from there.

Every time someone wanted to analyze the data, they had to update the data, making the manipulations and make use of Excel's graphing options.

Static, involved, and cumbersome.

At one point, we just embedded the Excel onto a SharePoint page:
![R1v3t1ng](/images/highchartsSP/excelIncidents.PNG)

## the implementation

The first step is to create a site page, and add a Content Editor. Within that editor edit the source and add the container for the graph:
```html
<div id="incident-container">No incidents to report.</div>
```

With the container in place, we'll now want to query the SharePoint list, manipulate the data to pass to Highcharts, and then create the graph.

First step is to pass our page the necessary Highcharts packages and JQuery:
```html
<script type="text/javascript" src="/Style%20Library/js/jquery-3.4.1.min.js"></script>
<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>
<script src="https://code.highcharts.com/modules/series-label.js"></script>
```

First I'm going to create some of the global arrays that I'll need to pass between functions:
```javascript
<script type="text/javascript">
var splistitems;
var seriesarray = new Array();
var output = new Array();
var finalData = new Array();

</script>
```

Then I can go ahead and start the query to SharePoint, using SharePoint's CAML query:
```javascript
function GetChartData() {
   seriesarray = [];
   var context = new SP.ClientContext.get_current();
   var list = context.get_web().get_lists().getByTitle("Incident Reporting");
   var view = list.get_views().getByTitle("thisFiscalYear");
   context.load(view);

   context.executeQueryAsync(
     function (sender, args)
        { getItemsFromList("Incident Reporting", "<View><Query>" + view.get_viewQuery() + "</Query></View>") },
     function (sender, args)
        { alert("error: " + args.get_message()); }
   );

}
```

The above gets the context of the query, finds my list and the associated view. (There's a lot of incidents, going off the thisFiscalYear view, limits my query, to provide more recent data and faster rendering times, because I'm querying that specific view, which is filtering out incidents outside of this fiscal year.)

Now I can query the list, and then enumerate through the list items:
```javascript
function getItemsFromList(listTitle, queryText) {
  var context = new SP.ClientContext.get_current();
  var list = context.get_web().get_lists().getByTitle(listTitle);
  var query = new SP.CamlQuery();
  query.set_viewXml(queryText);

  var items = list.getItems(query);
  context.load(items);

  context.executeQueryAsync(

    function()  {
      var listEnumerator = items.getEnumerator();
      while (listEnumerator.moveNext()) {
        var currentlistitem = listEnumerator.get_current();
        var itemProgramName = currentlistitem.get_item("SiteProgramName"); /*Use the internal column names  */
        var itemIncidentType = currentlistitem.get_item("CriticalIncidentType");
        var seriesitem = {
                    name: itemProgramName,
                    data: [ itemIncidentType ]
                };
                seriesarray.push(seriesitem);
              }
```

At this point my `seriesarray` array looks something like this:
```javascript
[{name: programName, data: incident}, {name: programName, data: incident}, {name: programName2, data: incident}]
```

For each incident I have an object. That's not very helpful. So I need to concatenate incidents from each program into one object. To this I'll need a function:
```javascript
function concatData (arr, arrOut) {
  arr.forEach(function(item) {
    var existing = arrOut.filter(function(v, i) {
      return v.name == item.name;
    });
    if (existing.length) {
      var existingIndex = arrOut.indexOf(existing[0]);
      arrOut[existingIndex].data = arrOut[existingIndex].data.concat(item.data);
    } else {
      if (typeof item.data == 'string')
        item.data = [item.data];
      arrOut.push(item);
    }
  });
}
```

Then:
```javascript
    output = [];
    concatData(seriesarray, output);
```

Now my `output` array has everything together, but I need to count the incidents of the same type in each program, so that I can graph it:
```javascript
function mapper (input) {
  for (var i = 0; i < input.length; i ++) {
    var mapOutput = input[i].data.flat().reduce(function(obj, b) {
      obj[b] = ++obj[b] || 1;
      return obj;
    }, {});
    var secondOutput = [];
    var totalIncidents = 0;
    for (var prop in mapOutput) {
      var seriesitem = {
                        name: prop,
                        value: mapOutput[prop]
                    };
      secondOutput.push(seriesitem);
      totalIncidents = totalIncidents + mapOutput[prop];
      totalAllPro = totalAllPro + mapOutput[prop];
    }

    var seriesItem = {
      name: input[i].name + ' <b>' + totalIncidents + '</b>',
      // total: totalIncidents,
      data: secondOutput
    }
    finalData.push(seriesItem);
  }
}
```

So one more time I have a function to better clean the data to accumulate same incidents in a program with the `reduce` function and also get the total number of incidents in each program.

Now the `finalData` array is looking like what Highcharts is expecting. To complete the `getItemsFromList` function from above:
```javascript
mapper(output);
              DrawChart('bubble-container', finalData);
            },
      );
}
```
Now we come to DrawChart:
```javascript
function DrawChart(render, data) {
        housingChart = new Highcharts.Chart({
            chart: {
                renderTo: render,
                type: 'packedbubble',
                height: '100%'
            },
            title: {
                text: 'Incidents by Program'
            },
            subtitle:{
                text: 'Data as of July 1, 2019: ' + totalAllPro.toString() + ' total incidents'
            },
            // colors: ['#DF5353', '#aaeeee', '#ff0066', '#eeaaee', '#DDDF0D', '#55BF3B', '#DF5353', '#7798BF', '#aaeeee', '#ff0066', '#eeaaee', '#DDDF0D', '#55BF3B', '#DF5353', '#7798BF', '#aaeeee'],
            tooltip: {
                useHTML: true,
                pointFormat: '<b>{point.name}:</b> {point.value}'
            },
            plotOptions: {
              series: {
                animation: false
              },
              packedbubble: {
                  boostThreshold: 1,
                  draggable: false,
                  minSize: '1%',
                  maxSize: '100%',
                  zMin: 0,
                  zMax: 50,
                  layoutAlgorithm: {
                      initialPositionRadius: 100,
                      gravitationalConstant: 0.1,
                      splitSeries: true,
                      seriesInteraction: false,
                      dragBetweenSeries: false,
                      parentNodeLimit: true,
                      parentNodeOptions: {
                          bubblePadding: 30
                      }
                  },
                  dataLabels: {
                      enabled: true,
                      parentNodeFormat: '{point.series.name}',
                      formatter: function() {
                        if (this.y > 10) {
                          return this.point.name;
                        }
                      },
                      style: {
                          color: 'black',
                          textOutline: 'none',
                          fontWeight: 'normal'
                          }
                      }
                  }
            },
            series: finalData
        });
    }
```

As you can see, I'm passing where I want to render the graph and also the data to the JSON that Highcharts is looking for.

## the end result
![Bubble GIF](/images/highchartsSP/fiscalGif.gif)
