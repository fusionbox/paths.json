paths.json
==========

I would like to see a countries.json file added here sometime.

Until then, this is a json file (us_states.json) that is useful if you need to draw US state borders.  But not just one state,
what if you need to draw the border around Colorado, Utah, and Nevada.

The format of the data is:

``` json
[
  // ...
  , { "name": "Arkansas",
      "paths": [
        [[33.0075,-91.2057],[33.1180,-91.1989],[33.1824,-91.1041],[33.3053,-91.1343],[33.4337,-91.2263],[33.5403,-91.2524],[33.6112,-91.1797],[33.6855,-91.2524],[33.6946,-91.1261],[33.7883,-91.1412],[33.7700,-91.0451],[33.8328,-91.0341],[33.9399,-91.0863],[33.9889,-90.9998],[34.0208,-90.9256],[34.0856,-90.9036],[34.1675,-90.9132],[34.1380,-90.8501],[34.2311,-90.9325],[34.2833,-90.8267],[34.3446,-90.6935],[34.3978,-90.6152],[34.4409,-90.5603],[34.5348,-90.5548],[34.7574,-90.5328],[34.8780,-90.4546],[34.8690,-90.2911],[35.0255,-90.3104]]
      , [[35.0255,-90.3104],[35.1154,-90.2843],[35.1323,-90.1772],[35.1985,-90.1112],[35.2826,-90.1524],[35.4383,-90.1332],[35.5579,-90.0206],[35.7287,-89.9547],[35.9169,-89.6594],[36.0013,-89.7130]]
      , [[36.0013,-89.7130],[35.9958,-90.3735],[36.1268,-90.2664],[36.2875,-90.0934],[36.3892,-90.0742],[36.4180,-90.1511],[36.4997,-90.1566],[36.4986,-94.6198]]
      , [[36.4986,-94.6198],[35.3801,-94.4412],[33.6421,-94.4522]]
      , [[33.6421,-94.4522],[33.5597,-94.4000],[33.5689,-94.3176],[33.5872,-94.1885],[33.5345,-94.0375],[33.4314,-94.0430],[33.0213,-94.0430]]
      , [[33.0213,-94.0430],[33.0075,-91.2057]]
        ]
    }
  // ...
]
```

You can use this data to combine multiple states' paths.  Just compare the start and end points of the paths, and remove paths
that share both start and end points.  Here's some php code that does exactly that.

``` php
<?php

$regions = array(/* the regions from us_states.json that you want to combine */);

// flatten the paths into one list
$all_paths = array();
foreach($regions as $region)
{
  $region['paths'] = json_decode($region['paths']);

  if ( empty($all_paths) )
    $all_paths = $region['paths'];
  else
    $all_paths = array_merge($all_paths, $region['paths']);
}

// remove duplicate paths by comparing start and end points
$paths = array();
while ( $path = array_shift($all_paths) )
{
  $first_point = $path[0];
  $last_point = $path[count($path) - 1];
  $include = TRUE;

  foreach($all_paths as $check => $check_path)
  {
    $check_path = $all_paths[$check];

    $check_first = $check_path[0];
    $check_last = $check_path[count($check_path) - 1];

    if ( $first_point == $check_first && $last_point == $check_last
      || $first_point == $check_last && $last_point == $check_first )
    {
      $include = FALSE;
      unset($all_paths[$check]);
      break;
    }
  }

  if ( $include )
    $paths[] = $path;
}
```
