Example Implementation:

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

import 'custom_info_widget.dart';

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
    child: Container(
      color: Colors.white,
      width: 50,
      height: 50,
      child: Text('Lorem Ipsum'),
    ),
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
          onCameraMove: (newPosition) {
            _mapIdleSubscription?.cancel();
            _mapIdleSubscription = Future.delayed(Duration(milliseconds: 150))
                .asStream()
                .listen((_) {
              if (_infoWidgetRoute != null) {
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

  /// now my _onTap Method. First it creates the Info Widget Route and then animates the Camera near the 
  /// location of the Marker and then on the Marker. The 2 animate Cameras are called to ensure that
  /// the faked on Map Idle always gets called.

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
