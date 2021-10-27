# flutter_http_networking

https://blog.logrocket.com/networking-flutter-using-http-package/

https://github.com/sbis04/flutter_http_networking

https://pub.dev/packages/http

http: ^0.13.4

zusätzlich:

json_annotation: ^4.0.1

läuft auf IOS und Android inkl Tests




======================

http: sample läuft auf IOS und Android

import 'dart:convert' as convert;

import 'package:http/http.dart' as http;

void main(List<String> arguments) async {
// This example uses the Google Books API to search for books about http.
// https://developers.google.com/books/docs/overview
var url =
Uri.https('www.googleapis.com', '/books/v1/volumes', {'q': '{http}'});

// Await the http get response, then decode the json-formatted response.
var response = await http.get(url);
if (response.statusCode == 200) {
var jsonResponse =
convert.jsonDecode(response.body) as Map<String, dynamic>;
var itemCount = jsonResponse['totalItems'];
print('Number of books about http: $itemCount.');
} else {
print('Request failed with status: ${response.statusCode}.');
}
}

output:
flutter: Number of books about http: 1239.

A new Flutter project.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://flutter.dev/docs/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://flutter.dev/docs/cookbook)

For help getting started with Flutter, view our
[online documentation](https://flutter.dev/docs), which offers tutorials,
samples, guidance on mobile development, and a full API reference.
