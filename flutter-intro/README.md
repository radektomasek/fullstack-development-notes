# Flutter Introduction - workshop

In this session we are going to implement a simple app which fetches data from weather service and display them on a screen.

## Prerequisites

To be able to practically follow this workshop, you need to make sure following criteria are met:

- Installation of the flutter environment (based on the OS) [https://flutter.dev/docs/get-started/install](https://flutter.dev/docs/get-started/install)
- Making sure either Android Emulator or iOS Simulator are working.
- Register an account at [https://openweathermap.org/api](https://openweathermap.org/api) and **obtain an API key**.
- Checking out the project (containing some extra resources like an image and font)
- Run the project from a terminal to make sure the code is up and running.

```shell
# list available emulators
flutter emulators
# launch a desired emulator
flutter emulators --launch <EMULATOR_ID>
# run the app in the emulator
flutter run
```

> Note: This code is written in [VSCode](https://code.visualstudio.com/). I strongly recommend using it during this workshop (despite the fact the Android Studio has an amazing support for Flutter). While using VSCode, make sure the **Flutter** and **Awesome Flutter Snippets** extensions are installed.

## Getting Started

Once the environment is setup, it's time to start coding.

### 1. Initial structure showing a simple button in the centre of the screen

```dart
// main.dart

import 'package:flutter/material.dart';

void main() => runApp(
      MaterialApp(
        title: 'Weather App',
        home: InitPage(),
      ),
    );

class InitPage extends StatefulWidget {
  @override
  _InitPageState createState() => _InitPageState();
}

class _InitPageState extends State<InitPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather App'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            RaisedButton(
              child: Text('Check'),
              onPressed: () {
                print('Checked button clicked');
              },
            )
          ],
        ),
      ),
    );
  }
}
```

### 2. Initialization of the navigation/routing

```dart
// result.dart

import 'package:flutter/material.dart';

class ResultPage extends StatefulWidget {
  @override
  _ResultPageState createState() => _ResultPageState();
}

class _ResultPageState extends State<ResultPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Result Page'),
      ),
      body: Center(
        child: Column(
          children: <Widget>[
            RaisedButton(
              child: Text('New search'),
              onPressed: () {
                Navigator.pop(context);
              },
            )
          ],
        ),
      ),
    );
  }
}
```

```dart
// main.dart

import 'package:flutter/material.dart';
import 'result.dart';

void main() => runApp(
      MaterialApp(
        title: 'Weather App',
        home: InitPage(),
      ),
    );

class InitPage extends StatefulWidget {
  @override
  _InitPageState createState() => _InitPageState();
}

class _InitPageState extends State<InitPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather App'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            RaisedButton(
              child: Text('Check'),
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => ResultPage(),
                  ),
                );
              },
            )
          ],
        ),
      ),
    );
  }
}
```

### 3. Add an image and text field which exceed the allowed dimension.

```dart
// main.dart

import 'package:flutter/material.dart';
import 'result.dart';

void main() => runApp(
      MaterialApp(
        title: 'Weather App',
        home: InitPage(),
      ),
    );

class InitPage extends StatefulWidget {
  @override
  _InitPageState createState() => _InitPageState();
}

class _InitPageState extends State<InitPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather App'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Image.asset('images/weather.png'),
            TextField(
              onChanged: (value) {
                print(value);
              },
            ),
            RaisedButton(
              child: Text('Check'),
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => ResultPage(),
                  ),
                );
              },
            )
          ],
        ),
      ),
    );
  }
}
```

### 4. Resolving the dimension issue.

```dart
// main.dart

import 'package:flutter/material.dart';
import 'result.dart';

void main() => runApp(
      MaterialApp(
        title: 'Weather App',
        home: InitPage(),
      ),
    );

class InitPage extends StatefulWidget {
  @override
  _InitPageState createState() => _InitPageState();
}

class _InitPageState extends State<InitPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather App'),
      ),
      body: SingleChildScrollView(
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Image.asset('images/weather.png'),
              TextField(
                onChanged: (value) {
                  print(value);
                },
              ),
              RaisedButton(
                child: Text('Check'),
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => ResultPage(),
                    ),
                  );
                },
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

### 5. Passing data via route + centre the column on the main axis on the Result route.

```dart
// result.dart

import 'package:flutter/material.dart';

class ResultPage extends StatefulWidget {
  final String cityName;

  ResultPage(this.cityName);

  @override
  _ResultPageState createState() => _ResultPageState();
}

class _ResultPageState extends State<ResultPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Result Page'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(widget.cityName),
            RaisedButton(
              child: Text('New search'),
              onPressed: () {
                Navigator.pop(context);
              },
            )
          ],
        ),
      ),
    );
  }
}
```

```dart
// main.dart

import 'package:flutter/material.dart';
import 'result.dart';

void main() => runApp(
      MaterialApp(
        title: 'Weather App',
        home: InitPage(),
      ),
    );

class InitPage extends StatefulWidget {
  @override
  _InitPageState createState() => _InitPageState();
}

class _InitPageState extends State<InitPage> {
  String cityName = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather App'),
      ),
      body: SingleChildScrollView(
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Image.asset('images/weather.png'),
              TextField(
                onChanged: (value) {
                  setState(() {
                    cityName = value;
                  });
                },
              ),
              RaisedButton(
                child: Text('Check'),
                onPressed: () {
                  print(cityName);
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => ResultPage(cityName),
                    ),
                  );
                },
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

### 6. Adding async methods.

> Weather API GET request string - http://api.openweathermap.org/data/2.5/weather?q=${cityName}&units=metric&appid=${apiKey}

```dart
// result.dart

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

class Weather {
  final double temperature;

  Weather({this.temperature});

  factory Weather.fromJson(Map<String, dynamic> json) {
    return Weather(temperature: json['main']['temp'].toDouble());
  }
}

Future<Weather> fetchWeather(String cityName, String apiKey) async {
  final response = await http.get(Uri.encodeFull(
      'http://api.openweathermap.org/data/2.5/weather?q=${cityName}&units=metric&appid=${apiKey}'));

  if (response.statusCode == 200) {
    return Weather.fromJson(json.decode(response.body));
  } else {
    throw Exception('Failed to fetch weather data');
  }
}

class ResultPage extends StatefulWidget {
  final String cityName;
  ResultPage(this.cityName);

  @override
  _ResultPageState createState() => _ResultPageState();
}

class _ResultPageState extends State<ResultPage> {
  Future<Weather> temperature;

  @override
  void initState() {
    super.initState();
    temperature = fetchWeather(widget.cityName, 'API_KEY');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Result Page'),
      ),
      body: FutureBuilder<Weather>(
        future: temperature,
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Text('The temperature in'),
                  Text(widget.cityName.toUpperCase()),
                  Text('is ${snapshot.data.temperature.toString()}'),
                  RaisedButton(
                    child: Text('New search'),
                    onPressed: () {
                      Navigator.pop(context);
                    },
                  )
                ],
              ),
            );
          } else if (snapshot.hasError) {
            return Center(
              child: Column(
                children: <Widget>[
                  Text('Problem with loading page: ${snapshot.error}'),
                  RaisedButton(
                    child: Text('New search'),
                    onPressed: () {
                      Navigator.pop(context);
                    },
                  )
                ],
              ),
            );
          }

          return Center(
            child: CircularProgressIndicator(),
          );
        },
      ),
    );
  }
}
```

### 7. Adding styles & final polishing.

```dart
// result.dart

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

class Weather {
  final double temperature;

  Weather({this.temperature});

  factory Weather.fromJson(Map<String, dynamic> json) {
    return Weather(temperature: json['main']['temp'].toDouble());
  }
}

Future<Weather> fetchWeather(String cityName, String apiKey) async {
  final response = await http.get(Uri.encodeFull(
      'http://api.openweathermap.org/data/2.5/weather?q=${cityName}&units=metric&appid=${apiKey}'));

  if (response.statusCode == 200) {
    return Weather.fromJson(json.decode(response.body));
  } else {
    throw Exception('Failed to fetch weather data');
  }
}

class ResultPage extends StatefulWidget {
  final String cityName;
  ResultPage(this.cityName);

  @override
  _ResultPageState createState() => _ResultPageState();
}

class _ResultPageState extends State<ResultPage> {
  Future<Weather> temperature;

  @override
  void initState() {
    super.initState();
    temperature = fetchWeather(widget.cityName, '');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Result Page'),
      ),
      body: FutureBuilder<Weather>(
        future: temperature,
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Text(
                    'The temperature in',
                    style: TextStyle(
                      fontSize: 30.0,
                      fontWeight: FontWeight.normal,
                    ),
                  ),
                  SizedBox(
                    height: 20.0,
                  ),
                  Text(
                    widget.cityName.toUpperCase(),
                    style: TextStyle(
                        fontSize: 40.0,
                        fontWeight: FontWeight.bold,
                        fontFamily: 'SedgwickAveDisplay'),
                  ),
                  SizedBox(
                    height: 20.0,
                  ),
                  Text(
                    'is ${snapshot.data.temperature.toString()}',
                    style: TextStyle(
                      fontSize: 30.0,
                      fontWeight: FontWeight.normal,
                    ),
                  ),
                  SizedBox(
                    height: 20.0,
                  ),
                  RaisedButton(
                    child: Text('New search'),
                    onPressed: () {
                      Navigator.pop(context);
                    },
                  )
                ],
              ),
            );
          } else if (snapshot.hasError) {
            return Center(
              child: Column(
                children: <Widget>[
                  Text('Problem with loading page: ${snapshot.error}'),
                  RaisedButton(
                    child: Text('New search'),
                    onPressed: () {
                      Navigator.pop(context);
                    },
                  )
                ],
              ),
            );
          }

          return Center(
            child: CircularProgressIndicator(),
          );
        },
      ),
    );
  }
}
```

```dart
// main.dart
import 'package:flutter/material.dart';
import 'result.dart';

void main() => runApp(
      MaterialApp(
        title: 'Weather App',
        home: InitPage(),
      ),
    );

class InitPage extends StatefulWidget {
  @override
  _InitPageState createState() => _InitPageState();
}

class _InitPageState extends State<InitPage> {
  String cityName = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather App'),
      ),
      body: SingleChildScrollView(
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Image.asset('images/weather.png'),
              SizedBox(
                height: 20.0,
              ),
              TextField(
                decoration: InputDecoration(
                  hintStyle: TextStyle(fontSize: 17.0),
                  hintText: 'Enter a city',
                  suffixIcon: Icon(Icons.search),
                  contentPadding: EdgeInsets.all(20),
                ),
                onChanged: (value) {
                  setState(() {
                    cityName = value;
                  });
                },
              ),
              SizedBox(
                height: 20.0,
              ),
              RaisedButton(
                child: Text('Check'),
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => ResultPage(cityName),
                    ),
                  );
                },
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com).
