
# Flutter Dock Application Code Documentation

This document provides an overview and explanation of the main components in the Flutter Dock application.

## Code Overview

### Main Application

The entry point of the application initializes the Flutter app and builds the main widget.

```dart
void main() {
  runApp(const MyApp());
}
```

#### MyApp Class

The `MyApp` class builds the main `MaterialApp` widget that serves as the application framework.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Dock<IconData>( // Specify that T is IconData
            items: const [
              Icons.person,
              Icons.message,
              Icons.call,
              Icons.camera,
              Icons.photo,
            ],
            builder: (e) {
              return Container(
                constraints: const BoxConstraints(minWidth: 48),
                height: 48,
                margin: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(8),
                  color: Colors.primaries[e.hashCode % Colors.primaries.length],
                ),
                child: Center(child: Icon(e, color: Colors.white)),
              );
            },
          ),
        ),
      ),
    );
  }
}
```

### Dock Widget

The `Dock` widget is a generic widget that displays a list of draggable items.

```dart
class Dock<T extends IconData> extends StatefulWidget {
  const Dock({
    super.key,
    this.items = const [],
    required this.builder,
  });

  final List<T> items; // Initial items to put in this Dock
  final Widget Function(T) builder; // Builder for the provided item

  @override
  State<Dock<T>> createState() => _DockState<T>();
}
```

### Dock State Management

The `_DockState` class manages the state for the `Dock` widget, handling item dragging and reordering.

```dart
class _DockState<T extends IconData> extends State<Dock<T>> {
  late List<T> _items; // Holds the draggable items
  T? _draggedItem; // Tracks the item being dragged
  int? _draggedItemIndex; // Tracks the index of the dragged item

  @override
  void initState() {
    super.initState();
    _items = List.from(widget.items); // Initialize the items
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(8),
        color: Colors.black12,
      ),
      padding: const EdgeInsets.all(4),
      height: 70, // Fixed height for the dock
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: _items.asMap().entries.map((entry) {
          int index = entry.key; // Current index of the item
          T item = entry.value; // Current item

          return DragTarget<T>( // Defines a target area for draggable items
            onAccept: (data) { // Action when an item is accepted
              setState(() {
                if (_draggedItemIndex != null && _draggedItemIndex != index) {
                  _items.remove(data); // Remove the item from its old position
                  _items.insert(index, data); // Insert it in the new position
                }
                _draggedItem = null; // Reset dragged item
                _draggedItemIndex = null; // Reset dragged index
              });
            },
            builder: (context, candidateData, rejectedData) {
              return Draggable<T>( // Makes its child draggable
                data: item,
                feedback: Material(
                  color: Colors.transparent,
                  child: Container(
                    constraints: const BoxConstraints(minWidth: 48),
                    height: 48,
                    decoration: BoxDecoration(
                      borderRadius: BorderRadius.circular(8),
                      color: Colors.grey[800],
                    ),
                    child: Center(child: Icon(item, color: Colors.white)),
                  ),
                ),
                childWhenDragging: Container(), // Hide the item while dragging
                onDragStarted: () {
                  setState(() {
                    _draggedItem = item; // Set the item being dragged
                    _draggedItemIndex = index; // Store the dragged item's index
                  });
                },
                child: AnimatedContainer( // Smooth transition for the dock item
                  duration: const Duration(milliseconds: 300),
                  curve: Curves.easeInOut,
                  constraints: const BoxConstraints(minWidth: 48),
                  height: 48,
                  margin: const EdgeInsets.all(8),
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(8),
                    color: (_draggedItem == item)
                        ? Colors.blue[100] // Highlight the dragged item
                        : Colors.primaries[item.hashCode % Colors.primaries.length],
                  ),
                  child: Center(
                    child: Icon(item, color: Colors.white),
                  ),
                ),
              );
            },
          );
        }).toList(),
      ),
    );
  }
}
```

## Conclusion

This documentation provides an overview of the main components and functionality of the Flutter Dock application. The code is structured to ensure easy readability and maintainability, following Flutter's best practices.
