Implementation

First I do have a Object point which consits of a location and a Widget. And i do calculate the taxicapDistance.

```dart
class Point {
  final Widget child;
  final Location location;
  
  Point({this.child, this.location});
}

Marker(
    markerId: MarkerId(point.location.latitude.toString() + point.location.longitude.toString()),
    position: point.location,
    onTap: () => _onTap(point),
    );



  double _taxicabDistance(LatLng p1, LatLng p2) =>
      (p1.latitude - p2.latitude).abs() + (p1.longitude - p2.longitude).abs();
```

now my _onTap Method. First it animates the Camera to the location of the Marker and then the Navigator pushes the PopUp Route of the Info Widget.

The Future delayed is needed, because the move animation takes some time.

```dart
_onTap(PointObject point) async {
    final RenderBox renderBox = context.findRenderObject();
    Rect _itemRect = renderBox.localToGlobal(Offset.zero) & renderBox.size;

    InfoWidgetRoute _infoWidgetRoute = InfoWidgetRoute(
      child: point.child,
      buildContext: context,
      textStyle: const TextStyle(
        fontSize: 14,
        color: Colors.black,
      ),
      mapsWidgetSize: _itemRect,
    );

     if (_taxicabDistance(_cameraPosition.target, point.location) > 0.0001) {
      await mapController.animateCamera(
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

      Future.delayed(
        Platform.isIOS
            ? Duration(milliseconds: 500)
            : Duration(milliseconds: 1250),
        () {

          Navigator.of(context, rootNavigator: true)
              .push(_infoWidgetRoute)
              .then<void>(
            (newValue) {
              _infoWidgetRoute = null;
            },
          );
        },
      );
    } else {
      Navigator.of(context, rootNavigator: true)
          .push(_infoWidgetRoute)
          .then<void>(
        (newValue) {
          _infoWidgetRoute = null;
        },
      );
    }
}
```
