# Flutter Solid Principles

## S — The Single Responsibility Principle (SRP):

### Bad

```dart
 @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: new Scaffold(
          body: FutureBuilder(
              future: getFeedsFromApi(),
              builder: (context,  snapshot) {
                  switch(snapshot.connectionState){
                    case ConnectionState.waiting:
                      return EmptySpendsPage();
                    case ConnectionState.none:
                      return EmptySpendsPage();
                    case ConnectionState.active:
                      return EmptySpendsPage();
                    case ConnectionState.done:
                      return ListView.builder(
                          itemCount: snapshot.data == null ? 0 : snapshot.data.length,
                          itemBuilder: (context, position) {
                            FeedItem feedItem = snapshot.data[position];
                            return null;
                          });
                  }

              }
          )
      ),
    );
  }
```

### Good

```dart
 @override
  Widget build(BuildContext context) {
    bloc.add(NoParams());
    return Scaffold(
      body: BlocBuilder<FeedsBloc, FeedsState>(
          bloc: bloc,
          builder: (BuildContext context, FeedsState feedState) {
            if (feedState is FeedsLoading) {
              return Center(child: CircularProgressIndicator());
            } else if (feedState is FeedsError) {
              return Text('Ayo! Error');
            } else if (feedState is FeedsLoaded) {
              return FeedList(feeds: feedState.feedItems);
            }
            return Container(
                color: Colors.orangeAccent,
                height: double.infinity,
                width: double.infinity);
          }),
    );
  }
```
## O — The Open-Closed Principle (OCP)

### Bad

```dart
void main() {
    MileageCalculator mileageCalculator = new MileageCalculator();
    mileageCalculator.showMileage(new Bike());
}

class MileageCalculator {
    void showMileage(dynamic anyVehicle) {
      if(anyVehicle is Bike) {
          print(anyVehicle.getBikeMileage());
      } else if(anyVehicle is Car) {
          print(anyVehicle.getCarMileage());
      }
    }
}

class Bike {
    String getBikeMileage() => "50";
}

class Car {
    String getCarMileage() => "12";
}
```

### Good

```dart
void main() {
    MileageCalculator mileageCalculator = new MileageCalculator();
    mileageCalculator.showMileage(new Car());
}

class MileageCalculator {
    void showMileage(Vehicle vehicle) {
       print(vehicle.getMileage());
    }
}

class Bike extends Vehicle {
    String getMileage() => "50";
}

class Car extends Vehicle {
    String getMileage() => "12";
}

abstract class Vehicle {
  String getMileage();
}
```

## L - The Liskov Substitution Principle (LSP)

### Bad

```dart
void main() {
    StatelessWidget adapter = new StatelessWidget();
    adapter.select(new RadioButtonWidget());
}

class StatelessWidget {
    void select(ClickListener clickListener) {
        if (clickListener is ListItemWidget) { 
            clickListener.changeTheBackground();
        } else if (clickListener is RadioButtonWidget) {
            clickListener.check();
        }
        clickListener.onClick(1);
    }
}

mixin ClickListener {
    void onClick(int position);
} 

class ListItemWidget implements ClickListener {
    @override 
    void onClick(int position){
       print("Clicked list item $position");
    }
    
    void changeTheBackground() {
       print("Change the background color of the item view");
    }
    
}

class RadioButtonWidget implements ClickListener {
    @override 
    void onClick(int position){
       print("Clicked radio button $position");
    }
    
    void check() {
       print("Enable the radio button");
    }
}
```

### Good

```dart
void main() {
    StatelessWidget adapter = new StatelessWidget();
    adapter.select(new RadioButtonWidget());
}

class StatelessWidget {
    void select(ClickListener clickListener) {
        clickListener.onClick(1);
    }
}

mixin ClickListener {
    void onClick(int position);
} 

class ListItemWidget implements ClickListener {
    @override 
    void onClick(int position){
        print("Clicked list item $position");
       _changeTheBackground();
    }
    
    void _changeTheBackground() {
       print("Change the background color of the item view");
    }
}

class RadioButtonWidget implements ClickListener {
    @override 
    void onClick(int position){
       print("Clicked radio button $position");
      _check();
    }
    
    void _check() {
       print("Enable the radio button");
    }
}
```
## I — The Interface Segregation Principle (ISP):

### Bad

```dart 
class ImageViewWidget with OnClickListener {
   @override 
   void onImageClick(int position) {
        // Yes, I have received a callback, go to the next activity.
        print("Clicked position is $position");
   }
   @override 
   void onRadioButtonClick(int position) {
        // This is no longer needed for this activity, but still I have been implemented for no use...
   }
}

 mixin OnClickListener {
    void onImageClick(int position);
    void onRadioButtonClick(int position);
 }
```

### Good

```dart
class ImageViewWidget with OnImageClickListener {
   @override 
   void onImageClick(int position) {
        // Yes, I have received a callback, go to the next activity.
        print("Clicked position is $position");
   }
}

 mixin OnImageClickListener {
    void onImageClick(int position);
 }
```

## D - The Dependency Inversion Principle (DIP)

### Bad
```dart

class RepositoryImpl {
  List<FeedItem> getDataFromApi() {
    // do your api call
  }
  
  List<FeedItem> getDataFromCache() {
    // get the data from the database
  }
}
```

### Good
```dart
class RepositoryImpl {
  
  RemoteDataSource remoteDataSource;
  LocalDataSource locaDataSource;
  
  RepositoryImpl(this.remoteDataSource, this.locaDataSource);
}

abstract class RemoteDataSource {
  List<FeedItem> getDataFromApi();
}

abstract class LocalDataSource {
  List<FeedItem> getDataFromCache();
}

class RemoteDataSourceImpl extends RemoteDataSource{
  // Lets assume this class has an access to http client
  List<FeedItem> getDataFromApi(){
        // do your api call
  }
}

class LocalDataSourceImpl extends LocalDataSource {
    // Lets assume this class has an access to database instance
  List<FeedItem> getDataFromCache(){
        // get the data from the database
  }
}
```
