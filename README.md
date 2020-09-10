Example Implementation:

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';



void main() => runApp(MyApp());

class PointObject {
  final Widget child;
  final LatLng location;

  PointObject({this.child, this.location});
}

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      initialRoute: "/",
      routes: {
        "/": (context) => HomePage(),
      },
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  PointObject point = PointObject(
    child: Text('Lorem Ipsum'),
    location: LatLng(47.6, 8.8796),
  );

  StreamSubscription _mapIdleSubscription;
  InfoWidgetRoute _infoWidgetRoute;
  GoogleMapController _mapController;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        color: Colors.green,
        child: GoogleMap(
          initialCameraPosition: CameraPosition(
            target: const LatLng(47.6, 8.6796),
            zoom: 10,
          ),
          circles: Set<Circle>()
            ..add(Circle(
              circleId: CircleId('hi2'),
              center: LatLng(47.6, 8.8796),
              radius: 50,
              strokeWidth: 10,
              strokeColor: Colors.black,
            )),
          markers: Set<Marker>()
            ..add(Marker(
              markerId: MarkerId(point.location.latitude.toString() +
                  point.location.longitude.toString()),
              position: point.location,
              onTap: () => _onTap(point),
            )),
          onMapCreated: (mapController) {
            _mapController = mapController;
          },

          /// This fakes the onMapIdle, as the googleMaps on Map Idle does not always work
          /// When the Map Idles and a _infoWidgetRoute exists, it gets displayed.
          ///  (see: https://github.com/flutter/flutter/issues/37682)
          
          /// The additional arguments in the if statement, are used to supress the info 
          /// widget if the user changes the camera position while the camera is animating
          /// towards the Location Marker
          onCameraMove: (newPosition) {
            _mapIdleSubscription?.cancel();
            _mapIdleSubscription = Future.delayed(Duration(milliseconds: 150))
                .asStream()
                .listen((_) {
              if (_infoWidgetRoute != null &&
                 ((newPosition.target.latitude -
                          point.location.latitude) *
                      1000)
                  .toInt() ==
              0 &&
          ((_currentPosition.target.longitude -
                          point.location.longitude) *
                      1000)
                  .toInt() ==
              0)
                Navigator.of(context, rootNavigator: true)
                    .push(_infoWidgetRoute)
                    .then<void>(
                  (newValue) {
                    _infoWidgetRoute = null;
                  },
                );
              }
            });
          },
        ),
      ),
    );
  }

  /// now my _onTap Method. First it creates the Info Widget Route and then
  /// animates the Camera twice:
  /// First to a place near the marker, then to the marker.
  /// This is done to ensure that onCameraMove is always called

  _onTap(PointObject point) async {
    final RenderBox renderBox = context.findRenderObject();
    Rect _itemRect = renderBox.localToGlobal(Offset.zero) & renderBox.size;

    _infoWidgetRoute = InfoWidgetRoute(
      child: point.child,
      buildContext: context,
      textStyle: const TextStyle(
        fontSize: 14,
        color: Colors.black,
      ),
      mapsWidgetSize: _itemRect,
    );

    await _mapController.animateCamera(
      CameraUpdate.newCameraPosition(
        CameraPosition(
          target: LatLng(
            point.location.latitude - 0.0001,
            point.location.longitude,
          ),
          zoom: 15,
        ),
      ),
    );
    await _mapController.animateCamera(
      CameraUpdate.newCameraPosition(
        CameraPosition(
          target: LatLng(
            point.location.latitude,
            point.location.longitude,
          ),
          zoom: 15,
        ),
      ),
    );
  }
}
```
