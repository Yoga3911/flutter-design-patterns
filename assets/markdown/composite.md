## Class diagram

![Composite Class Diagram](resource:assets/images/composite/composite.png)

## Implementation

### Class diagram

The class diagram below shows the implementation of **Composite** design pattern.

![Composite Implementation Class Diagram](resource:assets/images/composite/composite_implementation.png)

_IFile_ defines a common interface for both _File_ (leaf) and _Directory_ (composite) classes:

- _getSize()_ - returns size of the file;
- _render()_ - renders the component's UI.

_File_ class implements the _getSize()_ and _render()_ methods, additionally contains _title_, _size_ and _icon_ properties. _Directory_ implements the same required methods, but it also contains _title_, _isInitiallyExpanded_ and _files_ list, containing the _IFile_ objects, defines _addFile()_ method, which allows adding _IFile_ objects to the directory (_files_ list). _AudioFile_, _ImageFile_, _TextFile_ and _VideoFile_ classes extend the _File_ class to specify a concrete type of the file.

### IFile

An interface that defines methods to be implemented by leaf and composite components.

```
abstract interface class IFile {
  int getSize();
  Widget render(BuildContext context);
}
```

### File

A concrete implementation of the _IFile_ interface which matches the _leaf_ class in the Composite design pattern. In the _File_ class, the _getSize()_ method simply returns the file size, and _render()_ - returns file's UI widget which is used in the example screen.

```
base class File extends StatelessWidget implements IFile {
  final String title;
  final int size;
  final IconData icon;

  const File({
    required this.title,
    required this.size,
    required this.icon,
  });

  @override
  int getSize() => size;

  @override
  Widget render(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(left: LayoutConstants.paddingS),
      child: ListTile(
        title: Text(
          title,
          style: Theme.of(context).textTheme.bodyLarge,
        ),
        leading: Icon(icon),
        trailing: Text(
          FileSizeConverter.bytesToString(size),
          style: Theme.of(context)
              .textTheme
              .bodyMedium
              ?.copyWith(color: Colors.black54),
        ),
        dense: true,
      ),
    );
  }

  @override
  Widget build(BuildContext context) => render(context);
}
```

### Concrete classes extending File

All of these classes extend the _File_ class and specify the concrete file type by providing a unique icon for the corresponding file type.

```
final class AudioFile extends File {
  const AudioFile({
    required super.title,
    required super.size,
  }) : super(icon: Icons.music_note);
}

final class ImageFile extends File {
  const ImageFile({
    required super.title,
    required super.size,
  }) : super(icon: Icons.image);
}

final class TextFile extends File {
  const TextFile({
    required super.title,
    required super.size,
  }) : super(icon: Icons.description);
}

final class VideoFile extends File {
  const VideoFile({
    required super.title,
    required super.size,
  }) : super(icon: Icons.movie);
}
```

### Directory

A concrete implementation of the _IFile_ interface which matches the _composite_ class in the Composite design pattern. Similar as in _File_ class, _render()_ returns directory's UI widget which is used in the example screen. However, in this class _getSize()_ method calculates the directory size by calling the _getSize()_ method for each item in the _files_ list and adding up the results. This is the main idea of the Composite design pattern which allows the composite class to treat all the elements in the containing list uniformly as long as they implement the same interface.

```
class Directory extends StatelessWidget implements IFile {
  final String title;
  final bool isInitiallyExpanded;

  final List<IFile> files = [];

  Directory(this.title, {this.isInitiallyExpanded = false});

  void addFile(IFile file) => files.add(file);

  @override
  int getSize() {
    var sum = 0;

    for (final file in files) {
      sum += file.getSize();
    }

    return sum;
  }

  @override
  Widget render(BuildContext context) {
    return Theme(
      data: ThemeData(
        colorScheme: ColorScheme.fromSwatch().copyWith(primary: Colors.black),
      ),
      child: Padding(
        padding: const EdgeInsets.only(left: LayoutConstants.paddingS),
        child: ExpansionTile(
          leading: const Icon(Icons.folder),
          title: Text('$title (${FileSizeConverter.bytesToString(getSize())})'),
          initiallyExpanded: isInitiallyExpanded,
          children: files.map((IFile file) => file.render(context)).toList(),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) => render(context);
}
```

### Example

_CompositeExample_ widget contains the _buildMediaDirectory()_ method which builds the file structure for the example. This method illustrates the Composite design pattern - even though the components are of a different type, they could be handled in the same manner since the implemented interface of _IFile_ is the same for all components. This allows us to put _Directory_ objects inside other directories, mix them along with concrete _File_ objects hence building the tree structure of _IFile_ components.

```
class CompositeExample extends StatelessWidget {
  const CompositeExample();

  Widget _buildMediaDirectory() {
    final musicDirectory = Directory('Music')
      ..addFile(const AudioFile(title: 'Darude - Sandstorm.mp3', size: 2612453))
      ..addFile(const AudioFile(title: 'Toto - Africa.mp3', size: 3219811))
      ..addFile(
        const AudioFile(
          title: 'Bag Raiders - Shooting Stars.mp3',
          size: 3811214,
        ),
      );

    final moviesDirectory = Directory('Movies')
      ..addFile(const VideoFile(title: 'The Matrix.avi', size: 951495532))
      ..addFile(
        const VideoFile(title: 'The Matrix Reloaded.mp4', size: 1251495532),
      );

    final catPicturesDirectory = Directory('Cats')
      ..addFile(const ImageFile(title: 'Cat 1.jpg', size: 844497))
      ..addFile(const ImageFile(title: 'Cat 2.jpg', size: 975363))
      ..addFile(const ImageFile(title: 'Cat 3.png', size: 1975363));

    final picturesDirectory = Directory('Pictures')
      ..addFile(catPicturesDirectory)
      ..addFile(const ImageFile(title: 'Not a cat.png', size: 2971361));

    final mediaDirectory = Directory('Media', isInitiallyExpanded: true)
      ..addFile(musicDirectory)
      ..addFile(musicDirectory)
      ..addFile(moviesDirectory)
      ..addFile(picturesDirectory)
      ..addFile(Directory('New Folder'))
      ..addFile(
        const TextFile(title: 'Nothing suspicious there.txt', size: 430791),
      )
      ..addFile(const TextFile(title: 'TeamTrees.txt', size: 104));

    return mediaDirectory;
  }

  @override
  Widget build(BuildContext context) {
    return ScrollConfiguration(
      behavior: const ScrollBehavior(),
      child: SingleChildScrollView(
        padding: const EdgeInsets.symmetric(
          horizontal: LayoutConstants.paddingL,
        ),
        child: _buildMediaDirectory(),
      ),
    );
  }
}
```
