---
published: false
title: Geocoding in iOS
layout: post
---
- in project properties, under Capabilities, activate Maps.
- in the storyboard, add a Map Kit View, and a Search Bar and Search Display Controller
- in ViewController import MapKit, then create an outlet for the Map Kit View, and name it __mapView__. Also create a CLPlacemark property and name it __placemark__.