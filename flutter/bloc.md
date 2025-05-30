# 🔷 Understanding BLoC in Flutter

**BLoC (Business Logic Component)** is a Flutter design pattern used to separate business logic from
UI.
It uses Streams for reactive programming, making your app logic clean, testable, and scalable.

---

## 📦 1. Installing BLoC

To use BLoC in your Flutter project, add the required dependencies:

```yaml
dependencies:
  flutter_bloc: latest_version
```

Then, import it in your Dart files:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
```

---

## 🧠 2. Understanding Core Concepts

### ✅ Event

Events are inputs to the BLoC. They represent user interactions or lifecycle changes in the
application.

**Example:**

```dart
abstract class CounterEvent {}
class Increment extends CounterEvent {}
class Decrement extends CounterEvent {}
```

Events should be clearly defined for every action that changes the state.

---

### ✅ State

States are outputs from the BLoC. They hold the data needed to render the UI.

**Example:**

```dart
class CounterState {
  final int count;
  CounterState(this.count);
}
```

Every time the state changes, the UI is rebuilt to reflect the new data.

---

### ✅ BLoC Class

This class contains the business logic. It listens to events and emits new states in response.

**Example:**

```dart
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState(0)) {
    on<Increment>((event, emit) => emit(CounterState(state.count + 1)));
    on<Decrement>((event, emit) => emit(CounterState(state.count - 1)));
  }
}
```

---

## 🧱 3. Bloc Widgets Explained in Detail

### ✅ BlocProvider

`BlocProvider` is a widget that provides a BLoC instance to its child subtree.
It is commonly placed at the top level of your widget tree.

**Why Use It?**

- It manages the lifecycle of the BLoC.
- Makes the BLoC accessible to any widget below it in the widget tree.

**Example:**

```dart
BlocProvider(
  create: (context) => CounterBloc(),
  child: MyHomePage(),
)
```

### ✅ BlocBuilder

`BlocBuilder` is used to build widgets in response to new states. It listens to the BLoC and
rebuilds the UI accordingly.

**Key Points:**

- It only rebuilds when a new state is emitted.
- You can control rebuilds using `buildWhen`.

**Example:**

```dart
BlocBuilder<CounterBloc, CounterState>(
  builder: (context, state) {
    return Text('Count: ${state.count}');
  },
)
```

**With `buildWhen`:**

```dart
BlocBuilder<CounterBloc, CounterState>(
  buildWhen: (previous, current) => previous.count != current.count,
  builder: (context, state) => Text("${state.count}"),
)
```

### ✅ BlocListener

`BlocListener` listens to state changes and performs side effects (e.g., navigation or showing
snackbars) but doesn’t build widgets.

**Example:**

```dart
BlocListener<CounterBloc, CounterState>(
  listener: (context, state) {
    if (state.count == 10) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("Reached 10!")));
    }
  },
  child: MyWidget(),
)
```

### ✅ BlocConsumer

`BlocConsumer` combines both `BlocBuilder` and `BlocListener`.

**Use When:**

- You need to build UI based on the state **and** perform actions on state change.

**Example:**

```dart
BlocConsumer<CounterBloc, CounterState>(
  listener: (context, state) {
    if (state.count < 0) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("Negative value!")));
    }
  },
  builder: (context, state) {
    return Text("Count is: ${state.count}");
  },
)
```

### ✅ BlocSelector

`BlocSelector` allows you to select a specific value from the state and rebuild the UI only when
that value changes.

**Why Use It?**

- Useful for performance optimization when you want to rebuild only a portion of the widget based on
  a specific field of the state.

**Example:**

```dart
BlocSelector<CounterBloc, CounterState, int>(
  selector: (state) => state.count,
  builder: (context, count) {
    return Text('Selected count: $count');
  },
)
```

---

## 🔢 4. Complete Counter Example

```dart
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterBloc(),
      child: Scaffold(
        appBar: AppBar(title: Text('BLoC Counter')),
        body: Center(
          child: BlocBuilder<CounterBloc, CounterState>(
            builder: (context, state) => Text('Count: ${state.count}', style: TextStyle(fontSize: 30)),
          ),
        ),
        floatingActionButton: Column(
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            FloatingActionButton(
              onPressed: () => context.read<CounterBloc>().add(Increment()),
              child: Icon(Icons.add),
            ),
            SizedBox(height: 10),
            FloatingActionButton(
              onPressed: () => context.read<CounterBloc>().add(Decrement()),
              child: Icon(Icons.remove),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 🧭 5. Navigation in BLoC

Although BLoC doesn't manage navigation directly, it integrates cleanly with Flutter's `Navigator`.

### ✅ Push Navigation

```dart
Navigator.push(context, MaterialPageRoute(builder: (_) => SecondPage()));
```

### ✅ Pass BLoC to Another Screen

```dart
Navigator.push(
context,
MaterialPageRoute(
builder: (_) => BlocProvider.value(
value: context.read<CounterBloc>(),
child: SecondPage(),
),
),
);
```

---

## 🌐 6. BLoC with API Calls

Demonstrates handling asynchronous data with BLoC.

### ✅ States

```dart
abstract class ApiState {}
class ApiInitial extends ApiState {}
class ApiLoading extends ApiState {}
class ApiLoaded extends ApiState {
  final List<String> data;
  ApiLoaded(this.data);
}
class ApiError extends ApiState {
  final String message;
  ApiError(this.message);
}
```

### ✅ Events

```dart
abstract class ApiEvent {}
class FetchData extends ApiEvent {}
```

### ✅ Bloc

```dart
class ApiBloc extends Bloc<ApiEvent, ApiState> {
  ApiBloc() : super(ApiInitial()) {
    on<FetchData>((event, emit) async {
      emit(ApiLoading());
      try {
        await Future.delayed(Duration(seconds: 2));
        emit(ApiLoaded(["Apple", "Banana", "Orange"]));
      } catch (_) {
        emit(ApiError("Failed to fetch data"));
      }
    });
  }
}
```

### ✅ UI Integration

```dart
BlocBuilder<ApiBloc, ApiState>
(
builder: (context, state) {
if (state is ApiLoading) return CircularProgressIndicator();
if (state is ApiLoaded)
return ListView(
children: state.data.map((e) => ListTile(title: Text(e))).toList(),
);
if (state is ApiError) return Text(state.message);
return ElevatedButton(
onPressed: () => context.read<ApiBloc>().add(FetchData()),
child: Text("Load Data"),
);
},
)
```

---

## 🚀 7. Benefits of Using BLoC

- ✅ **Separation of Concerns** – Logic and UI are independent
- ✅ **Reactive Programming** – Powered by Streams
- ✅ **Testability** – Easy to write unit tests
- ✅ **Scalability** – Suitable for large applications
- ✅ **Consistency** – Follows a uniform architecture

---

