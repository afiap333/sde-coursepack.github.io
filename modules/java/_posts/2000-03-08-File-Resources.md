---
Title: File Resources
---

# Resources in Java

Often, our programs will need to use external resources. These could include:

* Files on our computer
* The Internet
* A Database
* etc.

In this module, we will first look at some simple usage of resource files in Java.

Second, we will look at some basic File I/O in Java for reading and writing to
Plain Text files (like .txt or .csv).

---

## Resource Files/Assets

Often, when making a program or website, we will need assets. These assets could
include images, like this picture of my cat Chloe sitting on my shoulder despite
my wishes that she not:

<img alt="The author's calico cat sitting on his shoulder, while the author scowls with annoyance, despite internally feeling loved like he has never felt before" src="{{site.baseurl}}/modules/java/images/8/chloe.jpg"/>

If I wanted to make this a company logo (and who wouldn't), then this image would
likely appear throughout any app I made. And so, I need to store that image with
the app's project code-base. And, in fact, the above picture [*is* in this website's
code base.](https://github.com/sde-coursepack/sde-coursepack.github.io/blob/main/modules/java/images/8/chloe.jpg)

Of course, this website is largely built from markdown files, not .java files. 
So the way I store and utilize the image in a Java project is a little different.

As we don't want to get into GUIs just yet, let's start simple.

### Creating a simple text file.

Let's make a simple text file in IntelliJ. I'm going to use my project
from the IntelliJ lecture.

First, I open up a text editor and write the file contents:

```
A Haiku for my cat:

Chloe, calico,
Why do you stand on shoulders?
I'm trying to work!
```

Now, wherever I saved that file, I'm going to copy and paste (or click and drag)
the file into my project's `src/main/resources` folder:

<img alt="My text file is now visible in the project folder sub/main/resources" src="{{site.baseurl}}/modules/java/images/8/haiku_resource.png"/>

### Accessing resource text file in Java

Now that we have this resource, we can access it as follows:

```java
    ClassLoader classLoader = getClass().getClassLoader();
    InputStream inputStream = classLoader.getResourceAsStream("haiku.txt");
```

You'll notice the filename "haiku.txt" is on the second line. We start by getting the classLoader, which is used to access the Path to the file (`resourceURL`) using the `getResource(filename)` function. We then call `getFile()` on that Path to get the String filename.

This gives us the String variable `inputStream` which we can then use to read via a `BufferedReader` as follows:

```java 
    InputStreamReader isr = new FileReader(inputStream);
    BufferedReader br = new BufferedReader(isr);
    for (String line = br.readLine(); line != null; line = br.readLine()) {
       System.out.println(line);
    }
```

Note that you may often see people combine the first two lines:

```java
    BufferedReader br = new BufferedReader(new InputStreamReader(filename));
```

However, we'll discuss later how this could be seen as bad style in many contexts because it makes the code harder to read and understand if you use this nest constructor call approach. That said, it's common enough for these "repetitive" file IO situations that we tend to overlook it.

Now, we can simply read the file via a BufferedReader. We cover reading from a BuffedReader below.


## Using a BufferedReader

A BufferedReader has one key method: `readLine()`

* This method returns a String of the next line in the file
* If the BufferedReader has reached the end of the file, it returns `null`

One simple use-case of a BufferedReader is to print the contents
of the file. One way to print the contents of a file is:

```java
    for (String line = br.readLine(); line != null; line = br.readLine()) {
            System.out.println(line);
    }
    br.close();
```

You will often see this using a while loop instead.

```java
    String line = br.readLine();
    while (line != null) {
        System.out.println(line);
        line = br.readLine();
    }
    br.close();
```

Of course, you don't have to print. For example, if you are reading
a csv file, you may split the line up by commas and use the file contents
to create or populate a data object. Or any number of other approaches.

## Files.lines()

Another approach for reading a file is to use `Files.lines()`, which is a functional
programming approach. We will discuss this when going over functional programming
and lambda bodies, later on.

---

## Main resources vs Test Resources

Remember that our source folder has both a `main` and `test` folder. In general,
any resources in `main` are accessible by either `main` or `test`. Resources in
`test` are only accessibly by classes in `test`.

For example, let's say you write a class that reads a CSV file and produces
an ArrayList of some class. You can include an example csv file in the 
`test` resources to test with, but that file won't be confused as part of the
`main` resources of the project.

---

## User specified files shouldn't be resources

If you program can take in a filename from the user, such as via command-line or text entry, you shouldn't open that file with `ClassLoader`. In that case, it's sufficient to just create a `BufferedReader` from the filename the user gives you.

Something like:

```java
    public ArrayList<String> getLinesFromFile(String filename) {
        try {
            FileReader fileReader = new FileReader(filename);
            BufferedReader bufferedReader = new BufferedReader(fileReader);
            ArrayList<String> lines = new ArrayList<>();
            String line = bufferedReader.readLine();
            while (line != null) {
                lines.add(line);
                line = bufferedReader.readLine();
            }
            return lines;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        }

```

You only use Resources when you are trying to access a resource file that you would include in a distribution of the project.

## Resource creation example

Consider the puzzle game Picross S, which involves solving [https://en.wikipedia.org/wiki/Nonogram]. If you are a programmer of the game Picross, which seems like the better approach for development?

![img.png](../images/8/nonogram_example.png)
Image Source: [Wikipedia](https://en.wikipedia.org/wiki/Nonogram#/media/File:Nonogram_wiki.svg)

1) Create a Java class for every single puzzle that describes how the puzzle works and hard-code what the data values are for each row/column
2) Describe a generic puzzle class that works for all puzzles, and store the *data* of each row and column as a file.

Saw we wanted to implement 300 puzzles. In the first case, we would need at least 300 classes. In the second case, we'd need probably two classes (one to read in the file, and the other to model the state of the puzzle) and 300 resource files. These files might be formatted something like (using the puzzle above):

```text
width: 30
height: 20

rows:
8, 7, 5, 7
5, 4, 3, 3
3, 3, 2, 3
...

columns:
1
1
2
...

```

Again, this is hypothetical. If I were implementing this, I would likely use something like JSON, rather than the text above. The point, however, is that this allows us to simply create 300 puzzle files with a consistent, human-readable format. This lets us separate *how the game works* from the individual puzzles in the game, and allows them to be edited separately.