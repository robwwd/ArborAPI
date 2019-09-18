# ArborAPI PHP class

PHP class for Arbor/Netscout SP/Siteline API

## What is the ArborAPI PHP Class?

ArborAPI is a php class to interface with Arbor/Netscout SP deployments.

## Features

ArborAPI supports the following:

-   Support for Arbor REST API
-   Support for Arbor Web services API
-   Currently testing with Arbor SP 8.2 but should work with versions above that.

## Requirements

ArborAPI PHP Class requires the following:

-   PHP 7.0 or higher
-   ext/curl, <https://www.php.net/manual/en/book.curl.php>

## Installation

Installation is via Composer (although can install the source directly into your project.)

## Usage

Initialising the class needs a configuration array as follows.

```php
$configiration = Array (
    [ipaddress] => 10.1.1.1,                      # IP address of the leader device
    [hostname] => "sp.example.com",               # Hostname of the leader device
    [apikey] => "9isdfpiosdpfokdssadsadasd",      # Web Service API Key
    [resttoken] => "sanbdnmsduihiunckasdjskldjas" # REST API Token.
);

// Create Arbor\API instance.
//
$arborApi = new Arbor\API($configuration);

// Get all peer managed objects.
//
$arborMOList = $arborApi->getManagedObjects('family', 'peer');

// Check if there is an error and print the error.
//
if ($arborApi->hasError()) {
    echo $arborApi->errorMessage();
    exit();
} else {
    [.. do something ..]
}
```

### Traffic Graphs

Traffic graph functions get a PNG image generated by Arbor SP based on a query and a graph definition both in XML.
Searching for Query XML or Graph XML in the online documentation for Arbor SP or through the API guides gives some
insite into how the XML should look.

There is also some helper functions to build the XML long with some wrapper functions for getting common graphs.

#### filters

Filters can be defined in the Query XML functions to build the filter matches (as per the Arbor SP documentation).
Filters are an array with the type (aspath, peer, customer, interface etc) and the query value applied to the filter.
If binby is true it displays each unique data stream on the graph as separate lines with a legend.

```php

// Aspath filter (see Arbor Docs for filter list) with the filter '_701_'. Will show each AS Path found
// as individual graph items.

$filters = [
    ['type' => 'aspath', 'value' => '_701_', 'binby' => true],
];

// Filter by peer managed object with ID of 10. Show traffic graphs with individual graph items on a per
// application basis.

$filters = [
    ['type' => 'peer', 'value' => '10', 'binby' => false],
    ['type' => 'application', 'value' => null, 'binby' => true]
];

```

You can use these filters to build the XML. Filters are limited to two filters per query by Arbor. There are
however no checks for this in the class.

#### Example 1: AS Path Graph.

```PHP
$arborApi = new Arbor\API($configuration);

// Build a filter line for the Query XML.

$filters = [
    ['type' => 'aspath', 'value' => '_701_', 'binby' => true],
];

// Query for date from the past for weeks (21 days go until now) and display bps.
//
$queryXML = buildQueryXML($filters, '21 days ago', 'now', 'bps')

// Build the graph XML with title 'ASN traffic'. By default units are bps.
$graphXML = $this->buildGraphXML('ASN traffic');

$image = base64_encode($arborApi->getTrafficGraph($queryXML, $graphXML));

// Echo image to the browser.

if ($image) {
    echo '<img src="data:image/png;base64,'.$image.'">';
}
```
#### Example 2: Detail graph for a peer

This example builds a detail graph for a peer managed object containing data from the last week. It displays a graph
item for in, out and total traffic.

```PHP

$arborApi = new Arbor\API($configuration);

$filters = [
    ['type' => 'peer', 'value' => $arborID, 'binby' => false],
];

// The last argument to the QueryXML build is for each class of traffic that should be retried (and displayed).
// The array can contain: in, out, total, backbone and dropped. (see Arbor documentation for up-to-date class fields)
//
$queryXML = $this->buildQueryXML($filters, '7 days ago', 'now', 'bps', ['in', 'out', 'total']);

// Build the graph, this time set the last argument 'detailed' to true to get a detailed graph.
$graphXML = $this->buildGraphXML('Peer traffic', 'bps', true);

$image = base64_encode($arborApi->getTrafficGraph($queryXML, $graphXML));

```
