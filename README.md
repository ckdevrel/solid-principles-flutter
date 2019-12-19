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
```kotlin
fun main() {
    val adapter = Adapter()
    adapter(object: Adapter.OnClickListener {
        override fun onItemClick(position: Int) {
            // Yes, I have received a callback, go to the next activity.
            println("Clicked position is $position")
        }
        override fun onRadioButtonClick(position: Int) {
            // This is no longer needed for this activity, but still I have been implemented for no use...
        }

    })
    adapter.execute()
}

class Adapter {

    private var onClickListener: OnClickListener? =null
   
    operator fun invoke (onClickListener: OnClickListener) {
        this.onClickListener = onClickListener
    }

    fun execute() {
        onClickListener?.onItemClick(4)
    }
    
    interface OnClickListener {
        fun onItemClick(position: Int)
        fun onRadioButtonClick(position: Int)
    }
}

```
### Good

```kotlin
fun main() {
    val adapter = Adapter()
    adapter(object: Adapter.OnItemClickListener {
        override fun onItemClick(position: Int) {
            // Yes, I have received a callback, go to the next activity.
            println("Clicked position is $position")
        }
    })
    adapter.execute()
}

class Adapter {

    private var onItemClickListener: OnItemClickListener? =null
   
    operator fun invoke (onItemClickListener: OnItemClickListener) {
        this.onItemClickListener = onItemClickListener
    }

    fun execute() {
        onItemClickListener?.onItemClick(4)
    }
    
    interface OnItemClickListener {
        fun onItemClick(position: Int)
    }
    
    interface OnRadioClickListener {
        fun onRadioButtonClick(position: Int)
    }

}
```

## D - The Dependency Inversion Principle (DIP)

### Bad
```kotlin
fun main() {
    val user = User()
    user.getMyData(3)
}

class User {

    fun getMyData(requestCode: Int) {
        when (requestCode) {
            1 -> {
                val firebase = Firebase()
                firebase.fetchData() 
            }
            2 -> {
                val restClient = RestClient()
                restClient.fetchData() 
            } 
            else -> print("I dont care about the user. Don't do anything. Keep quiet!")
        } 

    }
}

class Firebase {
    fun fetchData(){
        print("Syncing the data from the firebase storage")
    } 
}

class RestClient {
    fun fetchData(){ 
        print("Hitting the api and getting back the response")
    } 
}
```
### Good
```kotlin
fun main() {
    /* As a end user, I don't care about how will I retrieve the data. 
     * Do whatever you want and simply get my data. 
     */
    val user = User(dataFetcher = Firebase())
    user.getMyData()
}

class User(private val dataFetcher: DataFetcher) {

    fun getMyData() {
        dataFetcher.fetchData()
    }
}

interface DataFetcher {
    fun fetchData()
}

class Firebase: DataFetcher {
    override fun fetchData(){
       print("Syncing the data from the firebase storage")
    } 
}

class RestClient: DataFetcher {
    override fun fetchData(){ 
       print("Hitting the api and getting back the response")
    } 
}
```
