https://blog.logrocket.com/networking-flutter-using-http-package/

Networking in Flutter using the http package
July 13, 2021   7 min read

Networking in Flutter using the http package
Most apps have to perform network requests over the internet. As such, it’s important to handle network calls elegantly to avoid unnecessary errors in API calls.

In this article, we’ll take a look at how we can handle REST API requests in Flutter using the http package.

Getting started

Create a new Flutter project using the following command:

flutter create flutter_http_networking
You can open the project using your favorite IDE, but for this example, I’ll be using VS Code:

code flutter_http_networking
Add the http package to your pubspec.yaml file:

dependencies:
http: ^0.13.3
Replace the content of your main.dart file with the following basic structure:

import 'package:flutter/material.dart';
import 'screens/home_page.dart';

void main() {
runApp(MyApp());
}

class MyApp extends StatelessWidget {
@override
Widget build(BuildContext context) {
return MaterialApp(
title: 'Flutter Networking',
theme: ThemeData(
primarySwatch: Colors.teal,
),
debugShowCheckedModeBanner: false,
home: HomePage(),
);
}
}
We will create the HomePage after having a look at the API that will perform the network operations.

Requesting API data

For this demonstration of API requests, we will use the sample /posts data from JSONPlaceholder. We will start by fetching a single post data using the GET request. The endpoint that you have to use is:

GET https://jsonplaceholder.typicode.com/posts/<id>
Here, you have to replace the <id> with an integer value representing the post ID that you want to retrieve.

If your request is successful, the sample JSON response that you’ll receive will look like this:

{
"userId": 1,
"id": 1,
"title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
"body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
Specifying model class

The model class helps you to package the data returned by an API call or neatly send data using a network request.

We will define a model class for handling a single post data. You can use a JSON-to-Dart class conversion tool like quicktype to easily generate a model class. Copy and paste it inside a file called post.dart:

class Post {
Post({
this.id,
this.userId,
this.title,
this.body,
});

int? id;
int? userId;
String? title;
String? body;

factory Post.fromJson(Map<String, dynamic> json) => Post(
userId: json["userId"],
id: json["id"],
title: json["title"],
body: json["body"],
);

Map<String, dynamic> toJson() => {
"userId": userId,
"id": id,
"title": title,
"body": body,
};
}
Alternatively, you can also use JSON serialization and generate the fromJson and toJson methods automatically, which helps prevent any unnoticed errors that might occur while defining manually.

If you use JSON serialization, you will need the following packages:

json_serializable
json_annotation
build_runner
Add them to your pubspec.yaml file:

dependencies:
json_annotation: ^4.0.1

dev_dependencies:
json_serializable: ^4.1.3
build_runner: ^2.0.4
To use JSON serialization, you have to modify the Post class as follows:

import 'package:json_annotation/json_annotation.dart';

part 'post.g.dart';

@JsonSerializable()
class Post {
Post({
this.id,
this.userId,
this.title,
this.body,
});

int? id;
int? userId;
String? title;
String? body;

factory Post.fromJson(Map<String, dynamic> json) => _$PostFromJson(json);

Map<String, dynamic> toJson() => _$PostToJson(this);
}
You can trigger the code generation tool using the following command:

flutter pub run build_runner build
If you want to keep the code generation tool running in the background — which will automatically apply any further modifications you make to the model class — use the command:

flutter pub run build_runner serve --delete-conflicting-outputs
The --delete-conflicting-outputs flag helps to regenerate a part of the generated class if any conflicts are found.

Performing API requests

Now you can start performing the various network requests on the REST API. To keep your code clean, you can define the methods related to network requests inside a separate class.

Create a new file called post_client.dart, and define PostClient class inside it:

class PostClient {
// TODO: Define the methods for network requests
}
Define the base URL of the server along with the required endpoint in variables:

class PostClient {
static final baseURL = "https://jsonplaceholder.typicode.com";
static final postsEndpoint = baseURL + "/posts";
}
We will use these variables while performing the requests.

Fetching data

You can use the GET request to retrieve information from the API. To fetch a single post data, you can define a method like this:

Future<Post> fetchPost(int postId) async {
final url = Uri.parse(postsEndpoint + "/$postId");
final response = await http.get(url);
}
This method tries to retrieve a post data according to the ID passed to it. The http.get() method uses the URL to fetch data from the server which is stored in the response variable.

Verify whether or not the request was successful by checking the HTTP status code, which should be 200 if successful. You can now decode the raw JSON data and use Post.fromJson() to store it in a nice, structured manner using the model class.

Future<Post> fetchPost(int postId) async {
final url = Uri.parse(postsEndpoint + "/$postId");
final response = await http.get(url);

if (response.statusCode == 200) {
return Post.fromJson(jsonDecode(response.body));
} else {
throw Exception('Failed to load post: $postId');
}
}
Sending data

You can use the POST request for sending data to the API. We will create a new post by sending data using the following method:

Future<Post> createPost(String title, String body) async {
final url = Uri.parse(postsEndpoint);
final response = await http.post(
url,
headers: {
'Content-Type': 'application/json; charset=UTF-8',
},
body: jsonEncode({
'title': title,
'body': body,
}),
);
}
While sending data, you have to specify the header type in headers and the body that you want to send to the specified endpoint. Also, the JSON data should be sent in an encoded format using the jsonEncode method.

You can check whether your POST request was successful using the HTTP status code. If it returns a status code of 201, then the request was successful and we can return the post data.

Future<Post> createPost(String title, String body) async {
// ...

if (response.statusCode == 201) {
return Post.fromJson(jsonDecode(response.body));
} else {
throw Exception('Failed to create post');
}
}
Updating data

You can update any post information present on the API server by using the PUT request. Define the method like this:

Future<Post> updatePost(int postId, String title, String body) async {
final url = Uri.parse(postsEndpoint + "/$postId");
final response = await http.put(
url,
headers: {
'Content-Type': 'application/json; charset=UTF-8',
},
body: jsonEncode({
'title': title,
'body': body,
}),
);
}
Here, we have used the post ID to specify which post to update and send the request to. If the response status code is 200, then the request was successful and you can return the updated post sent by the server.

Future<Post> updatePost(int postId, String title, String body) async {
// ...

if (response.statusCode == 200) {
return Post.fromJson(jsonDecode(response.body));
} else {
throw Exception('Failed to update post');
}
}
Deleting data

You can remove a post from the API server by using the DELETE request. The method can be defined as follows:

Future<Post> deletePost(int postId) async {
final url = Uri.parse(postsEndpoint + "/$postId");
final response = await http.delete(
url,
headers: {
'Content-Type': 'application/json; charset=UTF-8',
},
);

if (response.statusCode == 200) {
return Post.fromJson(jsonDecode(response.body));
} else {
throw Exception('Failed to delete post: $postId');
}
}
We have used the post ID to specify which post to delete and send the request to the respective endpoint. You can verify whether the request was successful by checking if the HTTP status code is 200.

You should note that, in the case of deletion, the response returns empty data.

Future<Post> deletePost(int postId) async {
//...

if (response.statusCode == 200) {
return Post.fromJson(jsonDecode(response.body));
} else {
throw Exception('Failed to delete post: $postId');
}
}
Building the UI

The user interface will be defined within the HomePage widget. It will be a StatefulWidget because we will need to update its state after each network request.

import 'package:flutter/material.dart';
import 'package:flutter_http_networking/utils/post_client.dart';
import 'package:http/http.dart' as http;

class HomePage extends StatefulWidget {
@override
_HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
@override
Widget build(BuildContext context) {
return Scaffold();
}
}
First, we’ll define the two variables that will store the title and body of the post returned by an API call, and then we’ll initialize the PostClient class:

import 'package:flutter/material.dart';
import 'package:flutter_http_networking/utils/post_client.dart';
import 'package:http/http.dart' as http;

class HomePage extends StatefulWidget {
@override
_HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
final PostClient _postClient = PostClient();

String? _postTitle;
String? _postBody;

@override
Widget build(BuildContext context) {
return Scaffold();
}
}
We will now create buttons that will trigger the network request methods and update the variables with the information returned by the server. Below is the code snippet for triggering the fetchPost() method:

ElevatedButton(
onPressed: () async {
final post = await _postClient.fetchPost(1);
setState(() {
_postTitle = post.title;
_postBody = post.body;
});
},
child: Text('GET'),
)
You can trigger the remaining network request methods similarly.

The final app UI looks like this:

Example of the final app's UI

Testing network requests

You can test the API requests in Dart using Mockito, a package that helps to mimic network requests and test whether your app effectively handles the various types of requests, including null and error responses.

To test with Mockito, add it to your pubspec.yaml file and make sure that you also have the build_runner and flutter_test dependencies set:

dev_dependencies:
flutter_test:
sdk: flutter
build_runner: ^2.0.4
mockito: ^5.0.10
Now, you have to do a small modification to the network request methods that you want to test. We will do the modification to the fetchPost() method.

Provide a http.Client to the method, and use the client to perform the get() request. The modified method will look like this:

Future<Post> fetchPost(http.Client client, int postId) async {
final url = Uri.parse(postsEndpoint + "/$postId");
final response = await client.get(url);

if (response.statusCode == 200) {
return Post.fromJson(jsonDecode(response.body));
} else {
throw Exception('Failed to load post: $postId');
}
}
Create a test file called fetch_post_test.dart inside the test folder, and annotate the main function with @GenerateMocks([http.Client]).

import 'package:flutter_http_networking/models/post.dart';
import 'package:flutter_http_networking/utils/post_client.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

import 'fetch_post_test.mocks.dart';

@GenerateMocks([http.Client])
void main() {}
This will help to generate the MockClient class using the build_runner tool. Trigger the code generation by using the following command:

flutter pub run build_runner build
We will define just two tests: one for a successful API request, and one for an unsuccessful request with an error. You can test these conditions using the when() function provided by Mockito.

@GenerateMocks([http.Client])
void main() {
PostClient _postClient = PostClient();

final postEndpoint =
Uri.parse('https://jsonplaceholder.typicode.com/posts/1');

group('fetchPost', () {
test('successful request', () async {
final client = MockClient();

      when(
        client.get(postEndpoint),
      ).thenAnswer((_) async => http.Response(
          '{"userId": 1, "id": 2, "title": "mock post", "body": "post body"}',
          200));

      expect(await _postClient.fetchPost(client, 1), isA<Post>());
    });

    test('unsuccessful request', () {
      final client = MockClient();

      when(
        client.get(postEndpoint),
      ).thenAnswer((_) async => http.Response('Not Found', 404));

      expect(_postClient.fetchPost(client, 1), throwsException);
    });
});
}
You can run the tests using the command:

flutter test test/fetch_post_test.dart
Or, you can also run the tests directly inside your IDE. When I ran the tests using VS Code, I received this result:

Results of the tests when run in VS Code

Conclusion

The http package helps to perform all types of network requests in Flutter. It also provides support for Mockito that simplifies the testing of API calls.

If you want to have more advanced control over your requests, then you can use the Dio package, which helps to avoid some boilerplate code and has support for Global configuration, Interceptors, Transformers, and other features.

Thank you for reading this article! If you have any suggestions or questions about the article or examples, feel free to connect with me on Twitter or LinkedIn. You can find the repository of the sample app on my GitHub.